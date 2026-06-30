---
layout:     post
title:      "iOS4 Issue: NSURLConnection and NSOperation"
date:       2010-12-19 23:06:00
author:     "Daniel Vela"
og_image: "/img/post-bg-02.jpg"
locale:       en
lang-ref:   ios-issue-connection-operations
---

[http://blog.mugunthkumar.com/coding/ios4-issue-nsurlconnection-and-nsoperation/](http://blog.mugunthkumar.com/coding/ios4-issue-nsurlconnection-and-nsoperation/)  

There is some bug in **iOS4** when calling **NSURLConnection** from a thead different from the main one. To solve this issue use the following code:  

	//iOS 4 bug fix  
	if (![NSThread isMainThread]) {  
		[self performSelectorOnMainThread:@selector(start)  
		withObject:nil waitUntilDone:NO];  
		return;  
	}  