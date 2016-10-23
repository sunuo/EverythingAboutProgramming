###HitTest 应用
####改变点击区域
这应该是最常用的功能了，在不改变控件现有frame布局的情况下，扩大缩小甚至定制控件的响应区域
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

####容器控件（事件透传）

很多时候，我们对于UIView，只想用它作为一个容器来管理众多零碎的空间,同时有不能阻碍其兄弟视图子视图及父视图的手势响应，可以用hitest进行过滤
```
- (UIView*)hitTest:(CGPoint)point withEvent:(UIEvent *)event
{
    UIView *view = [super hitTest:point withEvent:event];
    view = (view==self?nil:view);//view==self，说明子view都不响应该事件,作为承载view，不应该响应任何事件，所以置为nil
    return view;
}
```




