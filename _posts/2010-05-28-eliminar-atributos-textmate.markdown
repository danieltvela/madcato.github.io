---
layout:     post
title:      "Eliminate extended attributes of files saved with Textmate"
date:       2010-05-28 20:13:00
author:     "Daniel Vela"
og_image: "/img/post-bg-04.jpg"
locale:       en
lang-ref:   eliminar-atributos-textmate
---

[Macromates](http://manual.macromates.com/en/saving_files.html)  

To eliminate the extended attributes of **Textmate** execute the following command on Terminal: 

	defaults write com.macromates.textmate OakDocumentDisableFSMetaData 1  

This will eliminate the **@** of the permissions of the files, those which we can see executing `ls -al` (-rw-r--r--@)
Also this will allow to copy this files to systems that doesn't suppport extended attributes (like Samba), and not see files ***._filename*** created at all.
