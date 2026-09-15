# H2DBPractice 📚

A simple **Spring Boot REST API** project for managing books using **H2 Database** and **JdbcTemplate**.

This project demonstrates a complete CRUD application:

```text
Client / Postman
       ↓
Controller
       ↓
Service
       ↓
JdbcTemplate
       ↓
H2 Database
       ↓
RowMapper
       ↓
Java Object
       ↓
JSON Response
```

---

## 🛠️ Technologies Used

| Technology | Version / Purpose |
|---|---|
| Java | 17 |
| Spring Boot | 2.7.8 |
| Spring Web | REST API |
| Spring JDBC | JdbcTemplate |
| Spring Data JPA | Included in pom.xml |
| H2 Database | In-memory database |
| Maven | Dependency management |
| Eclipse | Development |

---

# 📁 Project Structure

```text
h2DBPractice
│
├── src/main/java
│   │
│   ├── com.h2DBPractice
│   │   └── H2DbPracticeApplication.java
│   │
│   ├── com.h2DBPractice.controller
│   │   └── BookController.java
│   │
│   ├── com.h2DBPractice.model
│   │   ├── Book.java
│   │   └── BookRowMapper.java
│   │
│   ├── com.h2DBPractice.repository
│   │   └── BookRepository.java
│   │
│   └── com.h2DBPractice.service
│       └── BookH2Service.java
│
├── src/main/resources
│   ├── application.properties
│   ├── data.sql
│   └── schema.sql
│
├── src/test/java
├── pom.xml
└── README.md
```

---

# 1️⃣ Main Application

## `H2DbPracticeApplication.java`

```java
package com.h2DBPractice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class H2DbPracticeApplication {

    public static void main(String[] args) {
        SpringApplication.run(H2DbPracticeApplication.class, args);
    }
}
```

### 🔎 Explanation

This is the **main class** of the Spring Boot application.

### `@SpringBootApplication`

```java
@SpringBootApplication
```

This tells Spring Boot to start the application and perform the required auto-configuration and component scanning.

### `main()`

```java
public static void main(String[] args)
```

Java starts the program from the `main()` method.

### `SpringApplication.run()`

```java
SpringApplication.run(H2DbPracticeApplication.class, args);
```

This starts the Spring Boot application.

```text
main()
  ↓
SpringApplication.run()
  ↓
Spring Boot starts
  ↓
Embedded server starts
  ↓
REST API is ready
```

---

# 2️⃣ Controller Layer

## `BookController.java`

```java
package com.h2DBPractice.controller;

import java.util.ArrayList;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import com.h2DBPractice.model.Book;
import com.h2DBPractice.service.BookH2Service;

@RestController
public class BookController {

    @Autowired
    private BookH2Service bookService;

    @GetMapping("/")
    public String home() {

        return "hello world";
    }

    @GetMapping("/books")
    public ArrayList<Book> getBooks() {

        return bookService.getBooks();
    }

    @GetMapping("/books/{bookId}")
    public Book getBookById(
            @PathVariable("bookId") int bookId) {

        return bookService.getBookById(bookId);
    }

    @PostMapping("/books")
    public Book addBook(
            @RequestBody Book book) {

        return bookService.addBook(book);
    }

    @PutMapping("/books/{bookId}")
    public Book updateBook(
            @PathVariable("bookId") int bookId,
            @RequestBody Book book) {

        return bookService.updateBook(bookId, book);
    }

    @DeleteMapping("/books/{bookId}")
    public void deleteBook(
            @PathVariable("bookId") int bookId) {

        bookService.deleteBook(bookId);
    }
}
```

## 🔎 Explanation

The controller is responsible for receiving **HTTP requests**.

```text
Client
  ↓
BookController
  ↓
BookH2Service
```

### `@RestController`

```java
@RestController
```

Makes `BookController` a REST controller.

### `@Autowired`

```java
@Autowired
private BookH2Service bookService;
```

Spring injects the `BookH2Service` object into the controller.

### GET `/`

```java
@GetMapping("/")
public String home() {
    return "hello world";
}
```

Used to test whether the application is running.

URL:

```text
http://localhost:8080/
```

Response:

```text
hello world
```

### GET `/books`

```java
@GetMapping("/books")
public ArrayList<Book> getBooks() {
    return bookService.getBooks();
}
```

Gets all books from the service.

### GET `/books/{bookId}`

```java
@GetMapping("/books/{bookId}")
```

Gets a single book using its ID.

For:

```text
/books/3
```

`@PathVariable` gets:

```text
bookId = 3
```

### POST `/books`

```java
@PostMapping("/books")
```

Adds a new book.

`@RequestBody` converts the JSON request body into a `Book` object.

### PUT `/books/{bookId}`

Updates an existing book.

### DELETE `/books/{bookId}`

Deletes a book using its ID.

---

# 3️⃣ Model Layer

## `Book.java`

```java
package com.h2DBPractice.model;

public class Book {

    public Book() {

    }

    private int id;

    private String name;

    private String imageUrl;

    public Book(int id, String name, String imageUrl) {

        this.id = id;

        this.name = name;

        this.imageUrl = imageUrl;

    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getImageUrl() {
        return imageUrl;
    }

    public void setImageUrl(String imageUrl) {
        this.imageUrl = imageUrl;
    }
}
```

## 🔎 Explanation

`Book` is the **Java model class**.

It represents one book.

```text
Book
├── id
├── name
└── imageUrl
```

### Fields

```java
private int id;
private String name;
private String imageUrl;
```

These represent the data stored for each book.

### No-argument constructor

```java
public Book() {

}
```

Creates an empty `Book` object.

This is useful when the application creates an object first and then uses setters to assign values.

### Parameterized constructor

```java
public Book(int id, String name, String imageUrl)
```

Allows us to create a book with values directly.

Example:

```java
Book book = new Book(
    1,
    "Harry Potter",
    "harry.jpg"
);
```

### Getters and Setters

Getter:

```java
getName()
```

gets the value.

Setter:

```java
setName()
```

sets the value.

---

# 4️⃣ RowMapper

## `BookRowMapper.java`

```java
package com.h2DBPractice.model;

import org.springframework.jdbc.core.RowMapper;

import java.sql.ResultSet;
import java.sql.SQLException;

public class BookRowMapper implements RowMapper<Book> {

    @Override
    public Book mapRow(ResultSet rs, int rowNum) throws SQLException {

        Book b = new Book();

        b.setId(rs.getInt("id"));
        b.setName(rs.getString("name"));
        b.setImageUrl(rs.getString("imageUrl"));

        return b;
    }
}
```

## 🔎 Explanation

`BookRowMapper` converts a database row into a Java `Book` object.

The database may contain:

```text
id | name          | imageUrl
---|---------------|----------
1  | Harry Potter  | harry.jpg
2  | Rise          | rise.jpg
3  | Java          | java.jpg
```

The `RowMapper` converts a row into:

```text
Book object
     ↓
id = 1
name = Harry Potter
imageUrl = harry.jpg
```

### `implements RowMapper<Book>`

```java
implements RowMapper<Book>
```

Means this mapper produces `Book` objects.

### `ResultSet`

```java
ResultSet rs
```

Contains the result returned by the SQL query.

### Reading columns

```java
rs.getInt("id")
```

gets the `id`.

```java
rs.getString("name")
```

gets the `name`.

```java
rs.getString("imageUrl")
```

gets the image URL.

### Complete flow

```text
SQL Query
   ↓
ResultSet
   ↓
BookRowMapper
   ↓
Book Object
```

---

# 5️⃣ Repository Layer

## `BookRepository.java`

```java
package com.h2DBPractice.repository;

import java.util.ArrayList;

import com.h2DBPractice.model.Book;

public interface BookRepository {

    ArrayList<Book> getBooks();

    Book getBookById(int bookId);

    Book addBook(Book book);

    Book updateBook(int bookId, Book book);

    void deleteBook(int bookId);
}
```

## 🔎 Explanation

`BookRepository` is an interface.

It defines the operations that the application needs for books.

```text
BookRepository
│
├── getBooks()
├── getBookById()
├── addBook()
├── updateBook()
└── deleteBook()
```

The interface defines **what operations are available**.

The implementation is provided by:

```text
BookH2Service
```

### Why use an interface?

It separates the operation definitions from their implementation.

```text
BookRepository
      ↓
defines methods
      ↓
BookH2Service
      ↓
implements methods
```

---

# 6️⃣ Service Layer

## `BookH2Service.java`

```java
package com.h2DBPractice.service;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;
import org.springframework.web.server.ResponseStatusException;

import com.h2DBPractice.model.Book;
import com.h2DBPractice.model.BookRowMapper;
import com.h2DBPractice.repository.BookRepository;

@Service
public class BookH2Service implements BookRepository {

    @Autowired
    private JdbcTemplate db;

    @Override
    public ArrayList<Book> getBooks() {

        List<Book> bookList = db.query(
                "SELECT * FROM book",
                new BookRowMapper()
        );

        return new ArrayList<>(bookList);
    }

    @Override
    public Book getBookById(int bookId) {

        try {

            Book book = db.queryForObject(
                    "SELECT * FROM book WHERE id = ?",
                    new BookRowMapper(),
                    bookId
            );

            return book;

        } catch (Exception e) {

            throw new ResponseStatusException(
                    HttpStatus.NOT_FOUND,
                    "Book not found"
            );
        }
    }

    @Override
    public Book addBook(Book book) {

        db.update(
                "INSERT INTO book(name, imageUrl) VALUES (?, ?)",
                book.getName(),
                book.getImageUrl()
        );

        Book savedBook = db.queryForObject(
                "SELECT * FROM book WHERE name = ? AND imageUrl = ?",
                new BookRowMapper(),
                book.getName(),
                book.getImageUrl()
        );

        return savedBook;
    }

    @Override
    public Book updateBook(int bookId, Book book) {

        if (book.getName() != null) {

            db.update(
                    "UPDATE book SET name = ? WHERE id = ?",
                    book.getName(),
                    bookId
            );
        }

        if (book.getImageUrl() != null) {

            db.update(
                    "UPDATE book SET imageUrl = ? WHERE id = ?",
                    book.getImageUrl(),
                    bookId
            );
        }

        return getBookById(bookId);
    }

    @Override
    public void deleteBook(int bookId) {

        int rowsAffected = db.update(
                "DELETE FROM book WHERE id = ?",
                bookId
        );

        if (rowsAffected == 0) {

            throw new ResponseStatusException(
                    HttpStatus.NOT_FOUND,
                    "Book not found"
            );
        }
    }
}
```

## 🔎 Explanation

This is the main **service/database logic** of the application.

```java
@Service
public class BookH2Service implements BookRepository
```

`@Service` makes this class a Spring-managed service.

It also implements the methods defined in `BookRepository`.

---

## 🔌 JdbcTemplate

```java
@Autowired
private JdbcTemplate db;
```

`JdbcTemplate` is used to execute SQL statements.

The service uses it for:

```text
SELECT
INSERT
UPDATE
DELETE
```

---

## 📖 `getBooks()`

```java
List<Book> bookList = db.query(
    "SELECT * FROM book",
    new BookRowMapper()
);
```

SQL:

```sql
SELECT * FROM book;
```

The query gets all rows.

`BookRowMapper` converts the rows into `Book` objects.

Then:

```java
return new ArrayList<>(bookList);
```

returns the list.

Flow:

```text
GET /books
   ↓
Controller
   ↓
Service
   ↓
SELECT * FROM book
   ↓
RowMapper
   ↓
List<Book>
   ↓
JSON
```

---

## 🔎 `getBookById()`

```java
Book book = db.queryForObject(
    "SELECT * FROM book WHERE id = ?",
    new BookRowMapper(),
    bookId
);
```

The `?` is a parameter placeholder.

For:

```text
GET /books/3
```

the `bookId` value is `3`.

The query searches for the book with ID `3`.

If an exception occurs, the service returns:

```java
HttpStatus.NOT_FOUND
```

with:

```text
Book not found
```

So the client receives a **404 Not Found** response.

---

## ➕ `addBook()`

First, the book is inserted:

```java
db.update(
    "INSERT INTO book(name, imageUrl) VALUES (?, ?)",
    book.getName(),
    book.getImageUrl()
);
```

Example request:

```json
{
    "name": "Java Programming",
    "imageUrl": "java.jpg"
}
```

SQL:

```sql
INSERT INTO book(name, imageUrl)
VALUES (?, ?);
```

After insertion, the service queries the database again and returns the saved book.

---

## ✏️ `updateBook()`

The method allows the two fields to be updated independently.

### Update name

```java
if (book.getName() != null) {

    db.update(
        "UPDATE book SET name = ? WHERE id = ?",
        book.getName(),
        bookId
    );
}
```

### Update image URL

```java
if (book.getImageUrl() != null) {

    db.update(
        "UPDATE book SET imageUrl = ? WHERE id = ?",
        book.getImageUrl(),
        bookId
    );
}
```

For example:

```json
{
    "name": "New Book Name"
}
```

Only the name is updated.

For:

```json
{
    "imageUrl": "new-image.jpg"
}
```

only the image URL is updated.

Finally:

```java
return getBookById(bookId);
```

gets and returns the updated book.

---

## 🗑️ `deleteBook()`

```java
int rowsAffected = db.update(
    "DELETE FROM book WHERE id = ?",
    bookId
);
```

Deletes the book with the specified ID.

`rowsAffected` tells how many rows were affected.

If:

```java
rowsAffected == 0
```

the requested book was not found.

The application then returns:

```text
404 Book not found
```

---

# 7️⃣ Configuration

## `application.properties`

```properties
spring.application.name=h2DBPractice

spring.datasource.url=jdbc:h2:mem:goodreads

spring.datasource.driver-class-name=org.h2.Driver

spring.datasource.username=sa

spring.datasource.password=1234

spring.h2.console.enabled=true

spring.h2.console.path=/h2-console

spring.sql.init.mode=always

spring.jpa.hibernate.ddl-auto=update
```

## 🔎 Explanation

### Application name

```properties
spring.application.name=h2DBPractice
```

Sets the Spring Boot application name.

### H2 JDBC URL

```properties
spring.datasource.url=jdbc:h2:mem:goodreads
```

This configures an H2 database named:

```text
goodreads
```

The `mem` part means it is an **in-memory database**.

### H2 Driver

```properties
spring.datasource.driver-class-name=org.h2.Driver
```

Specifies the H2 JDBC driver.

### Username and password

```properties
spring.datasource.username=sa
spring.datasource.password=1234
```

These are the configured H2 database credentials.

### H2 Console

```properties
spring.h2.console.enabled=true
```

Enables the H2 web console.

```properties
spring.h2.console.path=/h2-console
```

Sets the console path.

Open:

```text
http://localhost:8080/h2-console
```

Use:

```text
JDBC URL: jdbc:h2:mem:goodreads
Username: sa
Password: 1234
```

### SQL initialization

```properties
spring.sql.init.mode=always
```

Enables SQL initialization using the SQL scripts in `src/main/resources`.

---

# 8️⃣ Database Schema

## `schema.sql`

```sql
CREATE TABLE IF NOT EXISTS book (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255),
    imageUrl VARCHAR(500)
);
```

## 🔎 Explanation

This SQL creates the `book` table.

```text
book
│
├── id
├── name
└── imageUrl
```

### `id`

```sql
id INT PRIMARY KEY AUTO_INCREMENT
```

The ID is an integer, is the primary key, and is automatically generated.

### `name`

```sql
name VARCHAR(255)
```

Stores the book name.

### `imageUrl`

```sql
imageUrl VARCHAR(500)
```

Stores the image URL or image filename.

---

# 9️⃣ Initial Database Data

## `data.sql`

```sql
insert into book(name, imageUrl)
values(
    'Harry Potter and the Philosopher''s Stone',
    'harry_potter_1.jpg'
);

insert into book(name, imageUrl)
values(
    'Harry Potter and the Chamber of Secrets',
    'harry_potter_2.jpg'
);

insert into book(name, imageUrl)
values(
    'Harry Potter and the Goblet of Fire',
    'harry_potter_3.jpg'
);

insert into book(name, imageUrl)
values(
    'Harry Potter and the Cursed Child',
    'harry_potter_4.jpg'
);

insert into book(name, imageUrl)
values(
    'The 3 Mistakes of My Life',
    'mistakes_life.jpg'
);

insert into book(name, imageUrl)
values(
    'Life of Pi',
    'life_of_pi.jpg'
);

insert into book(name, imageUrl)
values(
    'One Night at the Call Center',
    'one_night_acc.jpg'
);

insert into book(name, imageUrl)
values(
    'Half Girlfriend',
    'half_gf.jpg'
);

insert into book(name, imageUrl)
values(
    'The Secret',
    'secret.jpg'
);

insert into book(name, imageUrl)
values(
    'Rise',
    'rise.jpg'
);
```

## 🔎 Explanation

`data.sql` inserts initial book records into the database.

For example:

```sql
insert into book(name, imageUrl)
values(
    'Life of Pi',
    'life_of_pi.jpg'
);
```

This creates a book record.

Because the `id` is `AUTO_INCREMENT`, the database generates the ID.

The SQL scripts work together:

```text
schema.sql
    ↓
Creates book table
    ↓
data.sql
    ↓
Inserts initial books
```

---

# 🔟 Maven Configuration

## `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.8</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>h2DBPractice</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <name>goodreads</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>

        <!-- Spring Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring Data JDBC -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>

        <!-- Spring Data JPA -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- H2 Database -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
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

## 🔎 Explanation

`pom.xml` is Maven's project configuration file.

It defines:

- Project information
- Java version
- Spring Boot version
- Dependencies
- Build plugins

### Spring Web

```xml
spring-boot-starter-web
```

Used for creating REST APIs.

### Spring Data JDBC

```xml
spring-boot-starter-data-jdbc
```

Provides JDBC-related Spring functionality, including `JdbcTemplate`.

### Spring Data JPA

```xml
spring-boot-starter-data-jpa
```

Provides JPA functionality.

**In this particular code, the CRUD implementation uses `JdbcTemplate` and `RowMapper`, not `JpaRepository`.**

### H2

```xml
com.h2database
```

Provides the H2 database.

### Spring Boot Maven Plugin

```xml
spring-boot-maven-plugin
```

Supports building and running the Spring Boot application with Maven.

---

# 🔄 Complete CRUD Flow

## CREATE ➕

```text
POST /books
     ↓
BookController
     ↓
BookH2Service
     ↓
JdbcTemplate
     ↓
INSERT INTO book
     ↓
H2 Database
```

## READ 📖

```text
GET /books
     ↓
BookController
     ↓
BookH2Service
     ↓
JdbcTemplate
     ↓
SELECT * FROM book
     ↓
BookRowMapper
     ↓
Book Objects
     ↓
JSON
```

## UPDATE ✏️

```text
PUT /books/1
     ↓
BookController
     ↓
BookH2Service
     ↓
JdbcTemplate
     ↓
UPDATE book
     ↓
H2 Database
```

## DELETE 🗑️

```text
DELETE /books/1
     ↓
BookController
     ↓
BookH2Service
     ↓
JdbcTemplate
     ↓
DELETE FROM book
     ↓
H2 Database
```

---

# 🌐 REST API Endpoints

| HTTP Method | URL | Purpose |
|---|---|---|
| GET | `/` | Test application |
| GET | `/books` | Get all books |
| GET | `/books/{bookId}` | Get book by ID |
| POST | `/books` | Add book |
| PUT | `/books/{bookId}` | Update book |
| DELETE | `/books/{bookId}` | Delete book |

---

# 🧪 API Testing

## 1. Get all books

```http
GET http://localhost:8080/books
```

## 2. Get book by ID

```http
GET http://localhost:8080/books/1
```

## 3. Add book

```http
POST http://localhost:8080/books
Content-Type: application/json
```

Body:

```json
{
    "name": "Java Programming",
    "imageUrl": "java.jpg"
}
```

## 4. Update book

```http
PUT http://localhost:8080/books/1
Content-Type: application/json
```

Body:

```json
{
    "name": "Updated Java Programming"
}
```

## 5. Update image URL

```http
PUT http://localhost:8080/books/1
Content-Type: application/json
```

Body:

```json
{
    "imageUrl": "new-java.jpg"
}
```

## 6. Delete book

```http
DELETE http://localhost:8080/books/1
```

---

# 🖥️ H2 Console

After starting the application, open:

```text
http://localhost:8080/h2-console
```

Use:

```text
JDBC URL : jdbc:h2:mem:goodreads
User Name: sa
Password : 1234
```

Then you can execute SQL such as:

```sql
SELECT * FROM book;
```

---

# 🚀 How to Run

### 1. Clone the project

```bash
git clone <YOUR-GITHUB-REPOSITORY-URL>
```

### 2. Open the project in Eclipse.

### 3. Make sure Java 17 is configured.

### 4. Run:

```text
H2DbPracticeApplication.java
```

### 5. Test:

```text
http://localhost:8080/
```

Then:

```text
http://localhost:8080/books
```

---

# 🧠 What I Learned From This Project

This project demonstrates the following concepts:

```text
Spring Boot
   ↓
REST API
   ↓
Controller
   ↓
Service
   ↓
Repository Interface
   ↓
JdbcTemplate
   ↓
SQL
   ↓
H2 Database
   ↓
RowMapper
   ↓
Java Object
   ↓
JSON
```

### Important concepts

- `@SpringBootApplication`
- `@RestController`
- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PathVariable`
- `@RequestBody`
- `@Service`
- `@Autowired`
- `JdbcTemplate`
- `RowMapper`
- `ResponseStatusException`
- H2 Database
- SQL CRUD
- Maven

---

# ⭐ Project Summary

**H2DBPractice** is a Spring Boot CRUD REST API that manages books using an **H2 in-memory database**.

The application uses:

```text
Java 17
   +
Spring Boot 2.7.8
   +
Spring Web
   +
JdbcTemplate
   +
H2 Database
   +
SQL
```

The main purpose of this project is to understand how a Spring Boot REST API communicates with a database using `JdbcTemplate`.

---

## 👨‍💻 Author

**Prasanna Komati**

GitHub: `prasannakomati`

YouTube: **Prasanna Komati**
