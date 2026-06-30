---
layout:     post
title:      "Launch a clean Rails server every time the Xcode unit test are launched"
date:       2016-04-7 12:02:00
author:     "Daniel Vela"
og_image: "/img/post-bg-04.jpg"
locale:       en
lang-ref:   run-rails-from-tests
---

If you need to launch a Rails server in order to execute some Unit Tests from Xcode, you can use this method.

You must add this two scripts to the Scheme of the project, **Pre-actions** and **Post-actions** inside the Test section: 

![xcode test scheme scripts]({{ site.url }}/assets/xcode-test-acheme-scripts.png)


## Pre-actions

This is the script you must add to the **Pre-actions** section:


	osascript -e 'tell application "Terminal"' -e 'delay 0.5' -e "set currentTab to do script (\"cd $SRCROOT/__RAILSDIR__ && bundle exec rake db:reset RAILS_ENV=test && rails server -e test\")" -e 'end tell' &

*IMPORTANT: change __RAILSDIR__ string for the path where your rails project is stored. This script expects that this dir is a subdirectory of the xcode project directory. If the rails code is in other path, change cd $SRCROOT/__RAILSDIR__ for the complete path of the rails project.*

Note that the rails server is launched in the test enviroment, you can change this easily.

To allow the script to know the values of the project environment variables, you must provide the build settings, selecting the proper scheme:

![xcode test scheme scripts enviroment pre]({{ site.url }}/assets/script-rails-pre-action.png)

## Post-actions

This is the script you must add to the **Post-actions** section:


	osascript -e 'tell application "Terminal"' -e 'delay 0.5' -e "set currentTab to do script (\"cd $SRCROOT/__RAILSDIR__ && kill -INT \$(cat tmp/pids/server.pid)\")" -e 'end tell' &

*IMPORTANT: change __RAILSDIR__ string for the path where your rails project is stored. This script expects that this dir is a subdirectory of the xcode project directory. If the rails code is in other path, change cd $SRCROOT/__RAILSDIR__ for the complete path of the rails project.*

Note that the rails server is killed.

To allow the script to know the values of the project environment variables, you must provide the build settings, selecting the proper scheme:

![xcode test scheme scripts enviroment post]({{ site.url }}/assets/script-rails-post-action.png)


