###IOS手势处理的那些坑
####前言
在不考虑内部实现机制的情况下，IOS手势处理有三种形式：
1. Gesture Recongnizers
2. touches 事件响应 touchesBegan、touchesEnd..
3. UIControl Target-Action

本文探讨了这几种处理手势事件的混合使用可能会产生的冲突情况，并提供了解决方法，希望后人不在踩这些坑。
####Touch Events 冲突
通常我们自定义一个View的时候，使用UIView作为其父类，通常我们只是将其本身作为一个承载子view的容器，但是很多情况下我们不得不对其进行拓展使其能够响应用户手势（例如点击跳转），有3个方案可以做到这点
1. 实现UIVIew的touch事件touchesEnded:withEvent
   如果父视图响应touchesBegan:withEvent 事件的话，可能引起其他问题
2. 绑定UIGesture Recognizer
3. 将父类从UIView改成UIControl
   如果父视图响应UIGesture Recongnizer ,见下一节 UIGestureRecognizer和UIControl的冲突
   这种方式没有问题，而且是苹果推荐的做法
####UIGestureRecognizer和UIControl的冲突
最近把程序中丑陋的touchesBegan:withEvent:,touchesMoved:withEvent:,touchesEnded:withEvent:,touchesCancelled:withEvent: 这一套换成了UIGesture家族之后，感觉代码清晰了很多，但是蓦然发现很多UI不能用了，研究发现，是UIGesture recognizer和UIControl的问题，[源代码](http)
######问题分析
在上述场景中，竟然没有响应子控件的UIControl的touchUpInside事件，而是响应了父控件的tap事件。是因为UIGesture Recognizer的级别较高，而且由于cancelsTouchesInView默认是yes，所以UIControl不足以识别出touchUpInside事件，假如我们将UIControl的响应事件改成touchDown的话，你会发现，两者都响应。同样，保留touchUpInside将cancelsTouchesInView设置成NO，有同样的效果。
但是，并不是UIControl的所有子类都会遭遇同样的问题。假如我们把UIControl换成UIButton的话，只会响应UIButton的touchUpInside，不会响应父View的tap事件。原因见苹果官方说明：
> In iOS 6.0 and later, default control actions prevent overlapping gesture recognizer behavior. For example, the default action for a button is a single tap. If you have a single tap gesture recognizer attached to a button’s parent view, and the user taps the button, then the button’s action method receives the touch event instead of the gesture recognizer. This applies only to gesture recognition that overlaps the default action for a control, which includes:
> * A single finger single tap on a UIButton, UISwitch, UIStepper, UISegmentedControl, and UIPageControl.
> * A single finger swipe on the knob of a UISlider, in a direction parallel to the slider.
> * A single finger pan gesture on the knob of a UISwitch, in a direction parallel to the switch. 

#####解决方案

在父View使用UIGesture的情况下，子view也使用UIGesture或者UIButton, UISwitch, UIStepper, UISegmentedControl, and UIPageControl.控件

<!--#####TouchEvent和UIControl-->
####总结
1. UIGestureRecognizer 和 UITouch 都使用[HitTest](http://blog.csdn.net/woaihuangrong/article/details/52126346)机制来寻找响应控件,hitest机制是本文一切冲突的基础
2. 避免同时使用多套手势识别机制，在必须使用的时候，记得考虑上面提到的冲突
3. gesture recognizer 不参与响应链
4. 能用UIButton解决的问题不要用UIControl
5. Apple 推荐使用Gesture recognizers
6. UIControl的子类在手势方面有特殊处理