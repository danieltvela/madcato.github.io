---
layout:     post
title:      "Floating UIPickerView like the Safari one"
date:       2011-02-22 19:59:00
author:     "Daniel Vela"
og_image: "/img/post-bg-06.jpg"
locale:       en
lang-ref:   floating-view
---

![floating view]({{ site.url}}/assets/tumblr_inline_mjpfqfkirm1qz4rgp.png)

To create a floating UIPickerView like the one that is included in Safari use the following code:

{% highlight objc %}
//  
// FloatingPickerView.h  
// AppearingPickerView  
//  
// Created by Daniel Vela on 2/22/11.  
// Copyright 2011 velada.org . All rights reserved.  
//  
#import   
@protocol FloatingPickerViewDelegate  
-(void)donePressed;  
@end  

@interface FloatingPickerView : NSObject {  
    id delegate;  
    id dataSource;  
    id delegate2;  
    UIViewController* controller;  
    UIActionSheet* aac;  
    UIPickerView* pickerView;  
    NSInteger selectRowRow;  
    NSInteger selectRowComponent;  
    BOOL selectRowAnimated;  
}  
@property (nonatomic, retain) UIPickerView* pickerView;  
@property NSInteger selectRowRow;  
@property NSInteger selectRowComponent;  
@property BOOL selectRowAnimated;  
-(id)initWithController:(UIViewController*)control  
                     withDelegate:(id) dele  
                 withDataSource:(id) data  
                  withDelegate2:(id) dele2;  
- (void)showPicker;  
- (void)donePressed;  
- (void)selectRow:(NSInteger)row 
            inComponent:(NSInteger)component 
               animated:(BOOL)animated;  
- (void)doSelectRow;  
@end  

//  
// FloatingPickerView.m  
// AppearingPickerView  
//  
// Created by Daniel Vela on 2/22/11.  
// Copyright 2011 veladan.org . All rights reserved.  
//  

#import "FloatingPickerView.h"  

@implementation FloatingPickerView  
@synthesize pickerView;  
@synthesize selectRowRow;  
@synthesize selectRowComponent;  
@synthesize selectRowAnimated;  

#pragma mark -  
#pragma mark initialization  
-(id)initWithController:(UIViewController*)control  
           withDelegate:(id) dele  
                 withDataSource:(id) data  
                  withDelegate2:(id) dele2 {  
    controller = control;  
    delegate = dele;  
    dataSource = data;  
    delegate2 = dele2;  
    return self;  
}  

- (void)showPicker {  
    aac = [[UIActionSheet alloc] initWithTitle:@""  
                                                                        delegate:self  
                                                     cancelButtonTitle:nil  
                                          destructiveButtonTitle:nil  
                                                     otherButtonTitles:nil];  
    aac.actionSheetStyle = UIActionSheetStyleBlackTranslucent;  
    pickerView = [[UIPickerView alloc] initWithFrame:  
                                                                    CGRectMake(0.0, 44, 0.0, 0.0)];  
    pickerView.delegate = delegate;  
    pickerView.dataSource = dataSource;  
    pickerView.userInteractionEnabled = YES;  
    pickerView.showsSelectionIndicator = YES;  
    [aac addSubview:pickerView];   
    UISegmentedControl* doneButton = [[UISegmentedControl alloc] initWithItems:  
                                                                                    [NSArray arrayWithObject:@"OK"]];  
  doneButton.momentary = YES;  
    doneButton.frame = CGRectMake(260,7.0f, 50.0f, 30.0f);  
    doneButton.segmentedControlStyle = UISegmentedControlStyleBar;  
    doneButton.tintColor = [UIColor blueColor];  
    [doneButton addTarget:self action:@selector(donePressed) forControlEvents:UIControlEventValueChanged];  
    [aac addSubview:doneButton];  
    [aac showInView:controller.view];  
    [UIView beginAnimations:nil context:nil];  
    [aac setBounds:CGRectMake(0, 0, 320, 496)];  
    [UIView commitAnimations];  
    [doneButton release];  
} 

-(void)donePressed {  
    [aac dismissWithClickedButtonIndex:0 animated:YES];  
    [aac removeFromSuperview];  
    [aac autorelease];  
    [delegate2 donePressed];  
} 

- (void)selectRow:(NSInteger)row 
          inComponent:(NSInteger)component 
                 animated:(BOOL)animated {  
    // If you call selectRow from UIPickerView while calling a delegate method, the   
    // data will be not displayed.  
    // This cheap trick allows to select a row animated.  
    self.selectRowRow = row;  
    self.selectRowComponent = component;  
    self.selectRowAnimated = animated;  
    [self performSelector:@selector(doSelectRow) withObject:nil afterDelay:0.5];   
}

-(void)doSelectRow {  
    [self.pickerView selectRow:self.selectRowRow   
    inComponent:self.selectRowComponent   
    animated:self.selectRowAnimated];  
}  
{% endhighlight %}

Example of use:

{% highlight objc %}
pickerView = [[FloatingPickerView alloc] 
                            initWithController:sharedAppDelegate.myTabBarController 
                                  withDelegate:self 
                                    withDataSource:self
                                     withDelegate2:self];  
[pickerView showPicker];  
if(updateArticle != nil) {  
    NSInteger selectId = [dm.typesArray indexOfObject:updateArticle.type];  
    [pickerView selectRow:selectId inComponent:0 animated:YES];  
}
{% endhighlight %}