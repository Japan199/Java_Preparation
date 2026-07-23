# Spring Core, Dependency Injection, AOP, and MVC

## Spring Core

### 1. What is the Spring Framework?

**Interview-ready answer:**

Spring is a modular Java framework that provides dependency injection, web development, data access, transactions, security integration, testing support, and other infrastructure. Its main benefit is that application code depends on clear abstractions while the container manages wiring and lifecycle.

### 2. What are IoC and dependency injection?

**Interview-ready answer:**

Inversion of Control means object creation and wiring are controlled by a container rather than scattered through application code. Dependency injection is the technique Spring uses to supply an object’s collaborators. This reduces coupling and makes components easier to test.

### 3. Constructor, setter, or field injection?

**Interview-ready answer:**

I prefer constructor injection because required dependencies are explicit, objects can be immutable, and unit tests can instantiate them directly. Setter injection is appropriate for genuinely optional dependencies. I avoid field injection because it hides dependencies and makes testing and immutability harder.

### 4. `BeanFactory` versus `ApplicationContext`

**Interview-ready answer:**

`BeanFactory` is the basic IoC container. `ApplicationContext` builds on it with events, internationalization, environment support, resource loading, and automatic post-processor registration. Spring applications normally use an `ApplicationContext`.

### 5. What is a Spring bean?

**Interview-ready answer:**

A Spring bean is an object created, configured, and managed by the Spring container. It can be declared through component scanning, a `@Bean` method, or framework configuration.

### 6. `@Component`, `@Service`, `@Repository`, and `@Controller`

**Interview-ready answer:**

All are stereotype annotations and make classes eligible for component scanning. `@Service` expresses business logic, `@Repository` marks persistence components and enables exception translation, and `@Controller` is for MVC web components. The specialized annotation communicates architectural intent.

### 7. `@Bean` versus `@Component`

**Interview-ready answer:**

`@Component` is placed on a class discovered by scanning. `@Bean` is placed on a configuration method and is useful when I need explicit construction, customization, or cannot annotate a third-party class.

### 8. What are bean scopes?

**Interview-ready answer:**

Common scopes are singleton, prototype, request, session, and application. Singleton is the default and means one bean instance per application context, not one per JVM globally. Singleton services should normally be stateless or safely thread-safe.

### 9. Explain the bean lifecycle.

**Interview-ready answer:**

Spring instantiates the bean, injects dependencies, invokes aware callbacks and bean post-processors, then initialization callbacks such as `@PostConstruct`. On context shutdown it invokes destruction callbacks such as `@PreDestroy` for applicable scopes. Bean post-processors can wrap a bean in a proxy.

### 10. How does `@Autowired` resolve dependencies?

**Interview-ready answer:**

Spring first resolves primarily by type. If multiple candidates exist, I use `@Primary`, `@Qualifier`, or explicit configuration. A single constructor does not need `@Autowired` in modern Spring.

### 11. What is a circular dependency?

**Interview-ready answer:**

A circular dependency exists when bean A needs B and B needs A. Constructor cycles fail because neither object can be completed. Rather than using lazy injection as a default fix, I usually refactor responsibilities or introduce a third abstraction because the cycle often signals excessive coupling.

### 12. `@Configuration` and `@Bean` method behavior

**Interview-ready answer:**

`@Configuration` declares configuration classes. In full proxy mode, calls between `@Bean` methods can be intercepted so the container returns the managed singleton instead of constructing another object. I generally express dependencies as method parameters, which is clearer and also works well with lighter configuration modes.

## AOP

### 13. What is AOP?

**Interview-ready answer:**

Aspect-Oriented Programming separates cross-cutting concerns such as transactions, logging, authorization, and metrics from business logic. Spring AOP usually applies advice through runtime proxies at matched method join points.

### 14. Define aspect, advice, pointcut, and join point.

**Interview-ready answer:**

An aspect groups a cross-cutting concern. A join point is a point in execution, which in Spring AOP is usually a method invocation. A pointcut selects join points, and advice is the code executed before, after, around, or on exception.

### 15. JDK dynamic proxy versus CGLIB proxy

**Interview-ready answer:**

JDK dynamic proxies implement interfaces. Class-based proxies subclass the target class, commonly using CGLIB-style behavior. Final classes or methods cannot be advised through subclass overriding. I program to interfaces where it makes architectural sense but do not create meaningless interfaces only for proxying.

### 16. Why does self-invocation bypass `@Transactional` or `@Async`?

**Interview-ready answer:**

Spring commonly applies these features through a proxy. A call from one method to another on `this` does not pass through the proxy, so the advice is skipped. I solve it by moving the boundary to another bean or redesigning the method interaction rather than relying on self-proxy tricks.

### 17. What are AOP limitations?

**Interview-ready answer:**

Proxy-based AOP mainly intercepts eligible method calls that pass through the proxy. Self-invocation, final methods for class proxies, private methods, and objects not created by Spring can bypass advice. I keep cross-cutting boundaries at public service methods and test the actual configuration.

## Spring MVC

### 18. Explain a request flow in Spring MVC.

**Interview-ready answer:**

The request reaches `DispatcherServlet`, which finds a matching controller through handler mappings. A handler adapter invokes the controller, argument resolvers bind inputs, validation may run, and return-value handlers create a view or response body. Exceptions can be resolved through controller advice.

### 19. `@Controller` versus `@RestController`

**Interview-ready answer:**

`@Controller` normally returns view names unless methods use `@ResponseBody`. `@RestController` combines `@Controller` and `@ResponseBody`, so return values are serialized directly into the HTTP response.

### 20. `@RequestParam`, `@PathVariable`, and `@RequestBody`

**Interview-ready answer:**

`@PathVariable` identifies a resource within the URI, such as `/orders/{id}`. `@RequestParam` represents query options such as filtering or pagination. `@RequestBody` deserializes the HTTP body, normally JSON, into an object.

### 21. Filters versus interceptors versus aspects

**Interview-ready answer:**

Servlet filters work at the HTTP container level and are good for request wrapping, CORS, and security chains. MVC interceptors run around controller handling and can access handler metadata. Aspects apply to Spring method execution and are useful for service-level concerns. I choose the layer matching the concern.

### 22. How does validation work?

**Interview-ready answer:**

I put Bean Validation constraints such as `@NotBlank` or `@Size` on request DTOs and use `@Valid` or `@Validated` at the boundary. I return a consistent validation-error structure through global exception handling. Business rules that require database state belong in the service or domain layer.

### 23. How do you handle exceptions globally?

**Interview-ready answer:**

I use `@RestControllerAdvice` with focused `@ExceptionHandler` methods. I map domain exceptions to correct status codes and return a stable error contract containing code, message, timestamp, path, and correlation ID, without exposing stack traces or internal details.

### 24. What is content negotiation?

**Interview-ready answer:**

Content negotiation selects a representation based on request headers such as `Accept` and endpoint capabilities. Spring uses message converters to serialize or deserialize formats, most commonly JSON through Jackson.

### 25. How do you implement pagination?

**Interview-ready answer:**

I accept page, size, and validated sort fields, apply a maximum size, and return items plus pagination metadata. For large or frequently changing datasets, keyset or cursor pagination can be more stable and efficient than large offsets.

## Scenario questions

### A service has two implementations. How do you select one?

**Interview-ready answer:**

If one implementation is the normal default, I mark it `@Primary`. If the choice is explicit at one injection point, I use `@Qualifier`. If the implementation depends on configuration, I use conditional configuration or inject a map of strategies and select by a business key.

### A singleton service stores the current user in a field. Is it safe?

**Interview-ready answer:**

No. A singleton bean serves concurrent requests, so request-specific mutable fields can leak data and race. I keep services stateless and pass request data through method parameters or use correctly scoped request context where necessary.

