---
layout:     post
title:      "How To upload a Rails app to Heroku"
date:       2012-04-01 10:07:00
author:     "Daniel Vela"
og_image: "/img/post-bg-03.jpg"
locale:       en
lang-ref:   ios-dimensions-1
---


1 First, configure your app to use PostgreSQL: Edit your Gemfile to change the line :
{% highlight ruby %}
gem 'sqlite3'
{% endhighlight %}
To this:
{% highlight ruby %}
gem 'pg'
{% endhighlight %}
and execute:
{% highlight ruby %}
$ bundle install
{% endhighlight %}
2 Second, use git to create a local repository:
{% highlight bash %}
$ git init
$ git add .
$ git commit -m "new app"
{% endhighlight %}
3 Third, install Heroku gem, if it isn’t installed yet:
{% highlight bash%}
$ gem install heroku
$ heroku list   
{% endhighlight %}
4 Now create the Heroku app:
{% highlight bash %}
$ heroku create
$ git push heroku master
$ heroku run rake db:migrate
{% endhighlight %}
5 To rename the default Heroku app name:
{% highlight bash %}
$ heroku rename <newname>
{% endhighlight %}
6 To create a custom domain for the app:
{% highlight bash %}
$ heroku addons:add cutom_domains
$ heroku domains:add <new.domain.com>
{% endhighlight %}

## Next steps

1. Develop and test locally
2. Commit to git every change
3. Push to Heroku with: $ git push heroku master

## Utilities

1. Rename Heroku app $ heroku rename
2. Custom domain $ heroku addons:add custom_domains $ heroku domains:add
3. Add the following text to Gemfile to user Thin server instead of WebBrick: gem ‘thin’
4. [How to install PostgreSQL on Mac](https://devcenter.heroku.com/articles/local-postgresql#mac-os-x)
5. Sample file config/database.yml to use PostgreSQL:

{% highlight yaml %}
development:
  adapter: postgresql
  encoding: unicode
  database: blog_development
  pool: 5
  username: <set $USER>
  password:
  host: localhost
test:
  adapter: postgresql
  encoding: unicode
  database: blog_test
  pool: 5
  username: <set $USER>
  password:
  host: localhost
{% endhighlight %}

For Heroku **production:** entry is not necessary, because Heroku assigns one PostgreSQL to each app.

