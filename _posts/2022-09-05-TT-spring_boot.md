---
toc: true
layout: post
title: Java Spring Backend
description: The Spring Framework is the backend Java development platform that provides comprehensive support for Java applications. Spring handles the complicated infrastructure problems so students can focus on the purpose of your application.  Spring Boot is a micro Service that is used to build stand-alone application.
permalink: /techtalk/spring_boot
image: /images/spring_boot.png
tags: [spring, spring boot]
type: pbl
week: 4
---

## Tour of Backend
The backend description for this blog is minimal as there is NO backend business logic, this will come with a blog focussing on Spring MVC.  Mastering Spring simple definitions is critical as the backend will get much more complicated.

### Main.java
Main.java contains the Class that is used to bootstrap and launch a Spring application from a Java main method.  By default it will load index.html.  

```java
// @SpringBootApplication annotation is key to building web applications with Java https://spring.io/projects/spring-boot
@SpringBootApplication
public class Main {

    // Starts a spring application as a stand-alone application from the main method
    public static void main(String[] args) {
        SpringApplication.run(Main.class, args);
    }

}
```

### Birds.java
This is about as simple as an @Controller can get, it loads a page for view only.  Note the @GetMapping which is used to associate a Web URL (/birds) with a Java Method (public String birds())

```java
@Controller  // HTTP requests are handled as a controller, using the @Controller annotation
public class Birds {

    // CONTROLLER handles GET request for /birds, maps it to birds() method
    @GetMapping("/birds")
    public String birds() {

        // load HTML VIEW (birds.html)
        return "birds";

    }
}
```

### Greet.java
Purpose of Greet is to allow user to change Hello, World! greeting.  This page is VERY important to learn in order to master passing data from frontend to backend using @RequestParam with associated keys and values.  The key "name" HTML input and is mapped to the Java using wrapper Class, String name.  In the method, String name is made back into key value using model.addAttribute("name", name).   In this example, nothing is altered in, but this provides feedback and response framework that can be used for more sophisticated applications.

```java
@Controller  // HTTP requests are handled as a controller, using the @Controller annotation
public class Greet {

    // @GetMapping handles GET request for /greet, maps it to greeting() method
    @GetMapping("/greet")
    // @RequestParam handles variables binding to frontend, defaults, etc
    public String greeting(@RequestParam(name="name", required=false, defaultValue="World") String name, Model model) {

        // model attributes are visible to Thymeleaf when HTML is "pre-processed"
        model.addAttribute("name", name);

        // load HTML VIEW (greet.html)
        return "greet"; 

    }

}
```

## Hacks
Using creativity and research, can you come up with something that alters Hello, World application.  Perhaps a calculation, like counting words or letters in a phrase.