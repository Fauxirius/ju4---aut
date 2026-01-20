# Product Requirements Document (PRD): Project Modernization

## 1. Objective

The primary objective of this project is to modernize the `powermock-api` application by upgrading its core technology stack. This involves three key initiatives:

1.  **Upgrading the JDK** from version 8 to **17 (LTS)** to leverage new language features and performance improvements.
2.  **Upgrading the Spring Boot Framework** from `2.1.1.RELEASE` to **`3.2.5`** to enhance security, performance, and access to modern dependencies.
3.  **Modernizing the Testing Framework** by migrating from a legacy JUnit 4 and PowerMock stack to **JUnit 5 and standard Mockito**, improving testability and code quality.

## 2. Functional Requirements (FRs)

### FR-1: Update Build Configuration (`pom.xml`)
The project's build descriptor must be updated to support the new technology stack.

**Acceptance Criteria:**
- The `<java.version>` property in `pom.xml` must be set to `17`.
- The `spring-boot-starter-parent` version in `pom.xml` must be `3.2.5`.
- All PowerMock dependencies (`powermock-module-junit4`, `powermock-api-mockito2`) must be completely removed from `pom.xml`.
- The `spring-boot-starter-test` dependency must be configured to exclude the legacy `junit-vintage-engine`.

### FR-2: Refactor Production Code to Eliminate Static Dependencies
The application code must be refactored to follow modern dependency injection principles, removing the need for PowerMock.

**Acceptance Criteria:**
- The `NotificationUtil.java` class must be converted from a utility class with `static` methods to a standard Spring `@Service`.
- The `sendNotification` method within `NotificationUtil` must be refactored into a non-static instance method.
- The `OrderService.java` class must use constructor-based dependency injection to obtain an instance of `NotificationUtil`.
- All calls to `NotificationUtil.sendNotification` from within `OrderService` must be made on the injected instance, not statically.

### FR-3: Migrate Test Suite to JUnit 5 and Mockito
All existing tests must be migrated from the JUnit 4/PowerMock stack to JUnit 5 and standard Mockito.

**Acceptance Criteria:**
- The test class `PowermockApiApplicationTests.java` must not contain any imports from `org.junit.Test`, `org.junit.runner`, or `org.powermock`.
- The test class must be annotated with `@SpringBootTest` to leverage the Spring testing framework.
- The `NotificationUtil` dependency within the test class must be mocked using the `@MockBean` annotation.
- Test methods must use the JUnit 5 annotation (`org.junit.jupiter.api.Test`).
- The test logic must use standard `Mockito.verify()` to confirm that the `sendNotification` method is called as expected.

### FR-4: Finalize and Validate the Migration
The final migrated application must be stable, correctly configured, and fully functional.

**Acceptance Criteria:**
- All Java source files must be updated to use the `jakarta.*` package namespace where required by Spring Boot 3 (e.g., for persistence, validation).
- The command `mvn clean install` must execute successfully, compiling all code, passing all tests, and producing a valid build artifact.
- The application must start without errors using `mvn spring-boot:run`.
- A manual API test must be performed to verify that the application endpoints respond correctly and that the expected "Notification sent..." message is logged.

## 3. Non-Functional Requirements (NFRs)

-   **Performance:** The application's startup time and API response times must be equal to or better than the baseline established on the JDK 8 stack.
-   **Build Stability:** The project must build successfully using `mvn clean install` with no errors.
-   **Test Quality:** All tests must pass, and overall code coverage must be maintained or improved from the baseline. The final test suite must not contain any references to PowerMock or JUnit 4.
-   **Code Quality:** The refactored code must adhere to SOLID principles, with a specific focus on Dependency Inversion, demonstrated by the mandatory use of constructor injection for all services.

## 4. Documentation Requirements

-   **Audit Trail:** A clear, chronological record of key decisions, challenges encountered, and resolutions applied during the migration process.
-   **Exact Changes Made:** Detailed documentation (e.g., code diffs, structured summaries) outlining the specific code changes in each modified file.
-   **Learnings for Scaling:** A dedicated section capturing insights, challenges, and best practices observed during this migration. This document should serve as a guide for future, larger-scale modernization efforts.
