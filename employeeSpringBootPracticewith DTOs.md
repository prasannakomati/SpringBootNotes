# Employee Spring Boot Practice

## 1. EmployeeSpringBootPracticeApplication.java

``` java
package com.example.employeeSpringBootPractice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class EmployeeSpringBootPracticeApplication
{

    public static void main(String[] args)
    {
        SpringApplication.run(EmployeeSpringBootPracticeApplication.class, args);
    }

}
```

## 2. EmployeeController.java

``` java
package com.example.employeeSpringBootPractice.controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestController;

import com.example.employeeSpringBootPractice.dto.EmployeeRequestDto;
import com.example.employeeSpringBootPractice.dto.EmployeeResponseDto;
import com.example.employeeSpringBootPractice.service.EmployeeService;

@RestController
@RequestMapping("/employees")
public class EmployeeController
{

    @Autowired
    private EmployeeService empservice;


    //CREATE
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public EmployeeResponseDto addEmployee(@RequestBody EmployeeRequestDto requestDto)
    {

        return empservice.AddEmployee(requestDto);
    }


    //GETALLEMPLOYEES
    @GetMapping
    public List<EmployeeResponseDto> getEmployees()
    {
        return empservice.getEmployees();
    }


    @GetMapping("/{id}")
    public EmployeeResponseDto getEmployeeById(@PathVariable int id)
    {
        return empservice.getEmployeeById(id);


    }

       // =========================================
       // UPDATE
       // =========================================

       @PutMapping("/{id}")
       public EmployeeResponseDto updateEmployee(
               @PathVariable int id,
               @RequestBody EmployeeRequestDto requestDTO) {

           return empservice.updateEmployee(
                   id,
                   requestDTO
           );
       }


       // =========================================
       // DELETE
       //===========================================

       @DeleteMapping("/{id}")
       @ResponseStatus(HttpStatus.NO_CONTENT)
       public void deleteEmployee(
               @PathVariable int id) {

           empservice.deleteEmployee(id);
       }
}
```

## 3. EmployeeRequestDto.java

``` java
package com.example.employeeSpringBootPractice.dto;

public class EmployeeRequestDto
{
    private String name;
    private String email;
    private String password;
    private double salary;

    public EmployeeRequestDto()
    {
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    public String getEmail()
    {
        return email;
    }

    public void setEmail(String email)
    {
        this.email = email;
    }

    public String getPassword()
    {
        return password;
    }

    public void setPassword(String password)
    {
        this.password = password;
    }

    public double getSalary()
    {
        return salary;
    }

    public void setSalary(double salary)
    {
        this.salary = salary;
    }

    //   frotend -> controller -> requestdto -> entity -> response -> controller
}
```

## 4. EmployeeResponseDto.java

``` java
package com.example.employeeSpringBootPractice.dto;

public class EmployeeResponseDto
{
    private int id;
    private String name;
    private String email;
    private double salary;

    public EmployeeResponseDto()
    {
    }

    public EmployeeResponseDto(int id,String name,String email,double salary)
    {
        this.id=id;
        this.name=name;
        this.email=email;
        this.salary=salary;
    }

    public int getId()
    {
        return id;
    }

    public void setId(int id)
    {
        this.id = id;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    public String getEmail()
    {
        return email;
    }

    public void setEmail(String email)
    {
        this.email = email;
    }

    public double getSalary()
    {
        return salary;
    }

    public void setSalary(double salary)
    {
        this.salary = salary;
    }

}
```

## 5. Employee.java

``` java
package com.example.employeeSpringBootPractice.Entity;

import jakarta.persistence.*;

@Entity
@Table(name="Employee")
public class Employee
{
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private int id;
    private String name;
    private String email;
    private String password;
    private double salary;

    public Employee()
    {
    }

    public Employee(String name,String email,String password,double salary)
    {
        this.name=name;
        this.email=email;
        this.password=password;
        this.salary=salary;
    }

    public int getId()
    {
        return id;
    }

    public void setId(int id)
    {
        this.id = id;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    public String getEmail()
    {
        return email;
    }

    public void setEmail(String email)
    {
        this.email = email;
    }

    public String getPassword()
    {
        return password;
    }

    public void setPassword(String password)
    {
        this.password = password;
    }

    public double getSalary()
    {
        return salary;
    }

    public void setSalary(double salary)
    {
        this.salary = salary;
    }



}
```

## 6. EmployeeRepository.java

``` java
package com.example.employeeSpringBootPractice.repository;

import org.springframework.data.jpa.repository.JpaRepository;

import com.example.employeeSpringBootPractice.Entity.Employee;

public interface EmployeeRepository extends JpaRepository<Employee,Integer>
{


}
```

## 7. EmployeeService.java

``` java
package com.example.employeeSpringBootPractice.service;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.example.employeeSpringBootPractice.Entity.Employee;
import com.example.employeeSpringBootPractice.dto.EmployeeRequestDto;
import com.example.employeeSpringBootPractice.dto.EmployeeResponseDto;
import com.example.employeeSpringBootPractice.repository.EmployeeRepository;

@Service
public class EmployeeService
{
    @Autowired
    private EmployeeRepository employeerepository;



    //CREATE EMPLOYEE
    public EmployeeResponseDto AddEmployee(EmployeeRequestDto requestDto)
    {
        Employee employee =new Employee();
        employee.setName(requestDto.getName());
        employee.setEmail(requestDto.getEmail());
        employee.setPassword(requestDto.getPassword());
        employee.setSalary(requestDto.getSalary());

        //save entity to database

        Employee saveEmployee=employeerepository.save(employee);

        //Employeee->Response

        EmployeeResponseDto responseDTO=new EmployeeResponseDto(
                saveEmployee.getId(),
                saveEmployee.getName(),
                saveEmployee.getEmail(),
                saveEmployee.getSalary()
                );

        return responseDTO;

    }

    //GET ALL EMPLOYEES

    public List<EmployeeResponseDto> getEmployees()
    {
        List<Employee> employees=employeerepository.findAll();
        List<EmployeeResponseDto> responseList=new ArrayList<>();

        for(Employee employee:employees)
        {
            EmployeeResponseDto dto=new EmployeeResponseDto(
                    employee.getId(),
                    employee.getName(),
                    employee.getEmail(),
                    employee.getSalary()

                    );

            responseList.add(dto);
        }
        return responseList;
    }

    //GET EMPLOYEE BY ID

    public EmployeeResponseDto getEmployeeById(int id)
    {
        Employee employee=employeerepository.findById(id).orElseThrow();

        EmployeeResponseDto responseDto=new EmployeeResponseDto(
                employee.getId(),
                employee.getName(),
                employee.getEmail(),
                employee.getSalary()
                );

        return responseDto;
    }


    //UPDATE EMPLOYEE

    public EmployeeResponseDto updateEmployee(
               int id,
               EmployeeRequestDto requestDTO) {


        Employee employee =
                employeerepository
                        .findById(id)
                        .orElseThrow();


        // Request DTO → Entity

        employee.setName(requestDTO.getName());

        employee.setEmail(requestDTO.getEmail());

        employee.setPassword(requestDTO.getPassword());

        employee.setSalary(requestDTO.getSalary());


        Employee updatedEmployee =
                employeerepository.save(employee);


        // Entity → Response DTO

        return new EmployeeResponseDto(

                updatedEmployee.getId(),

                updatedEmployee.getName(),

                updatedEmployee.getEmail(),

                updatedEmployee.getSalary()
        );
    }


       // =========================================
       // DELETE EMPLOYEE
       // =========================================

       public void deleteEmployee(int id) {

           Employee employee =
                   employeerepository
                           .findById(id)
                           .orElseThrow();


           employeerepository.delete(employee);



        }

}
```

## 8. application.properties

``` properties
spring.application.name=employeeSpringBootPractice
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

## 9. pom.xml

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
         http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.5.4</version>
        <relativePath/>
    </parent>
    <groupId>com.example</groupId>
    <artifactId>employeeSpringBootPractice</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>employee-management</name>
    <description>
        Employee Management API using Spring Boot DTO
    </description>
    <properties>
        <java.version>17</java.version>
    </properties>
    <dependencies>
        <!-- Spring Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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
