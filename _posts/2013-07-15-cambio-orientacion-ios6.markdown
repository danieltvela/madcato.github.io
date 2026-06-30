---
layout:     post
title:      "Orientation change on iOS 6"
date:       2013-07-15 11:56:00
author:     "Daniel Vela"
og_image: "/img/post-bg-05.jpg"
locale:       en
lang-ref:   cambio-orientacion-ios6
---

With iOS 6 Apple modified the way to control the views orientation. Now is managed by the UIWindow root controller.

When the root controller is an object of classes INavigationController or UITabBarController, we hava a problem. This controllers allow allways the orientation change.

To solve this problem use the following code:

{% highlight objc %}
#pragma mark - UITabBarController

@implementation UITabBarController (Rotation_IOS6)

-(BOOL)shouldAutorotate
{
    return [[self.viewControllers lastObject] shouldAutorotate];
}

-(NSUInteger)supportedInterfaceOrientations
{
    return [[self.viewControllers lastObject] supportedInterfaceOrientations];
}

- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation
{
    return [[self.viewControllers lastObject] preferredInterfaceOrientationForPresentation];
}

@end

#pragma mark - UINAviagationController

@implementation UINavigationController (Rotation_IOS6)

-(BOOL)shouldAutorotate
{
    return [[self.viewControllers lastObject] shouldAutorotate];
}

-(NSUInteger)supportedInterfaceOrientations
{
    return [[self.viewControllers lastObject] supportedInterfaceOrientations];
}

- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation
{
    return [[self.viewControllers lastObject] preferredInterfaceOrientationForPresentation];
}

@end

{% endhighlight %}