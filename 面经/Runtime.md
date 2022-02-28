# Runtime

### 分类

##### 分类数据结构

* name：被分类的类名称
* cls ：isa指针
* instance_methods：对象方法列表
* class_methods：类方法列表
* protocols：协议列表
* properties：属性列表

##### 分类 OC

* 无法向现有的类中添加新的实例变量
* 多个分类override同一个方法(除load方法），执行最后编译的分类override方法

* Cocoa Framework有很多是用Category实现的，重写之后，会导致在Runtime的时候，只有一个方法会被执行，而哪个会被执行是undefined

##### 匿名分类（OC）

* 能为自定义类，**静态**附加额外的属性，成员变量，方法声明
* **私有**属性和方法写到类扩展
* 子类**不能继承**附加项目

### load方法

### initialize方法



### 扩展

##### Swift Extension

* 扩展就是向一个已有的类、结构体、枚举类型或者协议类型添加新功能（逆向建模）
* 支持被**继承**
* 注意扩展中不能有存储类型的属性，只可以添加计算型实例属性和计算型类型属性

### 消息转发

* `_objc_msgForward`函数是做什么的，直接调用它将会发生什么？

`_objc_msgForward`是 IMP 类型，用于消息转发的：

当向一个对象发送一条消息，但它并没有实现的时候，`_objc_msgForward`会尝试做消息转发

* `_objc_msgForward`消息转发做的几件事

  1.调用`+(BOOL)resolveInstanceMethod:(SEL)sel;`实例方法

   (或 `+(BOOL)resolveClassMethod:(SEL)sel;`)类方法

  2.调用`forwardingTargetForSelector:`方法

  3.调用`methodSignatureForSelector:`方法

  4.调用`forwardInvocation:`方法

  5.调用`doesNotRecognizeSelector:`

### Method Swizzling

### KVC&KVO

