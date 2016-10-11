###Gesture Recognizers 和 Touch-Event
####前言
IOS系统提供了两套处理手势的机制---Gesture Recognizers 和 Touch-Event。前者封装了常用的手势操作，使手势处理变的简单易用；后者更多的应用在自定义手势上。
今天我们来探讨一下这两者之间的关系。
####代码实例
####
####总结
1. UIGestureRecognizer 和 UITouch 都使用[++HitTest++](http://blog.csdn.net/woaihuangrong/article/details/52126346)机制来寻找响应控件
2. gestures 和 touches 的事件分发是独立的
	* 尽量不用同时使用这两套手势机制
	* 可以通过 cancelsTouchesInView、delaysTouchesBegan、delaysTouchesEnded来解决两者的冲突
	* Apple 推荐使用Gesture recognizers
3. gesture recognizer 不参与响应链
