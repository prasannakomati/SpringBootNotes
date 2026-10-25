# Java Database Connectivity, Hibernate, JPA, and Spring Data JPA
## Beginner-friendly notes

These notes explain four related technologies used to connect a Java application to a relational database.

---

## 1. JDBC

**JDBC** stands for **Java Database Connectivity**.

JDBC is a Java API that lets a Java application connect to a database, execute SQL statements, and read the results.

### How JDBC works

`Java application → JDBC API and driver → Database`

For example, Java can send this SQL query to a database:

```sql
SELECT * FROM employee;
```

The database returns matching employee records to the Java application.

### Advantages

- Connects Java applications to databases.
- Supports SQL operations such as `SELECT`, `INSERT`, `UPDATE`, and `DELETE`.
- Supports transactions, including commit and rollback.
- Provides direct control over SQL statements.
- Is a standard API supported by many database vendors through JDBC drivers.

### Disadvantages

- Requires more repetitive code for connections, statements, and results.
- Developers must map database rows to Java objects themselves in many cases.
- Resources must be closed correctly; try-with-resources helps.
- Requires SQL knowledge and exception handling.

### Simple JDBC example

```java
Connection con = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/company",
    "root",
    "password"
);

PreparedStatement ps =
    con.prepareStatement("SELECT * FROM employee");

ResultSet rs = ps.executeQuery();

while (rs.next()) {
    System.out.println(rs.getString("name"));
}

rs.close();
ps.close();
con.close();
```

This example assumes MySQL, a configured database, and an available MySQL JDBC driver. In real projects, prefer try-with-resources to ensure resources are closed even if an error occurs.

---

## 2. Hibernate

**Hibernate** is an **Object-Relational Mapping (ORM) framework** for Java.

ORM maps Java objects to database tables. Hibernate can generate SQL for common persistence operations, so developers often write less repetitive JDBC code.

### How Hibernate works

`Java objects → Hibernate ORM → JDBC → Database`

For example, an `Employee` Java object can be mapped to an `employee` database table.

```java
Employee emp = new Employee();
emp.setId(1);
emp.setName("Prasanna");
emp.setSalary(25000);

// Assuming a correctly configured Hibernate Session
session.persist(emp);
```

Hibernate can generate the SQL needed to persist the entity. The transaction must be committed for the database change to be committed.

### Advantages

- Reduces repetitive database-access code.
- Maps Java objects to database records.
- Supports relationships such as one-to-one and one-to-many.
- Can work with different relational databases using suitable configuration.
- Supports caching and transaction integration.

### Disadvantages

- Has concepts to learn, such as sessions, entity states, mappings, and transactions.
- Generated SQL can be difficult to debug without SQL logging.
- Poor fetch strategies can cause performance problems, including the N+1 query problem.
- Complex or database-specific queries may require HQL, JPQL, or native SQL.
- ORM processing can add overhead in some workloads.

---

## 3. JPA

**JPA** historically stands for **Java Persistence API**. The current specification is called **Jakarta Persistence**.

JPA is a **specification**, not a database engine and not, by itself, a concrete ORM implementation. It defines standard interfaces, annotations, and rules for mapping Java objects to relational databases.

Popular JPA implementations include Hibernate.

### Key JPA concepts

- `@Entity`: marks a class as a persistent entity.
- `@Id`: identifies the primary key.
- `EntityManager`: provides operations for managing entities.
- `persist()`: makes a new entity persistent.
- `find()`: retrieves an entity by its primary key.
- Transactions: group database operations into a unit of work.

### Simple entity example

```java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;

@Entity
public class Employee {
    @Id
    private Integer id;

    private String name;
    private double salary;

    public Employee() {
    }

    // Getters and setters
}
```

The example uses `jakarta.persistence` imports, as used by modern Jakarta-based versions. Older Java projects may use `javax.persistence` instead.

### JPA disadvantages and ways to reduce the problems

| Potential disadvantage | How to address it |
|---|---|
| Learning curve | Learn entities, CRUD, relationships, and transactions step by step |
| Performance issues | Retrieve only needed data; use pagination and measure query performance |
| Complex queries | Use JPQL, Criteria API, or native SQL when appropriate |
| Hard-to-understand generated SQL | Enable SQL logging in development and inspect query plans |
| N+1 query problem | Consider fetch joins, entity graphs, or batch fetching |
| Database-specific features | Use native SQL or JDBC when appropriate |
| Configuration complexity | Use well-documented framework defaults and test configuration |

These are potential issues, not guaranteed problems. The outcome depends on the JPA implementation, configuration, database, and query design.

---

## 4. JPQL, HQL, and Native SQL

These are ways to express database queries.

### Native SQL

Native SQL is SQL written for the target database. It uses database table and column names.

```sql
SELECT *
FROM employee
WHERE department = 'IT'
  AND salary > 30000;
```

Use it when you need database-specific SQL features or precise SQL control.

### HQL

**HQL** means **Hibernate Query Language**. It uses Hibernate entity names and properties rather than directly targeting table names in the usual way.

```java
String hql =
    "FROM Employee e " +
    "WHERE e.department = :dept " +
    "AND e.salary > :salary";

List<Employee> employees = session
    .createQuery(hql, Employee.class)
    .setParameter("dept", "IT")
    .setParameter("salary", 30000.0)
    .getResultList();
```

This assumes `Employee` has `department` and `salary` properties.

### JPQL

**JPQL** means **Jakarta Persistence Query Language**. It is part of the JPA/Jakarta Persistence specification and queries entities and their properties.

```java
String jpql =
    "SELECT e FROM Employee e " +
    "WHERE e.department = :dept " +
    "AND e.salary > :salary";

List<Employee> employees = entityManager
    .createQuery(jpql, Employee.class)
    .setParameter("dept", "IT")
    .setParameter("salary", 30000.0)
    .getResultList();
```

This also assumes the entity has the stated properties.

### Quick comparison

| Query type | Usually targets | Standard or framework |
|---|---|---|
| Native SQL | Database tables and columns | Database SQL dialect |
| HQL | Hibernate entities and properties | Hibernate |
| JPQL | JPA entities and properties | JPA/Jakarta Persistence |

---

## 5. Spring Data JPA

**Spring Data JPA** is a Spring project that simplifies data access by providing repository abstractions on top of JPA.

Instead of writing the implementation for common CRUD methods, developers can define a repository interface. Spring Data JPA creates a runtime implementation and registers it as a Spring bean.

### Typical flow

`Controller → Service → Spring Data JPA Repository → JPA provider (often Hibernate) → JDBC → Database`

This is a common arrangement in a Spring Boot application. Exact internal details depend on configuration and the selected provider.

### Step 1: Create an entity

```java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;

@Entity
public class Employee {
    @Id
    private Integer id;

    private String name;
    private double salary;

    public Employee() {
    }

    // Getters and setters
}
```

### Step 2: Create a repository

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface EmployeeRepository
        extends JpaRepository<Employee, Integer> {
}
```

Here:
- `Employee` is the entity type.
- `Integer` is the primary-key Java type.
- `JpaRepository` supplies common persistence methods.

You do not instantiate this interface with `new EmployeeRepository()`. Spring Data JPA creates and manages a repository implementation at runtime.

### Step 3: Use the repository in a service

```java
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class EmployeeService {
    private final EmployeeRepository repository;

    public EmployeeService(EmployeeRepository repository) {
        this.repository = repository;
    }

    public Employee addEmployee(Employee employee) {
        return repository.save(employee);
    }

    public List<Employee> getEmployees() {
        return repository.findAll();
    }

    public void deleteEmployee(Integer id) {
        repository.deleteById(id);
    }
}
```

### Common repository methods

| Method | Purpose |
|---|---|
| `save(entity)` | Saves or updates an entity, depending on its state and identifier strategy |
| `findAll()` | Retrieves all entities; be careful with large tables |
| `findById(id)` | Retrieves an entity by primary key and returns an `Optional` |
| `deleteById(id)` | Deletes by primary key; behavior for a missing ID depends on the method/version |
| `count()` | Counts entities |

Spring Data JPA can also derive queries from method names, for example:

```java
List<Employee> findByDepartment(String department);
```

This requires a matching `department` property on the entity.

### Advantages

- Reduces boilerplate code for common CRUD operations.
- Provides repository interfaces and query derivation.
- Supports pagination and sorting.
- Integrates with Spring dependency injection and transaction management.
- Can use JPA providers such as Hibernate.

### Limitations to remember

- Complex queries may still require JPQL or native SQL.
- Poor query design and fetch configuration can still cause performance problems.
- Understanding SQL, transactions, and entity relationships remains important.

---

## 6. JDBC vs Hibernate vs JPA vs Spring Data JPA

| Technology | What it is | Main purpose |
|---|---|---|
| JDBC | Java API | Execute SQL and communicate with the database |
| Hibernate | ORM framework and popular JPA implementation | Map objects to database records and manage persistence |
| JPA / Jakarta Persistence | Specification | Define standard persistence APIs and mapping behavior |
| Spring Data JPA | Spring data-access project | Simplify persistence with repositories |

### Important clarification

JPA and Hibernate are not exact alternatives: Hibernate commonly implements JPA. Spring Data JPA is a higher-level repository abstraction that commonly uses Hibernate underneath. JDBC is a lower-level database API that Hibernate commonly uses for database communication.

---

## 7. Final revision

Remember these four lines:

1. **JDBC:** Java code uses SQL to communicate with a database.
2. **Hibernate:** An ORM framework that maps Java objects to database records.
3. **JPA:** A standard specification for Java persistence; Hibernate is one implementation.
4. **Spring Data JPA:** Simplifies persistence by providing repository interfaces and common database methods.

**Interview-ready answer:** Spring Data JPA reduces boilerplate database-access code by providing repository abstractions over JPA. Hibernate is a common JPA implementation, and it commonly uses JDBC to communicate with a relational database.

---

## 8. Suggested learning order

1. Learn SQL basics and JDBC.
2. Understand entities, primary keys, and ORM.
3. Learn JPA concepts such as `EntityManager` and transactions.
4. Practice Hibernate entity mapping and relationships.
5. Use Spring Data JPA repositories in a Spring Boot project.
6. Learn JPQL, native queries, pagination, transactions, and query optimization.
