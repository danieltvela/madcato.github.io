---
layout:     post
title:      "Internationalization for iOS"
date:       2011-01-12 10:11:00
author:     "Daniel Vela"
og_image: "/img/post-bg-04.jpg"
locale:       en
lang-ref:   internationalization-ios
---

# iOS iphone SDK internationalization

MainWindow.xib  

es
en
fr  

To add different language nibs, follow this steps:

1. Add the resource file to Xcode:
	1. Expand your project in the Groups &amp; Files pane.
	2. Select the Resources folder.
	3. Choose Project &gt; Add to Project….
	4. Use the file browser to navigate to the resource, select it, and click Open.
	5. In the dialog that Xcode displays, select “Copy items into destination group’s folder”, choose one of the “relative” reference styles (for example, “Relative to Project”), and select the desired target. Click Add.
2. Select the resource file and open the Inspector window (File &gt; Get Info).
3. If the file is not already localizable, click the Make File Localizable button in the General tab of the Inspector window.
4. In the General pane again, click the Add Localization button. In the sheet that appears, select or type the locale you want to add. Xcode copies the resource to the other .lproj directory.

If your resource file is a plain text file, you should be sure to set the File Encoding for each localized copy. You can set the default localization of the file before you make the file localizable. You can use different encodings for different localized versions of the file. To set the encoding for a specific localization, choose that localization in the Groups and Files pane and change the setting in the Inspector window.
The language identifiers for each localized nib are ISO names, like (es,en, es_es, es_pu, en_us, etc)  

[Preparing Your Nib Files for Localization](http://developer.apple.com/library/ios/#documentation/MacOSX/Conceptual/BPInternational/Articles/LocalizingInterfaces.html%23//apple_ref/doc/uid/20002138-BBCBFFDF)  
[Language and Locale Designations](http://developer.apple.com/library/ios/#documentation/MacOSX/Conceptual/BPInternational/Articles/LanguageDesignations.html%23//apple_ref/doc/uid/20002144-BBCEGGFF)