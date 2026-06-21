# backend-interview-handbook
Complete backend interview preparation repository with Java, Spring Boot, Microservices, Security, SQL, System Design, and interview-ready explanations.
---

**hidden Spring/JPA internal interview questions

**JpaRepository is an interface. Then how does `save()` work if you never implemented it?**

### Interview Ready Answer:

> "JpaRepository is an interface so it only declares methods and does not contain implementation. During application startup, Spring Data JPA scans repository interfaces and dynamically generates implementation classes internally. Behind the scenes Spring uses `SimpleJpaRepository`, which implements JpaRepository methods like save(), findById(), findAll(). When we call `departmentRepo.save()`, actual execution happens inside generated repository implementation and finally goes through EntityManager and Hibernate to execute SQL."

```text
JpaRepository
↓

Spring generates implementation

↓

SimpleJpaRepository

↓

EntityManager

↓

Hibernate

↓

Database
```

---

**Who calls Controller method internally in Spring Boot?**
> "DispatcherServlet receives every request but it does not directly execute controller methods. It uses HandlerMapping to locate the endpoint and HandlerAdapter to prepare method arguments and invoke the controller method."

Flow:

```text
DispatcherServlet
↓

HandlerMapping

↓

HandlerAdapter

↓

Controller
```

---
**Who creates DTO object from JSON request?**
> "Spring creates DTO automatically using HttpMessageConverter. It converts incoming JSON into Java object and injects it into controller method parameters."

Example:

```json
{
"name":"IT"
}
```

↓

```java
DepartmentDto dto
```

---

**What happens internally when `save()` is called?**
> "When save() is called, Spring delegates execution to SimpleJpaRepository. Internally it uses EntityManager. If entity ID is null, EntityManager performs persist operation and Hibernate generates INSERT SQL. If ID exists, merge operation is performed and UPDATE SQL is generated."

Flow:

```text
save()

↓

SimpleJpaRepository

↓

EntityManager

↓

Hibernate

↓

SQL
```

---

**Why do we use DTO instead of Entity in Controller?**
> "DTO is used to transfer request and response data between client and application. Entity represents database structure. Using DTO avoids exposing internal database fields and provides loose coupling."

Simple:

```text
DTO → API

Entity → Database
```

---

**What is EntityManager?**
> "EntityManager is JPA's core interface used to manage entity lifecycle. It performs operations like persist, merge, remove and fetch while interacting with Hibernate."
Methods:
```text
persist()
merge()
remove()
find()
```

---

**How does Spring inject `@Autowired` dependency?**
> "Spring IoC container creates beans during application startup and manages their lifecycle. When a class needs dependency injection, Spring resolves the bean and injects it automatically."

Flow:

```text
Application Start

↓

Bean Creation

↓

Dependency Injection
```

---
**What is difference between `persist()` and `merge()`?**
> "`persist()` is used for inserting new entity and makes it managed by persistence context. `merge()` is used to update detached entities and synchronize changes with database."

Rule:

```text
persist()
→ Insert

merge()
→ Update
```

---

**Who converts Java object into JSON response?**
> "Spring uses HttpMessageConverter internally to serialize Java objects into JSON before sending response back to client."

Flow:

```text
Java Object

↓

JSON
```

---
**What is hidden behind Repository → Hibernate flow?**
> "Repository abstracts database operations. Internally generated repository implementation delegates to EntityManager, which further uses Hibernate to generate SQL and interact with database."

Flow:

```text
Repository

↓

SimpleJpaRepository

↓

EntityManager

↓

Hibernate

↓

Database
```

---------------------------------------------------------------
---
**FULL REST API INTERNAL FLOW** from beginning to end with **who calls whom + hidden methods + actual work 
---

# FULL POST REQUEST FLOW (SAVE DEPARTMENT)

User sends:

```http
POST /department
```

Body:

```json
{
 "departmentCode":"IT001",
 "departmentName":"IT"
}
```

---

```text
1. Postman / UI
(creates HTTP request and sends to server)
```

↓

```text
2. Tomcat Server
(receives HTTP request and forwards to Spring)
```

Hidden:

```java
tomcat.accept(request)
```

↓

---

```text
3. DispatcherServlet
(front controller → receives ALL requests)
```

Hidden:

```java
doDispatch()
```

Work:

```text
Find which controller method should execute
```

Reads:

```http
POST /department
```

↓

---

```text
4. HandlerMapping
(searches matching endpoint)
```

Hidden:

```java
getHandler()
```

Search:

```java
@PostMapping("/department")
```

Returns:

```text
DepartmentController.save()
```

↓

---

```text
5. HandlerAdapter
(executes controller method)
```

Hidden:

```java
handle()
```

Work:

```text
Prepare method parameters and invoke method
```

Sees:

```java
save(
@RequestBody
DepartmentDto dto
)
```

↓

---

```text
6. HttpMessageConverter
(JSON → Java Object)
```

Hidden:

```java
read()
```

Internally:

```java
DepartmentDto dto=
new DepartmentDto();
```

fills:

```java
dto.setDepartmentCode(
"IT001"
);
```

Result:

```java
dto={
 code:"IT001"
}
```

↓

---

```text
7. Controller Method Executes
(receives ready DTO)
```

Actual:

```java
controller.save(dto)
```

Controller:

```java
departmentService
.addDepartment(dto)
```

↓

---

```text
8. Spring IoC Container
(gives Service bean)
```

Hidden:

```java
getBean()
```

Work:

```text
Inject Service object
```

Controller already contains:

```java
departmentService
```

↓

---

```text
9. Service Layer Executes
(business logic layer)
```

Method:

```java
addDepartment(dto)
```

Work:

```text
DTO → Entity conversion
```

↓

---

```text
10. ModelMapper
(copy field values)
```

Hidden:

```java
map()
```

Internally:

```java
Department entity=
new Department();
```

Copies:

```java
dto
↓

entity
```

↓

---

```text
11. Repository Called
(request DB operation)
```

Code:

```java
departmentRepo.save(
entity
)
```

You call interface.

↓

---

```text
12. Spring Generates Repo Implementation
(actual worker object)
```

Generated internally:

```java
DepartmentRepoImpl
extends
SimpleJpaRepository
```

↓

---

```text
13. SimpleJpaRepository.save()
(real save method executes)
```

Hidden:

```java
save()
```

Checks:

```text
ID null?
```

If yes:

```text
Insert
```

If no:

```text
Update
```

↓

---

```text
14. EntityManager
(manages entity lifecycle)
```

Hidden:

```java
persist()
```

Work:

```text
Prepare entity for DB
```

↓

---

```text
15. Hibernate
(converts Entity → SQL)
```

Hidden:

```java
generate SQL
```

Creates:

```sql
insert into department...
```

↓

---

```text
16. Database
(executes SQL and saves row)
```

DB returns:

```text
Generated ID
```

Example:

```text
id=1
```

↓

---

```text
17. Hibernate Updates Entity
(populates generated values)
```

Before:

```java
id=null
```

After:

```java
id=1
```

↓

---

```text
18. Repository Returns Entity
(saved entity moves upward)
```

Flow:

```text
DB

↓

Hibernate

↓

EntityManager

↓

Repository

↓

Service
```

↓

---

```text
19. Service Converts Entity → DTO
(prepare response object)
```

Hidden:

```java
modelMapper.map()
```

Result:

```java
DepartmentDto
```

↓

---

```text
20. Controller Returns DTO
(response leaves controller)
```

Code:

```java
return dto;
```

↓

---

```text
21. HttpMessageConverter
(Java Object → JSON)
```

Hidden:

```java
write()
```

Creates:

```json
{
"id":1
}
```

↓

---

```text
22. DispatcherServlet
(receives final response)
```

↓

---

```text
23. Tomcat
(sends HTTP response)
```

↓

---

```text
24. Postman/UI
(displays response)
```

---

# COMPLETE ONE-LINE FLOW

```text
Postman
↓

Tomcat
↓

DispatcherServlet

↓

HandlerMapping

↓

HandlerAdapter

↓

HttpMessageConverter

(JSON→DTO)

↓

Controller

↓

Service

↓

ModelMapper

(DTO→Entity)

↓

Repository

↓

Generated Repo Impl

↓

SimpleJpaRepository

↓

EntityManager

↓

Hibernate

↓

Database

↓

Hibernate

↓

Repository

↓

Service

(Entity→DTO)

↓

Controller

↓

HttpMessageConverter

(Java→JSON)

↓

DispatcherServlet

↓

Tomcat

↓

Postman
```

---

# Debug Shortcut

```text
404
→ HandlerMapping

400
→ DTO conversion

500
→ Service/Repo

NullPointer
→ Bean Injection

SQL Error
→ Hibernate/DB
```

This is the complete request lifecycle used in Spring Boot REST APIs.

