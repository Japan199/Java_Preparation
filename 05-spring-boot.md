# Spring Boot: Basic to Production-Ready

## 1. What is Spring Boot?

**Interview-ready answer:**

Spring Boot is an opinionated layer over Spring that simplifies application setup through auto-configuration, starter dependencies, embedded servers, externalized configuration, and production features such as Actuator. It does not replace Spring; it configures Spring applications with sensible defaults that can be overridden.

## 2. What does `@SpringBootApplication` contain?

**Interview-ready answer:**

It combines `@SpringBootConfiguration`, `@EnableAutoConfiguration`, and `@ComponentScan`. I place it in a root package so component scanning covers the intended application packages without scanning unrelated code.

## 3. How does auto-configuration work?

**Interview-ready answer:**

Spring Boot examines the classpath, existing beans, properties, and application type, then conditionally registers configuration. Conditions include class presence, missing beans, and configured properties. If I define my own bean, many defaults back off.

## 4. What are starters?

**Interview-ready answer:**

Starters are curated dependency descriptors for a capability, such as web, validation, data JPA, or security. They reduce manual version coordination because Spring Boot’s dependency management provides compatible versions.

## 5. Why use an embedded server?

**Interview-ready answer:**

An embedded server packages the runtime with the application, making it executable and consistent across environments. It simplifies deployment, containerization, and local testing. Tomcat is a common default for servlet applications, but alternatives can be selected.

## 6. How is configuration externalized?

**Interview-ready answer:**

Spring Boot reads configuration from sources such as properties or YAML files, environment variables, system properties, and command-line arguments with a defined precedence. I keep environment-specific values outside the artifact and never store secrets in source control.

## 7. `@Value` versus `@ConfigurationProperties`

**Interview-ready answer:**

`@Value` is convenient for one or two values. For grouped application configuration, I prefer typed `@ConfigurationProperties` because it supports validation, metadata, structured binding, and easier testing.

```java
@ConfigurationProperties("payment.client")
@Validated
public record PaymentClientProperties(
    @NotBlank String baseUrl,
    @NotNull Duration timeout
) {}
```

## 8. What are profiles?

**Interview-ready answer:**

Profiles activate environment- or feature-specific beans and properties. I use them sparingly for genuine structural differences, not as the only mechanism for every configuration value. Environment variables and configuration properties usually handle value differences more cleanly.

## 9. How do you manage secrets?

**Interview-ready answer:**

I load secrets at runtime from a secret manager, container secret, or protected environment configuration. I restrict access, rotate credentials, avoid logging them, and keep placeholders rather than actual values in Git.

## 10. What is Actuator?

**Interview-ready answer:**

Actuator exposes production endpoints and metrics for health, readiness, configuration insight, and application behavior. I expose only required endpoints, secure them, and separate management access where needed. Health details should not leak internal dependency information publicly.

## 11. Liveness versus readiness

**Interview-ready answer:**

Liveness answers whether the process should be restarted. Readiness answers whether it should currently receive traffic. A temporary database outage should normally make a service unready, not necessarily kill a healthy process and create a restart loop.

## 12. How do you create custom health indicators?

**Interview-ready answer:**

I implement a health contributor for a meaningful dependency or internal state and include it in the appropriate health group. Checks must be fast, bounded by timeouts, and should not overload the dependency.

## 13. How do you handle environment-specific logging?

**Interview-ready answer:**

I configure log levels externally, use structured logs in deployed environments, include trace and correlation identifiers, and avoid sensitive data. I do not enable debug logging globally in production because it adds cost and may expose data.

## 14. What is the startup runner difference?

**Interview-ready answer:**

`CommandLineRunner` receives raw string arguments; `ApplicationRunner` receives parsed `ApplicationArguments`. Both run after the context is initialized. I keep them lightweight and avoid making application readiness depend on uncontrolled long-running work.

## 15. How do you customize auto-configuration?

**Interview-ready answer:**

I first use supported properties. If needed, I define my own bean so the conditional default backs off, or exclude a specific auto-configuration with a clear reason. I use the condition evaluation report when debugging why configuration was or was not applied.

## 16. What changed with Spring Boot 3?

**Interview-ready answer:**

Spring Boot 3 is based on Spring Framework 6, requires a modern Java baseline, uses Jakarta EE namespaces such as `jakarta.persistence`, and improves native-image and observability support. A migration from Boot 2 requires dependency compatibility and `javax` to `jakarta` changes, not just a version bump.

## 17. What is graceful shutdown?

**Interview-ready answer:**

Graceful shutdown stops accepting new traffic while allowing in-flight requests to complete within a configured timeout. In orchestration, I combine it with readiness changes and adequate termination grace periods so deployments do not drop requests.

## 18. How do you configure HTTP clients?

**Interview-ready answer:**

I configure connection and read timeouts, connection pooling, bounded retries for safe operations, tracing, metrics, and error mapping. I keep client configuration centralized and never rely on unlimited default timeouts.

## 19. `RestTemplate`, `WebClient`, and declarative clients

**Interview-ready answer:**

`RestTemplate` is a synchronous legacy-style client commonly found in existing applications. `WebClient` supports non-blocking reactive calls and can also be used in controlled synchronous flows. Declarative HTTP interfaces reduce boilerplate. I choose based on the application model and do not introduce reactive code unless the full call path benefits.

## 20. MVC versus WebFlux

**Interview-ready answer:**

Spring MVC is a servlet model and suits most blocking applications. WebFlux is non-blocking and useful for high-concurrency I/O when dependencies and code are reactive end to end. Blocking database or client calls inside a reactive event loop remove the benefit and can harm performance.

## 21. How do you improve startup time?

**Interview-ready answer:**

I measure first, then reduce unnecessary dependencies and scanning, avoid heavy bean initialization, make noncritical work lazy or asynchronous where safe, and examine startup-step diagnostics. Native images or AOT may help specific workloads but add build and compatibility trade-offs.

## 22. How do you package and run a Boot application?

**Interview-ready answer:**

I build an executable JAR using Maven or Gradle and run it with the required Java runtime and external configuration. For containers I use layered, minimal images, run as a non-root user, declare resource limits, and expose health probes.

## 23. How do you implement scheduled jobs safely?

**Interview-ready answer:**

`@Scheduled` is sufficient for simple single-instance tasks. In multiple replicas, I use a distributed scheduler, leader election, or a distributed lock so the job does not run unexpectedly on every instance. Jobs should be idempotent, observable, and retryable.

## 24. What does `@Async` do?

**Interview-ready answer:**

`@Async` executes an eligible proxied method using a task executor. I configure a bounded executor, propagate required context deliberately, return a future type when a result matters, and handle exceptions. Self-invocation will bypass the proxy.

## 25. How would you organize a Spring Boot service?

**Interview-ready answer:**

For a small service, controller-service-repository layers are adequate. For a growing service, I prefer package-by-feature with API, application, domain, and infrastructure responsibilities kept clear. Controllers validate transport data, services coordinate use cases, and repositories isolate persistence.

## Production checklist

- Explicit request and downstream timeouts
- Bounded thread pools and queues
- Health, metrics, logs, and traces
- Graceful shutdown and readiness probes
- Secrets outside the artifact
- Consistent error responses
- Database migration tool
- Connection-pool sizing
- Secure Actuator exposure
- Resource requests and limits
- Integration and contract tests

