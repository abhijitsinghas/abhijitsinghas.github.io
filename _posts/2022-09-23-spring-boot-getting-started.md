---
title: "Spring Boot - Getting Started with REST"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - Java
published: false
---

<!-- # Spring Boot - Getting Started with REST -->

In this quick tutorial we'll see how to get started with creating Spring Boot project. We'll be creating a simple RESTful service using Spring Boot.
{: .text-justify}

## Setup
To follow along, as a prerequisite you'll need to install following software: 

1. [JDK 17](https://www.oracle.com/java/technologies/downloads/) or newer
2. [Maven 3.8.6](https://maven.apache.org/download.cgi) or newer
3. IDE of your choice, [IntelliJ IDEA](https://www.jetbrains.com/idea/download/), [Spring Tools 4](https://spring.io/tools) supported for Eclipse, Visual Studio Code and Theia.

## Project Structure 
Let's navigate to [Spring Initializr](https://start.spring.io) to generate our project's base structure.

[![Spring Initializr]({{site.baseurl}}/assets/spring-boot/01-getting-started/1-spring-initializr.png)]({{site.baseurl}}/assets/spring-boot/01-getting-started/1-spring-initializr.png)

Let the following default project settings be selected :
* Project : Maven Project  
* Langauage : Java  
* Spring Boot : 2.7.4 or newest stable build  

[![Project Settings]({{site.baseurl}}/assets/spring-boot/01-getting-started/2-project-settings.png)]({{site.baseurl}}/assets/spring-boot/01-getting-started/2-project-settings.png)

Update/following details in Project Metadata section :
* Group : `com.abhijits`. It is the group id, the usual convention is to use reverse order of your domain name. For example int our example it would be . If you have your company domain name or personal domain you can use it, if not then you can keep using the example domain as is.
* Artifact : `student-service`. It is the project name.
* Name : `student-service`. This is the name of your main Application Class.
* Description : `Spring Boot Restful API demo project`. About the project.
* Package Name : `com.abhijits.studentservice`. Combination of Group and Artifact froms your package name, it's automatically filled by Spring-Initializr
{: .text-justify}

[![Project Metadata]({{site.baseurl}}/assets/spring-boot/01-getting-started/3-project-metadata.png)]({{site.baseurl}}/assets/spring-boot/01-getting-started/3-project-metadata.png)

On right hand side of the page click on `Add Dependencies` button and select following dependencies:
  1. Spring Web
  2. Spring HATEOAS

[![Dependencies]({{site.baseurl}}/assets/spring-boot/01-getting-started/4-dependencies.png)]({{site.baseurl}}/assets/spring-boot/01-getting-started/4-dependencies.png)

Now click the `Generate` button at bottom of the page to download the project structure as a zip file.

[![Generate]({{site.baseurl}}/assets/spring-boot/01-getting-started/5-generate.png)]({{site.baseurl}}/assets/spring-boot/01-getting-started/5-generate.png)

Unzip the downloaded zip. You will see following project structure:

[![Director Structure]({{site.baseurl}}/assets/spring-boot/01-getting-started/6-projectstructure.png)]({{site.baseurl}}/assets/spring-boot/01-getting-started/6-projectstructure.png)

Open the project in IDE of your choosing. For demonstration we are using Intellij Idea.

[![Intellij Idea]({{site.baseurl}}/assets/spring-boot/01-getting-started/7-intellij-idea.png)]({{site.baseurl}}/assets/spring-boot/01-getting-started/7-intellij-idea.png)

## Application Main
Project contains `StudentServiceApplication` class which serves as the main and entry point for the Spring Boot application.


`@SpringBootApplication` serves as primary application configuration. Behind the scene it is equivalent to `@Configuration`, `@EnableAutoConfiguration` and `@ComponentScan`.
{: .notice--info}
1. `@Configuration` Tags the class as a source of bean definitions for the application context.
2. `@EnableAutoConfiguration` Tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.
3. `@ComponentScan` Tells Spring to look for other components, configurations, and services in the `com.abhijits.studentservice` package.

``` java
package com.abhijits.studentservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StudentServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(StudentServiceApplication.class, args);
	}

}
```

### Run the Application

| | **Terminal** |**IDE** |
| ---- | ---- | ---- |
| Start | `mvn spring-boot:run`| `Run` button |
| Stop | `Ctrl+c`| `Stop` button |

## Creating Rest Controller

1. Create a package `controller` inside your parent package.
2. Create a class `StudentController.java`.
3. Annotate class with `@RestController`


