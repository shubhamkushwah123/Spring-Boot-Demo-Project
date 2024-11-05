# Spring-Boot-Demo-Project

Here's the modified solution that incorporates using the H2 database and preloading some sample data with `data.sql`. User authentication and AOP are excluded, as requested.

---

### Updated Project Overview
This Spring Boot project will use an H2 in-memory database to store and manage To-Do tasks, with CRUD (Create, Read, Update, Delete) operations.

### Prerequisites
- Java 11 or higher
- Spring Boot 3.x
- H2 Database
- Postman (for testing)

### Project Structure
The project structure remains similar, with the addition of a `data.sql` file to preload data in the H2 database:
```
src/
├── main/
│   ├── java/
│   │   └── com.example.todoapp/
│   │       ├── controller/
│   │       ├── model/
│   │       ├── repository/
│   │       ├── service/
│   │       └── TodoAppApplication.java
│   └── resources/
│       ├── application.properties
│       └── data.sql (preloads data)
```

### Step-by-Step Instructions

#### Step 1: Set Up the Project
1. Create a new Spring Boot project using [Spring Initializr](https://start.spring.io/).
   - **Dependencies**: Spring Web, Spring Data JPA, H2 Database, Lombok (optional for boilerplate code reduction).

#### Step 2: Configure the H2 Database
In `application.properties`, configure H2 as the database:
```properties
# H2 Database Configuration
spring.datasource.url=jdbc:h2:mem:todoappdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password

# JPA Settings
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=update
spring.h2.console.enabled=true
```

- The H2 console will be accessible at `http://localhost:8080/h2-console`. Use `jdbc:h2:mem:todoappdb` as the JDBC URL.

#### Step 3: Create the `Todo` Model
The `Todo` model remains the same:

```java
package com.example.todoapp.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import lombok.Data;

@Entity
@Data
public class Todo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private String description;
    private boolean completed;
}
```

#### Step 4: Create Repository and Service Layers
These layers remain as previously described.

**Repository**:
```java
package com.example.todoapp.repository;

import com.example.todoapp.model.Todo;
import org.springframework.data.jpa.repository.JpaRepository;

public interface TodoRepository extends JpaRepository<Todo, Long> {
}
```

**Service**:
```java
package com.example.todoapp.service;

import com.example.todoapp.model.Todo;
import com.example.todoapp.repository.TodoRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class TodoService {
    @Autowired
    private TodoRepository todoRepository;

    public List<Todo> getAllTodos() {
        return todoRepository.findAll();
    }

    public Optional<Todo> getTodoById(Long id) {
        return todoRepository.findById(id);
    }

    public Todo createTodo(Todo todo) {
        return todoRepository.save(todo);
    }

    public Todo updateTodo(Long id, Todo todoDetails) {
        Todo todo = todoRepository.findById(id).orElseThrow();
        todo.setTitle(todoDetails.getTitle());
        todo.setDescription(todoDetails.getDescription());
        todo.setCompleted(todoDetails.isCompleted());
        return todoRepository.save(todo);
    }

    public void deleteTodoById(Long id) {
        todoRepository.deleteById(id);
    }
}
```

#### Step 5: Create the Controller
The `TodoController` remains as before.

```java
package com.example.todoapp.controller;

import com.example.todoapp.model.Todo;
import com.example.todoapp.service.TodoService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/todos")
public class TodoController {
    @Autowired
    private TodoService todoService;

    @GetMapping
    public List<Todo> getAllTodos() {
        return todoService.getAllTodos();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Todo> getTodoById(@PathVariable Long id) {
        return todoService.getTodoById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public Todo createTodo(@RequestBody Todo todo) {
        return todoService.createTodo(todo);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Todo> updateTodo(@PathVariable Long id, @RequestBody Todo todoDetails) {
        return ResponseEntity.ok(todoService.updateTodo(id, todoDetails));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteTodoById(@PathVariable Long id) {
        todoService.deleteTodoById(id);
        return ResponseEntity.noContent().build();
    }
}
```

#### Step 6: Preload Data with `data.sql`
In `src/main/resources`, create a `data.sql` file to populate the database with initial data.

```sql
INSERT INTO TODO (ID, TITLE, DESCRIPTION, COMPLETED) VALUES (1, 'Buy groceries', 'Milk, Bread, Eggs, and Vegetables', false);
INSERT INTO TODO (ID, TITLE, DESCRIPTION, COMPLETED) VALUES (2, 'Pay bills', 'Electricity and Water bills', false);
INSERT INTO TODO (ID, TITLE, DESCRIPTION, COMPLETED) VALUES (3, 'Go to gym', 'Evening workout', true);
```

### Testing the Endpoints with Postman
The endpoints and JSON payloads remain the same:

#### 1. Create a To-Do
- **Endpoint:** `POST /api/todos`
- **JSON Payload:**
  ```json
  {
    "title": "Read a book",
    "description": "Finish reading the current novel",
    "completed": false
  }
  ```

#### 2. Get All To-Dos
- **Endpoint:** `GET /api/todos`

#### 3. Get a To-Do by ID
- **Endpoint:** `GET /api/todos/{id}`

#### 4. Update a To-Do
- **Endpoint:** `PUT /api/todos/{id}`
- **JSON Payload:**
  ```json
  {
    "title": "Update project",
    "description": "Finish the Spring Boot project",
    "completed": true
  }
  ```

#### 5. Delete a To-Do by ID
- **Endpoint:** `DELETE /api/todos/{id}`

---

This configuration completes the To-Do application with the H2 in-memory database and preloaded data. Let me know if you need additional changes or further assistance!
