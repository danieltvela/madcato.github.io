---
layout:     post
title:      "‘Version’ and ‘Build’ numbers in the Xcode projects"
date:       2013-03-22 12:01:00
author:     "Daniel Vela"
og_image: "/img/post-bg-06.jpg"
locale:       en
lang-ref:   version-buid-xcode
---


Both codes are in the Xcode Settings:

![Xcode: Version and Build]({{ site.url}}/assets/tumblr_inline_mk26jyIqZ41qz4rgp.png)


Both can be modified in the configuration file *.plist* of any Xcode project:

* Version: **CFBundleShortVersionString**
* Build: **CFBundleVersion**

## Differences

Both numbers can be used the same way. Both can hold the same number, but that is not the original propouse of Apple.

* The **Version** number usually is an identificator we use to tell final users what is the funcitonality set of a software version: *Version 1.2.1*
* The  **Build** number usually indicates an incremental code to indicate the installation package is subsequent to others.

## For what these two numbers are used?

Regarding Apple documentation, these are the characteristics that must fulfill the version and buld numebrs:

**Version** usually are compund of one, two or three integer numbers separated by a coma:

1. The fist number indicates the **mayor version** of the software.
2. The second indicates a **minor version**, that is, some additional functionality of a mayor version.
3. The third number indicates a **fix**

As an example: version **1.1** is equal to the **1.0** version with some new functionality. Version **2** could be a big change in the functionality of the app: a bigger change, like a whole redesign.

If, as an example, some bugs are found in the version **1.2**, we could fix those bugs and launch version **1.2.1**.

**Build** usually is used as an incremental value to distinguish the installation packages independently of its version. It's usefull when **we had distributed different installation packages of the same version**. This happends frequently in testing teams and distributing betas through out users. If we make beta tests of a particular version (like 1.3.1), we'll have to know how to distinguish between a version created this last week and a package created last month of the same version. To avoid confusions is very useful to use a **Build** code.

> In your Mac, click over the Xcode menu the first option "About Xcode" you'll see both numbers: **Version** and **Build**. Example: Version 4.6.1 (4H512). Many other Apple apps use this scheme.

## When uploading apps to iTunes Connect

There are two important points regarding the upload of a new version. In the screens of [iTunes Connect](https://itunesconnect.apple.com/)

1. When you create a new version, we have to indicate our **Version** number. This number must be equal to the project one. If it's different, an error would happen uploading the app.
	![iTunes Connenct: new Version]({{ site.url }}/assets/tumblr_inline_mk26le1gVo1qz4rgp.png)
2. The **Build** of the project must be superior than the uploaded previously. If is equal of inferior, the package can't be uploaded: **you must increment always before upload it**.  You can see the number of the **Build** of your apps already uploaded in the section *“Binary Details”* in “*Summary*” 
	![iTunes Connenct: binary details]({{ site.url }}/assets/tumblr_inline_mk26kveFyQ1qz4rgp.png)

**Version** is a number better changed manually each time functionality is added. However the **Build** increment can be automated using *scripts*.

## Mas información

* [stackoverflow: Version vs Build in XCode 4](http://stackoverflow.com/questions/6851660/version-vs-build-in-xcode-4/6965086#6965086)
* [wikipedia:Software versioning](http://en.wikipedia.org/wiki/Software_versioning)
* [semver versioning](https://semver.org)