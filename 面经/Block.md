# Block

auto变量block访问方式是值传递，static变量block访问方式是指针传递

auto自动变量可能会销毁的，内存可能会消失，不采用指针访问；static变量一直保存在内存中，指针访问即可

![img](https://upload-images.jianshu.io/upload_images/1915113-b684175082c9a822.png?imageMogr2/auto-orient/strip|imageView2/2/w/792/format/webp)

#### 如何判断block是哪种类型？

- 没有访问auto变量的block是__NSGlobalBlock __ ，放在数据段
- 访问了auto变量的block是__NSStackBlock __
- `[__NSStackBlock __ copy]`操作就变成了__NSMallocBlock __

#### 对每种类型block调用copy操作后是什么结果？

- __NSGlobalBlock __ 调用copy操作后，什么也不做
- __NSStackBlock __ 调用copy操作后，复制效果是：从栈复制到堆；副本存储位置是**堆**
- __NSMallocBlock __ 调用copy操作后，复制效果是：**引用计数增加**；副本存储位置是**堆**

#### 在ARC环境下，编译器会根据情况自动将栈上的block复制到堆上的几种情况？

- 1.block作为函数返回值时
- 2.将block赋值给__strong指针时
- 3.block作为Cocoa API中方法名含有usingBlock的方法参数时
- 4.block作为GCD API的方法参数时

**ARC下block属性的建议写法**
`@property (copy, nonatomic) void (^block)(void);`

- 如果block在`栈`空间，不管外部变量是强引用还是弱引用，block都会弱引用访问对象
- 如果block在`堆`空间，如果外部强引用，block内部也是强引用；如果外部弱引用，block内部也是弱引用

#### __block 修饰符作用？

- _block可以用于解决block内部无法修改auto变量值的问题
- __block不能修饰全局变量、静态变量（static）

## block循环引用

#### ARC下如何解决block循环引用的问题？

`__weak`：不会产生强引用，指向的对象销毁时，会自动让指针置为nil

`__unsafe_unretained`：不会产生强引用，不安全，指向的对象销毁时，指针存储的地址值不变

`__block`：必须把引用对象置位nil，并且要调用该block

