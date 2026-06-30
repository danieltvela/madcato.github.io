---
layout:     post
title:      "Install Bootstrap into Rails apps"
date:       2015-06-29 17:15:00
author:     "Daniel Vela"
og_image: "/img/post-bg-04.jpg"
locale:       en
lang-ref:   bootstrap-rails
---


Actually you don't need gem for this, here is the step to install Bootstrap 3 in RoR

* [Download](https://github.com/twbs/bootstrap/releases/download/v3.0.0/bootstrap-3.0.0-dist.zip) Bootstrap
* Copy:    

bootstrap/dist/css/bootstrap.css and bootstrap/dist/css/bootstrap.min.css

To: vendor/assets/stylesheets

* Copy:     

bootstrap/dist/js/bootstrap.js and bootstrap/dist/js/bootstrap.min.js

To: vendor/assets/javascripts

* Update: app/assets/stylesheets/application.css by adding:
{% highlight css %}
*= require bootstrap.min
{% endhighlight %}

* Update: app/assets/javascripts/application.jsby adding:
{% highlight css %}
//= require bootstrap.min
{% endhighlight %}

With this you can update bootstrap any time you want, don't need to wait gem to be updated. Also with this approach assets pipeline will use minified versions in production. 


(Source: [http://stackoverflow.com/questions/18371318/installing-bootstrap-3-on-rails-app](http://stackoverflow.com/questions/18371318/installing-bootstrap-3-on-rails-app))