---
layout:     post
title:      "App Transport Security has blocked a cleartext HTTP prevent http queries to success"
date:       2016-03-23 10:02:00
author:     "Daniel Vela"
og_image: "/img/post-bg-01.jpg"
locale:       en
lang-ref:   app-transport-blocked-error
---

if you see this message un your app log:     
   
*App Transport Security has blocked a cleartext HTTP (http://) resource load since it is insecure. Temporary exceptions can be configured via your app's Info.plist file.*

You must add some configuration to your <project>.plist file.    

**Short version** allow all:      

	<key>NSAppTransportSecurity</key>
	<dict>
	  <!--Include to allow all connections (DANGER)-->
	  <key>NSAllowsArbitraryLoads</key>
	      <true/>
	</dict>	

**Extended version** allow only some domains:    

	<key>NSAppTransportSecurity</key>
	<dict>
	  <key>NSExceptionDomains</key>
	  <dict>
	    <key>yourserver.com</key>
	    <dict>
	      <!--Include to allow subdomains-->
	      <key>NSIncludesSubdomains</key>
	      <true/>
	      <!--Include to allow HTTP requests-->
	      <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
	      <true/>
	      <!--Include to specify minimum TLS version-->
	      <key>NSTemporaryExceptionMinimumTLSVersion</key>
	      <string>TLSv1.1</string>
	    </dict>
	  </dict>
	</dict>

[http://stackoverflow.com/questions/31254725/transport-security-has-blocked-a-cleartext-http](http://stackoverflow.com/questions/31254725/transport-security-has-blocked-a-cleartext-http)