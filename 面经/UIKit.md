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
