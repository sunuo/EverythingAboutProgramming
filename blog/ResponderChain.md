UIGesture

#####实验三：Gesture Recognizer 与 touches:event 冲突
由于Gesture Recognizer对手势类型识别是需要时间的，所以会产生一些有趣的实验现象，
在实验对象一的Tap实验中，touch begin时还未检测出手势类型，无法处理，所以通过responder chain 传给了父视图，打印结touchesBegan:withEvent..
随后子视图识别除了Tap点击的模式，剩下的*事件*全被其绑定的UITapGestureRecognizer处理，所以，父视图没有机会输出touchesEnd:withEvent..