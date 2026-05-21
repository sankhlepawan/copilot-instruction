---
applyTo:
  - "**/*.java"
---

# Java review standards

## Required

- Target Java 17+ features. Use `var` for obvious local types, records for
  immutable data carriers, pattern matching for `instanceof`, switch expressions.
- Public APIs need Javadoc with `@param`, `@return`, `@throws`.
- Logging via SLF4J: `private static final Logger log = LoggerFactory.getLogger(Foo.class);`.
  Never `System.out.println` or `e.printStackTrace()` in production code.
- Parameterized log messages: `log.info("user {} logged in", userId)` — never
  string concatenation or `String.format` in log calls.
- `Optional<T>` as a return type only. Never as a field, parameter, or in collections.
- Constructor injection over field injection in Spring components. Flag
  `@Autowired` on fields as `[should-fix]`.
- Resource handling via try-with-resources. Flag manual `close()` in `finally` as `[should-fix]`.

## Disallowed

- Catching `Exception` or `Throwable` without re-throwing or logging the cause.
  Silent swallow `catch (Exception e) {}` is `[must-fix]`.
- `new Date()`, `Calendar`, `SimpleDateFormat`. Use `java.time` (`Instant`,
  `ZonedDateTime`, `DateTimeFormatter`).
- `Thread.sleep` in production code outside retry/backoff utilities. In tests,
  prefer Awaitility.
- `Runtime.exec`, `ProcessBuilder` with user-controlled input → command injection.
- `synchronized` on `String` literals, boxed primitives, or `this` in public
  classes — use a dedicated `private final Object lock = new Object()`.
- `volatile` for compound check-then-act operations — use `Atomic*` or proper locks.
- `equals()`/`hashCode()` overridden but not both.
- `==` for `String` or wrapper comparison. Use `.equals()` or `Objects.equals()`.

## Patterns to flag

- Mutable `static` fields without `final` — global state, thread-safety landmine.
- Returning `null` from collection-returning methods. Return empty collection.
- Returning internal mutable collections — wrap with `List.copyOf()` or
  `Collections.unmodifiableList()`.
- Stream operations with side effects in `map()` or `filter()`.
- `parallelStream()` without justification — usually slower and unsafe with
  shared mutable state.
- `String` concatenation inside loops on hot paths — use `StringBuilder`.
- Boxing in tight loops (`Integer` vs `int` in arithmetic).
- `Files.readAllBytes` / `readAllLines` on potentially large files — stream instead.

## Security

- SQL via `Statement` or string-concatenated queries → `[must-fix]`. Use
  `PreparedStatement` or JPA parameters. JPQL with `+ userInput +` → `[must-fix]`.
- `XMLInputFactory` / `DocumentBuilderFactory` / `SAXParserFactory` without XXE
  protections (`setFeature("http://apache.org/xml/features/disallow-doctype-decl", true)`).
- Jackson `enableDefaultTyping()` / `activateDefaultTyping()` → deserialization
  gadget risk. `[must-fix]`.
- `MessageDigest.getInstance("MD5"|"SHA-1")` for security → `[must-fix]`.
- Use `SecureRandom` for tokens/keys, never `Random` or `Math.random`.
- `TrustManager` accepting all certs, `HostnameVerifier` returning `true` →
  `[must-fix]` even in tests.

## Tests

- JUnit 5 (`org.junit.jupiter.api.*`). Flag JUnit 4 imports in new files.
- AssertJ over Hamcrest or raw `assertEquals`.
- Mockito: prefer `@ExtendWith(MockitoExtension.class)` over
  `MockitoAnnotations.openMocks`.
- Test naming: `methodUnderTest_condition_expectedResult` or BDD style.
- New service classes without a corresponding `*Test` → `[should-fix]`.
