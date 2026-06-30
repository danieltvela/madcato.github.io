---
layout:     post
title:      "Objective-C Singleton"
date:       2011-02-15 14:56:00
author:     "Daniel Vela"
og_image: "/img/post-bg-03.jpg"
locale:       en
lang-ref:   objectivec-singleton
---

Include the following declarations in an existing class:

{% highlight objc %}
// Declaration "YourClass.h"  
@interface YourClass {  
}  
+ (YourClass*)sharedInstance;  
@end  

// Implementation "YourClass.m"  
#import "YourClass.h"  

static YourClass *sharedInstance = nil;  

@implementation YourClass  
#pragma mark -  
#pragma mark Singleton methods  
+ (YourClass*)sharedInstance {  
    @synchronized(self)  {  
        if (sharedInstance == nil) {  
            sharedInstance = [[YourClass alloc] init];  
            // Some extra initialization can be done here  
        }  
    }  
    return sharedInstance;  
}  

+ (id)allocWithZone:(NSZone *)zone {  
    @synchronized(self) {  
        if (sharedInstance == nil) {  
            sharedInstance = [super allocWithZone:zone];  
            return sharedInstance; // assignment and return on first allocation  
        }  
    }  
    return nil; // on subsequent allocation attempts return nil  
}  

- (id)copyWithZone:(NSZone *)zone {  
    return self;  
}  

- (id)retain {  
    return self;  
}  

- (NSUInteger)retainCount {  
    return UINT_MAX; // denotes an object that cannot be released  
} 

- (void)release {  
    //do nothing  
} 

- (id)autorelease {  
    return self;  
} 

@end
{% endhighlight %}