---
layout:     post
title:      "Implement http digest authentication with Rails"
date:       2010-05-30 03:27:00
author:     "Daniel Vela"
og_image: "/img/post-bg-05.jpg"
locale:       en
lang-ref:   atutenticaion-digest-rails
---

In order to require an http connection to authenticate using **HTTP Digest** in Rails, execute the following code:

	# ... ApplicationController class
	private  
		DUsers = {"user" => "password"}  
		def digest_authenticate  
			realm = "application"  
			authenticate_or_request_with_http_digest(realm) do |name|  
			DUsers[name]  
		end  
	end  

... and execute this code before the invocation to our controller endpoint:

	 class ExampleController < ApplicationController  
	 before_filter :digest_authenticate, :only => ['index']  

	 	def index  
	 	end  
	 end  