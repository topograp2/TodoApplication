# Todo List Application Project

## 화면 구성

![image](https://github.com/user-attachments/assets/94b22a39-3e56-43a1-b824-450bd92fef49)


## Technologies & Tools

- Spring Boot
- JPA ( Hibernate ) + MySQL
- Lombok
- Thymeleaf + Bootstrap CSS

## API 구성

| HTTP Method | EndPoint | Descriptcion | Parameters | Return View |
| --- | --- | --- | --- | --- |
| GET | /tasks | 모든 task를 찾고 보여주기 | None | tasks(view) |
| POST | /tasks | 새로운 task를 만들기 | title(String, in request param)  | Redirect to /tasks |
| POST | /tasks/{id}/update | Updates the title of an existing task | id (Path), title (String) | Redirect to /tasks |
| GET | /tasks/{id}/delete | Deletes a specific task by ID | id(Path) | Redirect to /tasks |
| GET | /tasks/{id}/toggle | Toggles the completed status of a task | id(Paths) | Redirect to /tasks |

## Spring intializr

- Spring web
- Lombok
- JPA
- MySQL Driver
- Thymleaf

[generate]→ 압축풀고 IDE로 열기

<aside>
💡

프로젝트 진행 중에 dependency를 추가하고 싶다면?

start.spring.io에서 add… 눌러서 추가하고 싶은 거 추가→ [explore] → pom.xml에서 해당되는 dependency 복사 → 프로젝트에서 pom.xml에 붙여넣기 → [Load Maven Changes]

</aside>

## DB 연결

- 먼저 DB를 만들어주기
    - work bench에서 schema >마우스 우클릭> create schema
- todoapp/src/main/resources/application.properties에 추가

```
spring.datasource.url=jdbc:mysql://localhost:3306/todo-app
spring.datasource.username=root
spring.datasource.password=비빌번호
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
```

- url → <protocol>:<database type>:<port number>/<db-name>

## model 설정

```java
package com.app.todoapp.models;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import lombok.Data;

@Entity
@Data
public class Task {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String title;
    private boolean completed;

}

```

models/Task.java에서는 entity를 구성해줍니다.

- Jpa를 통해서 해당 class가 table로 변환이 되게 되는데 이를 나타내기 위해서는 **@Entity**  어노테이션을 사용해줍니다.
- **@Data** 는 Getter, Setter, ToString을 이미 포함하기 때문에 코드가 복잡해지는 것을 막아줍니다.
- Task는 id, title, completed로 세 개의 column으로 이루어집니다.
    - @Id : 말 그대로 Id라는 것을 나타내는 것입니다.
    - **@GeneratedValue( strategy = GenerationType.AUTO)** : Id를 자동으로 생성해주게 됩니다.

## Repository 설정

```java
package com.app.todoapp.repository;

import com.app.todoapp.models.Task;
import org.springframework.data.jpa.repository.JpaRepository;

public interface TaskRepository extends JpaRepository<Task, Long> {

}

```

- JpaRepository를 이용해서 repository를 만들어줍니다. 여기서는 Task를 담을 거고, Task에서 id를 key로 사용할 것이므로 key의 자료형인 Long을 넣어줍니다.

## Controller와 Service의 차이

- **Controller**:
    - 사용자의 요청을 받아 처리하는 역할.
    - HTTP 요청을 매핑하고, 응답을 생성.
    - `Service` 계층과 상호작용하여 비즈니스 로직을 실행하도록 위임.
    - 주로 입력 검증, URL 라우팅 등을 담당.
- **Service**:
    - 실제 비즈니스 로직을 담당.
    - 데이터베이스와의 상호작용 및 복잡한 처리를 처리.
    - `Controller`에서 요청을 받으면, 데이터를 처리하고 결과를 반환.

## getTask와 createTask 구현

### task.html

```java
<html lang="en" xmlns="http://www.thymeleaf.org" >
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Todo Application</title>
</head>
<body>
<h1>Todo Application</h1>
<div>
    <form action="/" method="post">
        <div>
            <input type="text" name="title" placeholde="Add new task" required/>
        </div>
        <div>
            <button type="submit">Add</button>
        </div>
    </form>
    <ul>
        <li th:each="task : ${tasks}" >
<p th:text="${task.title}">Task Title</p>
        </li>
    </ul>
</div>
</body>
</html>
```

### TaskController

```java
package com.app.todoapp.controller;

import com.app.todoapp.models.Task;
import com.app.todoapp.service.TaskService;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.util.List;

@Controller
//@RequestMapping("/tasks")
public class TaskController {

    private final TaskService taskService;

    public TaskController(TaskService taskService) {
        this.taskService = taskService;
    }

    @GetMapping
    public String getTasks(Model model){
        List<Task> tasks = taskService.getAllTasks();
        model.addAttribute("tasks", tasks);
        return "tasks";
    }

    @PostMapping
    public String createTasks(@RequestParam String title){
        taskService.createTask(title);
        return "redirect:/";
    }

}

```

- taskService에 있는 메서드를 황용하여 controller에서 view를 반환하게 됩니다!
- 여기서는 thymeleaf를 사용하였기 때문에 **@Controller**를 사용했습니다.
    - JSON/XML 반환을 위해서는 **@RestController**를 사용해주면 됩니다!
- **@RequestParam**은 **쿼리 파라미터** 또는 **폼 데이터**로 전달된 값을 메서드의 매개변수로 매핑할 때 사용됩니다.
    - 여기서는 타임리프에서 name=”title”과 매핑되게 됩니다.

### TaskService

```java
package com.app.todoapp.service;

import com.app.todoapp.models.Task;
import com.app.todoapp.repository.TaskRepository;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class TaskService {

    private final TaskRepository taskRepository;

    public TaskService(TaskRepository taskRepository) {
        this.taskRepository = taskRepository;
    }

    public List<Task> getAllTasks() {
        return taskRepository.findAll();
    }

    public void createTask(String title) {
        Task task = new Task();
        task.setTitle(title);
        task.setCompleted(false);
        taskRepository.save(task);
    }

}

```

- 실제 비지니스 로직을 구현했습니다!
- repository.findAll() ⇒ 레포지토리 내의 모든 task 반환
- repository.save(task) ⇒ 레포지토리에 생성한 task 저장

## Delete, Toggle 구현

### TaskController

```java
@GetMapping("/{id}/delete")
    public String deleteTask(@PathVariable Long id){
        taskService.deleteTask(id);
        return "redirect:/";
    }

    @GetMapping("/{id}/toggle")
    public String toggleTask(@PathVariable Long id){
        taskService.toggleTask(id);
        return "redirect:/";
    }
```

- {{baseurl}}/{id}/delete로 접근하면, 해당되는 id에 deleteTask를 수행하게 됩니다. 해당 비지니스 로직은 역시 taskService에서 구현합니다. 이후 다시 base url로 redirect하게 됩니다.
    - **@PathVariable**은 경로 변수를 표시하기 위해 사용되었습니다.
    - url 경로에서 {id}에 둘러싸인 값을 나타내는데, 값이 없는 경우에는 404가 발생하므로 주의해야합니다.
- toggle도 마찬가지입니다.

### TaskService

```java
public void deleteTask(Long id) {
        taskRepository.deleteById(id);
    }

    public void toggleTask(Long id) {
        Task task = taskRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("Invalid task id"));
        task.setCompleted(!task.isCompleted());
        taskRepository.save(task);
    }
```

- deleteTask에서는 id를 이용해서 해당되는 task를 삭제하게 됩니다.
- toggleTask에서는 id를 이용해서 해당되는 task를 찾아내고 task의 completed를 변경해주고 레포지토리에 저장해줍니다.
    - 이때에 예외처리도 꼭 해줍니다!!!

## 끝나고…

- MVC 패턴을 복습하고 controller와 service의 차이를 확실히 알 수 있는 계기가 되었습니다!
- 간단해서 재밌었고 어노테이션이 어떤 역할을 하는지 확실히 정리하는 게 중요하겠다는 생각이 들었습니다!
- 근데 여기서 Lombok이 말을 안 들어서 @Data로 하면 getter와 setter를 안 써도 되는데, 어쩔 수 없이 @Data를 빼고 getter, setter를 추가하게 되었어요… 뭐가 문제인지는 잘 모르겠네요…

> 참고 강의 :  https://youtu.be/olAva-QT0iw?si=n1a_xU4Tc4XIya0n
>
