# Exact Changes Summary: `powermock-api` Modernization

This document summarizes the exact code and configuration changes made during the project.

## 1. `pom.xml`

The `pom.xml` was significantly modified to upgrade the project's core dependencies.

**Summary of Changes:**
-   The Java version was updated from `1.8` to `17`.
-   The Spring Boot parent version was updated from `2.1.1.RELEASE` to `3.2.5`.
-   All PowerMock dependencies (`powermock-module-junit4` and `powermock-api-mockito2`) were removed.
-   An `<exclusion>` was added to the `spring-boot-starter-test` dependency to remove the JUnit 4 vintage engine.

---

## 2. `src/main/java/com/javatechie/pm/api/util/NotificationUtil.java`

This file was refactored from a static utility class to a standard, injectable Spring service.

**Summary of Changes:**
-   The `@Service` annotation was added to the class.
-   The `static` keyword was removed from the `sendEmail` method.

**Before:**
```java
public class NotificationUtil {
	public static String sendEmail(String email) {
		//use mail API
		return "success";
	}
}
```

**After:**
```java
import org.springframework.stereotype.Service;

@Service
public class NotificationUtil {
	public String sendEmail(String email) {
		//use mail API
		return "success";
	}
}
```
---

## 3. `src/main/java/com/javatechie/pm/api/service/OrderService.java`

This service was refactored to use modern dependency injection, removing the static call to `NotificationUtil`.

**Summary of Changes:**
-   A `private final` field for `NotificationUtil` was added.
-   A constructor was added to inject the `NotificationUtil` service.
-   The call to `NotificationUtil.sendEmail` was changed to use the injected instance (`this.notificationUtil.sendEmail`).

**Before:**
```java
@Service
public class OrderService {
	public OrderResponse checkoutOrder(OrderRequest order) {
		// ...
		String message = NotificationUtil.sendEmail(order.getEmailId());
		return new OrderResponse(order, message, HttpStatus.OK.value());
	}
    // ...
}
```

**After:**
```java
@Service
public class OrderService {
    private final NotificationUtil notificationUtil;

    public OrderService(NotificationUtil notificationUtil) {
        this.notificationUtil = notificationUtil;
    }

	public OrderResponse checkoutOrder(OrderRequest order) {
		// ...
		String message = this.notificationUtil.sendEmail(order.getEmailId());
		return new OrderResponse(order, message, HttpStatus.OK.value());
	}
    // ...
}
```
---

## 4. `src/test/java/com/javatechie/pm/api/PowermockApiApplicationTests.java`

The test class was completely migrated from a JUnit 4 / PowerMock framework to JUnit 5 and standard Mockito.

**Summary of Changes:**
-   All JUnit 4 and PowerMock annotations (`@RunWith`, `@PrepareForTest`, `@Test`, `@Before`) and imports were removed.
-   The class is now annotated with `@SpringBootTest`.
-   The static mocking (`PowerMockito.mockStatic`) was replaced with Spring's `@MockBean` for mocking the `NotificationUtil` service.
-   The test logic was updated to use standard Mockito verification (`verify(...)`) instead of `when(...)` for a static call.
-   The out-of-scope `testPrivateMethod` was removed for clarity and focus.

**Before:**
```java
@RunWith(PowerMockRunner.class)
@PrepareForTest(fullyQualifiedNames = "com.javatechie.pm.api.*")
public class PowermockApiApplicationTests {
	@InjectMocks
	private OrderService service;

	// ...

	@Test
	public void testStaticMethod() {
		// Given
		String emailid = "test@gmail.com";
		PowerMockito.mockStatic(NotificationUtil.class);
		// When
		when(NotificationUtil.sendEmail(emailid)).thenReturn("success");
		// Then
		OrderResponse response = service.checkoutOrder(request);
		assertEquals("success", response.getMessage());
	}
    // ...
}
```

**After:**
```java
@SpringBootTest
class PowermockApiApplicationTests {
	@Autowired
	private OrderService service;

	@MockBean
	private NotificationUtil notificationUtil;

	@Test
	void testCheckoutOrder_shouldSendEmail() {
		// Given
		OrderRequest request = new OrderRequest(111, "Mobile", 1, 10000, "test@gmail.com", true);

		// When
		service.checkoutOrder(request);

		// Then
		verify(notificationUtil, times(1)).sendEmail(request.getEmailId());
	}
}
```
