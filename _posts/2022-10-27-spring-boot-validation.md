---
title: "Spring Boot - Complete Guide to Validations for REST API's"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - Java
  - Validation
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

* `@NotNull`: Used when a field must not be null.
* `@NotEmpty`: Used when a list field must not empty.
* `@NotBlank`: Used when a string field must not be the empty string (i.e. it must have at least one character).
* `@Min` and `@Max`: Used when a numerical field is only valid when it’s value is above or below a certain value.
* `@Pattern`: Used when a string field is only valid when it matches a certain regular expression.
* `@Email`: Used when a string field must be a valid email address.

An example of class with annotations :
```java
public class StudentDto {

    @NotBlank(message = "Should not be null or empty.")
    private String firstName;

    @Email(message = "Not valid.")
    private String email;

    @Size(min = 4, max = 10, message = "Should be min 4 character and max 10 character in length.")
    private String password;
}
```

# Validation of Inputs to a REST Controller

There are 3 things which are to be validated for a incoming REST request:

1. Request Body
2. Path Variables
3. Query Parameters

## Request Body Validation
In REST `POST` and `PUT` requests JSON payload in request body is mapped to Java objects. We need to define this Java object using Validation Constraints.

Let's define our `StudentDto` which would be mapped to incoming payload:
```java
public class StudentDto {

    private UUID uuid;

    @NotBlank(message = "Should not be null or empty.")
    private String firstName;

    @NotBlank(message = "Should not be null or empty.")
    private String lastName;

    @NotBlank(message = "Should not be null or empty.")
    @Email(message = "Not valid.")
    private String email;

    @NotBlank(message = "Should not be null or empty.")
    @Size(min = 4, max = 10, message = "Should be min 4 character and max 10 character in length.")
    private String password;

    private String confirmPassword;
}
```

Let's define the `StudentController`

```java
@RestController
@RequestMapping("/student")
public class StudentController {

    @PostMapping(
            consumes = MediaType.APPLICATION_JSON_VALUE,
            produces = MediaType.APPLICATION_JSON_VALUE
    )
    public ResponseEntity<StudentDto> create(@Valid @RequestBody StudentDto studentDto) {
        return ResponseEntity.ok(studentDto.setUuid(UUID.randomUUID()));
    }
}
```

* `@Valid` In order for Spring to use constraints defined for validation in DTO we need to annotate RequestBody in Rest Controller with `@Valid` annotation (line 9).
* When the validation fails `MethodArgumentNotValidException` is thrown. Spring by default translates this exception to HTTP 400.

**Validation for Complex Types**  
If the Input class contains a field with another complex type that needs to be validated, this field, too, should be annotated with `@Valid`.
{: .notice--info}

## Path Variable Validation
Path Variable are of simple types like `Integer`, `String` etc. so Validation Constraints are directly added to method parameters in Controllers.

```java
@Validated
@RestController
@RequestMapping("/student")
public class StudentController {

    @GetMapping(
            path = "/{uuid}",
            produces = MediaType.APPLICATION_JSON_VALUE
    )
    public ResponseEntity<StudentDto> getByUuid(@NotNull((message = "Should not be null or empty.") @PathVariable UUID uuid) {
        return ResponseEntity.ok(createStudentDto(UUID.fromString(uuid)));
    }
}
```
* line 1 `@Validated` annotation on class tells Spring to validate parameters that are passed into a method of the annotated class. Without this annotation Spring would not use Validation constraints applied on method parameters.
* line 10 `@NotNull` constraint defines that the UUID in path variable should not be null.
* When the validation fails `ConstraintViolationException` is thrown. Spring by default translates this exception to HTTP 500.

## Query Parameter Validation
Query parameters marked with `@RequestParam` are also of Simple type i.e. `Integer`, `String` etc. and follow same validation approach as for Path Variables.
Validation constraints are directly added to method parameter in Controller and controller class is marked with `@Validated` annotation.

```java
@Validated
@RestController
@RequestMapping("/student")
public class StudentController {
    @GetMapping(
            path = "/search",
            produces = MediaType.APPLICATION_JSON_VALUE
    )
    public ResponseEntity<StudentDto> get(@NotBlank(message = "Should not be null or empty.") @Email(message = "Not Valid.") @RequestParam String email) {
        return ResponseEntity.ok(createStudentDto(email));
    }
}
```
* Same as for Path Variables when Validation for Query Parameter fails `ConstraintViolationException` is thrown. Spring by default translates this exception to HTTP 500.

# Validation at Input to Service Layer
Inputs to any Spring Component could be validated using a combination of `@Validated` and `@Valid` annotations.

```java
@Service
@Validated
public class StudentService {

    public StudentDto create(@Valid StudentDto studentDto) {
        return studentDto.setUuid(UUID.randomUUID());
    }

}
```
* `@Validated` annotation should be added to class for which method parameters needs to be validated.
* `@Valid` annotation should be added to method parameters.
* When the validation fails `MethodArgumentNotValidException` is thrown. Spring by default translates this exception to HTTP 400.

# Handling Validation Errors
When a validation fails REST API should return a meaningful error message in response. 
Let's first define our `ErrorResponse` class.

```java
public class ErrorResponse {

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "dd-MM-yyyy hh:mm:ss")
    private ZonedDateTime timestamp;

    private Integer statusCode;

    private List<String> message;


    public static ErrorResponse createInstance() {
        return new ErrorResponse().setTimestamp(ZonedDateTime.now());
    }

    public ErrorResponse addMessage(String message) {
        if (this.message == null) {
            this.message = new ArrayList<>();
        }
        this.message.add(message);
        return this;
    }

    // ... Getters and Setters

}
```
Now we need to create a `@RestControllerAdvice` to catch all the exceptions cascading up to the controller layer.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ResponseBody
    ErrorResponse handleConstraintViolationException(ConstraintViolationException e) {
        ErrorResponse response = ErrorResponse.createInstance();
        e.getConstraintViolations()
         .forEach(constraintViolation -> response.addMessage(constraintViolation.getPropertyPath().toString() + " : " + constraintViolation.getMessage()));

        return response;
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ResponseBody
    ErrorResponse handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
        ErrorResponse response = ErrorResponse.createInstance();

        e.getBindingResult().getFieldErrors()
         .forEach(fieldError -> response.addMessage(fieldError.getField() + " : " + fieldError.getDefaultMessage()));

        return response;
    }

    @ExceptionHandler(BindException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ResponseBody
    ErrorResponse handleBindException(BindException e) {
        ErrorResponse response = ErrorResponse.createInstance();

        e.getBindingResult().getFieldErrors()
                 .forEach(fieldError -> response.addMessage(fieldError.getField() + " : " + fieldError.getDefaultMessage()));

        return response;
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ResponseBody
    ErrorResponse handleException(Exception e) {
        ErrorResponse response = ErrorResponse.createInstance();

        response.addMessage(e.getMessage());

        return response;
    }

}
```
* `@RestControllerAdvice` annotation on line 1 is specialization of `@Component` annotation and is auto-detected via classpath scanning. `@RestControllerAdvice` is the combination of both `@ControllerAdvice` and `@ResponseBody` which acts as interceptor that surrounds the logic in Controllers and allows us to apply some common logic to them. Here we are using this to create exceptions thrown from Controllers or any layer below them. This makes the exception handler methods available globally for all the Controllers.
* `@ExceptionHandler` annotation marks methods for capturing exceptions and translating them to HTTP responses. `@ExceptionHandler` annotation indicates which type of `Exception` can be handled by following methods. The exception instance and the request is injected via method arguments.
* `@ResponseStatus` annoation defines the HTTP response code returned by the method handler.
* On line 4, 15 and 27, `@ExceptionHandler` is used to define handlers for `ConstraintViolationException`, `MethodArgumentNotValidException` and `BindException` respectively, thrown on validation failures.
* On line 39, we define handler for `Exception`, if code throws some other exception not defined in other specific handler then error flow would default to this method for handling the exception.

# Summary

In this tutorial we have gone through all the common validation methods used for REST API's in spring boot applications and Defining custom error handling response with validation messages.

Complete code for above tutorial could be found on [GitHub](https://github.com/abhijitsinghas/tutorials/tree/main/validation-basics).
