---
layout:     post
title:      "Aspect-oriented programming"
date:       2021-07-20 08:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-06.jpg"
locale:       en
lang-ref:   aspect-oriented-programming
---

As you can see in the [Wikipedia](https://en.wikipedia.org/wiki/Aspect_(computer_programming)), an aspect is: 

> An aspect of a program is a feature linked to many other parts of the program, but which is not related to the program's primary function. An aspect crosscuts the program's core concerns, therefore violating its separation of concerns that tries to encapsulate unrelated functions. For example, logging code can crosscut many modules, yet the aspect of logging should be separate from the functional concerns of the module it cross-cuts. Isolating such aspects as logging and persistence from business logic is at the core of the aspect-oriented programming (AOP) paradigm.[1]

But, as always, things are much better understood with an example...

#### Imagine that you are a young developer, in your first job, and...

... your boss ask you to create a function that add two numbers. (This is for a _very_ special client)

Then you program it and publish it in the company application server.

```java
public static int addTwoNumbers(int a, int b) {
  int sum = a + b;
  return sum;
}
```

Wow, your first day!

#### Next day, someone that presents himself as the systems operator, ...

... ask you to write a log line, because in that company _"every function has a log"_ and _"every one does it this way"_.

Then you type:

```java
public static int addTwoNumbers(int a, int b) {
  // Print log
  LOGGER.info("Funtion addTwoNumbers called, params {} and {}", a, b);

  int sum = a + b;
  return sum;
}
```

#### That same afternoon, someone that presents himself as the security officer, ...

... ask you to check if the user is logged, because in that company _"every function check it"_ and _"every one does it this way"_.

```java
public static int addTwoNumbers(int a, int b) throws {
  // Print log
  LOGGER.info("Funtion addTwoNumbers called, params {} and {}", a, b);
  // Check auth
  HttpSession session = request.getSession(false);
  if (session == null || session.getAttribute("loggedInUser") == null) {
    throw new UserNotAuthenticated();
  } 

  int sum = a + b;
  return sum;
}
```

... a few minutes later, the same guy returns and ask you to check the authorization level is enouh, because in that company _"every function check it"_ and _"every one does it this way"_.

```java
public static int addTwoNumbers(int a, int b) throws {\
  // Print log
  LOGGER.info("Funtion addTwoNumbers called, params {} and {}", a, b);
  // Check auth
  HttpSession session = request.getSession(false);
  if (session == null || session.getAttribute("loggedInUser") == null) {
    throw new UserNotAuthenticated();
  }
  // Check authorization
  int level = session.getAttribute("loggedUserAuthLevel")
  if level < LEVEL_7) {
    throw new UserAuthorizationNotEnough();
  }

  int sum = a + b;
  return sum;
}
```

#### Next day, someone that presents himself as the operations manager, ...

... ask you to save the results in a database, because in that company _"every function save it"_ and _"every one does it this way"_.

```java
public static int addTwoNumbers(int a, int b) throws {
  // Print log
  LOGGER.info("Funtion addTwoNumbers called, params {} and {}", a, b);
  // Check auth
  HttpSession session = request.getSession(false);
  if (session == null || session.getAttribute("loggedInUser") == null) {
    throw new UserNotAuthenticated();
  }
  // Check authorization
  int level = session.getAttribute("loggedUserAuthLevel")
  if level < LEVEL_7) {
    throw new UserAuthorizationNotEnough();
  }

  int sum = a + b;

  // Save on db
  String sql = "Insert into addTwoNumebrTable(a,b,result) values(?,?,?)";
  pst = conn.prepareStatement(sql);
  pst.setInt(1, a);
  pst.setInt(2, b);
  pst.setInt(3, sum);
  pst.execute();
  return sum;
}
```

#### Next day, someone that presents himself as marketing, ...

... ask you to send the function information to exploit the data. You answer that the information can found in hte log and also in the explotation database, but he answer: _"we are marketing, we need Google Analytics"_.

... and the you add it.

```java
public static int addTwoNumbers(int a, int b) throws {
  // Print log
  LOGGER.info("Funtion addTwoNumbers called, params {} and {}", a, b);
  // Check auth
  HttpSession session = request.getSession(false);
  if (session == null || session.getAttribute("loggedInUser") == null) {
    throw new UserNotAuthenticated();
  }
  // Check authorization
  int level = session.getAttribute("loggedUserAuthLevel")
  if level < LEVEL_7) {
    throw new UserAuthorizationNotEnough();
  }

  int sum = a + b;

  // Save on db
  String sql = "Insert into addTwoNumebrTable(a,b,result) values(?,?,?)";
  pst = conn.prepareStatement(sql);
  pst.setInt(1, a);
  pst.setInt(2, b);
  pst.setInt(3, sum);
  pst.execute();

  // Google Analytics
  Bundle parameters = new Bundle();
  params.putString("function", "addTwoNumbers");
  params.putInt("a", a);
  params.putInt("b", b);
  params.putInt("sum", sum);
  analytics.setDefaultEventParameters(parameters);
  return sum;
}
```

## And then, you read your own code and ...

![What the fuck am I looking at](/assets/wtf.jpg)

***This shit was a function to add two numbers!!!***

# And the worst of this code can't be seen between these lines, the worst is that the rest of project is the same.

Someone may think that this project requires an OOP framework that can solve auhtentication, authorization, persistence, logging, in a more designable way. But this approach tend to lack of the flexibility needed to add new kinds of functionality once the desing is settled.

There is other way to solve this: **Aspect-oriented programming**

## First: what's an aspect.

The definition in the wikipedia is perfectly correct. Any kind of code that affects the project in a transversal way, can be considered an aspect. Not everything is an aspect, there are some aspects of programming that are evident. Among them we find:
* log
* persistence
* authentication
* authorization
* the input / output console
* analytics
* transactions

## Then: what's aspect-oriented programming

In a simple way to say it: aspect-oriented programming is a way to add some functionality to an existing function, without editing or modifying it.

There are some techniques to accomplish this. One of them is the [interceptor pattern](https://en.wikipedia.org/wiki/Interceptor_pattern), other is the [Decorator pattern](https://en.wikipedia.org/wiki/Decorator_pattern) and almost every programming language has its own [aspect implementation](https://en.wikipedia.org/wiki/Aspect-oriented_programming#Implementations).

To understand this type of programming, first you must change your mind in the way you solve algorithms. Specially if you are used to do [OOP](https://en.wikipedia.org/wiki/Object-oriented_programming) and apply [Design Patterns](https://en.wikipedia.org/wiki/Design_pattern), because this king of programming requires to know every aspect you are going to use before implementing the project.

In Aspect-oriented programming, instead defining an structure of classes that interact to solve transversal functionality, you can design the bussines model and logic, and then add the different aspects to that functionality. [Wikipedia sample code is really good to see this way to solve use cases.](https://en.wikipedia.org/wiki/Aspect-oriented_programming#Motivation_and_basic_concepts)

I like this way to program for this reason:
- It's easier to add new functionality without modifing code.
- It doesn't require injections.
- Functions are much much easier to test.

What negative point are?
- This code is hard to debug, because most of languages are not desinged to do this king of progrramming. (Almost only Python and Ruby were created thinking in this king of programming)