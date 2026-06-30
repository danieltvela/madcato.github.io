---
layout:     post
title:      "Advanced Ruby: DSL"
date:       2020-10-16 05:44:00
author:     "Daniel Vela"
og_image: "/img/post-bg-05.jpg"
locale:       en
lang-ref:   advanced-ruby-8
---

# Advance Ruby: DSL

**DSL** means "Domain Specific Language". This king of programming languages are designed to be used in some king of specific subject: like maths, Machine Learning, networking, CI/CD, etc. 

A good example of a DSL is [fastlane](https://fastlane.tools). This tool is a great helper for developers in order to build and distribute mobile apps, and more. Here are a sample of this **DSL**.

```ruby
default_platform(:ios)

platform :ios do
  desc "Push a new beta build to TestFlight"
  lane :beta do
    build_app(workspace: "project-ios.xcworkspace", scheme: "project-ios")
    upload_to_testflight
  end
end
```

Then, in terminal you can execute `fastlane beta` to build project and publish it into TestFlight. Fastlane would ask for your "Apple Developer Program" credentials in order to have access to "App Store Connect". 

