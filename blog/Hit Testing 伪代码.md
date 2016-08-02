###Hit Testing 伪代码

####写作原因
虽然[官方文档](https://developer.apple.com/library/prerelease/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html#//apple_ref/doc/uid/TP40009541-CH4-SW4)上给出了Hit Testing的大略描述,但是**任何自然语言的描述都比不上源代码更能让程序员信服**
[baidu](http://www.baidu.com)
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
            /*此处见文档[UIView]()
            This method ignores view objects that are hidden, that have 			disabled user interactions, or have an alpha level less than 			0.01.
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
既是上一章的推导过程，也是对其验证
见[附件]()
####注意事项:
1. 如果一个subview 返回结果为nil，那么这一个层级所有view都将不再被考虑
2. hitTest 最终返回的view也许并不能处理touch事件（例如UILabel），此时该事件将交由响应者链处理
3. 

