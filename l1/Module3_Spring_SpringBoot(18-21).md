# MODULE 3 – SPRING & SPRING BOOT (Chapters 18–21)

---

# CHAPTER 18: CORE SPRING (IOC, DI, BEANS, ANNOTATIONS)

## 18.1 Why Spring Exists (The Problem It Solves)

In plain Java, if class `A` needs class `B`, you write:
```java
class OrderService {
    PaymentService payment = new PaymentService(); // tightly coupled
}
```
Problem: `OrderService` is glued to one specific `PaymentService`. If you want to swap it, mock it for testing, or change its config, you must edit `OrderService`'s code.

**Spring's fix:** Let a container create objects and "inject" them where needed. Your class just declares "I need a PaymentService" and Spring hands it one.

This is called **Inversion of Control (IoC)** — control of object creation is inverted, from your code to the Spring container.

## 18.2 IoC (Inversion of Control)

- Normally: your code controls the flow (you call `new`, you manage lifecycle).
- With IoC: the **Spring Container** controls object creation, wiring, and lifecycle.
- The container = **ApplicationContext** (an interface; `AnnotationConfigApplicationContext`, etc. are implementations).

**Analogy:** In a restaurant, you don't cook your own food (you don't `new Food()`); the kitchen (container) prepares it and serves it to you. You just consume it.

## 18.3 Dependency Injection (DI) — the Mechanism of IoC

DI = the *technique* by which IoC is achieved. Spring "injects" dependencies instead of you creating them manually.

### Types of DI

| Type | How | Pros | Cons |
|---|---|---|---|
| **Constructor Injection** | Dependency passed via constructor | Immutable, mandatory deps guaranteed, best for testing | Slightly more boilerplate |
| **Setter Injection** | Dependency set via setter method | Good for optional deps | Object can exist in incomplete state |
| **Field Injection** | `@Autowired` directly on field | Least code | Hard to test, hides dependencies, **not recommended** |

```java
// Constructor Injection (RECOMMENDED, especially with Spring Boot 4.3+ style)
@Service
class OrderService {
    private final PaymentService paymentService;

    @Autowired // optional if only ONE constructor exists
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}

// Field Injection (AVOID in real projects, common in MCQs though)
@Service
class OrderService {
    @Autowired
    private PaymentService paymentService;
}
```

⚠️ **Capgemini Trap:** If a class has only ONE constructor, `@Autowired` is optional (Spring auto-detects it). If there are MULTIPLE constructors, you MUST annotate the one Spring should use with `@Autowired`.

## 18.4 What is a Bean?

A **Bean** = any object that is created, configured, and managed by the Spring IoC container.

Ways to declare a bean:
```java
@Component      // generic bean
@Service        // business logic layer (semantically same as @Component)
@Repository     // DAO/persistence layer (adds exception translation)
@Controller     // MVC web layer
@RestController // @Controller + @ResponseBody combined

@Configuration
class AppConfig {
    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

### @Component vs @Service vs @Repository vs @Controller

| Annotation | Layer | Special Behavior |
|---|---|---|
| `@Component` | Generic | None extra |
| `@Service` | Business/Service | None extra (purely semantic marker) |
| `@Repository` | DAO/Persistence | Auto-translates DB exceptions into Spring's `DataAccessException` |
| `@Controller` | Web MVC | Returns view names by default |
| `@RestController` | Web REST | `@Controller` + `@ResponseBody` → returns data (JSON) directly |

⚠️ **Trap:** All four (`@Component`, `@Service`, `@Repository`, `@Controller`) are technically interchangeable to the container — they all register a bean. The differences are **semantic/readability** + `@Repository`'s exception translation feature. This is a VERY common MCQ trap ("Can you use @Component instead of @Service?" → Yes, functionally, but bad practice).

## 18.5 Bean Scopes

| Scope | Meaning | Default? |
|---|---|---|
| `singleton` | ONE instance per Spring container (shared everywhere) | ✅ Yes (default) |
| `prototype` | NEW instance every time it's requested | No |
| `request` | One instance per HTTP request (web apps only) | No |
| `session` | One instance per HTTP session (web apps only) | No |
| `application` | One instance per ServletContext | No |

```java
@Component
@Scope("prototype")
class ReportGenerator { }
```

⚠️ **Capgemini Trap:** Default scope is **singleton**, NOT prototype. Many students assume prototype is default — wrong.

⚠️ **Trap 2:** Singleton means one instance **per Spring container**, not one instance for the whole JVM (a subtle distinction if multiple contexts exist).

## 18.6 ApplicationContext vs BeanFactory

| Feature | BeanFactory | ApplicationContext |
|---|---|---|
| Loading | Lazy (creates bean on demand) | Eager (creates singleton beans at startup by default) |
| Features | Basic DI only | DI + AOP + Internationalization + Event publishing + more |
| Usage | Rarely used directly today | Used in almost all real Spring apps |

`ApplicationContext` **extends** `BeanFactory` — it's a more powerful superset.

## 18.7 @Autowired — How It Resolves Beans

1. Spring looks for a bean **by type**.
2. If multiple beans of the same type exist → conflict! Spring then tries to match **by name** (the field/parameter name).
3. If still ambiguous → use `@Qualifier("beanName")` to specify exactly which one.
4. If no matching bean exists → `NoSuchBeanDefinitionException` at startup.

```java
interface Notifier {}

@Component("emailNotifier")
class EmailNotifier implements Notifier {}

@Component("smsNotifier")
class SmsNotifier implements Notifier {}

@Service
class AlertService {
    @Autowired
    @Qualifier("smsNotifier")
    private Notifier notifier; // explicitly picks SmsNotifier
}
```

`@Primary` is another way to resolve ambiguity — marks one bean as the default choice when multiple candidates exist.

```java
@Component
@Primary
class EmailNotifier implements Notifier {}
```

⚠️ **Trap:** If both `@Primary` and `@Qualifier` are used somewhere in the wiring, `@Qualifier` at the injection point wins over `@Primary`.

## 18.8 Bean Lifecycle (High-Level)

1. Container starts → beans instantiated
2. Dependencies injected
3. `@PostConstruct` method called (if present) — "bean is ready"
4. Bean is used by the application
5. Container shuts down → `@PreDestroy` method called — "cleanup"

```java
@Component
class CacheService {
    @PostConstruct
    public void init() { System.out.println("Cache loaded"); }

    @PreDestroy
    public void cleanup() { System.out.println("Cache cleared"); }
}
```

## 18.9 Common Core Annotations Quick Reference

| Annotation | Purpose |
|---|---|
| `@Component` | Marks class as a Spring-managed bean |
| `@Autowired` | Injects a dependency |
| `@Qualifier` | Resolves which bean to inject when multiple candidates exist |
| `@Primary` | Marks default bean among multiple candidates |
| `@Configuration` | Marks class as a source of bean definitions |
| `@Bean` | Declares a bean inside a `@Configuration` class |
| `@Scope` | Defines bean scope (singleton/prototype/etc.) |
| `@Value` | Injects a value from properties file |
| `@PostConstruct` | Runs after bean initialization |
| `@PreDestroy` | Runs before bean destruction |
| `@Lazy` | Delays bean creation until first use |

---

## 🔑 CHAPTER 18 — ONE-PAGE REVISION

- **IoC** = container controls object creation, not you.
- **DI** = mechanism to inject dependencies (constructor > setter > field, in preference order).
- Constructor injection with **1 constructor** → `@Autowired` optional.
- **Bean** = object managed by Spring container.
- `@Component`/`@Service`/`@Repository`/`@Controller` are functionally similar; `@Repository` adds exception translation.
- Default scope = **singleton** (NOT prototype).
- `ApplicationContext` extends `BeanFactory`, adds AOP, events, i18n, and eager loading.
- Ambiguous autowiring → resolve with `@Qualifier` (wins) or `@Primary` (default fallback).
- Lifecycle hooks: `@PostConstruct` (start) and `@PreDestroy` (shutdown).

---

## CHAPTER 18 — MCQs

**Q1.** What does IoC stand for in Spring?
A) Internal object Creation B) Inversion of Control C) Instance of Class D) Interface oriented Coding
**Answer:** B
**Explanation:** IoC means the control of object creation and wiring is transferred from the developer to the Spring container.
**Why others wrong:** A, C, D are made-up/irrelevant terms.
**Difficulty:** Easy | **Topic:** IoC

---

**Q2.** Which is the default bean scope in Spring?
A) prototype B) request C) singleton D) session
**Answer:** C
**Explanation:** Singleton is the default — one shared instance per container.
**Why others wrong:** prototype/request/session all require explicit `@Scope` declaration.
**Difficulty:** Easy | **Topic:** Bean Scope

---

**Q3.** Which annotation is used to resolve ambiguity when multiple beans of the same type exist?
A) @Autowired B) @Component C) @Qualifier D) @Bean
**Answer:** C
**Explanation:** `@Qualifier` lets you specify the exact bean name to inject.
**Why others wrong:** `@Autowired` triggers injection but doesn't resolve ambiguity alone; `@Component`/`@Bean` just declare beans.
**Difficulty:** Medium | **Topic:** Autowiring

---

**Q4.** What happens if a class has two constructors and neither is annotated with `@Autowired`?
A) Spring picks the first one B) Compilation error C) Spring throws an exception since it can't decide which constructor to use D) Both constructors run
**Answer:** C
**Explanation:** With multiple constructors, Spring needs an explicit `@Autowired` to know which one to use for injection.
**Why others wrong:** Spring doesn't guess; it fails fast with an error instead of arbitrarily picking one.
**Difficulty:** Hard | **Topic:** Constructor Injection

---

**Q5.** Which of these is TRUE about `@Repository`?
A) It's purely cosmetic with zero functional difference from @Component
B) It adds automatic exception translation to Spring's DataAccessException
C) It changes the bean scope to prototype
D) It's only used for REST controllers
**Answer:** B
**Explanation:** `@Repository` translates persistence-specific exceptions into Spring's unified `DataAccessException` hierarchy.
**Why others wrong:** A ignores exception translation; C and D describe unrelated behavior.
**Difficulty:** Medium | **Topic:** Stereotype Annotations

---

**Q6.** Which method executes right after a bean's dependencies are injected?
A) @PreDestroy method B) Constructor C) @PostConstruct method D) main() method
**Answer:** C
**Explanation:** `@PostConstruct` runs once the bean is fully constructed and wired, signaling "ready for use."
**Why others wrong:** `@PreDestroy` runs at shutdown; constructor runs before injection completes in field injection; `main()` is unrelated to bean lifecycle.
**Difficulty:** Medium | **Topic:** Bean Lifecycle

---

**Q7.** `ApplicationContext` differs from `BeanFactory` because it:
A) Loads beans lazily only B) Cannot use annotations C) Eagerly initializes singleton beans and supports AOP, events, i18n D) Is deprecated in Spring Boot
**Answer:** C
**Explanation:** ApplicationContext is a superset of BeanFactory offering eager loading and enterprise features.
**Why others wrong:** BeanFactory (not ApplicationContext) is the lazy, minimal one; ApplicationContext supports annotations fully; it is not deprecated.
**Difficulty:** Medium | **Topic:** Container

---

**Q8.** Which injection type is generally recommended for mandatory dependencies?
A) Field injection B) Constructor injection C) Setter injection D) Static injection
**Answer:** B
**Explanation:** Constructor injection guarantees the object is fully initialized and immutable, and is easiest to unit test.
**Why others wrong:** Field injection hides dependencies and is hard to test; setter injection suits optional dependencies; "static injection" isn't a Spring concept.
**Difficulty:** Easy | **Topic:** DI Types

---

**Q9.** What does `@Primary` do?
A) Makes a bean immutable B) Marks a bean as the default choice among multiple candidates C) Forces prototype scope D) Deletes other beans of the same type
**Answer:** B
**Explanation:** When multiple beans qualify for injection, `@Primary` marks the default one to use unless overridden by `@Qualifier`.
**Why others wrong:** It has nothing to do with immutability, scope, or deletion.
**Difficulty:** Medium | **Topic:** Autowiring

---

**Q10.** In a conflict between `@Primary` on a bean and `@Qualifier` at the injection point, which wins?
A) @Primary always wins B) @Qualifier wins C) Spring throws an error D) Random selection
**Answer:** B
**Explanation:** `@Qualifier` is more specific (declared at the injection site) so it overrides the general `@Primary` default.
**Why others wrong:** A is reversed; C and D don't reflect actual Spring behavior.
**Difficulty:** Hard | **Topic:** Autowiring Precedence

---

# CHAPTER 19: SPRING BOOT REST APIs, VALIDATION, EXCEPTION HANDLING

## 19.1 Why Spring Boot (vs Plain Spring)?

Plain Spring required tons of XML/manual configuration. **Spring Boot** = Spring + auto-configuration + embedded server (Tomcat) + starter dependencies, so you can run a production-ready app with minimal setup.

```java
@SpringBootApplication // combines @Configuration + @EnableAutoConfiguration + @ComponentScan
public class GymProApplication {
    public static void main(String[] args) {
        SpringApplication.run(GymProApplication.class, args);
    }
}
```

⚠️ **Trap:** `@SpringBootApplication` is a **meta-annotation** — it bundles three annotations into one. A common MCQ asks "which three annotations does @SpringBootApplication include?"

| Included Annotation | Purpose |
|---|---|
| `@Configuration` | Marks class as bean definition source |
| `@EnableAutoConfiguration` | Auto-configures beans based on classpath |
| `@ComponentScan` | Scans package for `@Component`/`@Service`/etc. |

## 19.2 Starters

"Starters" = curated dependency bundles. E.g., `spring-boot-starter-web` pulls in Spring MVC, Jackson (JSON), embedded Tomcat, etc., in one line in `pom.xml`.

Common starters: `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `spring-boot-starter-security`, `spring-boot-starter-test`.

## 19.3 REST & HTTP Methods

REST = Representational State Transfer — an architectural style for APIs using HTTP methods to act on **resources**.

| HTTP Method | Purpose | Idempotent? | Body? |
|---|---|---|---|
| GET | Read data | ✅ Yes | No |
| POST | Create new resource | ❌ No | Yes |
| PUT | Update/replace ENTIRE resource | ✅ Yes | Yes |
| PATCH | Update PART of a resource | ❌ No (usually) | Yes |
| DELETE | Remove resource | ✅ Yes | No (usually) |

⚠️ **Capgemini Trap:** "Idempotent" means calling it multiple times has the same effect as calling it once. GET, PUT, DELETE are idempotent. **POST is NOT idempotent** (calling it twice creates two resources). This distinction is a favorite MCQ.

⚠️ **Trap 2:** PUT vs PATCH — PUT replaces the WHOLE resource (missing fields may get nulled out), PATCH updates only the given fields.

## 19.4 Building a Controller

```java
@RestController
@RequestMapping("/api/members")
public class MemberController {

    @Autowired
    private MemberService memberService;

    @GetMapping("/{id}")
    public ResponseEntity<Member> getMember(@PathVariable Long id) {
        Member m = memberService.findById(id);
        return ResponseEntity.ok(m);
    }

    @GetMapping
    public List<Member> getAll(@RequestParam(required = false) String city) {
        return memberService.findAll(city);
    }

    @PostMapping
    public ResponseEntity<Member> create(@Valid @RequestBody Member member) {
        Member saved = memberService.save(member);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }

    @PutMapping("/{id}")
    public Member update(@PathVariable Long id, @RequestBody Member member) {
        return memberService.update(id, member);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        memberService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### @PathVariable vs @RequestParam

| Feature | @PathVariable | @RequestParam |
|---|---|---|
| Comes from | URL path segment | Query string |
| Example | `/members/5` → `id=5` | `/members?city=Pune` → `city=Pune` |
| Typical use | Identifying a specific resource | Filtering/optional params |

⚠️ **Trap:** `@RequestParam` is **required by default** (`required = true`). If the parameter is missing and you didn't set `required = false`, Spring throws `MissingServletRequestParameterException`.

## 19.5 @RequestBody & ResponseEntity

- `@RequestBody` — converts incoming JSON into a Java object (deserialization via Jackson).
- `ResponseEntity<T>` — lets you control status code, headers, AND body of the response.

```java
return ResponseEntity.status(HttpStatus.CREATED).body(savedMember);
```

## 19.6 Validation

Use `@Valid` (or `@Validated`) + Bean Validation annotations on the DTO/Entity.

```java
public class MemberDto {
    @NotBlank(message = "Name is required")
    private String name;

    @Email(message = "Invalid email format")
    private String email;

    @Pattern(regexp = "^\\+91[6-9]\\d{9}$", message = "Invalid phone number")
    private String phone;

    @Min(18) @Max(100)
    private int age;

    @NotNull
    private LocalDate joiningDate;
}
```

| Annotation | Checks |
|---|---|
| `@NotNull` | Value must not be null (empty string OK) |
| `@NotBlank` | Not null AND not empty/whitespace (Strings only) |
| `@NotEmpty` | Not null AND size > 0 (Strings, Collections) |
| `@Size(min,max)` | Length/size range |
| `@Min` / `@Max` | Numeric range |
| `@Email` | Valid email format |
| `@Pattern` | Custom regex |

⚠️ **Capgemini Trap (very common):** Difference between `@NotNull`, `@NotEmpty`, `@NotBlank`:
- `@NotNull` → `""` (empty string) passes ✅
- `@NotEmpty` → `""` fails ❌, but `"  "` (spaces) passes ✅
- `@NotBlank` → `""` and `"  "` both fail ❌ (trims whitespace)

If `@Valid` fails, Spring throws `MethodArgumentNotValidException` — you must handle this in your exception handler, or the client gets a raw 400 with a messy default body.

## 19.7 Global Exception Handling

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(HttpStatus.NOT_FOUND.value(), ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors()
          .forEach(err -> errors.put(err.getField(), err.getDefaultMessage()));
        return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        ErrorResponse error = new ErrorResponse(500, "Something went wrong");
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

- `@RestControllerAdvice` = `@ControllerAdvice` + `@ResponseBody` — applies globally across all controllers.
- `@ExceptionHandler` — marks a method to handle a specific exception type.
- Custom exceptions typically extend `RuntimeException`.

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) { super(message); }
}
```

⚠️ **Trap:** If you define handlers for both `ResourceNotFoundException` and the generic `Exception`, Spring picks the **most specific matching handler** — not the order they're written in.

## 19.8 Common HTTP Status Codes MCQ Bait

| Code | Meaning |
|---|---|
| 200 OK | Success (GET/PUT/PATCH) |
| 201 Created | Successful POST (new resource created) |
| 204 No Content | Successful DELETE (no body returned) |
| 400 Bad Request | Validation failure/malformed request |
| 401 Unauthorized | Not authenticated |
| 403 Forbidden | Authenticated but not authorized |
| 404 Not Found | Resource doesn't exist |
| 409 Conflict | Duplicate/conflicting state |
| 500 Internal Server Error | Unhandled server-side exception |

---

## 🔑 CHAPTER 19 — ONE-PAGE REVISION

- `@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`.
- HTTP methods: GET/PUT/DELETE idempotent; **POST is NOT idempotent**; PUT replaces whole resource, PATCH partial.
- `@PathVariable` → from URL path; `@RequestParam` → from query string, **required by default**.
- `@RequestBody` deserializes JSON → object; `ResponseEntity` controls status+body+headers.
- Validation trio: `@NotNull` (allows `""`) < `@NotEmpty` (blocks `""`, allows `"  "`) < `@NotBlank` (blocks both).
- `@Valid` failure → `MethodArgumentNotValidException`.
- `@RestControllerAdvice` + `@ExceptionHandler` = centralized global exception handling.
- Most specific exception handler wins over generic `Exception` handler.
- 201=Created, 204=No Content, 400=Bad Request, 401=Unauthorized, 403=Forbidden, 404=Not Found.

---

## CHAPTER 19 — MCQs

**Q1.** `@SpringBootApplication` is a combination of which three annotations?
A) @Component, @Service, @Repository
B) @Configuration, @EnableAutoConfiguration, @ComponentScan
C) @RestController, @RequestMapping, @Autowired
D) @Bean, @Scope, @Lazy
**Answer:** B
**Explanation:** These three together enable configuration, auto-config based on classpath, and component scanning.
**Why others wrong:** A/C/D are unrelated annotation groups not bundled by this meta-annotation.
**Difficulty:** Easy | **Topic:** Spring Boot Basics

---

**Q2.** Which HTTP method is NOT idempotent?
A) GET B) PUT C) POST D) DELETE
**Answer:** C
**Explanation:** Calling POST multiple times typically creates multiple new resources, so results differ each time.
**Why others wrong:** GET, PUT, DELETE produce the same end-state no matter how many times they're called.
**Difficulty:** Medium | **Topic:** REST/HTTP Methods

---

**Q3.** What's the key difference between PUT and PATCH?
A) PUT is for reading, PATCH is for deleting
B) PUT replaces the entire resource, PATCH updates only specified fields
C) They are identical in every way
D) PATCH requires no request body
**Answer:** B
**Explanation:** PUT is a full replacement; PATCH is a partial update.
**Why others wrong:** Neither is for reading/deleting; they are not identical; PATCH does require a body describing what changes.
**Difficulty:** Medium | **Topic:** REST/HTTP Methods

---

**Q4.** By default, is `@RequestParam` required or optional?
A) Optional B) Required C) Depends on JVM version D) Always ignored if missing
**Answer:** B
**Explanation:** `@RequestParam` is required by default; missing it throws `MissingServletRequestParameterException`.
**Why others wrong:** Making it optional needs explicit `required = false`; JVM version is irrelevant; missing required params are never silently ignored.
**Difficulty:** Medium | **Topic:** Request Parameters

---

**Q5.** Given `@NotEmpty private String name;`, which value(s) will PASS validation?
A) null B) "" C) "   " (only spaces) D) Both null and ""
**Answer:** C
**Explanation:** `@NotEmpty` blocks null and `""`, but a string of only spaces has length > 0, so it technically passes.
**Why others wrong:** null and "" both fail the `@NotEmpty` check.
**Difficulty:** Hard | **Topic:** Validation

---

**Q6.** Which exception is thrown when `@Valid` validation fails on a `@RequestBody`?
A) IllegalArgumentException B) MethodArgumentNotValidException C) ValidationFailedException D) ConstraintViolationException
**Answer:** B
**Explanation:** Spring MVC throws `MethodArgumentNotValidException` specifically for `@Valid`-annotated `@RequestBody`/`@ModelAttribute` failures.
**Why others wrong:** `ConstraintViolationException` is more typical of method-level/bean validation outside MVC binding; the others aren't Spring's standard exception for this case.
**Difficulty:** Hard | **Topic:** Validation

---

**Q7.** What does `@RestControllerAdvice` combine?
A) @Service + @Repository B) @Controller + @ResponseBody applied globally as @ControllerAdvice + @ResponseBody C) @Configuration + @Bean D) @Component + @Scope
**Answer:** B
**Explanation:** It's `@ControllerAdvice` (global exception handling across controllers) plus `@ResponseBody` (returns data directly, not views).
**Why others wrong:** These combinations don't relate to global exception handling.
**Difficulty:** Medium | **Topic:** Exception Handling

---

**Q8.** What status code should a successful POST that creates a resource return?
A) 200 B) 201 C) 204 D) 400
**Answer:** B
**Explanation:** 201 Created signals a new resource was successfully created.
**Why others wrong:** 200 is generic success (GET/PUT); 204 means no content (DELETE); 400 is client error.
**Difficulty:** Easy | **Topic:** HTTP Status Codes

---

**Q9.** If both a specific exception handler (`ResourceNotFoundException`) and a generic `Exception` handler exist in the same `@RestControllerAdvice`, and a `ResourceNotFoundException` is thrown, which handler runs?
A) The generic Exception handler always B) The specific handler (most specific match wins) C) Both run in sequence D) Neither runs, app crashes
**Answer:** B
**Explanation:** Spring resolves to the most specific applicable exception handler.
**Why others wrong:** Generic handler is a fallback only when no more specific handler matches; only one handler executes.
**Difficulty:** Hard | **Topic:** Exception Handling

---

**Q10.** Which annotation converts incoming JSON into a Java object parameter?
A) @ResponseBody B) @RequestBody C) @PathVariable D) @RequestParam
**Answer:** B
**Explanation:** `@RequestBody` deserializes the HTTP request JSON body into the annotated Java object.
**Why others wrong:** `@ResponseBody` serializes outgoing Java objects to JSON (opposite direction); `@PathVariable`/`@RequestParam` extract values from URL, not the body.
**Difficulty:** Easy | **Topic:** REST Annotations

---

# CHAPTER 20: SPRING DATA JPA, RELATIONSHIPS

## 20.1 What is JPA vs Hibernate vs Spring Data JPA?

| Term | What it is |
|---|---|
| **JPA** | A *specification* (interface/rules) for ORM in Java — doesn't do anything by itself |
| **Hibernate** | The most popular *implementation* of JPA (actually does the work) |
| **Spring Data JPA** | A Spring abstraction layer on top of JPA that auto-generates repository code (less boilerplate) |

⚠️ **Trap:** JPA is NOT a library/tool — it's a **specification**. Hibernate is one implementation among others (EclipseLink is another).

## 20.2 Entity Basics

```java
@Entity
@Table(name = "members")
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "full_name", nullable = false, length = 100)
    private String name;

    private String email;
}
```

- `@Entity` — marks class as a JPA-managed table-mapped class.
- `@Id` — primary key.
- `@GeneratedValue` — auto-generates the ID. Common strategies:
  - `IDENTITY` — DB auto-increment handles it (common with MySQL).
  - `SEQUENCE` — uses a DB sequence object (common with PostgreSQL/Oracle).
  - `AUTO` — JPA provider picks the strategy.
- `@Column` — customizes column mapping (name, nullable, length, unique).

## 20.3 Repository Hierarchy

```
Repository (marker interface)
   └── CrudRepository<T, ID>       — basic CRUD: save, findById, findAll, delete, count
         └── PagingAndSortingRepository<T, ID>  — adds paging & sorting
               └── JpaRepository<T, ID>          — adds JPA-specific extras: batch ops, flush, etc.
```

| Interface | Provides |
|---|---|
| `CrudRepository` | save(), findById(), findAll(), deleteById(), count() |
| `PagingAndSortingRepository` | + findAll(Pageable), findAll(Sort) |
| `JpaRepository` | + saveAll(), flush(), deleteInBatch(), getOne()/getReferenceById() |

⚠️ **Trap:** `JpaRepository` extends `PagingAndSortingRepository` which extends `CrudRepository`. So `JpaRepository` is a superset — has EVERYTHING the others have, plus more. Most real projects use `JpaRepository` directly.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    // Query derivation - Spring auto-generates the query from the method name!
    List<Member> findByCity(String city);
    List<Member> findByAgeGreaterThan(int age);
    Optional<Member> findByEmail(String email);
    boolean existsByEmail(String email);
    List<Member> findByNameContainingIgnoreCase(String name);
}
```

## 20.4 Query Methods — Keyword Reference

| Keyword | Example | Meaning |
|---|---|---|
| `findBy` | `findByName` | WHERE name = ? |
| `And` / `Or` | `findByNameAndCity` | WHERE name=? AND city=? |
| `GreaterThan` / `LessThan` | `findByAgeGreaterThan` | WHERE age > ? |
| `Between` | `findByAgeBetween` | WHERE age BETWEEN ? AND ? |
| `Like` / `Containing` | `findByNameContaining` | WHERE name LIKE %?% |
| `OrderBy` | `findByCityOrderByNameAsc` | Adds ORDER BY |
| `IgnoreCase` | `findByNameIgnoreCase` | Case-insensitive match |
| `Top`/`First` | `findTop5ByOrderByAgeDesc` | LIMIT with ordering |

## 20.5 JPQL vs Native Query

```java
// JPQL — operates on ENTITY names/fields, not table/column names
@Query("SELECT m FROM Member m WHERE m.city = :city")
List<Member> customFindByCity(@Param("city") String city);

// Native SQL — actual table/column names
@Query(value = "SELECT * FROM members WHERE city = :city", nativeQuery = true)
List<Member> nativeFindByCity(@Param("city") String city);
```

⚠️ **Trap:** JPQL queries use **entity class name and field name** (`Member`, `m.city`), NOT the table/column name (`members`, `city` column) — this reversal is a favorite trick question.

## 20.6 Pagination & Sorting

```java
Pageable pageable = PageRequest.of(0, 10, Sort.by("name").ascending());
Page<Member> page = memberRepository.findAll(pageable);
```
- `Page<T>` — includes total pages, total elements, etc.
- `Slice<T>` — lighter, doesn't compute total count (only knows if there's a next page).

## 20.7 Relationships (THE Big MCQ Topic)

| Relationship | Example | Owning Side |
|---|---|---|
| `@OneToOne` | User ↔ Profile | Side with the foreign key |
| `@OneToMany` | Trainer → many Sessions | Usually mapped with `mappedBy` on the "one" side |
| `@ManyToOne` | Session → one Trainer | This side **owns** the FK column |
| `@ManyToMany` | Student ↔ Course | Needs a join table |

```java
// ManyToOne (owning side — has the foreign key)
@Entity
class Session {
    @ManyToOne
    @JoinColumn(name = "trainer_id")
    private Trainer trainer;
}

// OneToMany (inverse side — mappedBy points to field name in Session)
@Entity
class Trainer {
    @OneToMany(mappedBy = "trainer", cascade = CascadeType.ALL)
    private List<Session> sessions;
}
```

```java
// ManyToMany
@Entity
class Student {
    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private List<Course> courses;
}
```

⚠️ **Capgemini Trap #1:** The side WITHOUT `mappedBy` is the **owning side** (it has the actual foreign key column and controls the relationship in the DB). The side WITH `mappedBy` is the **inverse/mapped side** (read-only reflection of the relationship).

⚠️ **Trap #2:** `@ManyToOne` is almost always the owning side because "many" side holds the FK naturally (e.g., many Sessions each store one `trainer_id`).

⚠️ **Trap #3 — Fetch types:**

| Relationship | Default Fetch Type |
|---|---|
| `@OneToOne` | EAGER |
| `@ManyToOne` | EAGER |
| `@OneToMany` | LAZY |
| `@ManyToMany` | LAZY |

Common MCQ: "Which relationship is EAGER by default?" → OneToOne & ManyToOne (the "to-one" side). Collections default to LAZY (to avoid loading huge lists unnecessarily).

## 20.8 Cascade Types

| CascadeType | Effect |
|---|---|
| `PERSIST` | Save parent → children also saved |
| `MERGE` | Update parent → children also updated |
| `REMOVE` | Delete parent → children also deleted |
| `ALL` | All of the above + REFRESH + DETACH |

---

## 🔑 CHAPTER 20 — ONE-PAGE REVISION

- JPA = specification; Hibernate = implementation; Spring Data JPA = abstraction reducing boilerplate.
- `JpaRepository` > `PagingAndSortingRepository` > `CrudRepository` (superset relationship).
- Query derivation: method names like `findByCityAndAgeGreaterThan` auto-generate SQL.
- JPQL uses entity/field names; native query uses real table/column names.
- `@ManyToOne` = owning side (has FK), typically EAGER by default.
- `@OneToMany` = inverse side (`mappedBy`), typically LAZY by default.
- `@ManyToMany` needs a join table via `@JoinTable`.
- Cascade ALL = PERSIST + MERGE + REMOVE + REFRESH + DETACH.
- `Page<T>` computes total count; `Slice<T>` doesn't (lighter).

---

## CHAPTER 20 — MCQs

**Q1.** What is JPA?
A) A Java ORM library like Hibernate B) A specification/standard for ORM in Java C) A database engine D) A Spring Boot starter only
**Answer:** B
**Explanation:** JPA defines rules/interfaces; Hibernate (or others) implement them.
**Why others wrong:** JPA itself does no persistence work; it's not a DB engine; it's used across Spring Boot but is not exclusive to it.
**Difficulty:** Easy | **Topic:** JPA Basics

---

**Q2.** Which repository interface provides the MOST features (superset of the others)?
A) CrudRepository B) PagingAndSortingRepository C) JpaRepository D) Repository
**Answer:** C
**Explanation:** JpaRepository extends PagingAndSortingRepository which extends CrudRepository, inheriting all their methods plus JPA-specific extras.
**Why others wrong:** They provide progressively fewer features; `Repository` is just the empty marker interface.
**Difficulty:** Medium | **Topic:** Repository Hierarchy

---

**Q3.** What does the method name `findByAgeGreaterThanOrderByNameAsc` generate?
A) A syntax error B) WHERE age > ? ORDER BY name ASC C) WHERE age = ? ORDER BY name DESC D) DELETE query
**Answer:** B
**Explanation:** Spring Data parses method names into query conditions and ordering clauses automatically.
**Why others wrong:** It's valid, matches ">" (not "="), and orders ascending (Asc, not Desc); it's a SELECT, not DELETE.
**Difficulty:** Medium | **Topic:** Query Derivation

---

**Q4.** In JPQL, `SELECT m FROM Member m WHERE m.city = :city` refers to `Member` as:
A) The table name in the database B) The entity class name C) A column name D) A native SQL keyword
**Answer:** B
**Explanation:** JPQL operates on entity classes and their fields, not raw table/column names.
**Why others wrong:** The actual table might be named differently (e.g., `members`); `m.city` is a field, not a raw column reference; JPQL isn't native SQL.
**Difficulty:** Hard | **Topic:** JPQL

---

**Q5.** In a `@OneToMany`/`@ManyToOne` relationship between Trainer and Session, which side is the OWNING side?
A) Trainer (OneToMany side) B) Session (ManyToOne side) C) Both equally D) Neither — owning side doesn't apply here
**Answer:** B
**Explanation:** The `@ManyToOne` side holds the actual foreign key column, making it the owning side.
**Why others wrong:** Trainer's side uses `mappedBy`, making it the inverse/non-owning side; owning side concept always applies to bidirectional relations.
**Difficulty:** Hard | **Topic:** Relationships

---

**Q6.** What is the default fetch type for `@ManyToOne`?
A) LAZY B) EAGER C) Depends entirely on database D) There is no default
**Answer:** B
**Explanation:** `@ManyToOne` (and `@OneToOne`) default to EAGER loading.
**Why others wrong:** LAZY is default for collection-based relations (`@OneToMany`/`@ManyToMany`), not this one; fetch type is defined by JPA spec, not the DB; a default does exist.
**Difficulty:** Hard | **Topic:** Fetch Types

---

**Q7.** Which cascade type ensures deleting a parent also deletes its children?
A) CascadeType.MERGE B) CascadeType.PERSIST C) CascadeType.REMOVE D) CascadeType.REFRESH
**Answer:** C
**Explanation:** REMOVE propagates delete operations from parent to associated children.
**Why others wrong:** MERGE handles updates, PERSIST handles saves, REFRESH re-reads state from DB — none handle deletion propagation.
**Difficulty:** Medium | **Topic:** Cascade

---

**Q8.** What's the key difference between `Page<T>` and `Slice<T>`?
A) Page is for XML, Slice is for JSON B) Page computes total element/page counts, Slice does not (lighter-weight) C) Slice supports sorting, Page does not D) They're identical
**Answer:** B
**Explanation:** Page runs an extra COUNT query for totals; Slice skips this, only knowing if a next page exists.
**Why others wrong:** No such format restriction exists; both support sorting; they are not identical in behavior.
**Difficulty:** Hard | **Topic:** Pagination

---

**Q9.** Which annotation defines the join table for a `@ManyToMany` relationship?
A) @JoinColumn B) @JoinTable C) @Table D) @ManyToMany(table=...)
**Answer:** B
**Explanation:** `@JoinTable` specifies the intermediary table and its join columns for a many-to-many mapping.
**Why others wrong:** `@JoinColumn` is for single FK columns (one-to-one/many-to-one); `@Table` just names an entity's table; there's no such parameter on `@ManyToMany`.
**Difficulty:** Medium | **Topic:** Relationships

---

**Q10.** `@GeneratedValue(strategy = GenerationType.IDENTITY)` means:
A) JPA generates the ID in Java before insert B) The database auto-increments the ID column C) A UUID is generated D) The ID must be manually set
**Answer:** B
**Explanation:** IDENTITY relies on the database's own auto-increment feature (common in MySQL).
**Why others wrong:** It's the DB, not JPA, generating it; UUID generation is a different strategy; manual setting contradicts @GeneratedValue's purpose.
**Difficulty:** Medium | **Topic:** Entity Mapping

---

# CHAPTER 21: SPRING SECURITY/JWT BASICS, MICROSERVICES BASICS

## 21.1 Spring Security Core Concepts

| Term | Meaning |
|---|---|
| **Authentication** | "Who are you?" — verifying identity (username/password, token, etc.) |
| **Authorization** | "What are you allowed to do?" — checking permissions/roles |
| **Principal** | The currently authenticated user |
| **SecurityContext** | Holds the Authentication object for the current thread |
| **Filter Chain** | Series of filters HTTP requests pass through before reaching controller |

⚠️ **Trap:** Authentication happens FIRST, then Authorization. A common wrong answer swaps these.

## 21.2 Basic Security Config (Modern Style)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

⚠️ **Trap:** Passwords should NEVER be stored in plain text. `BCryptPasswordEncoder` is the standard — it's a one-way hash (can't be decrypted, only compared).

## 21.3 JWT (JSON Web Token) Basics

JWT = a compact, self-contained token used for stateless authentication.

**Structure:** `HEADER.PAYLOAD.SIGNATURE` (three Base64 parts separated by dots)

| Part | Contains |
|---|---|
| Header | Algorithm (e.g., HS256) + token type |
| Payload | Claims — user info (username, roles, expiry) |
| Signature | Ensures the token wasn't tampered with |

**Flow:**
1. User logs in with username/password.
2. Server validates credentials → generates JWT → sends to client.
3. Client stores token (localStorage/cookie) and sends it in `Authorization: Bearer <token>` header on every request.
4. Server validates the token's signature + expiry on each request — **no session stored server-side** (stateless!).

⚠️ **Trap:** JWT is **stateless** — the server does NOT store session data. This is why `SessionCreationPolicy.STATELESS` is set in config. Contrast with traditional session-cookie auth which IS stateful.

⚠️ **Trap 2:** JWT payload is **Base64-encoded, NOT encrypted** — anyone can decode and read it (don't put passwords/secrets in it!). Only the signature prevents tampering, not reading.

## 21.4 Method-Level Security

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) { ... }
```
`@PreAuthorize` checks authorization BEFORE the method executes.

## 21.5 Microservices Basics

**Microservices** = breaking a large application into small, independently deployable services, each owning its own data and communicating over the network (usually REST/HTTP or messaging).

| Monolith | Microservices |
|---|---|
| Single deployable unit | Multiple independent services |
| Single DB usually | Each service can have its own DB |
| Simple to start | Complex but scalable/flexible |
| Scaling = scale whole app | Scale individual services independently |

## 21.6 Key Microservices Components

| Component | Purpose |
|---|---|
| **Eureka (Service Registry)** | Services register themselves; other services discover them by name instead of hardcoded IP/port |
| **API Gateway** | Single entry point for all client requests; routes to appropriate microservice, can handle auth/rate-limiting |
| **Config Server** | Centralized externalized configuration for all microservices |
| **Feign Client** | Declarative REST client — lets one service call another as if calling a local method |
| **Load Balancer** | Distributes requests across multiple instances of a service |

```java
@FeignClient(name = "trainer-service")
public interface TrainerClient {
    @GetMapping("/api/trainers/{id}")
    TrainerDto getTrainer(@PathVariable Long id);
}
```

⚠️ **Trap:** Eureka enables **service discovery** — services find each other **by name**, not hardcoded IP:port. This decouples services from network topology (crucial when instances scale up/down or move).

⚠️ **Trap 2:** API Gateway sits in FRONT of all microservices (single entry point) — clients never call individual services directly in a well-designed system.

## 21.7 Quick Comparison: Session vs JWT Auth

| Feature | Session-based | JWT-based |
|---|---|---|
| State | Stateful (server stores session) | Stateless (server stores nothing) |
| Scalability | Harder (needs sticky sessions/shared store) | Easier (any server can validate) |
| Storage | Server-side session store | Client-side (token) |
| Use case | Traditional web apps | REST APIs, microservices, mobile |

---

## 🔑 CHAPTER 21 — ONE-PAGE REVISION

- Authentication ("who are you") happens BEFORE Authorization ("what can you do").
- Passwords → hash with `BCryptPasswordEncoder`, never store plain text.
- JWT structure: Header.Payload.Signature — payload is Base64-encoded (readable), NOT encrypted.
- JWT auth = stateless → `SessionCreationPolicy.STATELESS`.
- `@PreAuthorize` = method-level authorization check before execution.
- Eureka = service registry/discovery (find services by name, not IP).
- API Gateway = single entry point, routes requests to microservices.
- Feign Client = declarative REST client for inter-service calls.
- Config Server = centralized config management across services.

---

## CHAPTER 21 — MCQs

**Q1.** What is the correct order of security checks?
A) Authorization then Authentication B) Authentication then Authorization C) They happen simultaneously always D) Neither is required for REST APIs
**Answer:** B
**Explanation:** The system must first verify identity (authentication) before deciding what that identity can access (authorization).
**Why others wrong:** Reversing the order makes no sense (can't authorize an unknown identity); they are distinct sequential steps; REST APIs still require both for protected resources.
**Difficulty:** Easy | **Topic:** Security Basics

---

**Q2.** Is JWT payload encrypted?
A) Yes, fully encrypted B) No, it's only Base64-encoded and readable by anyone C) It's encrypted only in production D) Only the header is encrypted
**Answer:** B
**Explanation:** The payload is just Base64-encoded; the signature (not encryption) prevents tampering.
**Why others wrong:** Encryption is not applied by default to standard JWTs; environment doesn't change this; the header is likewise just encoded, not encrypted.
**Difficulty:** Hard | **Topic:** JWT

---

**Q3.** JWT-based authentication is considered:
A) Stateful B) Stateless C) Session-dependent D) Cookie-mandatory
**Answer:** B
**Explanation:** No session data is stored on the server; each request is validated independently using the token.
**Why others wrong:** Stateful/session-dependent describe traditional session-cookie auth, the opposite approach; cookies aren't mandatory for JWT (often sent via headers).
**Difficulty:** Medium | **Topic:** JWT

---

**Q4.** What is the purpose of Eureka in a microservices architecture?
A) API documentation B) Service registry/discovery — services find each other by name C) Database migration D) Load testing
**Answer:** B
**Explanation:** Eureka lets services register and discover each other dynamically by name instead of hardcoded network addresses.
**Why others wrong:** It's unrelated to documentation, DB migrations, or load testing tools.
**Difficulty:** Easy | **Topic:** Microservices

---

**Q5.** What is the role of an API Gateway?
A) Stores application data B) Single entry point that routes requests to appropriate microservices C) Compiles Java code D) Replaces the database
**Answer:** B
**Explanation:** It centralizes routing, and often handles cross-cutting concerns like auth and rate-limiting, for all incoming client requests.
**Why others wrong:** It doesn't store data, compile code, or act as a database.
**Difficulty:** Easy | **Topic:** Microservices

---

**Q6.** Which annotation declares a Feign client interface?
A) @RestController B) @FeignClient C) @Service D) @EnableFeign
**Answer:** B
**Explanation:** `@FeignClient(name="service-name")` marks an interface as a declarative REST client for inter-service calls.
**Why others wrong:** These are unrelated annotations for controllers/services; `@EnableFeign` isn't the correct annotation name (it's `@EnableFeignClients` at the app level, different purpose).
**Difficulty:** Medium | **Topic:** Feign

---

**Q7.** Which class is typically used to hash passwords securely in Spring Security?
A) MD5Encoder B) BCryptPasswordEncoder C) Base64Encoder D) PlainTextEncoder
**Answer:** B
**Explanation:** BCrypt is a strong, adaptive one-way hashing algorithm recommended for password storage.
**Why others wrong:** MD5 is considered weak/broken for passwords; Base64 is encoding (reversible), not hashing; "PlainTextEncoder" defeats the purpose of security entirely.
**Difficulty:** Medium | **Topic:** Password Security

---

**Q8.** In microservices, each service typically:
A) Must share one common database with all other services B) Can own its own independent database C) Cannot have its own database D) Must use only in-memory storage
**Answer:** B
**Explanation:** A core microservices principle is decentralized data management — each service can own its data store.
**Why others wrong:** Sharing one DB reintroduces monolithic coupling; services CAN have databases; storage type isn't restricted to in-memory.
**Difficulty:** Medium | **Topic:** Microservices Principles

---

**Q9.** What does `@PreAuthorize("hasRole('ADMIN')")` do?
A) Runs after the method executes B) Checks role-based authorization before the method executes C) Only works on controllers D) Encrypts the return value
**Answer:** B
**Explanation:** It's a method-security annotation that blocks execution unless the authenticated user has the required role.
**Why others wrong:** It checks BEFORE, not after; it can be applied to any Spring-managed bean method, not just controllers; it doesn't encrypt data.
**Difficulty:** Medium | **Topic:** Method Security

---

**Q10.** Compared to session-based auth, JWT-based auth is generally considered:
A) Harder to scale horizontally B) Easier to scale since no server-side session state is needed C) Impossible to use in microservices D) Identical in scalability
**Answer:** B
**Explanation:** Since no session store is required, any server instance can validate a JWT independently, aiding horizontal scaling.
**Why others wrong:** Session-based auth is the one that's harder to scale (needs sticky sessions/shared stores); JWT is actually a natural fit for microservices; the two approaches differ meaningfully in scalability.
**Difficulty:** Medium | **Topic:** Auth Comparison

---

# MODULE 3 — MASTER REVISION SHEET (All 4 Chapters)

- **IoC/DI:** container creates & wires objects; constructor injection preferred; singleton = default scope.
- **Stereotypes:** `@Component`/`@Service`/`@Repository`/`@Controller` functionally similar; `@Repository` adds exception translation.
- **Ambiguity resolution:** `@Qualifier` (specific, wins) > `@Primary` (default fallback).
- **Spring Boot:** `@SpringBootApplication` = Configuration + AutoConfig + ComponentScan.
- **REST:** POST is not idempotent; PUT = full replace, PATCH = partial; `@PathVariable` (URL) vs `@RequestParam` (query, required by default).
- **Validation:** NotNull < NotEmpty < NotBlank (increasing strictness); failure → `MethodArgumentNotValidException`.
- **Exception handling:** `@RestControllerAdvice` + `@ExceptionHandler`; most specific handler wins.
- **JPA:** spec, not a tool; Hibernate implements it; `JpaRepository` ⊃ `PagingAndSortingRepository` ⊃ `CrudRepository`.
- **JPQL** uses entity/field names, not table/column names.
- **Relationships:** `@ManyToOne` = owning side + EAGER default; `@OneToMany` = inverse side (`mappedBy`) + LAZY default.
- **Security:** Authentication before Authorization; BCrypt for passwords.
- **JWT:** stateless, payload readable (not encrypted), only signature prevents tampering.
- **Microservices:** Eureka = discovery, Gateway = single entry point, Feign = inter-service REST calls, Config Server = centralized config.

---

*Next up: Chapter 22+ (MySQL Module) whenever you're ready — say "continue" or jump straight to a specific chapter number.*
