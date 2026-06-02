---
layout:     post
title:      "Usando Ruby on Rails sin base de datos"
date:       2010-05-16 01:18:00
author:     "Daniel Vela"
background: "/img/post-bg-04.jpg"
locale:       en
lang-ref:   rails-sin-basededatos
---

<p><a href="http://blog.veladan.org/post/45344135930/usando-ruby-on-rails-sin-base-de-datos" class="tumblr_blog">danieltvela</a>:</p>

<blockquote><p>Ruby On Rails can be used without database. To do that, <strong>active_record</strong> gem must not be loaded. This gem is loaded automatically by Rails, to avoid that set this line in the file <strong><em>enviroment.rb</em></strong>:</p>

{% highlight ruby %}
config.frameworks -= [:active_record]  
{% endhighlight %}

<p>This way we can deactivate other unneeded gems like <strong>action_mailer</strong>.</p></blockquote>

