---
layout:     post
title:      "Git for Mac"
date:       2011-10-08 10:55:00
author:     "Daniel Vela"
og_image: "/img/post-bg-05.jpg"
locale:       en
lang-ref:   git-for-mac
---

## 1. Install git from:

{% highlight bash %}
git-osx-installer for Mac OS X
{% endhighlight %}

## 2. Configure:

{% highlight bash %}
$git config --global user.name "Your name here"   
$git config --global user.email "yourmail@here.com"   
{% endhighlight %}

**optional**    

{% highlight bash %}
$git config --global merge.tool vimdiff    
$git config --global core.editor vim   
$git config --global color.ui true   
{% endhighlight %}

## 3. Commands:

* Help: 
{% highlight bash %}
$ git help   
{% endhighlight %}
* Initialization: In the project root directory execute this:
{% highlight bash %}
$ git init
{% endhighlight %}
* Common usage:
{% highlight bash %}
$ git add  # add new files or modified ones to be commited
$ git commit -m "message"
{% endhighlight %}
* Delete last 3 commits:
{% highlight bash %}
$ git reset --hard HEAD~3  
{% endhighlight %}
* Undo last changes:
{% highlight bash %}
$ git revert  # This option creates a new commit with the last changes removed  
{% endhighlight %}

## 4. Branches

* Create branch

{% highlight bash %}
$ git branch   
{% endhighlight %}

* Change current branch

{% highlight bash %}
$ git checkout   
{% endhighlight %}

* Merge two branches

{% highlight bash %}
$ git merge   # Merges current branch with   
{% endhighlight %}

* Delete branch

{% highlight bash %}
$ git branch -d   
{% endhighlight %}

## 5. Tags

* Create tag

{% highlight bash %}
 $ git tag   
 # The tag names can be used with other   
 # git commands like $ git checkout   
{% endhighlight %}

## 6. Remote repository

* Configure remote repository

{% highlight bash %}
$ git remote add origin git@github.com:madcat/cachirulo.git  
{% endhighlight %}

* Send files to remote repository

{% highlight bash %}
$ git push -u origin master   
{% endhighlight %}

* Retrive files from remote repository

{% highlight bash %}
$ git pull   
{% endhighlight %}

## 7. Misc

* Hide the current changes

{% highlight bash %}
$ git stash  
{% endhighlight %}

* Recover last hidden changes

{% highlight bash %}
$ git stash pop     
{% endhighlight %}

* Download all tracked branches

{% highlight bash %}
$ git remote update      
$ git pull --all            
{% endhighlight %}

* To track a branch

{% highlight bash %}
$ git checkout -b localbranchName origin/remotebranchname        
{% endhighlight %}

* List all branches, even those not tracked

{% highlight bash %}
$ git branch -a
{% endhighlight %}
