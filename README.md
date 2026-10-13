# Interview Preparation Guide: JDK 8 → 17, Spring Boot 2.7.17 → 3.x, and JBoss EAP 7.4 → 8.0

> **Purpose:** Prepare to discuss enterprise application upgrades confidently in Java backend interviews. Use this as a study guide, not as a claim that every checklist item occurred in your project. Be precise about what you personally changed, what your team handled, and what you verified.

## 1. How to explain the upgrade work in an interview

### A concise 60–90 second answer

“My application upgrade work covered the Java runtime, the Spring application framework, and the application-server platform. We moved the Java baseline from JDK 8 to JDK 17, upgraded Spring Boot from 2.7.17 to the 3.x line, and moved the deployment platform from JBoss EAP 7.4 to EAP 8. These changes had to be treated as a compatibility exercise rather than just version-number changes. I would check the supported Java and server versions, review dependency and namespace changes, update build configuration, compile and run automated tests, deploy to a lower environment, verify startup and integrations, and then validate logs and application behaviour. The most important areas to understand are the Java language/runtime changes, Spring Boot 3’s Java 17 and Jakarta EE baseline, and the differences between the old and new application-server environments.”

Adapt this answer to what you actually did. If asked for specifics, explain the exact modules, code changes, test evidence, issues found, and your own contribution. Do not claim that you personally performed a step unless you did.

### Keep the three upgrades distinct

| Upgrade | What changed | Main interview focus |
|---|---|---|
| JDK 8 → 17 | Java compiler/runtime baseline and JDK APIs | Compatibility, removed/deprecated APIs, strong encapsulation, TLS/security defaults, build/runtime alignment |
| Spring Boot 2.7.17 → 3.x | Framework, dependency ecosystem, configuration and Jakarta baseline | Java 17 minimum, Spring Framework 6, `javax.*` → `jakarta.*`, dependency upgrades, security/config changes, tests |
| JBoss EAP 7.4 → 8.0 | Application-server/runtime platform and supported specifications | Certified JDK, Jakarta EE support, deployment descriptors, server modules, datasources/JMS/security, logging and deployment validation |

**Terminology note:** JBoss EAP and WildFly are related but not identical products. EAP is Red Hat’s supported enterprise application platform; WildFly is the upstream community application server. Say “JBoss EAP 8.0” if that was the actual target. Do not call an EAP deployment a WildFly migration unless that is what the project used.

---

# Part A — JDK 8 to JDK 17

## 2. What does JDK mean?

- **JDK (Java Development Kit):** tools for developing Java applications, including the compiler (`javac`), runtime components, and development utilities.
- **JRE (Java Runtime Environment):** historically described the runtime needed to run Java applications. Since Java 9, the JDK distribution no longer follows the old separate-JRE packaging model in the same way.
- **JVM (Java Virtual Machine):** executes Java bytecode and provides runtime services such as memory management and garbage collection.
- **JDK version vs source level:** the JDK used to compile/run an app, the language source/target level, and the deployed runtime should be deliberately aligned. Setting a Maven `source` level alone does not install or select the runtime.

## 3. Why upgrade from Java 8 to 17?

Typical reasons include long-term support, vendor support, security updates, performance/runtime improvements, modern language features, and compatibility with newer frameworks. For this project, the most defensible explanation is that a newer Java baseline was needed to support the modernized framework and application-server stack, provided that matches the actual project decision.

Java 17 is an LTS release. The migration from 8 to 17 crosses several Java releases, so review the cumulative changes rather than treating it as one small update.

## 4. Java 8 vs Java 17 — key changes to know

### Language and API features introduced after Java 8

| Feature | Introduced | What to say |
|---|---:|---|
| Local-variable type inference (`var`) | Java 10 | Can reduce repeated local variable types; only for local variables with an initializer, not a general replacement for static typing. |
| Standard HTTP Client API (`java.net.http`) | Java 11 | Modern HTTP client supporting synchronous and asynchronous calls. |
| Text blocks (`"""`) | Java 15 | Easier multiline strings; useful for SQL/JSON snippets, subject to project style. |
| Records | Java 16 | Concise data-carrier classes with generated accessors, constructor, `equals`, `hashCode`, and `toString`; not a drop-in replacement for every DTO. |
| Pattern matching for `instanceof` | Java 16 | Combines type check and cast in a more concise form. |
| Switch expressions | Standard in Java 14 | `switch` can return a value and use arrow labels. |
| Sealed classes | Standard in Java 17 | Restrict which classes/interfaces may extend or implement a type. |
| Helpful NullPointerException messages | Java 14 | JVM can identify which expression was null in many NPE cases. |

You do **not** need to rewrite a Java 8 codebase to use these features. A migration can keep most code unchanged while compiling and running it on a newer JDK.

### Runtime/platform changes

- **Strong encapsulation of JDK internals:** Java 9 introduced the module system; access to internal JDK APIs became more restricted over time. Libraries relying on internal APIs or reflective access may fail or emit warnings.
- **Removed Java EE/CORBA modules:** several Java EE and CORBA modules were removed from the JDK in Java 11. Applications that relied on JDK-bundled JAXB, JAX-WS, activation or related APIs may need explicit dependencies or replacements.
- **Garbage collectors and runtime behaviour:** defaults and available collectors changed across releases. Do not claim that a newer JDK automatically improves every workload; benchmark representative workloads.
- **TLS/security defaults:** newer JDKs may disable older protocols, algorithms, or certificates. This can expose outdated downstream endpoints or trust-store configuration.
- **Tooling and dependencies:** Maven/Gradle plugins, bytecode instrumentation, test libraries, agents, and vendor-specific libraries must support the selected JDK.

## 5. JDK migration checklist

### Build and code

1. Confirm the target JDK distribution/vendor and exact version used in local development, CI, test environments, and production.
2. Update Maven/Gradle compiler configuration and CI build images.
3. Prefer Maven compiler `release` configuration when appropriate, so compilation targets the intended Java API level.
4. Compile from a clean build; fix compilation errors instead of suppressing them blindly.
5. Search for removed JDK-bundled APIs, internal `sun.*`/`com.sun.*` usage, illegal reflective access, and dependencies that require older runtimes.
6. Update libraries, Maven plugins, test frameworks, code coverage tools, agents, and static-analysis tools to versions compatible with JDK 17.
7. Check shell scripts, Docker images, service launch commands, JVM options, environment variables, and operational scripts.

### Testing and operations

1. Run unit tests, integration tests, and regression tests on JDK 17—not only on the developer’s machine.
2. Validate database drivers, TLS connections, outbound HTTP calls, MQ/Kafka clients, file encoding, date/time behaviour, and serialization where used.
3. Compare startup logs and warning/error counts before and after the upgrade.
4. Exercise representative load and observe CPU, heap, garbage collection, latency, thread counts, and memory usage.
5. Confirm monitoring, log collection, health checks, JVM diagnostics, and rollback steps.
6. Deploy to a lower environment first, obtain evidence, then progress through the release process.

### Example Maven configuration

```xml
<properties>
    <maven.compiler.release>17</maven.compiler.release>
</properties>
```

The project also needs a compatible Maven Compiler Plugin version and a JDK 17 installed/configured in the build environment. Do not assume this property alone changes the JDK used by the IDE or deployment server.

## 6. JDK 8 → 17 interview questions and sample answers

**Q1. Is upgrading the JDK just changing the version in `pom.xml`?**

No. The build compiler, source/API level, runtime, dependencies, build plugins, CI agents, application server, JVM arguments, and deployment images must all be compatible. I would compile and test on the target JDK, validate integrations and operational behaviour, and deploy through lower environments before production.

**Q2. What is the difference between JDK, JRE, and JVM?**

The JVM executes bytecode. The JDK provides development tools plus runtime components. JRE is the traditional term for the runtime environment; modern JDK distributions no longer necessarily ship a separate JRE package as they did historically.

**Q3. What problems can occur when moving from Java 8 to 17?**

Common risks are removed Java EE modules, internal JDK API usage, stronger encapsulation affecting reflection, incompatible libraries or agents, TLS/certificate issues, changed JVM behaviour, and build tools that do not support the target JDK.

**Q4. Does Java 17 require using records, `var`, or text blocks?**

No. These are optional language features. A migration can retain Java 8-style source code and still run on Java 17, subject to compatibility. Feature adoption should be a separate, deliberate code-quality decision.

**Q5. How would you prove the migration was successful?**

A clean reproducible build on JDK 17, automated test results, successful deployment and startup, verified downstream integrations, regression tests, log/health checks, performance comparison, and documented release/rollback evidence.

**Q6. How do you investigate an `InaccessibleObjectException`?**

Identify the library and reflective access path from the stack trace, check whether it relies on JDK internals, and upgrade or replace the library where possible. A targeted `--add-opens` can sometimes be a temporary compatibility measure, but it should not replace a durable fix without a clear risk assessment.

**Q7. What is the difference between `source`, `target`, and `release`?**

`source` controls accepted Java language syntax; `target` controls generated class-file version. Neither alone guarantees the code only uses APIs available in that Java version. `release` combines language/class-file/API constraints for a target release when supported by the compiler.

---

# Part B — Spring Boot 2.7.17 to Spring Boot 3.x

## 7. The essential version story

Spring Boot 3.0 requires **Java 17 or later** and is based on **Spring Framework 6**. It adopts Jakarta EE 9+ APIs, including the important package namespace shift from `javax.*` to `jakarta.*` for APIs covered by that migration. Spring Boot 2.7 is the last 2.x line and was commonly used as a migration stepping stone.

Do not say every `javax.*` package becomes `jakarta.*`. Java SE packages such as `javax.sql` and `javax.crypto` remain under `javax`. The change applies to the relevant Java EE/Jakarta EE APIs, for example Servlet, Persistence, Validation and related specifications.

## 8. Major changes to explain

### 8.1 Java 17 baseline

The application must build and run on Java 17 or later. Ensure local IDE settings, Maven/Gradle, CI, test containers, application server and production runtime all use compatible versions.

### 8.2 Spring Framework 6

Spring Boot 3 is built on Spring Framework 6. This introduces framework-level compatibility changes and a Jakarta EE baseline. Review framework APIs, third-party integrations, configuration, testing, and observability libraries.

### 8.3 `javax.*` → `jakarta.*`

Typical examples:

| Older namespace | New namespace |
|---|---|
| `javax.servlet.*` | `jakarta.servlet.*` |
| `javax.persistence.*` | `jakarta.persistence.*` |
| `javax.validation.*` | `jakarta.validation.*` |
| `javax.transaction.*` | `jakarta.transaction.*` |
| `javax.annotation.*` (Jakarta EE annotations) | `jakarta.annotation.*` |

Check imports, dependency coordinates, generated code, XML descriptors, annotations, and transitive dependencies. A dependency that still expects the old `javax` API may not be binary-compatible with a component expecting `jakarta` APIs.

### 8.4 Persistence and Hibernate

Spring Boot 3 moved to the Hibernate 6 generation. Review ORM queries, custom types, dialect configuration, identifier generation, SQL generation, and deprecated/changed Hibernate APIs. Do not assume every application requires query changes; run persistence-focused integration tests against the actual database and driver.

### 8.5 Spring Security

Spring Security 6 changes configuration patterns and removes older approaches such as the deprecated `WebSecurityConfigurerAdapter` pattern. Modern configuration generally defines a `SecurityFilterChain` bean. If the application uses security, OAuth2, JWT, method security, or custom filters, review the relevant migration notes and test authentication/authorization end to end. Do not claim a security migration if the application did not use those components.

### 8.6 Configuration and observability

Review changed/deprecated configuration properties and use the Spring Boot properties migrator only as a temporary diagnostic aid if suitable. Validate Actuator endpoints, metrics, tracing, logging, health indicators, and any custom management endpoint configuration. An endpoint being exposed does not automatically mean it is secured appropriately.

### 8.7 Third-party dependencies

Check compatibility of database drivers, messaging clients, OpenAPI/Swagger libraries, validation, JSON libraries, logging, test libraries, APM agents, and any in-house shared libraries. Prefer a coherent Spring Boot dependency-management baseline instead of manually pinning every transitive dependency without a reason.

## 9. Recommended migration sequence

1. **Inventory:** record current Boot, Spring Framework, Java, Hibernate, Security, servlet container, database driver and key library versions.
2. **Baseline:** capture clean build results, test results, application startup logs, endpoint behaviour, integration behaviour and performance baselines.
3. **Move to Java 17:** make the build and runtime compatible first, and resolve JDK-specific issues.
4. **Update the Boot parent/BOM:** move to a chosen supported Boot 3.x version compatible with the target platform; do not select an arbitrary minor version without checking support policy and dependency compatibility.
5. **Resolve Jakarta migration:** update imports and dependencies where needed; inspect XML/configuration and generated sources.
6. **Update integrations:** Hibernate/JPA, security, validation, web, messaging, observability, and vendor libraries.
7. **Build and test iteratively:** compile errors first, then unit tests, integration tests, regression and contract tests.
8. **Deploy to a lower environment:** confirm server/runtime compatibility, configuration, secrets, connectivity and operational monitoring.
9. **Regression/performance checks:** validate representative business flows and non-functional requirements.
10. **Release safely:** follow change approvals, deployment/rollback plan, smoke tests and post-release monitoring.

## 10. What to test in a banking backend

Use only the categories relevant to your application, and be prepared to explain actual tests you performed.

- API contract: request/response fields, status codes, validation errors and backward compatibility.
- Authentication/authorization: access controls and role behaviour, if applicable.
- Database: connection pool, transactions, rollback, locking, SQL generation, data types and query performance.
- Messaging: queue/topic connectivity, message conversion, retry, duplicate handling and transaction boundaries, if applicable.
- Downstream services: TLS, certificates, timeouts, error mapping and retry behaviour.
- Batch/scheduled jobs: startup, scheduling, restartability, job parameters and data correctness.
- Deployment: WAR/JAR packaging, context path, server libraries, environment-specific configuration and health checks.
- Observability: logs, correlation IDs, metrics, traces and alerting.
- Non-functional: startup time, throughput, latency, CPU, memory, thread pools and GC.

## 11. Spring Boot 2.7 → 3.x interview questions and sample answers

**Q1. What are the biggest breaking changes in Spring Boot 3?**

The Java 17 minimum, Spring Framework 6, Jakarta EE namespace changes, and upgrades to core integrations such as Hibernate and Spring Security are the major areas. There can also be configuration-property and third-party dependency changes, so I would use the official migration guide and test the complete application rather than focus only on compilation.

**Q2. Why is `javax.servlet` not interchangeable with `jakarta.servlet`?**

They are different Java packages and types. A class compiled against one namespace does not automatically satisfy an API expecting the other. Imports, libraries, servlet container and deployment platform need to agree on the Jakarta API generation.

**Q3. Do all `javax` imports need to change?**

No. Only packages that moved as part of the Jakarta EE transition should change. Java SE packages such as `javax.sql` remain `javax`. Blindly replacing every `javax` string can break the application.

**Q4. What would you do if the application compiles but fails during deployment?**

Read the earliest meaningful cause in the server/application logs, check dependency and API namespace mismatches, validate the supported JDK/server combination, inspect deployment descriptors and server modules, verify external configuration and data sources, and reproduce in the closest lower environment. Then make one controlled change at a time and retest.

**Q5. Why can an old third-party library break after Boot 3 migration?**

It may reference `javax` APIs, depend on an older Spring/Hibernate/Security API, use unsupported reflection, or bring conflicting transitive dependencies. I would inspect the dependency tree, check the vendor’s compatibility matrix, upgrade or replace the library, and add focused tests.

**Q6. How do you handle configuration changes?**

Compare the old configuration with the target Boot version’s migration notes, inspect startup warnings, validate property names and defaults, and test each environment’s effective configuration. A properties migrator can help identify renamed properties, but should not be treated as a permanent substitute for updating configuration.

**Q7. What is the role of the Spring Boot parent or BOM?**

It provides a tested set of dependency versions and plugin defaults. Using its dependency management reduces version drift. Override a managed version only when there is a documented compatibility or security reason and tests validate the combination.

**Q8. How do you explain your personal contribution honestly?**

State the modules/services you handled, the code/configuration/dependency changes you personally made, the issues you investigated, and the test/deployment checks you executed. Separate those from work performed by teammates, platform teams or release/operations teams.

---

# Part C — JBoss EAP 7.4 to JBoss EAP 8.0

## 12. Understand the platform first

**JBoss EAP** is Red Hat’s enterprise application platform. It provides an application-server runtime and enterprise services such as web container support, naming, datasource connection pools, transaction management, security integration, messaging integration and deployment management.

**WildFly** is the upstream community application server from which EAP is derived. EAP has its own supported release cadence, tested combinations, patches and support guarantees. Check Red Hat’s official EAP 8.0 documentation for the exact supported JDKs, operating systems, migration requirements and feature differences applicable to the deployed patch level.

## 13. Why the application-server upgrade matters

Moving from EAP 7.4 to EAP 8 is not just copying a WAR file to a new server. The target server can change the supported Jakarta EE APIs, runtime dependencies, subsystem behaviour, server modules, security defaults and configuration model. Application compatibility depends on both the app’s framework baseline and the server’s supported specification level.

If the application was migrated to Spring Boot 3 and deployed as a WAR to an external servlet container/application server, the servlet API namespace and container compatibility are especially important. If it is an executable Spring Boot JAR, clarify whether EAP hosts it directly, whether it is deployed as a WAR, or whether the architecture uses another packaging model. Do not mix these models in your answer.

## 14. Migration areas to review

### 14.1 Supported Java and server configuration

- Verify the target EAP 8 patch level’s supported JDK and operating-system matrix.
- Confirm the actual Java executable used by the server service, not just the `java -version` from an interactive terminal.
- Review JVM heap, GC, system properties, trust stores, environment variables and service scripts.
- Compare standalone/domain mode and server configuration management with the existing deployment model.

### 14.2 Jakarta EE and deployment compatibility

- Review the application’s Servlet/Jakarta API usage and any EE APIs supplied by the server.
- Inspect `web.xml`, `jboss-web.xml`, `jboss-deployment-structure.xml`, persistence descriptors and other deployment descriptors where present.
- Ensure the application does not package conflicting copies of APIs that should be provided by the container.
- Check classloading and server module dependencies if the app relies on server-provided libraries.

### 14.3 Datasources and database drivers

- Verify datasource names/JNDI names, driver modules, credentials/secret references, pool sizes, validation queries and transaction settings.
- Confirm the correct JDBC driver version is installed and supported.
- Test connection acquisition, transaction commit/rollback, timeout behaviour and pool exhaustion handling.
- Never place credentials in source control or paste real credentials into interview notes.

### 14.4 Messaging and external integrations

Where used, check JMS/Jakarta Messaging configuration, IBM MQ or other client libraries, connection factories, queue/topic names, TLS, credentials, message conversion, transaction behaviour and reconnection policy. Confirm whether messaging is managed by EAP or accessed through an application-level client; these are different configuration paths.

### 14.5 Security, TLS and identity

Review security realms/domains or Elytron-based configuration as applicable to the existing setup, application authentication integration, role mapping, certificates, trust stores, cipher/protocol compatibility and secrets. Validate both successful and rejected access paths.

### 14.6 Logging, monitoring and operational behaviour

Check log categories and formats, log locations/rotation, health checks, metrics, deployment scanner/management deployment mechanism, server startup, thread pools, connection pools, scheduled jobs and alerting. Compare startup logs and warnings with the baseline.

## 15. A practical EAP migration verification checklist

- [ ] Confirm official support matrix for EAP 8 patch level, JDK, OS, database and required integrations.
- [ ] Document the old server configuration and deployment topology.
- [ ] Build the application with its intended Java and Spring Boot baseline.
- [ ] Review deployment descriptors, namespaces, server modules and classloading exclusions/dependencies.
- [ ] Configure datasources, drivers, JNDI names, security and external resources in the target environment.
- [ ] Deploy to a lower environment and inspect the full startup/deployment logs.
- [ ] Run smoke tests and business regression tests.
- [ ] Test database transactions, messaging, downstream HTTP/TLS and batch/scheduled workloads where applicable.
- [ ] Compare application behaviour, resource use, logs, health indicators and response times.
- [ ] Confirm operational runbooks, deployment procedure, rollback and post-deployment validation.

## 16. JBoss EAP interview questions and sample answers

**Q1. What is JBoss EAP?**

It is Red Hat’s supported enterprise application platform based on the WildFly application-server project. It provides a managed runtime for enterprise Java applications and services such as web deployment, datasources, transactions, security integration and messaging integration.

**Q2. Is JBoss EAP the same as WildFly?**

No. WildFly is the upstream community project. EAP is Red Hat’s enterprise product with its own release, support, certification and patch lifecycle. Their relationship does not mean every version or feature is identical.

**Q3. What are the risks in moving EAP 7.4 to EAP 8?**

The main risks are supported JDK/runtime changes, Jakarta EE API compatibility, deployment descriptors, server modules and classloading, datasource/driver setup, security/TLS configuration, messaging integration and differences in operational configuration. The exact risks depend on the application packaging and features it uses.

**Q4. What is JNDI and why does it matter for a datasource?**

JNDI is Java Naming and Directory Interface. An application can look up a server-managed resource using a logical name, such as a datasource JNDI name, rather than embedding connection details in code. The application’s lookup name must match the resource configured on the server.

**Q5. What is the difference between an application-managed datasource and a server-managed datasource?**

With a server-managed datasource, the application server manages the connection pool and exposes a resource, often through JNDI. An application-managed pool is configured and managed inside the application/runtime libraries. The choice affects lifecycle, credentials, pooling, transaction integration and operational ownership.

**Q6. What do you check if a WAR deploys but the application cannot connect to the database?**

Check the first relevant exception, datasource/JNDI name, server datasource status, driver module/version, network access, credentials, TLS/trust configuration, pool limits and database availability. Then run a datasource connection test and a minimal application-level query, while keeping secrets out of logs.

**Q7. What causes classloading conflicts?**

The server and application may provide different versions of the same class/API, or the deployment may package libraries that conflict with server modules. Inspect dependency contents and server module configuration, and avoid bundling container-provided APIs unnecessarily.

**Q8. How do you validate the server upgrade?**

Confirm supported versions and configuration, deploy in a lower environment, check startup/deployment logs, test web endpoints and business flows, validate DB/messaging/security integrations, compare performance/monitoring, and prove rollback readiness.

---

# Part D — The combined upgrade story

## 17. How the three migrations relate

```text
JDK 8 → JDK 17
    └─ Establishes the modern Java compiler/runtime baseline

Spring Boot 2.7.17 → Spring Boot 3.x
    └─ Requires Java 17+ and moves to Spring Framework 6 / Jakarta EE APIs

JBoss EAP 7.4 → EAP 8.0
    └─ Requires deployment compatibility with the target server/Jakarta APIs/JDK

Combined result
    └─ Build compatibility + application compatibility + runtime/deployment compatibility
       + regression evidence + operational readiness
```

The exact order can vary by project. A safe approach is to establish the supported target versions first, then migrate incrementally in a branch/environment where issues can be isolated. The dependency chain means that the JDK, Spring Boot and server upgrades should be validated together before production rollout.

## 18. Strong answer to “What was the most challenging part?”

Use this structure and fill in a genuine example:

1. **Context:** “The challenge appeared while moving [specific module/service] from [old baseline] to [target baseline].”
2. **Symptom:** “We observed [a real compile error, startup failure, namespace mismatch, dependency conflict, datasource issue, or test regression].”
3. **Investigation:** “I traced the first meaningful error, checked the relevant dependency/configuration and reproduced it in [environment].”
4. **Fix:** “I changed [the exact code, dependency, import, configuration or deployment setting] and verified it with [test or command].”
5. **Outcome:** “The service then [built/deployed/passed regression tests], and we confirmed [evidence].”

Do not memorize a fabricated issue. Interviewers often ask follow-ups such as the exception name, file changed, root cause, test case, and why the fix worked.

## 19. Questions that connect all three topics

**Q1. Why must you consider the application server during a Spring Boot upgrade?**

Because the target Spring Boot version can change the Java and Jakarta API baseline, while the server supplies runtime APIs and services. If the app is deployed as a WAR, the servlet/Jakarta API level and classloading behaviour must be compatible with the target server.

**Q2. Could Java 17 and Boot 3 work locally but fail on the target server?**

Yes. Local execution may use the embedded server and a different classpath, while deployment to EAP uses server-provided APIs, modules and configuration. Validate the actual packaging and deployment mode on the target platform.

**Q3. How would you separate a code issue from an environment issue?**

Use the earliest failure in logs, reproduce in a controlled environment, compare runtime versions and effective configuration, inspect dependency/classloading differences, and run a minimal test for the affected integration. Avoid changing several unrelated settings at once.

**Q4. What is your rollback strategy?**

Follow the release plan: preserve the previous deployable artifact and compatible configuration, identify database/schema changes that are or are not backward-compatible, define health/smoke-test gates, and agree on rollback ownership and triggers before production. Never assume rolling back the application automatically reverses a database migration.

**Q5. How do you make the upgrade reproducible?**

Pin and document supported toolchain versions, use dependency management, build from a clean checkout in CI, externalize environment configuration, retain test evidence, and document deployment/rollback steps.

---

# Part E — Practical commands and evidence

## 20. Useful commands

These are examples; adapt paths and tooling to the project.

```bash
# Confirm the Java executable on PATH
java -version
javac -version

# Build and run tests with Maven
mvn -version
mvn clean verify

# Inspect Maven dependency tree for version conflicts
mvn dependency:tree

# Search source for likely namespace migration points (PowerShell example)
Get-ChildItem -Recurse -Include *.java,*.xml,*.yml,*.yaml |
  Select-String -Pattern 'javax\.servlet|javax\.persistence|javax\.validation|jakarta\.'
```

For an application server, use the approved server management/CLI procedure for the environment to inspect deployments, datasources and server configuration. Exact commands depend on standalone/domain mode, permissions and organizational runbooks.

## 21. Evidence to keep for interview preparation

Create a small private notes table from your real project records:

| Evidence | Your notes |
|---|---|
| Exact source and target versions | Fill in actual versions/patches |
| Modules/services you personally changed | Fill in actual names or describe safely without confidential identifiers |
| Dependency/import/config changes | List real examples |
| Build/test commands and results | Fill in actual evidence |
| Defects discovered and root causes | Use genuine examples only |
| Integration tests performed | DB, HTTP, MQ, batch, security, etc. only if applicable |
| Deployment environment and validation | Fill in actual details |
| Your role vs other teams’ role | Be precise about ownership |

Do not include proprietary code, internal hostnames, customer data, secrets, or restricted system details in public notes or GitHub.

---

# Part F — A focused study plan

## 22. Five-session revision plan

1. **Session 1 — Java 8 → 17:** JDK/JVM/JDK terminology, language feature timeline, removed APIs, reflection/encapsulation, TLS, Maven compiler `release`.
2. **Session 2 — Boot 2.7 → 3:** Java 17 baseline, Spring Framework 6, Jakarta namespace migration, dependency management, configuration changes.
3. **Session 3 — Hibernate/Security/integrations:** Hibernate 6/JPA, Security 6 if used, validation, database drivers, messaging and observability libraries.
4. **Session 4 — EAP 7.4 → 8:** EAP vs WildFly, supported version matrix, WAR vs executable JAR, descriptors, classloading, JNDI/datasources, TLS/logging.
5. **Session 5 — Your real story:** practise the 90-second summary, one genuine challenge, your exact contribution, test evidence, and rollback approach.

## 23. Final interview checklist

Before an interview, make sure you can answer without notes:

- [ ] Why the upgrades were needed.
- [ ] What changed between the source and target versions.
- [ ] Which compatibility issue was most relevant to your application.
- [ ] Which code/configuration/dependency changes you personally made.
- [ ] How you tested the change and what the results were.
- [ ] How you validated database, messaging, security and downstream integrations that your service uses.
- [ ] How the app was packaged and deployed: executable JAR, WAR, embedded server, or external EAP runtime.
- [ ] What you monitored after deployment and how rollback would work.
- [ ] What was handled by your team versus platform/DevOps/release teams.

## 24. Official documentation to consult

Always check documentation matching the **exact target patch/version** used by the project:

- Oracle/OpenJDK Java migration guides and release notes for Java 9 through 17.
- Spring Boot 3.x migration guide and Spring Boot system requirements.
- Spring Framework 6 migration documentation.
- Hibernate ORM 6 migration guide, if JPA/Hibernate is used.
- Spring Security 6 migration guide, if Spring Security is used.
- Red Hat JBoss EAP 8.0 Installation Guide, Configuration Guide, supported configurations and migration guide.

The exact supported versions and migration notes can differ by patch release. Treat the official version-specific documentation as the final authority for a production upgrade.
