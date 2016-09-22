###SDWebImage 加载gif动图的缺陷与解决方法
[SDWebImage](https://github.com/rs/SDWebImage.git) 集缓存管理、异步下载、下载次数控制和优化等功能于一身，基本上是IOS开发的一个标配组件了。然而，在实际开发过程中发现SDWebImage对动图gif的支持不尽如人意。
####示例图片如下：
![gif](http://img1.ph.126.net/1AOHxZrzrj1HfV4Xun3Ccw==/6631544953747346185.gif)

使用SDWebImage加载显示效果比实际效果要慢的多
![IOS](http://img2.ph.126.net/t_jKlUNKBerkz5a3Zd4k2g==/6631585635677588507.gif)

####原因分析
在SDWebImage的核心文件中有对UIImage的gif扩展，在这个扩展里面，程序获取到了gif每一帧图像以及对应的显示时间
```
for (size_t i = 0; i < count; i++) {
            //获取gif每一帧图像
            CGImageRef image = CGImageSourceCreateImageAtIndex(source, i, NULL);
            if (!image) {
                continue;
            }
			//sd_frameDurationAtIndex 获取每一帧图像对应的显示时间
            duration += [self sd_frameDurationAtIndex:i source:source];

            [images addObject:[UIImage imageWithCGImage:image scale:[UIScreen mainScreen].scale orientation:UIImageOrientationUp]];

            CGImageRelease(image);
        }

        if (!duration) {
            duration = (1.0f / 10.0f) * count;
        }
		//创建动图
        animatedImage = [UIImage animatedImageWithImages:images duration:duration];
```
问题出在***作者获取每一帧图像的显示时间的目的仅仅是为了计算gif动画的总时长，并没有给每一帧图像的显示时间分配相应的权重***，导致每一帧图像显示的时间为平均时间，视觉上给人带来了卡顿效果
####解决方法
解决方法其实很简单，由于系统提供的方法
```
+ (UIImage *)animatedImageWithImages:(NSArray<UIImage *> *)images duration:(NSTimeInterval)duration
```
会将duration平分给每一帧image，为了达到权重分配的效果，可以增加显示时间长的图片的帧数

github上有个[mayoff/uiimage-from-animated-gif](https://github.com/mayoff/uiimage-from-animated-gif.git)，就是使用了这个原理
我们将替换SDWebImage的`UIImage+GIF.m`文件的`sd_animatedGIFWithData`方法替换为新的方法即可
![UIImage+GIF.m](http://img1.ph.126.net/h0OKOaA-CCYSB3kfCQNl1Q==/6631476784026439534.png)
手懒的同学点这里下载替换好的文件**[UIImage+GIF.m](http://download.csdn.net/download/woaihuangrong/9621499)**
如果抛开对SDWebImage的依赖，还有[其他GIF的显示方法](http://www.2cto.com/kf/201603/491897.html)，不再一一赘述