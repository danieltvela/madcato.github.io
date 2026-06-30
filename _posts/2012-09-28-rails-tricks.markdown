---
layout:     post
title:      "Rails tricks"
date:       2012-09-28 10:07:00
author:     "Daniel Vela"
og_image: "/img/post-bg-03.jpg"
locale:       en
lang-ref:   rails-tricks
---



---
(Mac only) When trying to execute **“$ bundle install”** and the next error occurs: **“can’t find header files for ruby at /System/Library/Frameworks/Ruby.framework/Versions/1.8/usr/lib/ruby/ruby.h”** you must install **“Command Line Tools of Xcode”**.

---
Use $ bundle exec \ to execute any command (like rake) using the gems defined in the Gemfile of your project.

---
Every thing os generated with:
{% highlight bash %}
$ rails generate
{% endhighlight %}

can be undone with:
{% highlight bash %}
$rails destroy
{% endhighlight %}

**$ rale db:migrate** it’s undone with **$ rake db:rollback**

---

**$ rails generate controller PageController method1 method2** to generate static pages

---

[Upload to Heroku](http://blog.veladan.org/2012/04/how-to-upload-rails-app-to-heroku.html)

---

**scss - sass** app/assets/stylesheets/

{% highlight scss %}
variable: $grayMediumLight: #eaeaea;
tree:
.center {
  text-align: center;
  h1 {
    margin-bottom: 10px;
  }
}
@mixin box {
 -moz-box-sizing:border-box;
}
.class1 {
 clear:both;
 @include box;
}
{% endhighlight %}

---

**rake db:reset** # empty db and reload migrations

---

{% highlight ruby %}
SampleApp::Application.configure do
  .
  .
  .
  # Force all access to the app over SSL, use Strict-Transport-Security,
  # and use secure cookies.
  config.force_ssl = true
  .
  .
  .
end
{% endhighlight %}