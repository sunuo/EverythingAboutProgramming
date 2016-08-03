###Hit Testing 伪代码 及 IOS消息机制的相关问题

####写作原因
虽然[官方文档](https://developer.apple.com/library/prerelease/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html#//apple_ref/doc/uid/TP40009541-CH4-SW4)上给出了Hit Testing的大略描述,但是发现很多同学在使用中还会有各种疑惑以及错误，而且**任何自然语言的描述都比不上源代码更能让程序员信服**，so~~
####先上结果：
```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event
{
    if (![self pointInside:point withEvent:event]) {
        return nil;//如果这个点不在本身处理范围内，返回nil
    }

    NSArray* sortedSubViews=[self subviews];/*对subview按照index由大到小排序,index为视图层级，0为添加的第一个视图*/;

    for (UIView* subview in [self subviews]) {

        if(subview.hidden||subview.alpha < 0.01||subview.userInteractionEnabled==NO)
        {
            /*此处见文档
            This method ignores view objects that are hidden, that have disabled user interactions, or have an alpha level less than 0.01.
            */
            continue;
        }

		//将父视图的坐标点转换为子视图坐标点
        CGPoint covertPoint=[self convertPoint:point toView:subview];
		//递归子视图
        UIView* hitView=[subview hitTest:covertPoint withEvent:event];
        
        if (hitView) {
            return hitView;
        }
    }
    return self;
}
```
####实验：
#####实验一：Hit Testing

#####实验二：Responder Chain


#####实验三：Gesture Recognizer 与 touches:event 冲突
由于Gesture Recognizer对手势类型识别是需要时间的，所以会产生一些有趣的实验现象，
在实验对象一的Tap实验中，touch begin时还未检测出手势类型，无法处理，所以通过responder chain 传给了父视图，打印结touchesBegan:withEvent..
随后子视图识别除了Tap点击的模式，剩下的*事件*全被其绑定的UITapGestureRecognizer处理，所以，父视图没有机会输出touchesEnd:withEvent..

####总结:
1. 如果一个subview 返回结果为nil，那么这一个层级所有view都将不再被考虑
2. hitTest 最终返回的view也许并不能处理touch事件（例如UILabel），此时该事件将交由响应者链处理
3. Responsder chain 传递顺序，->父视图->controller->UIWindow->UIApplication->丢弃

