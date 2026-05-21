---
applyTo:
  - "**/pom.xml"
  - "**/build.gradle"
  - "**/build.gradle.kts"
  - "**/settings.gradle"
  - "**/settings.gradle.kts"
  - "**/*Controller.java"
  - "**/*Service.java"
  - "**/*Repository.java"
  - "**/*Configuration.java"
  - "**/*Config.java"
---

# Spring & Java build file standards

## Spring Transactions

- `@Transactional` on `private` methods has no effect — Spring proxies bypass it.
  Flag as `[must-fix]`.
- `@Transactional` method called via `this.method()` from the same class bypasses
  the proxy. Flag as `[must-fix]` and suggest extracting to another bean or
  using self-injection.
- `@Transactional(readOnly = true)` should be used for query-only methods —
  enables performance optimizations and prevents accidental writes.
- Avoid `@Transactional` on the class level when only some methods need it.

## Spring Controllers

- Declare explicit `produces` and `consumes` on `@RequestMapping` /
  `@GetMapping` / `@PostMapping`.
- `@RequestParam` without `required = false` or a default — confirm the param
  is genuinely required, otherwise document the 400 it will throw.
- Validation: `@Valid` on `@RequestBody` parameters, `@Validated` on classes
  using parameter-level validation annotations.
- Return `ResponseEntity<T>` when you need to control status code or headers,
  otherwise return the body directly.
- Never expose JPA entities directly from controllers — use DTOs. Flag entity
  classes appearing in `@RestController` signatures.

## Spring Repositories

- Repository methods returning `List<T>` for potentially large result sets —
  prefer `Stream<T>`, `Page<T>`, or `Slice<T>` with pagination.
- `@Query` with `nativeQuery = true` and concatenated user input → `[must-fix]`.
- Custom `@Query` JPQL/SQL should use named parameters (`:userId`) over
  positional (`?1`) for readability.
- `findAll()` without pagination on entities with potential growth → `[should-fix]`.

## Spring Configuration

- `@Value("${prop}")` scattered across the codebase — prefer
  `@ConfigurationProperties` with a typed POJO and `@Validated`.
- Hardcoded URLs, timeouts, pool sizes in `@Bean` methods — externalize to
  `application.yml` and inject.
- Profile-specific beans should use `@Profile`, not runtime `if` checks on
  `${spring.profiles.active}`.
- `@Bean` methods returning concrete classes — prefer the interface type for
  testability.

## Maven (pom.xml)

- Pin versions explicitly. Flag missing `<version>` (unless inherited from a
  managed parent BOM).
- Use `<dependencyManagement>` for version coordination across modules.
- `spring-boot-starter-parent` or `spring-boot-dependencies` BOM should manage
  Spring versions — don't override individual Spring deps without justification.
- Flag `<scope>compile</scope>` on test-only deps — use `test` scope.
- Snapshot deps (`-SNAPSHOT`) in release branches → `[must-fix]`.

## Gradle (build.gradle / build.gradle.kts)

- Pin versions explicitly. Flag `+`, `latest.release`, or dynamic ranges like `1.+`.
- Prefer `implementation` over `api` unless the dep is genuinely part of your
  public API surface.
- `testImplementation` for test deps, never `implementation`.
- Use the Spring Boot Gradle plugin's dependency management rather than pinning
  Spring versions manually.

## Known-bad dependencies (any build file)

- `log4j-core` < 2.17.1 → `[must-fix]` (Log4Shell range).
- `commons-collections` 3.x without documented reason → `[should-fix]`
  (deserialization gadget chains).
- `xstream` < 1.4.20 → `[must-fix]` (deserialization CVEs).
- `jackson-databind` < 2.15.x → flag and suggest upgrade.
- `snakeyaml` < 2.0 without explicit safe-mode usage → `[should-fix]`.
- `spring-security` and `spring-core` versions out of sync → `[must-fix]`.
