# 📚 Goodreads with JPA

A simple **Spring Boot + Spring Data JPA + H2 Database** CRUD application for managing books.

---

## 📁 Project Structure

```text
goodreadswithjpa
│
├── src/main/java
│   └── com.example.goodreadswithjpa
│       ├── GoodreadswithjpaApplication.java
│       │
│       ├── controller
│       │   └── BookController.java
│       │
│       ├── model
│       │   └── Book.java
│       │
│       ├── repository
│       │   ├── BookJpaRepository.java
│       │   └── BookRepository.java
│       │
│       └── service
│           └── BookJpaService.java
│
├── src/main/resources
│   ├── application.properties
│   ├── data.sql
│   └── schema.sql
│
├── src/test/java
├── pom.xml
├── mvnw
├── mvnw.cmd
└── HELP.md
```

---

# 🔄 Application Flow

```text
🌐 Client / Postman
        ↓
🎮 BookController
        ↓
⚙️ BookJpaService
        ↓
📦 BookJpaRepository
        ↓
🗄️ H2 Database
```

### Simple explanation

- **Controller** receives HTTP requests.
- **Service** contains the application logic.
- **Repository** communicates with the database through JPA.
- **Model** represents the database entity.
- **H2** stores the book data.
- **JPA/Hibernate** maps the `Book` class to the `book` table.

---

# 1️⃣ GoodreadswithjpaApplication.java

## Source Code

```java
package com.example.goodreadswithjpa;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class GoodreadswithjpaApplication {

	public static void main(String[] args) {
		SpringApplication.run(GoodreadswithjpaApplication.class, args);
	}

}
```

## Explanation

This is the **main class** of the Spring Boot application.

### `@SpringBootApplication`

```java
@SpringBootApplication
```

This tells Spring Boot to start and configure the application.

### `main()`

```java
public static void main(String[] args)
```

Java execution starts from this method.

```java
SpringApplication.run(GoodreadswithjpaApplication.class, args);
```

This starts the Spring Boot application.

---

# 2️⃣ BookController.java

## Source Code

```java
package com.example.goodreadswithjpa.controller;

import java.util.ArrayList;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import com.example.goodreadswithjpa.model.Book;
import com.example.goodreadswithjpa.service.BookJpaService;


@RestController
public class BookController 
{

    @Autowired
    private BookJpaService bookService;

    @GetMapping("/")
    public String home() 
    {
        return "hello world";
    }

    @GetMapping("/books")
    public ArrayList<Book> getBooks() 
    {
        return bookService.getBooks();
    }

    @GetMapping("/books/{bookId}")
    public Book getBookById(
            @PathVariable("bookId") int bookId) 
    {
        return bookService.getBookById(bookId);
    }

    @PostMapping("/books")
    public Book addBook(
            @RequestBody Book book) 
    {
        return bookService.addBook(book);
    }

    @PutMapping("/books/{bookId}")
    public Book updateBook(
            @PathVariable("bookId") int bookId,
            @RequestBody Book book) 
    {
        return bookService.updateBook(bookId, book);
    }

    @DeleteMapping("/books/{bookId}")
    public void deleteBook(
            @PathVariable("bookId") int bookId) 
    {
        bookService.deleteBook(bookId);
    }

}
```

## Explanation

`BookController` handles the application's **REST API requests**.

### `@RestController`

```java
@RestController
```

Marks this class as a REST controller. It allows methods to return data directly as HTTP responses.

### Dependency Injection

```java
@Autowired
private BookJpaService bookService;
```

Spring automatically provides the `BookJpaService` object.

### API Endpoints

| HTTP Method | URL | Purpose |
|---|---|---|
| GET | `/` | Check the application |
| GET | `/books` | Get all books |
| GET | `/books/{bookId}` | Get one book |
| POST | `/books` | Add a book |
| PUT | `/books/{bookId}` | Update a book |
| DELETE | `/books/{bookId}` | Delete a book |

### `@PathVariable`

```java
@PathVariable("bookId") int bookId
```

Gets the ID from the URL.

Example:

```text
GET /books/5
```

Here:

```text
bookId = 5
```

### `@RequestBody`

```java
@RequestBody Book book
```

Converts the JSON request body into a Java `Book` object.

---

# 3️⃣ Book.java

## Source Code

```java
package com.example.goodreadswithjpa.model;

import javax.persistence.*;

@Entity
@Table(name="book")
public class Book 
{
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="id")
	private int id;

	@Column(name="name")
	private String name;

	@Column(name="imageUrl")
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

	public Book() {
		// TODO Auto-generated constructor stub
	}

	public String getImageUrl() {
		return imageUrl;
	}

	public void setImageUrl(String imageUrl) {
		this.imageUrl = imageUrl;
	}

}
```

## Explanation

`Book` is the **model/entity class**.

It represents a book and is mapped to the database table named `book`.

### `@Entity`

```java
@Entity
```

Tells JPA that this Java class is a database entity.

### `@Table`

```java
@Table(name="book")
```

Maps the Java class to the database table:

```text
Book → book
```

### `@Id`

```java
@Id
```

Marks `id` as the primary key.

### `@GeneratedValue`

```java
@GeneratedValue(strategy=GenerationType.IDENTITY)
```

Allows the database to generate the ID automatically.

### Fields

```text
id       → Book ID
name     → Book name
imageUrl → Book image name/URL
```

### Empty Constructor

```java
public Book() {
}
```

JPA requires a no-argument constructor for entity creation.

---

# 4️⃣ BookJpaRepository.java

## Source Code

```java
package com.example.goodreadswithjpa.repository;

import org.springframework.data.jpa.repository.JpaRepository;

import com.example.goodreadswithjpa.model.Book;

public interface BookJpaRepository extends JpaRepository<Book,Integer>
{

}
```

## Explanation

This repository uses **Spring Data JPA**.

The important line is:

```java
extends JpaRepository<Book,Integer>
```

Here:

```text
Book    → Entity type
Integer → ID type
```

Because `Book` has an integer ID.

`JpaRepository` already provides common database operations such as:

```text
findAll()
findById()
save()
deleteById()
```

Therefore, you don't need to write SQL for these basic CRUD operations.

---

# 5️⃣ BookRepository.java

## Source Code

```java
package com.example.goodreadswithjpa.repository;

import java.util.ArrayList;

import org.springframework.stereotype.Repository;

import com.example.goodreadswithjpa.model.Book;

@Repository
public interface BookRepository
{
	ArrayList<Book> getBooks();

    Book getBookById(int bookId);

    Book addBook(Book book);

    Book updateBook(int bookId, Book book);

    void deleteBook(int bookId);
}
```

## Explanation

This interface defines the operations that the service should provide.

It contains:

```text
getBooks()
getBookById()
addBook()
updateBook()
deleteBook()
```

The `BookJpaService` implements this interface:

```java
public class BookJpaService implements BookRepository
```

So the interface defines **what operations are available**, while the service provides the implementation.

---

# 6️⃣ BookJpaService.java

## Source Code

```java
package com.example.goodreadswithjpa.service;
import java.util.*;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;

import org.springframework.stereotype.Service;
import org.springframework.web.server.ResponseStatusException;

import com.example.goodreadswithjpa.model.Book;
import com.example.goodreadswithjpa.repository.BookJpaRepository;
import com.example.goodreadswithjpa.repository.BookRepository;


@Service
public class BookJpaService implements BookRepository {

    @Autowired
    private BookJpaRepository bookJpaRepository;

    @Override
    public ArrayList<Book> getBooks() {

        List<Book> bookList = bookJpaRepository.findAll();

        ArrayList<Book> books = new ArrayList<>(bookList);

        return books;
    }

    @Override
    public Book getBookById(int bookId) {

        try {

            Book book = bookJpaRepository.findById(bookId).get();

            return book;

        } catch (Exception e) {

            throw new ResponseStatusException(
                    HttpStatus.NOT_FOUND
            );
        }
    }

    @Override
    public Book addBook(Book book) {

        bookJpaRepository.save(book);

        return book;
    }

    @Override
    public Book updateBook(int bookId, Book book) {

        try {

            Book newBook = bookJpaRepository.findById(bookId).get();

            if (book.getName() != null) {
                newBook.setName(book.getName());
            }

            if (book.getImageUrl() != null) {
                newBook.setImageUrl(book.getImageUrl());
            }

            bookJpaRepository.save(newBook);

            return newBook;

        } catch (Exception e) {

            throw new ResponseStatusException(
                    HttpStatus.NOT_FOUND
            );
        }
    }

    @Override
    public void deleteBook(int bookId) {

        try {

            bookJpaRepository.deleteById(bookId);

        } catch (Exception e) {

            throw new ResponseStatusException(
                    HttpStatus.NOT_FOUND
            );
        }

        throw new ResponseStatusException(
                HttpStatus.NO_CONTENT
        );
    }
}
```

## Explanation

`BookJpaService` contains the application's **service logic**.

### `@Service`

```java
@Service
```

Tells Spring to manage this class as a service component.

### Repository injection

```java
@Autowired
private BookJpaRepository bookJpaRepository;
```

Spring injects the JPA repository into the service.

---

## `getBooks()`

```java
List<Book> bookList = bookJpaRepository.findAll();
```

`findAll()` gets all books from the database.

Then:

```java
ArrayList<Book> books = new ArrayList<>(bookList);
```

converts the result into an `ArrayList`.

---

## `getBookById()`

```java
bookJpaRepository.findById(bookId).get();
```

Finds a book using its ID.

If the book cannot be found, the code throws:

```java
HttpStatus.NOT_FOUND
```

which represents HTTP:

```text
404 Not Found
```

---

## `addBook()`

```java
bookJpaRepository.save(book);
```

Saves a new book to the database.

---

## `updateBook()`

First, the existing book is retrieved:

```java
Book newBook = bookJpaRepository.findById(bookId).get();
```

Then the provided fields are checked.

```java
if (book.getName() != null) {
    newBook.setName(book.getName());
}
```

If a new name was provided, it is updated.

Similarly:

```java
if (book.getImageUrl() != null) {
    newBook.setImageUrl(book.getImageUrl());
}
```

Finally:

```java
bookJpaRepository.save(newBook);
```

saves the updated book.

---

## `deleteBook()`

```java
bookJpaRepository.deleteById(bookId);
```

Deletes the book using its ID.

If an error occurs, the code returns:

```text
404 Not Found
```

After successful deletion, the code throws:

```java
HttpStatus.NO_CONTENT
```

which represents:

```text
204 No Content
```

---

# 7️⃣ application.properties

## Source Code

```properties
spring.application.name=goodreadswithjpa

spring.datasource.url=jdbc:h2:mem:goodreads

spring.datasource.driver-class-name=org.h2.Driver

spring.datasource.username=sa

spring.datasource.password=1234

spring.h2.console.enabled=true

spring.h2.console.path=/h2-console

spring.sql.init.mode=always

spring.jpa.hibernate.ddl-auto=update
```

## Explanation

### Application name

```properties
spring.application.name=goodreadswithjpa
```

Sets the application name.

### H2 Database URL

```properties
spring.datasource.url=jdbc:h2:mem:goodreads
```

Uses an H2 **in-memory database** named `goodreads`.

### H2 Driver

```properties
spring.datasource.driver-class-name=org.h2.Driver
```

Specifies the H2 JDBC driver.

### Username and Password

```properties
spring.datasource.username=sa
spring.datasource.password=1234
```

Credentials used to connect to the H2 database.

### H2 Console

```properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

Enables the H2 browser console.

If the application runs on port 8080:

```text
http://localhost:8080/h2-console
```

### Hibernate DDL

```properties
spring.jpa.hibernate.ddl-auto=update
```

Allows Hibernate to create/update database tables based on JPA entity mappings.

For this project:

```text
Book.java
   ↓
@Entity
   ↓
Hibernate
   ↓
book table
```

---

# 8️⃣ schema.sql

## Source Code

```sql
CREATE TABLE IF NOT EXISTS book (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255),
    imageUrl VARCHAR(500)
);
```

## Explanation

This SQL manually creates the `book` table.

The table contains:

```text
id
name
imageUrl
```

However, this project also uses:

```properties
spring.jpa.hibernate.ddl-auto=update
```

which allows Hibernate to create/update the table from `Book.java`.

---

# 9️⃣ data.sql

## Source Code

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

## Explanation

`data.sql` inserts initial book records into the database.

For example:

```sql
insert into book(name, imageUrl)
values(
    'Life of Pi',
    'life_of_pi.jpg'
);
```

adds a book to the `book` table.

Because your database is:

```text
jdbc:h2:mem:goodreads
```

the in-memory database is recreated when the application starts.

---

# 🔟 pom.xml

## Source Code

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
    <artifactId>goodreadswithjpa</artifactId>
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

## Explanation

`pom.xml` is the **Maven configuration file**.

It defines the Spring Boot version, Java version, dependencies, and build configuration.

### Spring Boot version

```xml
<version>2.7.8</version>
```

This project uses Spring Boot 2.7.8.

### Java version

```xml
<java.version>17</java.version>
```

The project is configured for Java 17.

### Spring Web

```xml
spring-boot-starter-web
```

Provides REST API functionality such as:

```text
@RestController
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
```

### Spring Data JPA

```xml
spring-boot-starter-data-jpa
```

Provides JPA/Hibernate support and `JpaRepository`.

### H2

```xml
h2
```

Provides the H2 database.

### Spring Data JDBC

```xml
spring-boot-starter-data-jdbc
```

Provides Spring Data JDBC support. The shown CRUD implementation specifically uses Spring Data JPA through `JpaRepository`.

---

# 🌐 REST API Examples

## 1. Home

```http
GET /
```

Response:

```text
hello world
```

---

## 2. Get All Books

```http
GET /books
```

Example response:

```json
[
  {
    "id": 1,
    "name": "Harry Potter and the Philosopher's Stone",
    "imageUrl": "harry_potter_1.jpg"
  }
]
```

---

## 3. Get Book by ID

```http
GET /books/1
```

Gets the book whose ID is `1`.

---

## 4. Add Book

```http
POST /books
```

Request body:

```json
{
  "name": "Java Programming",
  "imageUrl": "java.jpg"
}
```

---

## 5. Update Book

```http
PUT /books/1
```

Request body:

```json
{
  "name": "Updated Java Programming"
}
```

---

## 6. Delete Book

```http
DELETE /books/1
```

Deletes the book with ID `1`.

---

# 🧠 Complete CRUD Flow

```text
                    🌐 REST API
                        │
       ┌────────────────┼────────────────┐
       │                │                │
      GET              POST             PUT/DELETE
       │                │                │
       └────────────────┼────────────────┘
                        ↓
               🎮 BookController
                        ↓
                ⚙️ BookJpaService
                        ↓
                📦 BookJpaRepository
                        ↓
                  🔗 JPA/Hibernate
                        ↓
                   🗄️ H2 Database
```

---

# ⭐ Key Concepts Used

```text
Spring Boot
   ↓
REST API
   ↓
Spring Data JPA
   ↓
Hibernate
   ↓
H2 Database
   ↓
CRUD Operations
```

### CRUD

```text
C → Create → POST
R → Read   → GET
U → Update → PUT
D → Delete → DELETE
```

---

# 📌 Important Annotations

| Annotation | Purpose |
|---|---|
| `@SpringBootApplication` | Starts Spring Boot |
| `@RestController` | Creates REST controller |
| `@Service` | Creates service component |
| `@Repository` | Marks repository component |
| `@Entity` | Maps Java class to database entity |
| `@Table` | Specifies database table |
| `@Id` | Defines primary key |
| `@GeneratedValue` | Generates ID automatically |
| `@Column` | Maps field to table column |
| `@Autowired` | Dependency injection |
| `@GetMapping` | Handles GET request |
| `@PostMapping` | Handles POST request |
| `@PutMapping` | Handles PUT request |
| `@DeleteMapping` | Handles DELETE request |
| `@PathVariable` | Gets value from URL |
| `@RequestBody` | Gets data from request body |

---

# 🎯 Final Summary

This project is a **Book CRUD REST API** built using:

```text
☕ Java 17
🚀 Spring Boot 2.7.8
🌐 Spring Web
📦 Spring Data JPA
⚙️ Hibernate
🗄️ H2 Database
🔨 Maven
```

The main architecture is:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
JPA / Hibernate
    ↓
H2 Database
```

The `Book` entity defines the database structure, `BookController` handles HTTP requests, `BookJpaService` contains the service logic, and `BookJpaRepository` provides JPA database operations.


---

# 🏗️ Do We Need `schema.sql` and `data.sql`?

This project can work **without `schema.sql` and `data.sql`** when Hibernate/JPA is configured to create the database table and you do not need initial sample data.

## `schema.sql` — Not Compulsory ❌

`schema.sql` is used when you want to manually create database tables using SQL.

Example:

```sql
CREATE TABLE IF NOT EXISTS book (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255),
    imageUrl VARCHAR(500)
);
```

However, this project has:

```properties
spring.jpa.hibernate.ddl-auto=update
```

and the `Book` class is an entity:

```java
@Entity
@Table(name="book")
public class Book
```

Therefore, Hibernate can create/update the `book` table from the entity definition.

### Hibernate table creation flow

```text
📚 Book.java
     ↓
@Entity
     ↓
@Table(name="book")
     ↓
⚙️ Hibernate / JPA
     ↓
🏗️ Creates or updates `book` table
     ↓
🗄️ H2 Database
```

So:

```text
schema.sql → ❌ Not compulsory
```

---

## `data.sql` — Not Compulsory ❌

`data.sql` is used to insert initial/sample records into the database.

For example:

```sql
INSERT INTO book(name, imageUrl)
VALUES ('Life of Pi', 'life_of_pi.jpg');
```

If you remove `data.sql`, Hibernate can still create the table.

The difference is that the table will initially contain no sample books.

```text
Without data.sql:

🏗️ Table → Created by Hibernate ✅
📚 Records → No initial records
```

You can then add books through the REST API:

```http
POST /books
```

Example:

```json
{
  "name": "Java Programming",
  "imageUrl": "java.jpg"
}
```

---

## 🔎 What Happens When Both Files Are Removed?

If you remove:

```text
schema.sql ❌
data.sql   ❌
```

and keep:

```properties
spring.jpa.hibernate.ddl-auto=update
```

Hibernate can create the table from your `Book` entity.

```text
🚀 Spring Boot starts
        ↓
📚 Book.java is detected as @Entity
        ↓
⚙️ Hibernate reads the entity mapping
        ↓
🏗️ Hibernate creates/updates `book` table
        ↓
🗄️ H2 database is ready
        ↓
🌐 REST API starts
```

For this project, this means the basic application can run without either SQL file.

---

## 📌 Difference Between the Three

| Feature | `schema.sql` | `data.sql` | Hibernate/JPA |
|---|---|---|---|
| Create table | ✅ Yes | ❌ No | ✅ Yes |
| Insert initial data | ❌ No | ✅ Yes | ❌ Not automatically |
| Uses SQL file | ✅ | ✅ | ❌ |
| Uses `@Entity` mapping | ❌ | ❌ | ✅ |
| Required for this JPA project | ❌ Not compulsory | ❌ Not compulsory | ✅ Used through JPA/Hibernate |

### Simple rule to remember

```text
schema.sql
   ↓
🏗️ Manually create table

data.sql
   ↓
📚 Insert initial/sample data

Hibernate + @Entity
   ↓
⚙️ Automatically create/update table
```

### ⭐ In this project

```properties
spring.jpa.hibernate.ddl-auto=update
```

is the configuration that allows Hibernate to create/update the database schema based on your entity classes.

Therefore:

**`schema.sql` is not compulsory.**  
**`data.sql` is not compulsory.**  
**Hibernate can automatically create the `book` table from `Book.java`.** ✅

If `data.sql` is used together with Hibernate schema generation, initialization order can matter. In such a setup, Spring Boot can be configured to defer SQL data initialization until after Hibernate creates the schema.
