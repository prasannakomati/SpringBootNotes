# Employee Spring Boot Practice --- Project Structure and Explanation

## 1. Complete Project Structure

``` text
employeeSpringBootPractice
├── src/main/java
│   └── com.example.employeeSpringBootPractice
│       ├── EmployeeSpringBootPracticeApplication.java
│       ├── controller
│       │   └── EmployeeController.java
│       ├── dto
│       │   ├── EmployeeRequestDto.java
│       │   └── EmployeeResponseDto.java
│       ├── Entity
│       │   └── Employee.java
│       ├── repository
│       │   └── EmployeeRepository.java
│       └── service
│           └── EmployeeService.java
├── src/main/resources
│   └── application.properties
├── src/test/java
├── target
├── HELP.md
├── mvnw
├── mvnw.cmd
└── pom.xml
```

## 2. Explanation of Each File

### 1. `EmployeeSpringBootPracticeApplication.java` --- Main Class

**Purpose:** Starts the Spring Boot application.

-   `@SpringBootApplication` enables Spring Boot configuration and
    component scanning.
-   `main()` is the entry point of the Java application.
-   `SpringApplication.run()` starts the Spring application context and
    embedded web server.

### 2. `EmployeeController.java` --- Controller Layer

**Purpose:** Receives HTTP requests from a client, such as Postman or a
frontend application, and calls the service layer.

  ---------------------------------------------------------------------------
  Annotation                  Endpoint                Purpose
  --------------------------- ----------------------- -----------------------
  `@PostMapping`              `POST /employees`       Create an employee

  `@GetMapping`               `GET /employees`        Get all employees

  `@GetMapping("/{id}")`      `GET /employees/1`      Get one employee by ID

  `@PutMapping("/{id}")`      `PUT /employees/1`      Update an employee

  `@DeleteMapping("/{id}")`   `DELETE /employees/1`   Delete an employee
  ---------------------------------------------------------------------------

Important annotations:

-   `@RestController` --- handles REST requests and returns response
    data.
-   `@RequestMapping("/employees")` --- establishes the common URL path.
-   `@RequestBody` --- converts incoming JSON into a Java DTO.
-   `@PathVariable` --- extracts the employee ID from the URL.
-   `@ResponseStatus(HttpStatus.CREATED)` --- returns HTTP `201 Created`
    after successful creation.
-   `@ResponseStatus(HttpStatus.NO_CONTENT)` --- returns HTTP
    `204 No Content` after successful deletion.

### 3. `EmployeeRequestDto.java` --- Request Data

**Purpose:** Holds the data that the client sends to your API.

Fields:

-   `name`
-   `email`
-   `password`
-   `salary`

Example JSON for creating an employee:

``` json
{
  "name": "Ravi",
  "email": "ravi@example.com",
  "password": "examplePassword123",
  "salary": 25000.0
}
```

Spring converts this JSON into an `EmployeeRequestDto` object through
`@RequestBody`.

### 4. `EmployeeResponseDto.java` --- Response Data

**Purpose:** Defines the employee information returned to the client.

Fields:

-   `id`
-   `name`
-   `email`
-   `salary`

The response DTO does not include the password.

Example response:

``` json
{
  "id": 1,
  "name": "Ravi",
  "email": "ravi@example.com",
  "salary": 25000.0
}
```

### 5. `Employee.java` --- Entity Layer

**Purpose:** Represents an employee record in the database.

Important annotations:

-   `@Entity` --- marks the class as a JPA entity.
-   `@Table(name="Employee")` --- specifies the database table name.
-   `@Id` --- marks the primary key.
-   `@GeneratedValue(strategy=GenerationType.IDENTITY)` --- configures
    database-generated IDs.

The entity contains `id`, `name`, `email`, `password`, and `salary`.

### 6. `EmployeeRepository.java` --- Repository Layer

**Purpose:** Provides database operations.

``` java
public interface EmployeeRepository extends JpaRepository<Employee, Integer>
{
}
```

Here:

-   `Employee` is the entity type.
-   `Integer` is the type of the entity's ID.

By extending `JpaRepository`, the repository inherits methods such as:

  Method               Purpose
  -------------------- ----------------------------
  `save(employee)`     Insert or update an entity
  `findAll()`          Retrieve all employees
  `findById(id)`       Retrieve an employee by ID
  `delete(employee)`   Delete an employee

### 7. `EmployeeService.java` --- Service Layer

**Purpose:** Implements the application logic between the controller and
repository.

The service contains five operations:

1.  `AddEmployee()` --- converts request DTO data into an entity, saves
    it, and creates a response DTO.
2.  `getEmployees()` --- retrieves all employees and converts them into
    response DTOs.
3.  `getEmployeeById()` --- retrieves one employee and returns a
    response DTO.
4.  `updateEmployee()` --- finds an employee, updates its fields, saves
    it, and returns the response.
5.  `deleteEmployee()` --- finds the employee and deletes it.

The `@Service` annotation marks the class as a Spring-managed service
component.

### 8. `application.properties` --- Application Configuration

**Purpose:** Configures the application and its database.

The current settings specify:

-   Application name: `employeeSpringBootPractice`
-   Database: H2 in-memory database named `employee_db`
-   Database username: `sa`
-   Hibernate schema setting: `create`
-   SQL logging: enabled
-   H2 console: enabled at `/h2-console`

Because the database URL uses `jdbc:h2:mem:employee_db`, the database is
held in memory and normally does not persist after the application shuts
down.

### 9. `pom.xml` --- Maven Configuration

**Purpose:** Defines project configuration and dependencies.

The project uses Spring Boot `3.5.4` and Java `17`.

  Dependency                       Purpose
  -------------------------------- ----------------------------
  `spring-boot-starter-web`        REST APIs and web server
  `spring-boot-starter-data-jpa`   JPA and repository support
  `h2`                             In-memory database
  `spring-boot-starter-test`       Testing support

Maven reads this file to resolve dependencies and build the project.

### 10. Other Folders and Files

-   `src/test/java` --- contains test classes.
-   `target` --- contains generated build output, such as compiled
    classes and packaged artifacts.
-   `mvnw` and `mvnw.cmd` --- Maven Wrapper scripts for Unix-like
    systems and Windows.
-   `HELP.md` --- project help or reference documentation.
-   **JRE System Library \[JavaSE-17\]** --- provides the Java runtime
    libraries selected for the project.
-   **Maven Dependencies** --- displays dependencies resolved by Maven.

## 3. How the Layers Work Together

``` text
Client / Postman
       |
       v
EmployeeController
       |
       v
EmployeeRequestDto
       |
       v
EmployeeService
       |
       v
EmployeeRepository
       |
       v
Employee Entity
       |
       v
H2 Database
```

The response returns through the service and controller as an
`EmployeeResponseDto`, which Spring serializes into JSON.

For example, when you send `POST /employees`:

1.  The controller receives the JSON.
2.  Spring maps the JSON to `EmployeeRequestDto`.
3.  The controller calls `empservice.AddEmployee(requestDto)`.
4.  The service copies the request fields into an `Employee` entity.
5.  The repository saves the entity.
6.  The service creates an `EmployeeResponseDto`.
7.  The controller returns the response as JSON.

## 4. Quick Summary

  Component                  Responsibility
  -------------------------- ----------------------------------------------------
  Main class                 Starts the application
  Controller                 Handles HTTP requests
  Request DTO                Holds incoming data
  Response DTO               Defines outgoing data
  Entity                     Represents a database record
  Repository                 Performs database operations
  Service                    Contains application logic
  `application.properties`   Configures the application
  `pom.xml`                  Defines Maven dependencies and build configuration

**Remember:** Controller = HTTP requests, DTO = data transfer, Entity =
database record, Repository = database access, Service = application
logic.
