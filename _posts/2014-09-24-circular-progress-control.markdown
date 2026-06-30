---
layout:     post
title:      "Circular progress indicator control"
date:       2014-09-24 15:41:00
author:     "Daniel Vela"
og_image: "/img/post-bg-03.jpg"
locale:       en
lang-ref:   circular-progress-control
---

![ciruclar control]({{ site.url }}/assets/tumblr_nldejg9DJx1rjbh19o1_1280.png)

{% highlight objc %}
override func drawRect(rect: CGRect) {        
    let π = M_PI
    let context = UIGraphicsGetCurrentContext()
    CGContextSetLineWidth(context, 20.0)

    // Back circle
    CGContextSetStrokeColorWithColor(context,UIColor.blueColor().colorWithAlphaComponent(0.2).CGColor)
    CGContextAddArc(context, 150,150, 100, CGFloat(0), CGFloat(2 * π), 0)
    CGContextStrokePath(context)


    let radius = 100.0

    // Create the circle layer
    var circle = CAShapeLayer()

    // Set the center of the circle to be the center of the view
    let center = CGPointMake(150,150)

    let fractionOfCircle = 3.0 / 4.0

    let twoPi = 2.0 * Double(M_PI)
    // The starting angle is given by the fraction of the circle that the point is at, divided by 2 * Pi and less
    // We subtract M_PI_2 to rotate the circle 90 degrees to make it more intuitive (i.e. like a clock face with zero at the top, 1/4 at RHS, 1/2 at bottom, etc.)
    let startAngle = Double(fractionOfCircle) / Double(twoPi) - Double(M_PI_2)
    let endAngle = 0.0 - Double(M_PI_2)
    let clockwise: Bool = true

    // `clockwise` tells the circle whether to animate in a clockwise or anti clockwise direction
    circle.path = UIBezierPath(arcCenter: center, radius: CGFloat(radius), startAngle: CGFloat(startAngle), endAngle: CGFloat(endAngle), clockwise: clockwise).CGPath



    // Configure the circle
    circle.fillColor = UIColor.clearColor().CGColor
    circle.strokeColor = UIColor.blueColor().CGColor
    circle.lineWidth = 20.0
    circle.lineCap = kCALineCapRound

    // When it gets to the end of its animation, leave it at 0% stroke filled
    circle.strokeEnd = CGFloat(fractionOfCircle)

    // Add the circle to the parent layer
    self.layer.addSublayer(circle)

    // Configure the animation
    var drawAnimation = CABasicAnimation(keyPath: "strokeEnd")
    drawAnimation.repeatCount = 1.0

    // Animate from the full stroke being drawn to none of the stroke being drawn
    drawAnimation.fromValue = NSNumber(float: 0.0)
    drawAnimation.toValue = NSNumber(double: fractionOfCircle)

    drawAnimation.duration = 0.8

    drawAnimation.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut)

    // Add the animation to the circle
    circle.addAnimation(drawAnimation, forKey: "drawCircleAnimation")
}
{% endhighlight %}