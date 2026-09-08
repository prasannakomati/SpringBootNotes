# 1. Final Project Structure

```text
goodreads
│
├── pom.xml
│
├── src
│   └── main
│       ├── java
│       │   └── com
│       │       └── example
│       │           └── goodreads
│       │               │
│       │               ├── GoodreadsApplication.java
│       │               │
│       │               ├── controller
│       │               │   └── BookController.java
│       │               │
│       │               ├── model
│       │               │   ├── Book.java
│       │               │   └── BookRowMapper.java
│       │               │
│       │               ├── repository
│       │               │   └── BookRepository.java
│       │               │
│       │               └── service
│       │                   └── BookH2Service.java
│       │
│       └── resources
│           ├── application.properties
│           └── schema.sql
```

**Do not keep `BookService.java` for this H2 version.**

Your old `BookService` stores books in:

```java
HashMap<Integer, Book>
```

Your new `BookH2Service` stores books in:

```text
H2 Database
```

You only need one implementation for this project.

---

# 2. `pom.xml`

Use Spring Web, Spring JDBC and H2.

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
        <version>3.5.4</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>goodreads</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <name>goodreads</name>
    <description>Goodreads Spring Boot H2 JDBC Application</description>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>

        <!-- Spring Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring JDBC -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
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

Important:

For the code you have shown, **`spring-boot-starter-jdbc`** is the important dependency for `JdbcTemplate`.

You don't actually need `spring-boot-starter-data-jdbc` if you're manually using `JdbcTemplate`.

---

# 3. `application.properties`

Location:

```text
src/main/resources/application.properties
```

```properties
spring.application.name=goodreads

spring.datasource.url=jdbc:h2:mem:goodreads
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

spring.sql.init.mode=always
```

Here:

```text
jdbc:h2:mem:goodreads
```

means an H2 **in-memory database** named `goodreads`.

---

# 4. `schema.sql`

Location:

```text
src/main/resources/schema.sql
```

```sql
CREATE TABLE IF NOT EXISTS book (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255),
    imageUrl VARCHAR(500)
);
```

You don't write:

```sql
CREATE DATABASE goodreads;
```

for this H2 in-memory setup.

Spring Boot starts H2 and creates the `book` table using `schema.sql`.

---

# 5. `Book.java`

Location:

```text
src/main/java/com/example/goodreads/model/Book.java
```

```java
package com.example.goodreads.model;

public class Book {

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

---

# 6. `BookRowMapper.java`

Location:

```text
src/main/java/com/example/goodreads/model/BookRowMapper.java
```

```java
package com.example.goodreads.model;

import org.springframework.jdbc.core.RowMapper;

import java.sql.ResultSet;
import java.sql.SQLException;

public class BookRowMapper implements RowMapper<Book> {

    @Override
    public Book mapRow(ResultSet rs, int rowNum) throws SQLException {

        return new Book(
                rs.getInt("id"),
                rs.getString("name"),
                rs.getString("imageUrl")
        );
    }
}
```

This class converts:

```text
Database Row
       ↓
Book object
```

For example:

```text
id = 1
name = "Harry Potter"
imageUrl = "harry.jpg"

       ↓

new Book(
    1,
    "Harry Potter",
    "harry.jpg"
)
```

---

# 7. `BookRepository.java`

Location:

```text
src/main/java/com/example/goodreads/repository/BookRepository.java
```

```java
package com.example.goodreads.repository;

import java.util.ArrayList;

import com.example.goodreads.model.Book;

public interface BookRepository {

    ArrayList<Book> getBooks();

    Book getBookById(int bookId);

    Book addBook(Book book);

    Book updateBook(int bookId, Book book);

    void deleteBook(int bookId);
}
```

---

# 8. `BookH2Service.java`

This is the most important file.

Location:

```text
src/main/java/com/example/goodreads/service/BookH2Service.java
```

Use this complete version:

```java
package com.example.goodreads.service;

import java.util.ArrayList;
import java.util.List;

import com.example.goodreads.model.Book;
import com.example.goodreads.model.BookRowMapper;
import com.example.goodreads.repository.BookRepository;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;
import org.springframework.web.server.ResponseStatusException;

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

---

# 9. `BookController.java`

Your controller needs the correct package and imports.

Location:

```text
src/main/java/com/example/goodreads/controller/BookController.java
```

```java
package com.example.goodreads.controller;

import java.util.ArrayList;

import com.example.goodreads.model.Book;
import com.example.goodreads.service.BookH2Service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class BookController {

    @Autowired
    private BookH2Service bookService;

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

---

# 10. `GoodreadsApplication.java`

Location:

```text
src/main/java/com/example/goodreads/GoodreadsApplication.java
```

```java
package com.example.goodreads;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class GoodreadsApplication {

    public static void main(String[] args) {

        SpringApplication.run(
                GoodreadsApplication.class,
                args
        );
    }
}
```

---

# 11. Delete `BookService.java`

You currently have this:

```java
public class BookService implements BookRepository {

    private HashMap<Integer, Book> hmap = new HashMap<>();

    ...
}
```

**Remove it from this H2 project.**

That class represents the earlier version where the data was stored in Java memory:

```text
BookService
    ↓
HashMap
    ↓
Books
```

Your new version is:

```text
BookH2Service
    ↓
JdbcTemplate
    ↓
H2
    ↓
book table
```

Don't mix both implementations while learning this project.

---

# 12. The biggest errors in your pasted code

### Error 1: Missing imports

Your `BookH2Service` uses:

```java
ResponseStatusException
HttpStatus
List
```

but the imports weren't all present.

You need:

```java
import java.util.List;

import org.springframework.http.HttpStatus;
import org.springframework.web.server.ResponseStatusException;
```

---

### Error 2: `BookController` package is missing

You need:

```java
package com.example.goodreads.controller;
```

at the top.

---

### Error 3: `BookController` needs the `Book` import

You use:

```java
Book
```

so you need:

```java
import com.example.goodreads.model.Book;
```

---

### Error 4: Your pasted imports contain invalid Markdown characters

You pasted:

```text
import org.springframework.web.bind.annotation.*;*
```

That is invalid Java.

It should be either:

```java
import org.springframework.web.bind.annotation.*;
```

or individual imports, which I used in the corrected version.

---

### Error 5: Duplicate `BookH2Service`

You pasted `BookH2Service` **twice**.

Keep only one:

```text
service
└── BookH2Service.java
```

---

### Error 6: Old `BookService`

You have both:

```text
BookService
BookH2Service
```

For this project, keep:

```text
BookH2Service
```

and delete:

```text
BookService
```

---

# 13. How the complete application works

Now the architecture becomes very simple:

```text
Client / Postman
       |
       | HTTP Request
       ↓
BookController
       |
       | calls
       ↓
BookH2Service
       |
       | uses
       ↓
JdbcTemplate
       |
       | SQL
       ↓
H2 Database
       |
       ↓
BOOK TABLE
```

For a GET request:

```text
GET /books
     ↓
BookController
     ↓
bookService.getBooks()
     ↓
JdbcTemplate
     ↓
SELECT * FROM book
     ↓
BookRowMapper
     ↓
ArrayList<Book>
     ↓
JSON Response
```

For POST:

```text
POST /books
     ↓
@RequestBody Book
     ↓
BookController
     ↓
bookService.addBook()
     ↓
JdbcTemplate
     ↓
INSERT INTO book
     ↓
H2
     ↓
SELECT newly inserted book
     ↓
Book
     ↓
JSON Response
```

For PUT:

```text
PUT /books/1
     ↓
BookController
     ↓
updateBook(1, book)
     ↓
JdbcTemplate
     ↓
UPDATE book
     ↓
H2
     ↓
getBookById(1)
     ↓
Book
```

For DELETE:

```text
DELETE /books/1
     ↓
BookController
     ↓
deleteBook(1)
     ↓
JdbcTemplate
     ↓
DELETE FROM book
     ↓
H2
```

## The key point for your Spring learning

In your **old Spring JDBC project**, you manually configured:

```text
DataSource
    ↓
JdbcTemplate
```

inside XML.

In this **Spring Boot project**, you don't manually create that XML.

You have:

```text
pom.xml
    ↓
spring-boot-starter-jdbc
    ↓
Spring Boot auto-configuration
    ↓
DataSource
    ↓
JdbcTemplate
    ↓
@Autowired
private JdbcTemplate db;
```

So this line:

```java
@Autowired
private JdbcTemplate db;
```

is the point where **your service receives the `JdbcTemplate` that Spring Boot has automatically configured**.

And then:

```java
db.query(...)
db.queryForObject(...)
db.update(...)
```

