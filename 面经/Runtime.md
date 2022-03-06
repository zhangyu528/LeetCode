# Runtime

### 实例对象的数据结构？

```objective-c
struct objc_object {
	isa_t isa;//指向 类对象 的内存地址
	//...
}
union isa_t 
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }
    Class cls;
    uintptr_t bits;
}

@interface Object { 
    Class isa; 
}

@interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}
```

### 类对象的数据结构?

```objective-c
struct objc_class : objc_object {//继承实例对象
    // Class ISA;     //isa：指向元类 内存地址
    Class superclass; //父类指针
    cache_t cache;             // formerly cache pointer and vtable 方法缓存
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags 用于获取地址

    class_rw_t *data() { 
        return bits.data(); // &FAST_DATA_MASK 获取地址值
    }
```

### 元类对象的数据结构? 

```objective-c
//元类对象 和 类对象 数据结构是一致的（同上）
```

### Obj-C 对象、类的本质是通 过什么数据结构实现的？

* 结构体

* **类对象和元类对象是唯一的**，对象是可以在运行时创建无数个的

### `Obj-C` 中的类信息存放在哪里？

类方法存储在元类。

- 对象方法、属性、成员变量、协议等存放在 Class 对象中。
- 类方法存放在 meta-class 对象中。
- 成员变量的具体指，存放在 instance 对象中。

### 一个 `NSObject` 对象占用多少内存空间？

受限于内存分配的机制，一个 `NSObject`对象都会分配**16字节**的内存空间。但是实际上在64位下，只使用了 `8字节`，在32位下，只使用了 `4字节`。

`iOS` 中，分配内存空间都是 `16字节` 的倍数。**如果存在继承关系，则需要父类的大小**

### 说一下对 `isa` 指针的理解， 对象的`isa` 指针指向哪里？`isa` 指针有哪两种类型

* isa：是一个Class 类型的指针. 每个实例对象有个isa的指针,他指向对象的类，而Class里也有个isa的指针, 指向meteClass(元类)，而Class里还有个指向superclass对象的类

* **Class 是一个 objc_class 结构类型的指针, id是一个 objc_object 结构类型的指针.**

* ```objectivec
  typedef struct objc_class *Class;
  typedef struct objc_object *id;
  ```

### Category

* 被添加在了 `class_rw_t` 的对应结构里

* `Category` 在刚刚编译完的时候，和原来的类是分开的，只有在程序运行起来后，通过 `Runtime` ，`Category` 和原来的类才会合并到一起

##### 分类数据结构

* name：被分类的类名称
* cls ：isa指针
* instance_methods：对象方法列表
* class_methods：类方法列表
* protocols：协议列表
* properties：属性列表(只读)

##### 分类 OC

* 无法向现有的类中添加新的实例变量
* 多个分类override同一个方法**(除load方法）**，执行最后编译的分类override方法

* Cocoa Framework有很多是用Category实现的，重写之后，会导致在Runtime的时候，只有一个方法会被执行，而哪个会被执行是undefined

##### 匿名分类（OC）

* 能为自定义类，**静态**附加额外的属性，成员变量，方法声明
* **私有**属性和方法写到类扩展
* 子类**不能继承**附加项目

### 如何给 `Category` 添加属性？关联对象以什么形式进行存储？

* `关联对象` 以哈希表的格式，存储在一个全局的单例中

* ```objective-c
  @interface NSObject (Extension)
  
  @property (nonatomic,copy  ) NSString *name;
  
  @end
  
  
  @implementation NSObject (Extension)
  
  - (void)setName:(NSString *)name {
      
      objc_setAssociatedObject(self, @selector(name), name, OBJC_ASSOCIATION_COPY_NONATOMIC);
  }
  
  
  - (NSString *)name {
      
      return objc_getAssociatedObject(self,@selector(name));
  }
  
  @end
  ```

### 扩展

##### Swift Extension

* 扩展就是向一个已有的类、结构体、枚举类型或者协议类型添加新功能（逆向建模）
* 支持被**继承**
* 注意扩展中不能有**存储类型**的属性，只可以添加**计算型实例属性和计算型类型属性**

### Runtime 消息解析&转发

如果当前类没有对应的实例方法，系统会调用如下方法，可以选择在这个时机动态添加

```objective-c
+(BOOL)resolveInstanceMethod:(SEL)sel {

    if (sel == @selector(eat)) {
        class_addMethod([self class], sel, (IMP)vc_eat, "v@:");
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
void vc_eat(id obj,SEL _cmd) {
    NSLog(@"这是Eat方法");
}
```

如果当前类没有对应的类方法，系统会调用如下方法，可以选择在这个时机动态添加

```objective-c
+(BOOL) resolveClassMethod:(SEL)sel {
    if(sel == @selector(newPerson)) {
        // 找到当前类的 metaClass
        Class MetaClass = objc_getMetaClass([NSStringFromClass(self) UTF8String]);
        class_addMethod(MetaClass,sel,(IMP)vc_newPerson,"v@:");
        return YES;
    }
    return [super resolveClassMethod:sel];
}
void vc_newPerson(id obj,SEL _cmd) {
    NSLog(@"当前是类方法 newPerson")；
}
```

如果在当前类，以上两个方法都没有实现，可以将消息转发给其他的类处理

```objective-c
- (id)forwardingTargetForSelector:(SEL)aSelector {
    if(aSelector == @selector(someMethod)) {
        return [SomeClass new];
    }
       return [super forwardingTargetForSelector:aSelector];
}
```

### 如何运用 `Runtime` 字典转模型？

`Runtime` 遍历 `ivar_list`,结合 `KVC` 赋值。

### 如何运用 `Runtime` 进行模型的归解档

```
Runtime` 遍历 `ivar_list
```

### `Objective-C` 如何实现多重继承？

* 使用协议
* 消息转发

### Method Swizzling

* 实现异常保护
* 实现埋点统计
* 实现AOP

```objectivec
#import <objc/runtime.h>
@implementation UIViewController (Swizzling)
+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];
        // When swizzling a class method, use the following:
        // Class class = object_getClass((id)self);
        SEL originalSelector = @selector(viewWillAppear:);
        SEL swizzledSelector = @selector(xxx_viewWillAppear:);
        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
        BOOL didAddMethod = class_addMethod(class,
                                            originalSelector,
                                            method_getImplementation(swizzledMethod),
                                            method_getTypeEncoding(swizzledMethod));
        if (didAddMethod) {//以前没有，现在添加成功
            class_replaceMethod(class,
                                swizzledSelector,
                                method_getImplementation(originalMethod),
                                method_getTypeEncoding(originalMethod));
        } else {//已经有，添加失败
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}
#pragma mark - Method Swizzling
- (void)xxx_viewWillAppear:(BOOL)animated {
    [self xxx_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", self);
}
@end
```



### Isa Swizzling

```objectivec
- (Class)class {//实例的类对象
    return object_getClass(self);
}

Class object_getClass(id obj)  //实例isa的类对象
{
    if (obj) return obj->getIsa();
    else return Nil;
}
```

### Objective-C的+load方法调用原理分析区别?

**调用时机：**`+load`方法会在**Runtime**加载类对象(`class`)和分类(`category`)的时候调用

**调用频率：** 每个类对象、分类的`+load`方法，在工程的整个生命周期中只调用一次

**调用顺序：**

> 1. 先调用类对象(class)的
>
>    ```objective-c
>    +load
>    ```
>
>    方法：
>
>    - 类对象的`load`调用顺序是按照  **类文件的编译顺序**  进行先后调用；
>    - 调用子类`+load`之前会先调用父类的`+load`方法
>
> 2. 再调用分类(`category`)的+load方法：按照编译先后顺序调用（先编译的，先被调用）

### Objective-C的+initialize方法调用原理分析

* `+initialize`方法会在类对象  ***第一次***  接收到消息的时候调用

* 调用顺序：调用某个类的`+initialize`之前，会先调用其父类的`+initialize`（前提是父类的`+initialize`从来没有被调用过）

* 由于`+initialize`的调用，是通过消息机制，也就是`objc_msgSend()`，因此如果子类的`+initialize`没有实现，就会去调用父类的`+initialize`

* 基于同样的原因，如果分类实现的`+initialize`，那么就会“覆盖”类对象本身的`+initialize`方法而被调用。

