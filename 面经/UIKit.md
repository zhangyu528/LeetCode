# UIKit

### 1.`UIView` 和 `CALayer` 是什么关系？

- `CALayer` 基于 `QuartzCore` 框架
- `UIView` 基于 `UIKit` 框架

`CALayer` 是继承自 `NSObject` 的,而 `UIView` 是继承自 `UIResponder` 的

`UIView` 和 `CALayer` 算是相互补充的关系。

```
UIView` = `CALayer.delegate
```

`UIView` : 负责用户的交互事件。

`CALayer`: 负责图像和动画的渲染。

### 2.`Bounds` 和 `Frame` 的区别?

- `Bounds`：一般是相对于自身来说的，是控件的内部尺寸。如果你修改了 `Bounds`，那么子控件的相对位置也会发生改变。
- `Frame` ：是相对于父控件来说的，是控件的外部尺寸。

### 3.`TableViewCell` 如何根据 `UILabel` 内容长度自动调整高度?

### 4.`LoadView`方法了解吗？

自定义控制器的 `View`。可以在这个方法中做一些个性化的设置

### 5.`UIButton` 的父类是什么？`UILabel` 的父类又是什么？

- UIButton -> UIControl -> UIView -> UIResponder
- UILabel -> UIView -> UIResponder

### 6.实现一个控件，可以浮在任意界面的上层并支持拖动？

两个方面

- 1.这个空间要添加在程序的主 `window` 上。
- 2.随手指拖动更新位移

### 7.说一下控制器 `View` 的生命周期，一旦收到内存警告会如何处理

当系统内存告急时，会接收到`didReceiveMemoryWarning`。 这是属于 `ViewController` 的方法，当`ViewController` 接收到`didReceiveMemoryWarning`，首先会判断当前的 `ViewController` 是否还显示在 `window`上，如果不在就会移除当前的 `ViewController`，销毁`ViewController` 上面的子控件，并执行 `ViewDidUnload` 方法。

### 8.如何暂停一个 `UIView` 中正在播放的动画？暂停后如何恢复？

### 9.说一下 `UIView` 的生命周期？

### 10.`UIViewController` 的生命周期？

```objective-c
-[ViewController initWithCoder:]
-[ViewController awakeFromNib]
-[ViewController loadView]
-[ViewController viewDidLoad]
-[ViewController viewWillAppear:]
-[ViewController viewWillLayoutSubviews]
-[ViewController viewDidLayoutSubviews]
-[ViewController viewDidAppear:]
-[ViewController viewWillDisappear:]
-[ViewController viewDidDisappear:]
-[ViewController dealloc]
-[ViewController didReceiveMemoryWarning]
```

### 11.通过UIView获取所在的UIViewController? 

```objectivec
- (UIViewController *)viewController {
    UIView *next = self;
    while ((next = [next superview])) {
        UIResponder *nextResponder = [next nextResponder];
        if ([nextResponder isKindOfClass:[UIViewController class]]) {
            return (UIViewController *)nextResponder;
        }
    }
    return nil;
}
```

### 12.`setNeedsDisplay` 和 `layoutIfNeeded` 两者是什么关系？

`setNeedsDisplay` 是给当前的视图做了标记。

`layoutIfNeeded` 查找是否有标记，如果有标记及立刻刷新。

只有这二者合起来使用，才会起到立刻刷新的效果。

### 13.要求详细的描述事件响应链

##### 1.1 物理层面事件的生成

iPhone 采用电容触摸传感器，利用人体的电流感应工作，由一块四层复合玻璃屏的内表面和夹层各涂有一层导电层，最外层是一层矽土玻璃保护层。当我们手指触摸感应屏的时候，人体的电场让手指和触摸屏之间形成一个耦合电容，对高频电流来说电容是直接导体。于是手指从接触点吸走一个很小的电流，这个电流分从触摸屏的四脚上的电极流出，并且流经这四个电极的电流和手指到四个电极的距离成正比。控制器通过对这四个电流的比例做精确的计算，得出触摸点的距离。

##### 1.2 iOS 操作系统下封装和分发事件

iOS 操作系统看做是一个处理复杂逻辑的程序，不同进程之间彼此通信采用消息发送方式，即 IPC (Inter-Process Communication)。现在继续说上面电容触摸传感器产生的 Touch Event，它将交由 IOKit.framework 处理封装成 IOHIDEvent 对象；下一步很自然想到通过消息发送方式将事件传递出去，至于发送给谁，何时发送等一系列的判断逻辑又该交由谁处理呢？

答案是 **SpringBoard.app**，它接收到封装好的 **IOHIDEvent** 对象，经过逻辑判断后做进一步的调度分发。例如，它会判断前台是否运行有应用程序，有则将封装好的事件采用 mach port 机制传递给该应用的主线程。

Port 机制在 IPC 中的应用是 Mach 与其他传统内核的区别之一，在 Mach 中，用户进程调用内核交由 IPC 系统。与直接系统调用不同，用户进程首先向内核申请一个 port 的访问许可；然后利用 IPC 机制向这个 port 发送消息，本质还是系统调用，而处理是交由其他进程完成的

##### 1.3 IOHIDEvent -> UIEvent

应用程序主线程的 runloop 申请了一个 mach port 用于监听 `IOHIDEvent` 的 `Source1` 事件，回调方法是 `__IOHIDEventSystemClientQueueCallback()`，内部又进一步分发 `Source0` 事件，而 `Source0`事件都是自定义的，非基于端口 port，包括触摸，滚动，selector选择器事件，它的回调方法是 `__UIApplicationHandleEventQueue()`，将接收到的 `IOHIDEvent` 事件对象封装成我们熟悉的 `UIEvent` 事件；然后调用 `UIApplication` 实例对象的 `sendEvent:` 方法，将 `UIEvent` 传递给 `UIWindow` 做一些逻辑判断工作：比如触摸事件产生于哪些视图上，有可能有多个，那又要确定哪个是最佳选项呢？ 等等一系列操作。这里先按下不表。

##### 1.4 Hit-Testing 寻找最佳响应者

`Source0` 回调中将封装好的触摸事件 UIEvent（里面有多个UITouch 即手势点击对象），传递给视图 `UIWindow`，其目的在于找到最佳响应者，这个过程称之为 `Hit-Testing`，字面上理解：hit 即触碰了屏幕某块区域，这个区域可能有多个视图叠加而成，那么这个触摸讲道理响应者有多个喽，那么“最佳”又该如何评判？这里要牢记几个规则：

1. 事件是自下而上传递，即 `UIApplication -> UIWindow -> 子视图 -> ...->子视图中的子视图`;
2. 后加的视图响应程度更高，即更靠近我们的视图;
3. 如果某个视图不想响应，则传递给比它响应程度稍低一级的视图，若能响应，你还得继续往下传递，若某个视图能响应了，但是没有子视图 它就是最佳响应者。
4. 寻找最佳响应者的过程中， UIEvent 中的 UITouch 会不断打上标签：比如 `HitTest View` 是哪个，`superview` 是哪个？关联了什么 `Gesture Recognizer`?

那么如何判定视图为响应者？由于 OC 中的类都继承自 `NSObject` ，因此默认判断逻辑已经在`hitTest:withEvent`方法中实现，它有两个作用： 1.询问当前视图是否能够响应事件 2.事件传递的桥梁。若当前视图无法响应事件，返回 nil 。代码如下：

```objective-c
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{ 
  // 1. 前置条件要满足       
  if (self.userInteractionEnabled == NO || 
  self.hidden == YES ||  
  self.alpha <= 0.01) return nil;
  
  // 2. 判断点是否在视图内部 这是最起码的 note point 是在当前视图坐标系的点位置
    if ([self pointInside:point withEvent:event] == NO) return nil;

  // 3. 现在起码能确定当前视图能够是响应者 接下去询问子视图
    int count = (int)self.subviews.count;
    for (int i = count - 1; i >= 0; i--)
    {
      // 子视图
        UIView *childView = self.subviews[i];
    
    // 点需要先转换坐标系        
        CGPoint childP = [self convertPoint:point toView:childView];  
        // 子视图开始询问
        UIView *fitView = [childView hitTest:childP withEvent:event]; 
        if (fitView)
        {
      return fitView;
    }
    }
                         
    return self;
}
```

##### 1.5 UIResponder Chain 响应链

`Hit-Testing` 过程中我们无法确定当前视图是否为“最佳”响应者，此时自然还不能处理事件。因此处理机制应该是找到所有响应者以及最佳响应者(**自下而上**)，由它们构成了一条响应链；接着将事件沿着响应链**自上而下**传递下去 ——最顶端自然是最佳响应者，事件除了被响应者消耗，还能被手势识别器或是 `target-action` 模式捕获并消耗。有时候，最佳响应者可能对处理 `Event` “毫无兴趣”，它们不会重写 `touchBegan` `touchesMove`..等四个方法；也不会添加任何手势；但如果是 `control(控件)` 比如 UIButton ，那么事件还是会被消耗掉的。
