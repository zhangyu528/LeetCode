# Foundation

### 1.`nil`、`NIL`、`NSNULL` 有什么区别？

- `nil`、`NIL` 可以说是等价的，都代表内存中一块空地址。
- `NSNULL` 代表一个指向 `nil` 的对象。

### 2.如何实现一个线程安全的 `NSMutableArray`?

主要是多线程读写时，需要加锁

### 3.如何定义一台 `iOS` 设备的唯一性?

- 1.`IMEI`：国际移动设备身份码，iOS 5 以后苹果不再允许获取 IMEI

- 2.`UDID`: iOS 设备的唯一识别码，在 iOS 6（2013 年 5 月 ） 以后被 Apple 禁止

- 3.`IDFA`：广告标识符，每台设备的唯一ID。用户可以禁止、重置、还原，iOS 6提出。

- 4.`MAC地址`：网络唯一标识符，iOS 7 之后

- 5.`UUID`:（Universally Unique IDentifier）：通用唯一识别码,NSUUID 每次获取的值都会发生变化，但是它会保持唯一性

- 6.`IDFV`：Vendor 标示符，也被称为厂商标识符。只要用户的设备中没有卸载当前 Vendor 的所有 APP，则不会发生变化。什么是 Vendor ？可以理解成 bundleID 的前两部分

  结论：2017 年的我们想要通过 `UDID`、`Mac` 地址、`OpenUDID` 来定位用户设备已经是不可能啦，因为它们要么是无效，要么是受到了 `App Store` 的限制。但是如果你想要追踪广告的话，可以使用 `IDFA`，想要用来分析用户行为可以使用 `IDFV + KeyChain` 来解决。

### 4.`atomic` 修饰的属性是绝对安全的吗？为什么？

不是，所谓的安全只是局限于 `Setter`、`Getter` 的访问器方法而言的，你对它做 `Release` 的操作是不会受影响的。这个时候就容易崩溃了。

### 5.实现 `isEqual` 和 `hash` 方法时要注意什么？

### 6.`id` 和 `instanceType` 有什么区别？

##### 相同点

`instancetype` 和 `id` 都是万能指针，指向对象。

##### 不同点：

- 1.`id` 在编译的时候不能判断对象的真实类型，`instancetype` 在编译的时候可以判断对象的真实类型。
- 2.`id` 可以用来定义变量，可以作为返回值类型，可以作为形参类型；`instancetype` 只能作为返回值类型。

### 7.简述事件传递、事件响应机制。

概括的讲，事件传递是由内到外，事件响应是由外到内。

### 8.说一下对 `Super` 关键字的理解。

这是一个编译器的特性，优先到父类中寻找方法。

### 9.了解 `逆变` 和 `协变` 吗？

iOS 9 之后的新特性，基于泛型，在有父子继承关系的时候会用到逆变与协变。

### 10.`@synthesize` 和 `@dynamic` 分别有什么作用？

`@synthesize` 是自动合成 `Setter`，`Getter` 方法。

`@dynamic`是动态添加 `Setter`，`Getter` 方法。

### 11.`Obj-C` 中的反射机制了解吗？

```objective-c
// SEL和字符串转换
FOUNDATION_EXPORT NSString *NSStringFromSelector(SEL aSelector);
FOUNDATION_EXPORT SEL NSSelectorFromString(NSString *aSelectorName);
// Class和字符串转换
FOUNDATION_EXPORT NSString *NSStringFromClass(Class aClass);
FOUNDATION_EXPORT Class __nullable NSClassFromString(NSString *aClassName);
// Protocol和字符串转换
FOUNDATION_EXPORT NSString *NSStringFromProtocol(Protocol *proto) NS_AVAILABLE(10_5, 2_0);
FOUNDATION_EXPORT Protocol * __nullable NSProtocolFromString(NSString *namestr) NS_AVAILABLE(10_5, 2_0);
```

### 12.`typeof` 和 `__typeof`，`__typeof__` 的区别?

`__typeof __()` 和 `__typeof()` 是 `C语言` 的编译器特定扩展，因为标准 `C` 不包含这样的运算符。 标准 `C` 要求编译器用双下划线前缀语言扩展（这也是为什么你不应该为自己的函数，变量等做这些）

`typeof()` 与前两者完全相同的，只不过去掉了下划线，同时现代的编译器也可以理解。

所有这三个意思是相同的，但没有一个是标准C，不同的编译器会按需选择符合标准的写法。

### 13.如何判断一个文件在沙盒中是否存在？

### 14.头文件导入的方式？

```objective-c
#if __has_include(<AFNetworking/AFNetworking.h>)
#import <AFNetworking/AFNetworking.h>
#else
#import "AFNetworking.h"
#endif
```

### 15.如何将 `Obj-C` 代码改变为 `C++/C` 的代码？

```
/ 在当前路径下执行
clang rewrite-objc main.m -o main.cpp
```

上面的方法会生成不同平台下的源代码，大约有10万行，但我们只想要 iOS 平台下的相关代码，我们可取以下的方式：

```makefile
// xcrun == Xcode Run
// -sdk 指定系统平台
// -arch 指定的架构
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc main.m -o main-arm64.cpp
```

### 16.知不知道在哪里下载苹果的源代码？

[https://openSource.apple.com/tarballs](https://opensource.apple.com/tarballs)

### 17.`objc_getClass()`、`object_getClass()`、`Class` 这三个方法用来获取类对象有什么不同？

导入 `#import <objc/runtime.h>`

- `object_getClass`:        传一个对象，返回这个对象的类对象。
- `class_getSuperclass`:    传一个类，返回类的父类。
- `objc_getClass`:          传一个类名称，返回对应的类对象。
- `objc_getMetaClass`:      传一个类名称，返回对应的元类对象。
