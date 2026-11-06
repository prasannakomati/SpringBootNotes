# Student Project -- Create Student Feature

This document contains the code for the current **Create Student**
feature in the Spring Boot project.

> **Code preservation:** The Java code and configuration values below
> are transcribed from the code you provided. No logic, class names,
> field names, endpoint names, dependency versions, or configuration
> values have been intentionally changed.

## 1. Project structure

``` text
student-project
└── src
    └── main
        ├── java
        │   └── com.example.studentproject
        │       ├── controller
        │       │   └── StudentController.java
        │       ├── entity
        │       │   └── Student.java
        │       ├── repository
        │       │   └── StudentJpaRepository.java
        │       ├── service
        │       │   └── StudentService.java
        │       └── StudentProjectApplication.java
        └── resources
            └── application.properties
└── pom.xml
```

## 2. StudentController.java

**Location:**
`src/main/java/com/example/studentproject/controller/StudentController.java`

``` java
package com.example.studentproject.controller;


import com.example.studentproject.entity.Student;
import com.example.studentproject.repository.StudentJpaRepository;
import com.example.studentproject.service.StudentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/students")
public class StudentController {

    @Autowired
    private StudentService studentService;

    @PostMapping
    public Student createStudent(@RequestBody Student student)
    {
        return studentService.createStudent(student);
    }


}
```

### Explanation

-   `@RestController`: Marks this class as a REST API controller. The
    returned Java object is written as a response body, normally as
    JSON.
-   `@RequestMapping("/students")`: Sets `/students` as the base URL for
    the endpoints in this controller.
-   `@Autowired`: Injects the `StudentService` object into the
    controller.
-   `@PostMapping`: Handles HTTP POST requests at `/students`.
-   `@RequestBody`: Converts the JSON request body into a `Student` Java
    object.
-   `studentService.createStudent(student)`: Sends the student data to
    the service layer and returns the saved student.

**Note:** `StudentJpaRepository` is imported in this controller but is
not used in the shown code. It is retained exactly as provided.

## 3. Student.java

**Location:**
`src/main/java/com/example/studentproject/entity/Student.java`

``` java
package com.example.studentproject.entity;


import jakarta.persistence.*;

@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;

    private String password;

    private String department;

    // Getters and Setters

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getDepartment() {
        return department;
    }

    public void setDepartment(String department) {
        this.department = department;
    }
}
```

### Explanation

-   `@Entity`: Tells JPA that this class represents a database entity (a
    table).
-   `@Id`: Marks `id` as the primary key.
-   `@GeneratedValue(strategy = GenerationType.IDENTITY)`: Lets the
    database generate the ID when a row is inserted.
-   `id`: Stores the student's unique ID.
-   `name`: Stores the student's name.
-   `email`: Stores the student's email address.
-   `password`: Stores the password field.
-   `department`: Stores the student's department.
-   Getters read field values; setters assign field values.

**Security note:** This code currently stores and returns the `password`
field as part of the entity. For a real application, do not store
plain-text passwords or return them in API responses. Use password
hashing and a response DTO. This is a note only; the supplied code above
has not been changed.

## 4. StudentJpaRepository.java

**Location:**
`src/main/java/com/example/studentproject/repository/StudentJpaRepository.java`

``` java
package com.example.studentproject.repository;


import com.example.studentproject.entity.Student;
import org.springframework.data.jpa.repository.JpaRepository;

public interface StudentJpaRepository extends JpaRepository<Student, Long> {

}
```

### Explanation

-   `JpaRepository<Student, Long>` provides common database operations
    for the `Student` entity.
-   `Student` is the entity type.
-   `Long` is the Java type of the entity's primary key (`id`).
-   Spring Data JPA provides the repository implementation at runtime;
    you do not write the basic `save()` implementation yourself.
-   The service uses `save(student)` to insert the student.

## 5. StudentService.java

**Location:**
`src/main/java/com/example/studentproject/service/StudentService.java`

``` java
package com.example.studentproject.service;


import com.example.studentproject.entity.Student;
import com.example.studentproject.repository.StudentJpaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class StudentService {

    @Autowired
    private StudentJpaRepository studentRepository;

    public Student createStudent(Student student) {

        return studentRepository.save(student);
    }
}
```

### Explanation

-   `@Service`: Marks this class as the service layer component.
-   `@Autowired`: Injects the repository into the service.
-   `createStudent(Student student)`: Receives the student object from
    the controller.
-   `studentRepository.save(student)`: Saves the student entity in the
    database and returns the saved entity.

## 6. StudentProjectApplication.java

**Location:**
`src/main/java/com/example/studentproject/StudentProjectApplication.java`

``` java
package com.example.studentproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StudentProjectApplication {

    public static void main(String[] args)
    {
        SpringApplication.run(StudentProjectApplication.class, args);
    }

}
```

### Explanation

-   `@SpringBootApplication`: Enables Spring Boot auto-configuration and
    component scanning, and marks this class as the application
    configuration entry point.
-   `main()`: The Java application entry point.
-   `SpringApplication.run(...)`: Starts the Spring Boot application.

## 7. application.properties

**Location:** `src/main/resources/application.properties`

``` properties
spring.application.name=student-project


spring.datasource.url=jdbc:h2:mem:employee_db
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=


#JPA
spring.jpa.hibernate.ddl-auto=create
spring.jpa.show-sql=true

#H2 CONSOLE

spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

### Explanation

-   `spring.application.name`: Sets the application name.
-   `spring.datasource.url=jdbc:h2:mem:employee_db`: Creates/uses an
    in-memory H2 database named `employee_db`.
-   `spring.datasource.driver-class-name`: Specifies the H2 JDBC driver.
-   `spring.datasource.username` and `spring.datasource.password`: Set
    the database login; here the username is `sa` and the password is
    blank.
-   `spring.jpa.hibernate.ddl-auto=create`: Creates the database schema
    when the application starts. Because this is an in-memory database,
    its data is not persistent across application restarts.
-   `spring.jpa.show-sql=true`: Prints SQL statements in the application
    console.
-   `spring.h2.console.enabled=true`: Enables the H2 web console.
-   `spring.h2.console.path=/h2-console`: Sets the H2 console path.

## 8. pom.xml

**Location:** `pom.xml`

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.2</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>student-project</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name/>
    <description/>
    <url/>
    <licenses>
        <license/>
    </licenses>
    <developers>
        <developer/>
    </developers>
    <scm>
        <connection/>
        <developerConnection/>
        <tag/>
        <url/>
    </scm>
    <properties>
        <java.version>17</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-h2console</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webmvc</artifactId>
        </dependency>

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webmvc-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

### Explanation

-   `<parent>`: Inherits dependency and plugin management from Spring
    Boot.
-   `<version>4.1.2</version>`: The Spring Boot parent version specified
    in your file.
-   `<java.version>17</java.version>`: Sets the Java version property to
    17.
-   `spring-boot-starter-data-jpa`: Adds Spring Data JPA and Hibernate
    integration.
-   `spring-boot-starter-webmvc`: Adds Spring MVC web application
    support.
-   `h2`: Adds the H2 database at runtime.
-   The two `*-test` dependencies provide test support.
-   `spring-boot-maven-plugin`: Provides Spring Boot Maven build/run
    support.

**Version check:** The `pom.xml` is reproduced as provided. I have not
verified whether every dependency artifact listed is available for the
specified Spring Boot version. If Maven reports a dependency-resolution
error, check the artifact names and version against the official Spring
Boot documentation before changing anything.

## 9. How the Create Student request works

``` text
Postman / Client
      |
      | POST /students
      | JSON request body
      v
StudentController
      |
      | studentService.createStudent(student)
      v
StudentService
      |
      | studentRepository.save(student)
      v
StudentJpaRepository
      |
      v
H2 Database
      |
      | Saved Student
      v
JSON response to client
```

### Step-by-step flow

1.  The client sends an HTTP `POST` request to `/students`.
2.  The request body contains student details in JSON format.
3.  `@RequestBody` converts the JSON into a `Student` object.
4.  The controller calls `studentService.createStudent(student)`.
5.  The service calls `studentRepository.save(student)`.
6.  Spring Data JPA saves the entity in the H2 database.
7.  The saved `Student` object is returned to the controller.
8.  Spring MVC converts the returned object into a JSON response.

## 10. Test the API using Postman

**Method:** `POST`

**URL:** `http://localhost:8080/students`

**Body → raw → JSON:**

``` json
{
  "name": "Prasanna Kumar",
  "email": "prasanna@example.com",
  "password": "test123",
  "department": "CSE"
}
```

The `id` is not included in the request because it is generated by the
database.

A successful response should contain the saved student fields and the
generated ID. The exact ID value is assigned by the database.

## 11. H2 Console

Open:

`http://localhost:8080/h2-console`

Use these connection details from your properties file:

  Field       Value
  ----------- ---------------------------
  JDBC URL    `jdbc:h2:mem:employee_db`
  User Name   `sa`
  Password    Leave blank

After connecting, you can inspect the student table created by JPA. The
exact table name follows the entity/table naming rules used by
Hibernate.

## 12. Important points to understand

1.  **Controller:** Receives the HTTP request.
2.  **Service:** Contains the application/business logic.
3.  **Repository:** Performs database operations.
4.  **Entity:** Represents the database table and its rows.
5.  **H2:** The in-memory database used by this project.
6.  **POST:** Used here to create a new student.
7.  **JSON:** The format used to send student details to the API.

### Current feature

✅ Create Student

The code in this document covers only the create-student feature. It
does not add update, delete, get-all, or get-by-ID endpoints.
