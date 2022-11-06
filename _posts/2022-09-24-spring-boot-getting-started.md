---
title: "Spring Boot - Getting Started with RESTful service"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - Java
---

# Overview
This is a quick tutorial on getting started with creating RESTful service with Spring Boot. This tutorial would be covering following aspects :
1. Generating Project using Spring Initializr.
2. Creating RESTful Controller.
3. DTO Validations for Requests.
4. Service Layer.
5. Spring JPA for Persistance using In Memory H2 DB.
6. Database Entities and Validations.
7. Exception handling using `@RestControllerAdvice`.

# Setup
To follow along, as a prerequisite you'll need to install following toolkits: 

1. [JDK 17](https://www.oracle.com/java/technologies/downloads/) or newer
2. [Maven 3.8.6](https://maven.apache.org/download.cgi) or newer
3. IDE of your choice, [IntelliJ IDEA](https://www.jetbrains.com/idea/download/), [Spring Tools 4](https://spring.io/tools) supported for Eclipse, Visual Studio Code and Theia.

# Creating Project
Navigate to [Spring Initializr](https://start.spring.io) to generate project's base structure.

Enter data for following fields :
* **Group** : Follows java package rules
* **Artifact** : Project Name
* **Name** : Name of the Applications Main Class
* **Description** : Project Description
* **Package Name** : Combination of **Group** and **Artifact**

Click on **`Add Dependencies`** button and search/select following dependencies:
1. Spring Web
2. Spring HATEOAS
3. Validation
4. Spring Data JPA
5. H2 Database

Spring Initializr window should look something like:

[![Spring Initializr]({{site.baseurl}}/assets/posts/01-spring-boot-getting-started/01-spring-initializr.png)]({{site.baseurl}}/assets/posts/01-spring-boot-getting-started/01-spring-initializr.png)

Now click on **`Generate`** button to download project structure. Unzip it and open the project in an IDE.

# Spring Boot Essentials

Project contains `StudentServiceApplication` class which serves as the main and entry point for the Spring Boot application.

`@SpringBootApplication` serves as primary application configuration. Behind the scene it is equivalent to `@Configuration`, `@EnableAutoConfiguration` and `@ComponentScan`.
{: .notice--info}
1. `@Configuration` tags the class as a source of bean definitions for the application context.
2. `@EnableAutoConfiguration` enables Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.
3. `@ComponentScan` enables Spring to look for other components, configurations, and services in the `com.abhijits.studentservice` package.

```java
@SpringBootApplication
public class StudentServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(StudentServiceApplication.class, args);
	}

}
```
Run the application on Terminal/Console using `mvn spring-boot:run` and stop using `Ctrl+c` or use IDE's `start` and `stop` button to run and stop application respectively.
{: .notice--info}

Successful execution will produce following logs for application start up:

[![Run Console]({{site.baseurl}}/assets/posts/01-spring-boot-getting-started/02-run-console.png)]({{site.baseurl}}/assets/posts/01-spring-boot-getting-started/02-run-console.png)

# Controller

Let's create a controller class `StudentController`.

## StudentController

```java
@RestController
@RequestMapping("/student")
public class StudentController {

    @GetMapping(
            path = "/{uuid}",
            produces = MediaType.APPLICATION_JSON_VALUE
    )
    public ResponseEntity<StudentDto> get(@PathVariable UUID uuid) {
        return ResponseEntity.ok().body(new StudentDto().setUuid(uuid)
                .setAge(18)
                .setGender(Gender.MALE)
                .setStandard(10)
                .setFirstName("Amit")
                .setLastName("Sharma")
        );
    }

    @PostMapping(
            consumes = MediaType.APPLICATION_JSON_VALUE,
            produces = MediaType.APPLICATION_JSON_VALUE
    )
    public ResponseEntity<StudentDto> create(@RequestBody StudentDto studentDto) {
        studentDto.setUuid(UUID.randomUUID());
        return ResponseEntity.created(
                WebMvcLinkBuilder.linkTo(
                        WebMvcLinkBuilder.methodOn(StudentController.class)
                                         .get(studentDto.getUuid())).toUri()
        ).body(studentDto);
    }

    @PatchMapping(
            path = "/{uuid}",
            consumes = MediaType.APPLICATION_JSON_VALUE
    )
    public ResponseEntity<StudentDto> update(@PathVariable UUID uuid, @RequestBody StudentDto studentDto) {
        return ResponseEntity.ok()
                             .body(studentDto.setUuid(uuid));
    }

    @DeleteMapping(
            path = "/{uuid}",
            consumes = MediaType.APPLICATION_JSON_VALUE
    )
    public ResponseEntity delete(@PathVariable UUID uuid) {
        return ResponseEntity.ok().build();
    }
}
```  

* `@RestController` on line 1, tells spring that this class is a Rest Controller. It is a Convenience Annotation which includes `@Controller` and `@ResponseBody` 
* `@RequestMapping` - Maps HTTP request with request handler methods. On line 2 `@RequestMapping("/student")` is going to redirect all the requests call starting with `/student` to methods contained in StudentController class.
* `@GetMapping` on line 5 is mapping the `get()` method to HTTP GET request with path variable `uuid`. As there is a `@RequestMapping` defined on class level so final path would be `/student/{uuid}` where `{uuid}` defines a place holder for actual uuid. `produces = MediaType.APPLICATION_JSON_VALUE` on line 7 defines the response content type and is mached against the `Accept` header of HTTP Request.
* `@PathVariable` on line 9 maps the place holder defined in `@GetMapping` on line 6 to corresponding java varialbe `UUID uuid`. If name of the corresponding java variable name is different from the place holder, then it could be mapped using `@PathVariable("uuid")`
* `ResponseEntity` class is used to fully configure the complete HTTP response: status code, headers and body. On line 9 `@ResponseEntity<StudentDto>` defines that body of the response would contain `StudentDto`.
* `@PostMapping` on line 16 is mapping `create()` method to POST `/student` request. No explicit path is defined on this annotation so path defined by `@RequestMapping` on class level at line 1 would be mapped.
* `WebMvcLinkBuilder` on line 26 is used to easily build `Link` instances pointing to Spring MVC controllers. The benefit of using this is that links are not hard coded and would be updated automatically if there are any changes in the method or the link associated with it.
* `@RequestBody` on line 23 maps the body of the POST request to `StudentDto`
* `@PatchMapping` on line 32 is mapping `update()` method to PATCH `/student/{uuid}` request. For this particular request we are using both `@PathVariable` and `@RequestBody` annotation to map path variable and request body contained in PATCH request.
* `@DeleteMapping` on line 41 is mapping `delete()` method to DELETE `/student/{uuid}`.

Now let's create `StudentDto` and `Gender` Enum used in `StudentController`

## StudentDto
``` java
public class StudentDto {

    private UUID uuid;
    private String firstName;
    private String lastName;
    private double age;
    private Gender gender;
    private int standard;

    public UUID getUuid() {
        return uuid;
    }

    public StudentDto setUuid(UUID uuid) {
        this.uuid = uuid;
        return this;
    }

    // ... add getter/setter for rest of the fields using above pattern where setter is returning this
    // ... add toString()

}
```

## Gender Enum
``` java
public enum Gender {

    MALE("MALE"),
    FEMALE("FEMALE"),
    OTHER("OTHER");

    private String value;

    private Gender(String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }

    public Gender setValue(String value) {
        this.value = value;
        return this;
    }

    @Override
    public String toString() {
        return "Gender{" +
                "value='" + value + '\'' +
                '}';
    }
}
```

# Test Run for Endpoints

Before trying `run` the applicaton let's comment following dependencies in our `pom.xml` file as our DB is not yet configured and running the application would throw error.

```xml
<!--		<dependency>-->
<!--			<groupId>org.springframework.boot</groupId>-->
<!--			<artifactId>spring-boot-starter-data-jpa</artifactId>-->
<!--		</dependency>-->
<!--		<dependency>-->
<!--			<groupId>com.h2database</groupId>-->
<!--			<artifactId>h2</artifactId>-->
<!--			<scope>runtime</scope>-->
<!--		</dependency>-->
```

Now let's Run the application and test the REST endpoints. You can either use `curl` to send http request or use `Postman` or any other http request tool of you choosing. In this blog we are defining the `curl` request for testing.

Getting familiarised with `curl` early on is benefitial as it can be used for testing/debugging REST requests inside containers/servers as well.
{: .notice--info}


## Create Student
Request :

`curl -v -X POST -d '{"firstName":"Varun","lastName":"Dube","age":18,"gender":"MALE","standard":10}' -H 'Content-Type: application/json' http://localhost:8080/student`

Response :
```console
< HTTP/1.1 201 
< Location: http://localhost:8080/student/1d9a0b9d-0832-4198-97dd-4235f568265b
< Content-Type: application/json
< Transfer-Encoding: chunked
< Date: Tue, 27 Sep 2022 07:36:14 GMT
< 
* Connection #0 to host localhost left intact
{"uuid":"1d9a0b9d-0832-4198-97dd-4235f568265b","firstName":"Varun","lastName":"Dube","age":18.0,"gender":"MALE","standard":10}
```


## Get Student
Request :

`curl -v -X GET -H 'Accept: application/json' http://localhost:8080/student/2577edba-5ade-4783-b653-0294bc2703e2`

Response :
```console
< HTTP/1.1 200 
< Content-Type: application/json
< Transfer-Encoding: chunked
< Date: Tue, 27 Sep 2022 09:27:23 GMT
< 
* Connection #0 to host localhost left intact
{"uuid":"2577edba-5ade-4783-b653-0294bc2703e2","firstName":"Amit","lastName":"Sharma","age":18.0,"gender":"MALE","standard":10}
```

## Update Student
Request :

`curl -v -X PATCH -d '{"firstName":"Varun","lastName":"Dube","age":40,"gender":"MALE","standard":12}' -H 'Content-Type: application/json' http://localhost:8080/student/fac201c6-353c-4c57-9349-0397df285901`

Response :
```console
< HTTP/1.1 200 
< Content-Type: application/json
< Transfer-Encoding: chunked
< Date: Tue, 27 Sep 2022 09:29:26 GMT
< 
* Connection #0 to host localhost left intact
{"uuid":"fac201c6-353c-4c57-9349-0397df285901","firstName":"Varun","lastName":"Dube","age":40.0,"gender":"MALE","standard":12}
```

## Delete Student
Request :

`curl -v -X DELETE -H 'Content-Type: application/json' http://localhost:8080/student/fac201c6-353c-4c57-9349-0397df285901 `

Response :
```console
< HTTP/1.1 200 
< Content-Length: 0
< Date: Tue, 27 Sep 2022 09:31:12 GMT
< 
* Connection #0 to host localhost left intact
```

# DTO Validation

Now let's try to execute following POST request where names are empty.

`curl -v -X POST -d '{"firstName":"","lastName":"","age":18,"gender":"MALE","standard":10}' -H 'Content-Type: application/json' http://localhost:8080/student`

Application provided following response:
```console
{"uuid":"7974baf3-c6f2-4784-b74b-3d41ecb18b68","firstName":"","lastName":"","age":18.0,"gender":"MALE","standard":10}%
```

This is not a valid request as `firstName` and `lastName` should not be null or empty and should be a valid name.

Let's add validations on our `StudentDto`.

```java
public class StudentDto {

    private UUID uuid;

    @NotBlank(message = "'firstName' should not be Empty or null.")
    @Pattern(regexp = "^[A-Za-z]+$", message = "'firstName' should only contain alphabets.")
    private String firstName;

    @NotBlank(message = "'firstName' should not be Empty or null.")
    @Pattern(regexp = "^[A-Za-z]+$", message = "'firstName' should only contain alphabets.")
    private String lastName;

    @NotNull(message = "'age' should not be null.")
    @Min(value = 4, message = "'age' minimum valid age is 4.")
    @Max(value = 99, message = "'age' maximum valid age is 99.")
    private double age;

    @NotNull(message = "'gender' should not be null.")
    private Gender gender;

    @NotNull(message = "'standard' should not be null.")
    @Min(value = 1, message = "'standard' minimum valid value is 1.")
    @Max(value = 12, message = "'standard' maximum valid value is 12.")
    private int standard;

}
```

* `@NotBlank` is applied on CharSequence fields and validates that the field is not null and trimmed length is greater than zero.
* `@Pattern` is also applied to CharSequence fields and validates that field values match the provided regex.
* `@NotNull` validates that the field is not null.
* `@Min` and `@Max` are used on number fields to validate minimum and maximum allowed values.

In order for these validations to be used we need to annotate DTO in our controller methods with `@Valid`. Update the `create()` and `update()` methods as follows:

1. `public ResponseEntity<StudentDto> create(@Valid @RequestBody StudentDto studentDto)`
2. `public ResponseEntity<StudentDto> update(@PathVariable UUID uuid, @Valid @RequestBody StudentDto studentDto)`

Let's not again run the application and fire post request with invalid data.

`curl -v -X POST -d '{"firstName":"","lastName":"","age":18,"gender":"MALE","standard":10}' -H 'Content-Type: application/json' http://localhost:8080/student`

Application is now returning Bad Request 400 in response.

```console
< HTTP/1.1 400 
< Content-Type: application/json
< Transfer-Encoding: chunked
< Date: Sun, 02 Oct 2022 00:57:24 GMT
< Connection: close
< 
* Closing connection 0
{"timestamp":"2022-10-02T00:57:24.809+00:00","status":400,"error":"Bad Request","path":"/student"}
```

Let's not connect our applicaton with database, then later we will use `@RestControllerAdvice` to return custom error response.

# Database Configurations

Let's add/uncomment JPA dependencies in pom.xml file.
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```
You can either use `application.properties` file or `application.yml` and add following properties to configure in memory H2 database. For this blog we are using `application.yml` file.

```yml
spring:
  datasource:
    url: jdbc:h2:mem:mydb
    username: admin
    password: password
    driverClassName: org.h2.Driver
  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
  h2:
    console:
      enabled: true
```

* Above configs define the `url`, `username`, `password` and `driverClassName` used to access the database. 
* `spring.jpa.database-platform` is used to set the database `Dialect`.
* `h2.console.enabled` is used to enable the h2 database console for debugging. After running the application console can be accessed at `http://localhost:8080/h2-console`

# Entity and Repository

Let's create entity class to map with database table.

```java
@Entity
public class Student {

    @Id
    @Column(columnDefinition = "UUID")
    private UUID uuid = UUID.randomUUID();

    @Column(nullable = false)
    private String firstName;

    @Column
    private String lastName;

    @Column(nullable = false)
    private double age;

    @Column(nullable = false)
    @Enumerated(EnumType.ORDINAL)
    private Gender gender;

    @Column(nullable = false)
    private int standard;

    // Add getter's, setter's and toString() method
}
```
* `@Entity` on line 1 defines that `Student` class is an Entity and is mapped to a database table.
* `@Id` on line 4 specifies the primary key for the table.
* `@Column` is used to map variable to corresponding column name in table. `nullable` parameter defines if the column could have `null` values, default is `true`. `columnDefinition` on line 5 is used to define database column type. This is not usually required but in current version of hibernate mapping for H2 database `UUID` column type is not automatically inferred, so needs to be defined explicitly.

Now let's create `StudentRepository` class extending from `JpaRepository` for accessing database methods.

```java
public interface StudentRepository extends JpaRepository<Student, UUID> {
}
```

Extending `StudentRepository` from `JpaRepository` adds some basic CRUD methods and functionality. 


# Service and Transformer
Now let's create a `StudentService` class

```java
@Service
public class StudentService {

    private final StudentRepository studentRepository;

    public StudentService(StudentRepository studentRepository) {
        this.studentRepository = studentRepository;
    }

    public Student get(UUID uuid) {
        return studentRepository.findById(uuid).orElseThrow(() -> new IllegalArgumentException("Student not found."));
    }

    public Student create(Student student) {
        return studentRepository.save(student);
    }

    public void delete(UUID uuid) {
        studentRepository.deleteById(uuid);
    }

    public Student update(Student student) {
        return studentRepository.save(
                get(student.getUuid())
                        .setFirstName(student.getFirstName())
                        .setLastName(student.getLastName())
                        .setGender(student.getGender())
                        .setAge(student.getAge())
                        .setStandard(student.getStandard())
        );
    }
}
```

* `@Service` annotation on line 1 is a specialization of `@Component` annotation and defines that the class is a service layer and does not encapsulate any state.
* `StudentService` class is using `constructor` based injection for `StudentRepository`. `@Autowired` attonation is used to mark dependency injection, but this annotation is optional for constructor injection.

Let's create `Transformer` interface to convert DTO to Entity and vice-versa.

```java
public interface Transformer<TDomain, TDto> {
    TDto toDto (TDomain entity);

    TDomain toEntity(TDto dto);
}
```

Now let's extend `Transformer` interface to create `StudentTransformer`.

```java
@Component
public class StudentTransformer implements Transformer<Student, StudentDto> {

    @Override
    public StudentDto toDto(Student student) {
        return new StudentDto()
                .setUuid(student.getUuid())
                .setFirstName(student.getFirstName())
                .setLastName(student.getLastName())
                .setGender(student.getGender())
                .setAge(student.getAge())
                .setStandard(student.getStandard());
    }

    @Override
    public Student toEntity(StudentDto studentDto) {
        return new Student()
                .setFirstName(studentDto.getFirstName())
                .setLastName(studentDto.getLastName())
                .setAge(studentDto.getAge())
                .setGender(studentDto.getGender())
                .setStandard(studentDto.getStandard());
    }
}
```
# Controller Update
Now let's wire the `StudentService` and `StudentTransformer` in `StudentController` and update the `get()`, `create()`, `update()` and `delete()` methods.

```java
@RestController
@RequestMapping("/student")
public class StudentController {

    private final StudentService studentService;

    private final StudentTransformer studentTransformer;

    public StudentController(StudentService studentService, StudentTransformer studentTransformer) {
        this.studentService = studentService;
        this.studentTransformer = studentTransformer;
    }

    @GetMapping(
            path = "/{uuid}",
            produces = MediaType.APPLICATION_JSON_VALUE
    )
    public ResponseEntity<StudentDto> get(@PathVariable UUID uuid) {
        return ResponseEntity.ok().body(studentTransformer.toDto(studentService.get(uuid)));
    }

    @PostMapping(
            consumes = MediaType.APPLICATION_JSON_VALUE,
            produces = MediaType.APPLICATION_JSON_VALUE
    )
    public ResponseEntity<StudentDto> create(@Valid @RequestBody StudentDto studentDto) {
        StudentDto responseStudentDto = studentTransformer.toDto(studentService.create(studentTransformer.toEntity(studentDto)));
        return ResponseEntity.created(
                WebMvcLinkBuilder.linkTo(
                        WebMvcLinkBuilder.methodOn(StudentController.class)
                                         .get(responseStudentDto.getUuid())).toUri()
        ).body(responseStudentDto);
    }

    @PatchMapping(
            path = "/{uuid}",
            consumes = MediaType.APPLICATION_JSON_VALUE
    )
    public ResponseEntity<StudentDto> update(@PathVariable UUID uuid, @Valid @RequestBody StudentDto studentDto) {
        return ResponseEntity.ok()
                             .body(studentTransformer.toDto(studentService.update(studentTransformer.toEntity(studentDto).setUuid(uuid))));
    }

    @DeleteMapping(
            path = "/{uuid}",
            consumes = MediaType.APPLICATION_JSON_VALUE
    )
    public ResponseEntity delete(@PathVariable UUID uuid) {
        studentService.delete(uuid);
        return ResponseEntity.ok().build();
    }
}
```

Let's run the application and navigate to `http://localhost:8080/h2-console` in browser.

Enter database `username` and `password` set in `application.yml` file and click on `Connect` button.

[![H2 Console]({{site.baseurl}}/assets/posts/01-spring-boot-getting-started/03-h2-console.png)]({{site.baseurl}}/assets/posts/01-spring-boot-getting-started/03-h2-console.png)

Let's now send a post request to application.

`curl -v -X POST -d '{"firstName":"Varun","lastName":"Dube","age":18,"gender":"MALE","standard":10}' -H 'Content-Type: application/json' http://localhost:8080/student`

Getting following response in console.

```console
< HTTP/1.1 201 
< Location: http://localhost:8080/student/b83f54d9-b9db-4bef-877f-43e50243c5ea
< Content-Type: application/json
< Transfer-Encoding: chunked
< Date: Sun, 02 Oct 2022 01:49:47 GMT
< 
* Connection #0 to host localhost left intact
{"uuid":"b83f54d9-b9db-4bef-877f-43e50243c5ea","firstName":"Varun","lastName":"Dube","age":18.0,"gender":"MALE","standard":10}
```

Now let verify in h2 console that data is actually entered in DB. Enter following query in `SQL Statement` text box.

```sql
SELECT * FROM student;
```

Click on the `Run` button.

Results in Database should match response in your POST request.

[![H2 Console SQL]({{site.baseurl}}/assets/posts/01-spring-boot-getting-started/04-h2-console-sql.png)]({{site.baseurl}}/assets/posts/01-spring-boot-getting-started/04-h2-console-sql.png)

# Error Handling

Let's first create `ErrorResponse` class. This would define the format returned when error has occured.

```java
public class ErrorResponse {

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "dd-MM-yyyy hh:mm:ss")
    private ZonedDateTime timestamp;

    private Integer statusCode;

    private List<String> message;

    private String stackTrace;

    public static ErrorResponse createInstance() {
        return new ErrorResponse().setTimestamp(ZonedDateTime.now());
    }

    public List<String> getMessage() {
        return message;
    }

    public ErrorResponse setMessage(List<String> message) {
        this.message = message;
        return this;
    }

    public ErrorResponse addMessage(String message) {
        if (this.message == null) {
            this.message = new ArrayList<>();
        }
        this.message.add(message);
        return this;
    }
    // Getter's/Setter's for remaining private fields
}
```

`@JsonFormat` on line 3 defines the format to serialize timestamp field.

Let's create a class `GlobalExceptionHandler`
```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @Value("${stackTrace.enabled}")
    private boolean printStackTrace;

    @Override
    public ResponseEntity<Object> handleExceptionInternal(
            Exception ex,
            Object body,
            HttpHeaders headers,
            HttpStatus status,
            WebRequest request) {

        return buildErrorResponse(ex, status);
    }

    @ExceptionHandler(value = { Exception.class })
    protected ResponseEntity<Object> handleAllException(Exception ex) {
        ResponseStatus responseStatus = ex.getClass().getAnnotation(ResponseStatus.class);
        if (responseStatus != null)
            return buildErrorResponse(ex, responseStatus.value());
        return buildErrorResponse(ex, HttpStatus.INTERNAL_SERVER_ERROR);
    }

    private ResponseEntity<Object> buildErrorResponse(Exception exception, HttpStatus httpStatus) {

        ErrorResponse errorResponse = ErrorResponse.createInstance()
                                                   .setStatusCode(httpStatus.value());

        if (exception instanceof MethodArgumentNotValidException) {
            errorResponse.setMessage(((MethodArgumentNotValidException) exception).getBindingResult().getAllErrors().stream().map(
                    MessageSourceResolvable::getDefaultMessage).collect(Collectors.toList()));
        } else {
            errorResponse.addMessage(exception.getMessage());
        }

        if (printStackTrace) {
            errorResponse.setStackTrace(getStackTrace(exception));
        }

        return ResponseEntity.status(httpStatus).body(errorResponse);
    }

    private String getStackTrace(Exception exception) {
        StringWriter stringWriter = new StringWriter();
        PrintWriter printWriter = new PrintWriter(stringWriter);
        exception.printStackTrace(printWriter);
        return stringWriter.toString();
    }

}
```

* `@RestControllerAdvice` annotation on line 1 is specialization of `@Component` annotation and is auto-detected via classpath scanning. `@RestControllerAdvice` is the combination of both `@ControllerAdvice` and `@ResponseBody` which acts as interceptor that surrounds the logic in Controllers and allows us to apply some common logic to them.  
* Rest Controller Advice’s methods (annotated with `@ExceptionHandler`) are shared globally across multiple `@Controller` components to capture exceptions and translate them to HTTP responses. 
* `@ExceptionHandler` on line number 18, marks `handleAllException()` method for capturing exceptions and translating them to HTTP responses. This method is shared globally across multiple `@Controller`s. The `@ExceptionHandler` annotation indicates which type of `Exception` can be handled. The exception instance and the request is injected via method arguments.
* By using two annotations together, we can:
    1. control the body of the response along with status code
    2. handle several exceptions in the same method
* `@Value` annotation on line 4, defines a property driven dependency injection where `printStackTrace` varialbe recieves value from the property file.
* `@GlobalExceptionHandler` is extended from `ResponseEntityExceptionHandler`, which is a convenience base class for `@ControllerAdvice` to provide centralized exception handling across all `@RequestMapping` methods through `@ExceptionHandler` methods.

Add following property to `application.yml` file to enable/disable returning of stacktrace in error response.
```yml
 stackTrace:
  enabled: false
```

 Let's create a `NotFoundException` and throw it from `StudentService` class when get does not find anything.
 ```java
 @ResponseStatus(HttpStatus.NOT_FOUND)
public class NotFoundException extends RuntimeException {

    public NotFoundException() {
        super();
    }

    public NotFoundException(String message) {
        super(message);
    }
}
```
`@ResponseStatus` annotation defines the HttpStatus code which this exception is translated too.

Update `StudentService` class to use `NotFoundException`.
```java
   public Student get(UUID uuid) {
        return studentRepository.findById(uuid).orElseThrow(() -> new NotFoundException("Student not found."));
    }
```

Now let's again try to test with Invalid POST request.

`curl -v -X POST -d '{"firstName":"","lastName":"","age":18,"gender":"MALE","standard":20}' -H 'Content-Type: application/json' http://localhost:8080/student`

Response is 400 Bad Request.

```console
< HTTP/1.1 400 
< Content-Type: application/json
< Transfer-Encoding: chunked
< Date: Sun, 02 Oct 2022 01:25:39 GMT
< Connection: close
< 
* Closing connection 0
{"timestamp":"02-10-2022 06:55:39","statusCode":400,"message":["'firstName' should not be Empty or null.","'lastName' should only contain alphabets.","'firstName' should only contain alphabets.","'standard' maximum valid age is 12.","'lastName' should not be Empty or null."],"stackTrace":null}%    
```

Let's `enable` stackTrace in `application.yml`
```yml
 stackTrace:
  enabled: true
```
Get request for data is not available.

`curl -v -X GET -H 'Accept: application/json' http://localhost:8080/student/2577edba-5ade-4783-b653-0294bc2703e2`

Response is a 404 Not Found with complete stackTrace.

```console
< HTTP/1.1 404 
< Content-Type: application/json
< Transfer-Encoding: chunked
< Date: Sun, 02 Oct 2022 01:34:56 GMT
< 
{"timestamp":"02-10-2022 07:04:56","statusCode":404,"message":["Student not found."],"stackTrace":"com.abhijits.studentservice.errorhandling.exceptions.NotFoundException: Student not found.\n\tat com.abhijits.studentservice.service.StudentService.lambda$get$0(StudentService.java:23)\n\tat java.base/java.util.Optional.orElseThrow(Optional.java:403)\n\tat com.abhijits.studentservice.service.StudentService.get(StudentService.java:23)\n\tat com.abhijits.studentservice.controller.StudentController.get(StudentController.java:36)\n\tat java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)\n\tat java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)\n\tat java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)\n\tat java.base/java.lang.reflect.Method.invoke(Method.java:568)\n\tat org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:205)\n\tat org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:150)\n\tat org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:117)\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:895)\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:808)\n\tat org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)\n\tat org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1071)\n\tat org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:964)\n\tat org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)\n\tat org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:898)\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:655)\n\tat org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:764)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:227)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)\n\tat org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)\n\tat org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)\n\tat org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)\n\tat org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)\n\tat org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:197)\n\tat org.apache.catalina.core.Standa* Connection #0 to host localhost left intact rdContextValve.invoke(StandardContextValve.java:97)\n\tat org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:541)\n\tat org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:135)\n\tat org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)\n\tat org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:78)\n\tat org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:360)\n\tat org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:399)\n\tat org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)\n\tat org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:890)\n\tat org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1789)\n\tat org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)\n\tat org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1191)\n\tat org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)\n\tat org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)\n\tat java.base/java.lang.Thread.run(Thread.java:833)\n"}%  
```

# Summary

Congratulations! now you have successfully created a SpringBoot based REST Application which is able to perform basic CRUD operations, along with basic error handling. 

Complete code for above tutorial could be found on [Github](https://github.com/abhijitsinghas/sb-student-service).