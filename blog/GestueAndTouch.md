###IOS手势处理的那些坑
####前言
在不考虑内部实现机制的情况下，我们使用三种方式来处理IOS手势：
1. Gesture Recongnizers --- UIGestureRecognizer 及其子类
2. touches 响应 --- touchesBegan、touchesEnd..等
3. Target-Action 机制 --- UIControl及其子类

本文探讨了这几种处理手势事件的混合使用可能会产生的冲突情况，并提供了解决方法，希望看过这篇文章的朋友们不再踩这些坑。

[从这里下载本片文章涉及的源码](https://github.com/sunuo/TouchEventHandler.git)

想象这样一个场景，我们自定义了一个View，使用UIView作为其父类，通常我们只是将其本身作为一个承载子view的容器，但是很多情况下我们不得不对其进行拓展使其能够响应用户手势（例如点击跳转），有3个方案可选。
1.  绑定UIGesture Recognizer ,**Apple推荐做法**
2.  实现UIVIew的touch事件touchesEnded:withEvent
    * 如果父视图响应touchesBegan:withEvent 事件的话，见第二节  Touch Events 冲突
    * 如果父视图响应UIGesture Recongnizer 事件的话，见第三节  Touch Events 冲突
3.  将父类从UIView改成UIControl
    * 如果父视图响应UIGesture Recongnizer ,见第四节 UIGestureRecognizer和UIControl的冲突

####Touch Events 冲突
在这个例子中，我们让父view响应touchesBegan事件，子view（背景红色，占据父view的上半部分）的touchesEnded处理跳转事件
* 当点击下半部分区域时，父view背景改变
* 当点击上半部分区域时，弹出跳转提示(期望结果)，同时，父view背景改变(非期望结果)
#####问题分析
* 当点击下半区域时，Hitest寻找到响应控件父view，touchesBegan更改了背景颜色
* 当点击下半区域时，在touchesBegan阶段，hitest寻找响应控件子view，但是由于子view无法响应touchesBegan，所以通过响应链寻找到父view处理此touch事件，父view背景因此更改。
  同理，在touchesEnded阶段，Hitest寻找到了响应控件子view，所以弹出了跳转提示。
#####解决方案
除了使用UIGestureRecognizer代替touchesEnded之外，还可以通过实现子view所有的touches事件来防止touches event 被响应链传递。
```
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
//    防止此事件被响应链传递
}
```
####UIGestureRecognizer和touches事件的冲突
> A window delivers touch events to a gesture recognizer before it delivers them to the hit-tested view attached to the gesture recognizer.

为了搞清楚这两者同时存在时响应的时序问题，我们在view上绑定tapGestureRecognizer的同时实现所有的touches事件，尝试以下操作并观察结果

1. 点击
```
2016-10-30 15:37:47.711 LOG:::VCTouchesVSGestures___view:touchesBegan:withEvent:___<VCTouchesVSGestures: 0x7f9183ea8510>
2016-10-30 15:37:47.716 VCTouchesVSGestures: 0x7f9183ea8510>
2016-10-30 15:37:47.717 LOG:::VCTouchesVSGestures___view:touchesCancelled:withEvent:___<VCTouchesVSGestures: 0x7f9183ea8510>
```
2. 拖动
```
2016-10-30 15:44:16.222 LOG:::VCTouchesVSGestures___view:touchesBegan:withEvent:___<VCTouchesVSGestures: 0x7f9183ea8510>
2016-10-30 15:44:16.236 LOG:::VCTouchesVSGestures___view:touchesMoved:withEvent:___<VCTouchesVSGestures: 0x7f9183ea8510>
2016-10-30 15:44:16.267 LOG:::VCTouchesVSGestures___view:touchesMoved:withEvent:___<VCTouchesVSGestures: 0x7f9183ea8510>
....
....
2016-10-30 15:44:18.137 LOG:::VCTouchesVSGestures___view:touchesEnded:withEvent:___<VCTouchesVSGestures: 0x7f9183ea8510>
```
#####问题分析

1. 我们发现，在手势事件刚触发时，view便接到了touchesBegan事件，同时随着tap gesture被识别出，系统向view发送了touchesCancelled事件，于是view的touches响应就此结束
2. 如果我们的手势被解析失败，那么view的touches事件将会持续触发，直到截止

#####解决方案

UIGestureRecognizer类提供了三个属性来处理这两者之间的关系，它们分别是:

* cancelsTouchesInView

  默认为YES，当手势识别成功时，UIApplication将发送touchesCancel消息。

  设置为NO时，即使手势识别成功，UIView仍然会正常响应touches事件

* delaysTouchesBegan

  默认为NO，如果为YES，只要gestureRecognizer没有识别失败（识别中or识别成功），那么view永远接受不到touchesbegan。在识别失败的情况下，gesture绑定的view将会被触发touchesBegan（可能跟着一系列的touchesMove）

* delaysTouchesEnded

  默认为YES，其效果类似delaysTouchesBegan

有兴趣的读者可以将上述例子中的gesture的上述三个属性修改一下，查看效果

####UIGestureRecognizer和UIControl的冲突
点击程序的任意区域，都没有跳转提示，只有父view的背景不断变化，似乎子view添加的手势处理事件完全没有被触发。
#####问题分析
经过调试发现，完全没有响应子view的UIControl的touchUpInside事件，而是响应了父控件的tap事件。是因为UIGesture Recognizer的级别较高，而且由于cancelsTouchesInView默认是yes，所以UIControl不足以识别出touchUpInside事件，假如我们将UIControl的响应事件改成touchDown的话，你会发现，两者都响应。同样，保留touchUpInside将cancelsTouchesInView设置成NO，有同样的效果。
实际上，UIGestureRecognizer和UIControl的冲突本质上是UIGestureRecognizer和Touches响应的冲突，可参考上一节。
但是，并不是UIControl的所有子类都会遭遇同样的问题。假如我们把UIControl换成UIButton的话，只会响应UIButton的touchUpInside，不会响应父View的tap事件。原因见苹果官方说明：
> In iOS 6.0 and later, default control actions prevent overlapping gesture recognizer behavior. For example, the default action for a button is a single tap. If you have a single tap gesture recognizer attached to a button’s parent view, and the user taps the button, then the button’s action method receives the touch event instead of the gesture recognizer. This applies only to gesture recognition that overlaps the default action for a control, which includes:
> * A single finger single tap on a UIButton, UISwitch, UIStepper, UISegmentedControl, and UIPageControl.
> * A single finger swipe on the knob of a UISlider, in a direction parallel to the slider.
> * A single finger pan gesture on the knob of a UISwitch, in a direction parallel to the switch. 

#####解决方案

* 在父View使用UIGesture的情况下，子view也使用UIGesture或者UIButton, UISwitch, UIStepper, UISegmentedControl, and UIPageControl.控件。

####总结
1. UIGestureRecognizer 和 UITouch 都使用[HitTest](http://blog.csdn.net/woaihuangrong/article/details/52126346)机制来寻找响应控件,hitest机制是本文一切冲突的基础
2. 避免同时使用多套手势识别机制，在必须使用的时候，记得考虑上面提到的冲突
3. gesture recognizer 不参与响应链(这篇文章并未涉及,但是由于其高优先级，响应链对其根本无用武之地)
4. 尽量不要直接使用UIControl