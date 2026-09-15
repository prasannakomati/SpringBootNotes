# H2DBPractice 📚

A simple **Spring Boot REST API** project for managing books using **H2
Database** and **JdbcTemplate**.

The project demonstrates:

-   Spring Boot application setup
-   REST API development
-   Controller → Service → Repository flow
-   H2 in-memory database
-   `JdbcTemplate` for SQL operations
-   `RowMapper` for converting database rows into Java objects
-   CRUD operations: Create, Read, Update, Delete
-   SQL initialization using `schema.sql` and `data.sql`
-   HTTP status handling with `ResponseStatusException`

------------------------------------------------------------------------

## 🛠️ Technologies Used

  Technology                   Purpose
  ---------------------------- -----------------------------------
  Java 17                      Programming language
  Spring Boot 2.7.8            Backend framework
  Spring Web                   REST API
  Spring JDBC / JdbcTemplate   Database operations
  H2 Database                  In-memory relational database
  Maven                        Dependency and project management
  Eclipse                      Development environment

> **Note:** The project also includes `spring-boot-starter-data-jpa` in
> `pom.xml`, but the current CRUD implementation uses **JdbcTemplate**,
> not JPA repositories.

------------------------------------------------------------------------

## 📁 Project Structure

``` text
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
│   ├── schema.sql
│   └── data.sql
│
├── src/test/java
│
├── pom.xml
└── README.md
```

------------------------------------------------------------------------

# 🔄 Application Flow

The request travels through the application approximately like this:

``` text
Client / Postman / Browser
          │
          ▼
   BookController
          │
          ▼
    BookH2Service
          │
          ▼
     JdbcTemplate
          │
          ▼
      H2 Database
          │
          ▼
     BookRowMapper
          │
          ▼
       Book Object
          │
          ▼
        JSON
```

### Example

When the client sends:

``` http
GET /books/1
```

the flow is:

``` text
GET /books/1
     ↓
BookController
     ↓
getBookById(1)
     ↓
BookH2Service
     ↓
JdbcTemplate
     ↓
SELECT * FROM book WHERE id = 1
     ↓
BookRowMapper
     ↓
Book Java Object
     ↓
JSON Response
```

------------------------------------------------------------------------

# 1️⃣ Main Application Class

### `H2DbPracticeApplication.java`

``` java
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

### Explanation

`@SpringBootApplication` tells Spring Boot to configure and start the
application.

It combines important Spring Boot functionality, including component
scanning and auto-configuration.

The `main()` method starts the Spring Boot application:

``` java
SpringApplication.run(H2DbPracticeApplication.class, args);
```

After starting successfully, the application can receive HTTP requests.

------------------------------------------------------------------------

# 2️⃣ Model Class

### `Book.java`

The `Book` class represents one book.

``` text
Book
├── id
├── name
└── imageUrl
```

The fields are:

``` java
private int id;
private String name;
private String imageUrl;
```

The class contains:

-   No-argument constructor
-   Parameterized constructor
-   Getters
-   Setters

For example:

``` java
Book book = new Book(
    1,
    "Harry Potter",
    "harry.jpg"
);
```

This represents:

``` text
id       = 1
name     = Harry Potter
imageUrl = harry.jpg
```

------------------------------------------------------------------------

# 3️⃣ RowMapper

### `BookRowMapper.java`

`BookRowMapper` converts one row from the SQL result into a `Book` Java
object.

``` java
public class BookRowMapper implements RowMapper<Book>
```

The important method is:

``` java
@Override
public Book mapRow(ResultSet rs, int rowNum) throws SQLException {

    Book b = new Book();

    b.setId(rs.getInt("id"));
    b.setName(rs.getString("name"));
    b.setImageUrl(rs.getString("imageUrl"));

    return b;
}
```

### Database → Java Object

Suppose the database contains:

``` text
id | name          | imageUrl
---|---------------|----------
1  | Harry Potter  | harry.jpg
2  | Rise          | rise.jpg
3  | Java          | java.jpg
```

`RowMapper` converts each row into:

``` text
Book(1, "Harry Potter", "harry.jpg")
Book(2, "Rise", "rise.jpg")
Book(3, "Java", "java.jpg")
```

So:

``` text
SQL Row
  ↓
ResultSet
  ↓
BookRowMapper
  ↓
Book Object
```

------------------------------------------------------------------------

# 4️⃣ Repository Interface

### `BookRepository.java`

The repository interface defines the operations that the application
supports.

``` java
public interface BookRepository {

    ArrayList<Book> getBooks();

    Book getBookById(int bookId);

    Book addBook(Book book);

    Book updateBook(int bookId, Book book);

    void deleteBook(int bookId);
}
```

It defines five operations:

  Method            Purpose
  ----------------- ---------------
  `getBooks()`      Get all books
  `getBookById()`   Get one book
  `addBook()`       Add a book
  `updateBook()`    Update a book
  `deleteBook()`    Delete a book

The interface describes **what operations are available**. The service
class provides the implementation.

------------------------------------------------------------------------

# 5️⃣ Service Layer

### `BookH2Service.java`

The service class contains the database logic.

``` java
@Service
public class BookH2Service implements BookRepository
```

`@Service` tells Spring to create this class as a Spring-managed bean.

The service uses:

``` java
@Autowired
private JdbcTemplate db;
```

`JdbcTemplate` makes it easier to execute SQL statements from Java.

------------------------------------------------------------------------

## 📖 Get All Books

``` java
List<Book> bookList = db.query(
    "SELECT * FROM book",
    new BookRowMapper()
);

return new ArrayList<>(bookList);
```

Flow:

``` text
SELECT * FROM book
       ↓
    JdbcTemplate
       ↓
  BookRowMapper
       ↓
 List<Book>
```

Endpoint:

``` http
GET /books
```

------------------------------------------------------------------------

## 🔎 Get Book by ID

``` java
Book book = db.queryForObject(
    "SELECT * FROM book WHERE id = ?",
    new BookRowMapper(),
    bookId
);
```

The `?` is replaced by the `bookId` parameter.

Example:

``` http
GET /books/3
```

SQL effectively searches for:

``` sql
SELECT * FROM book WHERE id = 3;
```

If the book cannot be found, the code throws:

``` java
throw new ResponseStatusException(
    HttpStatus.NOT_FOUND,
    "Book not found"
);
```

The client receives HTTP **404 Not Found**.

------------------------------------------------------------------------

# ➕ Add Book

The service inserts a new book:

``` java
db.update(
    "INSERT INTO book(name, imageUrl) VALUES (?, ?)",
    book.getName(),
    book.getImageUrl()
);
```

Example request:

``` http
POST /books
Content-Type: application/json
```

Request body:

``` json
{
    "name": "Java Programming",
    "imageUrl": "java.jpg"
}
```

The SQL operation is:

``` sql
INSERT INTO book(name, imageUrl)
VALUES (?, ?);
```

The values are supplied separately by `JdbcTemplate`.

------------------------------------------------------------------------

# ✏️ Update Book

The project supports updating the book name and image URL independently.

### Update name

``` java
if (book.getName() != null) {

    db.update(
        "UPDATE book SET name = ? WHERE id = ?",
        book.getName(),
        bookId
    );
}
```

### Update image URL

``` java
if (book.getImageUrl() != null) {

    db.update(
        "UPDATE book SET imageUrl = ? WHERE id = ?",
        book.getImageUrl(),
        bookId
    );
}
```

Example:

``` http
PUT /books/1
Content-Type: application/json
```

``` json
{
    "name": "Updated Book Name"
}
```

Only the name is changed because `imageUrl` is `null`.

Another example:

``` json
{
    "imageUrl": "new-image.jpg"
}
```

Only the image URL is changed.

------------------------------------------------------------------------

# 🗑️ Delete Book

``` java
int rowsAffected = db.update(
    "DELETE FROM book WHERE id = ?",
    bookId
);
```

If no database row was deleted:

``` java
if (rowsAffected == 0) {

    throw new ResponseStatusException(
        HttpStatus.NOT_FOUND,
        "Book not found"
    );
}
```

Example:

``` http
DELETE /books/5
```

If book `5` exists, it is deleted.

If it does not exist, the API returns **404 Not Found**.

------------------------------------------------------------------------

# 6️⃣ Controller Layer

### `BookController.java`

The controller receives HTTP requests and calls the service.

``` java
@RestController
public class BookController
```

`@RestController` tells Spring that this class handles REST requests and
its return values are written as HTTP response data, typically JSON for
objects.

The service is injected using:

``` java
@Autowired
private BookH2Service bookService;
```

------------------------------------------------------------------------

## Controller Endpoints

  HTTP Method   Endpoint            Operation
  ------------- ------------------- --------------------
  GET           `/`                 Test/home endpoint
  GET           `/books`            Get all books
  GET           `/books/{bookId}`   Get one book
  POST          `/books`            Add a book
  PUT           `/books/{bookId}`   Update a book
  DELETE        `/books/{bookId}`   Delete a book

------------------------------------------------------------------------

# 7️⃣ GET `/`

``` java
@GetMapping("/")
public String home() {
    return "hello world";
}
```

Open:

``` text
http://localhost:8080/
```

Expected response:

``` text
hello world
```

------------------------------------------------------------------------

# 8️⃣ GET `/books`

``` java
@GetMapping("/books")
public ArrayList<Book> getBooks() {
    return bookService.getBooks();
}
```

Open:

``` text
http://localhost:8080/books
```

This returns the books stored in the H2 database.

------------------------------------------------------------------------

# 9️⃣ GET `/books/{bookId}`

``` java
@GetMapping("/books/{bookId}")
public Book getBookById(
        @PathVariable("bookId") int bookId) {

    return bookService.getBookById(bookId);
}
```

Example:

``` text
http://localhost:8080/books/1
```

`@PathVariable` extracts `1` from the URL and passes it to the service.

------------------------------------------------------------------------

# 🔟 POST `/books`

``` java
@PostMapping("/books")
public Book addBook(
        @RequestBody Book book) {

    return bookService.addBook(book);
}
```

`@RequestBody` converts incoming JSON into a `Book` object.

Example JSON:

``` json
{
    "name": "Spring Boot",
    "imageUrl": "spring.jpg"
}
```

------------------------------------------------------------------------

# 1️⃣1️⃣ PUT `/books/{bookId}`

``` java
@PutMapping("/books/{bookId}")
public Book updateBook(
        @PathVariable("bookId") int bookId,
        @RequestBody Book book) {

    return bookService.updateBook(bookId, book);
}
```

It uses:

-   `@PathVariable` → identifies the book
-   `@RequestBody` → contains the new data

------------------------------------------------------------------------

# 1️⃣2️⃣ DELETE `/books/{bookId}`

``` java
@DeleteMapping("/books/{bookId}")
public void deleteBook(
        @PathVariable("bookId") int bookId) {

    bookService.deleteBook(bookId);
}
```

Example:

``` text
DELETE /books/2
```

This deletes book ID `2`.

------------------------------------------------------------------------

# 🗄️ H2 Database Configuration

### `application.properties`

The application uses an H2 in-memory database:

``` properties
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

Important setting:

``` properties
spring.datasource.url=jdbc:h2:mem:goodreads
```

`mem` means the database is stored in memory.

Therefore, the database data is not persistent across application
restarts.

------------------------------------------------------------------------

# 🖥️ H2 Console

The H2 console is enabled using:

``` properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

Open:

``` text
http://localhost:8080/h2-console
```

Use the configured JDBC URL:

``` text
jdbc:h2:mem:goodreads
```

Username:

``` text
sa
```

Password:

``` text
1234
```

------------------------------------------------------------------------

# 🏗️ Database Schema

### `schema.sql`

The project creates a `book` table:

``` sql
CREATE TABLE IF NOT EXISTS book (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255),
    imageUrl VARCHAR(500)
);
```

The table contains:

``` text
book
│
├── id
├── name
└── imageUrl
```

------------------------------------------------------------------------

# 📦 Initial Data

### `data.sql`

The project inserts initial books when the application initializes the
database.

Examples include:

``` text
Harry Potter and the Philosopher's Stone
Harry Potter and the Chamber of Secrets
Harry Potter and the Goblet of Fire
Harry Potter and the Cursed Child
The 3 Mistakes of My Life
Life of Pi
One Night at the Call Center
Half Girlfriend
The Secret
Rise
```

These records are inserted using SQL `INSERT` statements.

------------------------------------------------------------------------

# 🧪 Testing the API

You can test the API using **Postman**, **Insomnia**, or another HTTP
client.

## Get all books

``` http
GET http://localhost:8080/books
```

## Get one book

``` http
GET http://localhost:8080/books/1
```

## Add a book

``` http
POST http://localhost:8080/books
```

Body:

``` json
{
    "name": "Java Programming",
    "imageUrl": "java.jpg"
}
```

## Update a book

``` http
PUT http://localhost:8080/books/1
```

Body:

``` json
{
    "name": "Updated Java Programming"
}
```

## Delete a book

``` http
DELETE http://localhost:8080/books/1
```

------------------------------------------------------------------------

# 🔁 CRUD Summary

``` text
              BOOK API
                 │
       ┌─────────┼─────────┐
       │         │         │
     CREATE     READ     UPDATE     DELETE
       │         │         │          │
      POST       GET       PUT       DELETE
       │         │         │          │
       └─────────┴─────────┴──────────┘
                 │
                 ▼
           H2 Database
```

### CRUD Mapping

  CRUD     HTTP     SQL
  -------- -------- --------
  Create   POST     INSERT
  Read     GET      SELECT
  Update   PUT      UPDATE
  Delete   DELETE   DELETE

------------------------------------------------------------------------

# 📌 Key Spring Concepts Demonstrated

### `@SpringBootApplication`

Starts and configures the Spring Boot application.

### `@RestController`

Creates a REST controller.

### `@GetMapping`

Handles HTTP GET requests.

### `@PostMapping`

Handles HTTP POST requests.

### `@PutMapping`

Handles HTTP PUT requests.

### `@DeleteMapping`

Handles HTTP DELETE requests.

### `@PathVariable`

Reads a value from the URL.

### `@RequestBody`

Converts request JSON into a Java object.

### `@Service`

Marks the service class as a Spring-managed component.

### `@Autowired`

Injects a Spring-managed dependency.

### `JdbcTemplate`

Executes SQL statements against the configured database.

### `RowMapper`

Converts SQL result rows into Java objects.

### `ResponseStatusException`

Allows the application to return an HTTP error status such as
`404 NOT_FOUND`.

------------------------------------------------------------------------

# 🧠 Simple Architecture Explanation

Think of the application like a restaurant:

``` text
Customer
   ↓
Controller
   ↓
Service
   ↓
JdbcTemplate
   ↓
Database
```

-   **Controller** = Waiter → receives the request
-   **Service** = Kitchen → performs the required operation
-   **JdbcTemplate** = Database assistant → executes SQL
-   **Database** = Storage → stores the books
-   **RowMapper** = Translator → converts database rows into Java
    objects

------------------------------------------------------------------------

# ⚠️ Important Project Note

The current `pom.xml` contains both:

``` xml
spring-boot-starter-data-jdbc
```

and:

``` xml
spring-boot-starter-data-jpa
```

However, the code shown in this project directly uses:

``` java
JdbcTemplate
```

and:

``` java
RowMapper<Book>
```

It does **not** use a JPA `Repository` such as:

``` java
JpaRepository<Book, Integer>
```

Therefore, the actual CRUD implementation demonstrated by this project
is **Spring JDBC/JdbcTemplate + H2**, even though JPA is also included
as a Maven dependency.

------------------------------------------------------------------------

# 🚀 How to Run

### 1. Clone the repository

``` bash
git clone <YOUR-GITHUB-REPOSITORY-URL>
```

### 2. Open the project in Eclipse or another Java IDE.

### 3. Make sure Java 17 is available.

### 4. Run:

``` text
H2DbPracticeApplication.java
```

### 5. Test the application

``` text
http://localhost:8080/
```

Then:

``` text
http://localhost:8080/books
```

H2 console:

``` text
http://localhost:8080/h2-console
```

------------------------------------------------------------------------

# 📚 Learning Outcome

After completing this project, you can understand how a basic Spring
Boot REST API connects to a relational database:

``` text
HTTP Request
     ↓
Controller
     ↓
Service
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
JSON Response
```

This project is a good practice project for learning **Spring Boot +
REST API + JDBC + H2 + CRUD**.
