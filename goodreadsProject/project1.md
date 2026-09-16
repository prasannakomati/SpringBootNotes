# Goodreads API – Phase 2 (Spring Boot REST APIs)

## Overview

This project implements CRUD operations using Spring Boot REST APIs.
The application manages book data using an in-memory HashMap.

---

## Project Structure

com.example.goodreads

* GoodreadsApplication.java
* BookController.java
* BookService.java
* BookRepository.java
* Books.java

---

## Source Code

### 1. GoodreadsApplication.java

```java
package com.example.goodreads;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class GoodreadsApplication {

    public static void main(String[] args) {
        SpringApplication.run(GoodreadsApplication.class, args);
    }
}
```

---

### 2. BookRepository.java

```java
package com.example.goodreads;

import java.util.ArrayList;

public interface BookRepository {

    ArrayList<Books> getBooks();

    Books getBookById(int bookId);

    Books addBook(Books book);

    Books updateBook(int bookId, Books book);

    void deleteBook(int bookId);
}
```

---

### 3. BookService.java

```java
package com.example.goodreads;

import java.util.*;
import org.springframework.web.server.ResponseStatusException;
import org.springframework.http.HttpStatus;

public class BookService implements BookRepository {

    private HashMap<Integer, Books> hmap = new HashMap<>();
    int uniqueBookId = 3;

    public BookService() {
        Books b1 = new Books(1, "harry potter", "harry_potter.jpg");
        Books b2 = new Books(2, "Rise", "rise.jpeg");
        hmap.put(b1.getId(), b1);
        hmap.put(b2.getId(), b2);
    }

    @Override
    public ArrayList<Books> getBooks() {
        Collection<Books> bookCollection = hmap.values();
        ArrayList<Books> books = new ArrayList<>(bookCollection);
        return books;
    }

    @Override
    public Books getBookById(int bookId) {
        Books book = hmap.get(bookId);
        if (book == null) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND);
        }
        return book;
    }

    @Override
    public Books addBook(Books book) {
        book.setId(uniqueBookId);
        hmap.put(uniqueBookId, book);
        uniqueBookId += 1;
        return book;
    }

    @Override
    public Books updateBook(int bookId, Books book) {
        Books existingBook = hmap.get(bookId);
        if (existingBook == null) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND);
        }
        if (book.getName() != null) {
            existingBook.setName(book.getName());
        }
        if (book.getImageUrl() != null) {
            existingBook.setImageUrl(book.getImageUrl());
        }
        return existingBook;
    }

    @Override
    public void deleteBook(int bookId) {
        Books book = hmap.get(bookId);
        if (book == null) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND);
        } else {
            hmap.remove(bookId);
        }
    }
}
```

---

### 4. BookController.java

```java
package com.example.goodreads;

import org.springframework.web.bind.annotation.*;
import java.util.*;

@RestController
class BookController {

    BookService bookService = new BookService();

    @GetMapping("/books")
    public ArrayList<Books> getBooks() {
        return bookService.getBooks();
    }

    @GetMapping("/books/{bookId}")
    public Books getBookById(@PathVariable("bookId") int bookId) {
        return bookService.getBookById(bookId);
    }

    @PostMapping("/books")
    public Books addBook(@RequestBody Books book) {
        return bookService.addBook(book);
    }

    @PutMapping("/books/{bookId}")
    public Books updateBook(@PathVariable("bookId") int bookId, @RequestBody Books book) {
        return bookService.updateBook(bookId, book);
    }

    @DeleteMapping("/books/{bookId}")
    public void deleteBook(@PathVariable("bookId") int bookId) {
        bookService.deleteBook(bookId);
    }
}
```

---

### 5. Books.java

```java
package com.example.goodreads;

public class Books {

    private int id;
    private String name;
    private String imageUrl;

    public Books() {
    }

    public Books(int id, String name, String imageUrl) {
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

## Endpoints

GET /books
GET /books/{bookId}
POST /books
PUT /books/{bookId}
DELETE /books/{bookId}

---
# Introduction

In the previous unit, we created a goodreads application and have written APIs to read the books data.

In this unit, we'll learn to write APIs to handle the Create, Update and Delete books data.

---

## 1. Add Book API

To add a book to our hmap, we need to write its logic. Let's update our BookRepository interface with an abstract method to add a book.

### File: BookRepository.java

```java
package com.example.goodreads;

public interface BookRepository {
    ArrayList getBooks();
    Books getBookById(int bookId);
    Books addBook(Books book);
}
```

We have added an abstract method addBook() that takes a Books object as an argument and returns a Books object.

In the BookService class we have to implement the addBook() method as it is an abstract method.

### File: BookService.java

```java
package com.example.goodreads;

import com.example.goodreads.Books;
import org.springframework.http.HttpStatus;
import org.springframework.web.server.ResponseStatusException;
import java.util.*;

public class BookService implements BookRepository {

    private HashMap<Integer, Books> hmap = new HashMap<>();
    int uniqueBookId = 3;

    public BookService() {
        Books b1 = new Books(1, "harry potter","harry_potter.jpg");
```

In the above code, the addBook() method adds a book to the hmap and returns the Books object.

We have also intialized an attribute as int uniqueBookId = 3. While creating books we need to take care that every book should be identified by a unique ID. To achieve that, we have defined uniqueBookId and assigned the initial value as 3 because we are already creating two books using the BookService constructor. After adding the book, we are increasing the id as uniqueBookId += 1 so that the next book we add will get the new ID.

Now, our logic to add a book to the hmap is ready. So, we have to write a method in our Restcontroller class to handle the HTTP POST request.

So, let's create a addBook() method that handles the HTTP POST request to the path as /books.

```java
@PostMapping("/books")
public Books addBook(@RequestBody Books book){
    // method body
}
```

The addBook() method in the above code is annotated with the @PostMapping("/books"), which is a request mapping annotation that maps a specific HTTP POST request to the method. In our case, the addBook() method is mapped to the URL /books and will handle HTTP POST requests to that URL.

### Accessing the Request Body using RequestBody Annotation

The @RequestBody annotation is used to bind the request body to a method parameter, which can be used to process the request data.

The @RequestBody annotation is typically used with an HTTP POST or PUT request, which typically has a request body that contains data in the form of a JSON.

The @RequestBody annotation in the addBook() method specifies that the request body should be bound to the method parameter.

Here, Spring Boot converts the request body to the Books object and sends it to the addBook() method. This means that the client can send a JSON representation of the book as the request body and the addBook() method will receive it as a Books object.

### File: BookController.java

```java
package com.example.goodreads;

import org.springframework.web.bind.annotation.*;
import java.util.*;
import com.example.goodreads.BookService;

@RestController
public class BookController {
    BookService bookService = new BookService();
```

Request
post request book

Response
post response book

---

## 2. Update Book API

To update the details of an existing book in our hmap, we need to write its logic. Let's update our BookRepository interface with an abstract method to update the details of an existing book.

### File: BookRepository.java

```java
package com.example.goodreads;

public interface BookRepository {
    ArrayList getBooks();
    Books getBookById(int bookId);
    Books addBook(Books book);
    Books updateBook(int bookId, Books book);
}
```

We have added an abstract method updateBook() that takes a bookId and Books object as arguments and returns the updated Books object.

In the BookService class we have to implement the updateBook() method as it is an abstract method.

### File: BookService.java

```java
package com.example.goodreads;

import com.example.goodreads.Books;
import org.springframework.http.HttpStatus;
import org.springframework.web.server.ResponseStatusException;
import java.util.*;

public class BookService implements BookRepository {

    private HashMap<Integer, Books> hmap = new HashMap<>();
    int uniqueBookId = 3;
```

In the above code, the updateBook() method updates the details of an existing book in the hmap and returns the updated Books object.

Now, our logic to update a book in the hmap is ready. So, we have to write a method in our BookController class to handle the HTTP PUT request.

Let's use /books/{bookId} as a path to identify a single book resource, where bookId is a path parameter.

So, let's create a updateBook() method that handles the HTTP PUT request to the path as /books/{bookId}.

```java
@PutMapping("/books/{bookId}")
public Books updateBook(@PathVariable("bookId") int bookId, @RequestBody Books book) {
    // method body
}
```

The updateBook() method in the above code is annotated with the @PutMapping("/books/{bookId}"), which is a request mapping annotation that maps a specific HTTP PUT request to the method. In our case, the updateBook() method is mapped to the URL /books/{bookId} and will handle HTTP PUT requests to that URL.

We have also used the @PathVariable and @RequestBody annotations to access the path parameters and request body, respectively.

### File: BookController.java

```java
package com.example.goodreads;

import org.springframework.web.bind.annotation.*;
import java.util.*;
import com.example.goodreads.BookService;

@RestController
public class BookController {
    BookService bookService = new BookService();
```

Request
post request book

Response
post response book

---

## 3. Delete Book API

To delete the details of an existing book in our hmap, we need to write its logic. Let's update our BookRepository interface with an abstract method to delete the details of an existing book.

### File: BookRepository.java

```java
package com.example.goodreads;

public interface BookRepository {
    ArrayList getBooks();
    Books getBookById(int bookId);
    Books addBook(Books book);
    Books updateBook(int bookId, Books book);
    void deleteBook(int bookId);
}
```

We have added an abstract method deleteBook() that takes a bookId as an argument.

In the BookService class we have to implement the deleteBook() method as it is an abstract method.

### File: BookService.java

```java
package com.example.goodreads;

import com.example.goodreads.Books;
import org.springframework.http.HttpStatus;
import org.springframework.web.server.ResponseStatusException;
import java.util.*;

public class BookService implements BookRepository {

    private HashMap<Integer, Books> hmap = new HashMap<>();
```

In the above code, the deleteBook() method deletes the details of an existing book in the hmap and the method throws a ResponseStatusException with the HttpStatus.NO_CONTENT status code, which indicates that the DELETE request was successful and not returning any content. If the hmap.get(id) returns null, the method throws a ResponseStatusException with the HttpStatus.NOT_FOUND status code, which indicates that the requested resource was not found.

Now, our logic to delete a book in the hmap is ready. So, we have to write a method in our BookController class to handle the HTTP DELETE request.

Let's use /books/{bookId} as a path to identify a single book resource, where bookId is a path parameter.

So, let's create a deleteBook() method that handles the HTTP DELETE request to the path as /books/{bookId}

```java
@DeleteMapping("/books/{bookId}")
public void deleteBook(@PathVariable("bookId") int bookId) {
    // method body
}
```

The deleteBook() method in the above code is annotated with the @DeleteMapping("/books/{bookId}"), which is a request mapping annotation that maps a specific HTTP DELETE request to the method. In our case, the deleteBook() method is mapped to the URL /books/{bookId} and will handle HTTP DELETE requests to that URL.

We have also used the @PathVariable annotation to access the path parameters.

### File: BookController.java

```java
package com.example.goodreads;

import org.springframework.web.bind.annotation.*;
import java.util.*;
import com.example.goodreads.BookService;

@RestController
public class BookController {
    BookService bookService = new BookService();
```

Request
post request book

Response
post response book

---

## Summary

### Creating a Book (POST request)

The Spring Boot annotation @PostMapping() is used to handle the HTTP POST requests to the specified path.
The @RequestBody annotation is used to bind the request body to the method parameter.

### Updating a Book

The Spring Boot annotation @PutMapping() is used to handle the HTTP PUT requests to the specified path.

### Deleting a Book

The Spring Boot annotation @DeleteMapping() is used to handle the HTTP DELETE requests to the specified path.
