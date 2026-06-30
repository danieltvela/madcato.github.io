---
layout:     post
title:      "How to generate an static web from a Rails project"
date:       2016-09-16 12:02:00
author:     "Daniel Vela"
og_image: "/img/post-bg-06.jpg"
locale:       en
lang-ref:   generate-static-web-from-rails
---

**IMPORTANT:** *This method only works if the output static web files are served from a web server and the files are served from the root directory of the web server*

Precompile assets with:
{% highlight bash %}
  $ bundle exec rake assets:precompile
{% endhighlight %}

Open config/enviroments/production.rb and change the following attribute to **true**:
{% highlight ruby %}
  config.serve_static_assets = true
{% endhighlight %}

Run the server in production mode
{% highlight bash %}
  $ rails s -e production
{% endhighlight %}

then execute **wget** command:

{% highlight bash %}
  $ wget -P static -nH -m http://localhost:3000
{% endhighlight %}

Parameters:

* **-P** prefix directory. the base directory to write web
* **-nH** no-host-directories. Don't create the base direcotry with the site name 
* **-m** man mirror equivalent to -r -N -l inf --no-remove-listing



