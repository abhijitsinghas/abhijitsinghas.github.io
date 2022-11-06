---
title: "Spring Boot - Complete Guide to Validations for REST API's"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - Java
  - Validation
published: false
---

# Overview
Generally there is a need to validate user Input to an application. The de-facto standard for doing so in Spring is to use Hibernate Validators, reference implementation for Bean Validation Framework. This it is integrated quite well in Spring and Spring Boot.

# Dependencies
To use the validation framework in our application we need to add following dependencies to our project.

``` maven
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-validation</artifactId> 
</dependency>
```

# Bean Validation
Bean Validation works by adding constraint annotations on the fields of the class to be validated. Commonly used annotations are defined in [javax.validation.constraint](https://docs.jboss.org/hibernate/beanvalidation/spec/2.0/api/javax/validation/constraints/package-summary.html) package.



