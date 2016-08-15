###HitTest 应用
####扩大点击区域
这应该是最常用的功能了，在不改变控件现有frame布局的情况下，扩大控件的响应区域
```
@interface KVTouchExpandView : UIView
@property(nonatomic,assign)UIEdgeInsets expandInset;
@end

@implementation KVTouchExpandView
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event {
    CGPoint parentLocation = [self convertPoint:point toView:[self superview]];
    CGRect expandRect = self.frame;
    expandRect.origin.x -= _expandInset.left;
    expandRect.origin.y -= _expandInset.top;
    expandRect.size.width += (_expandInset.left + _expandInset.right);
    expandRect.size.height += (_expandInset.top + _expandInset.bottom);
    return CGRectContainsPoint(expandRect, parentLocation);
}
@end
```

####管理零碎控件（事件透传）

以一个典型的播放器操控UI为例
![dddd](http://img0.ph.126.net/9PRVsvwXNdd092tWcUPcWg==/6631554849351457191.jpg)
这个操控页面可以分为，三块topview、bottomview、其他，对于其他里面的三个零碎控件，若是把他们和topview以及bottomview平行的话，代码会很难看，而且不好管理,*对于比较零散的功能控件，可以创建一个承载view把它们聚合到一起,统一管理，承载view只有承载功能，触摸事件可被子视图捕获，承载view本身不响应任何事件*
```
- (UIView*)hitTest:(CGPoint)point withEvent:(UIEvent *)event
{
    UIView *view = [super hitTest:point withEvent:event];
    view = (view==self?nil:view);//view==self，说明子view都不响应该事件,作为承载view，不应该响应任何事件，所以置为nil
    return view;
}
```




