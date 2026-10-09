# Core Components of Microservices Architecture

Microservices architecture is a software design approach where an application is split into small, independent services. Each service focuses on one specific business capability and can be developed, deployed, scaled, and maintained separately.

## 1. Microservices

A microservice is an independent application component that performs a specific task. For example, in a food ordering system, there may be separate services for users, food items, orders, payments, and delivery.

Each microservice usually has its own codebase, business logic, API, and sometimes its own database. This makes the system easier to maintain and scale because changes in one service do not always require changes in the whole application.

## 2. API Gateway

An API Gateway acts as the entry point for clients such as web apps, mobile apps, or external systems. Instead of calling each microservice directly, the client sends requests to the API Gateway.

The API Gateway routes requests to the correct service. It can also handle common tasks such as authentication, request logging, rate limiting, and response transformation.

## 3. Service Discovery

In a microservices system, services may run on different servers, ports, or containers. Their locations can change when services restart or scale.

Service discovery helps services find and communicate with each other dynamically. Tools such as Eureka, Consul, or Kubernetes service discovery are commonly used for this purpose.

## 4. Database Per Service

Microservices commonly follow the database-per-service pattern. This means each service owns and manages its own data.

For example, an order service may have an order database, while a payment service may have a payment database. This improves independence between services, but it also requires careful handling of data consistency across the system.

## 5. Inter-Service Communication

Microservices need to communicate with each other to complete business workflows. This communication can be synchronous or asynchronous.

Synchronous communication usually happens through REST APIs or gRPC, where one service waits for another service to respond. Asynchronous communication uses messaging systems such as Kafka, RabbitMQ, or AWS SQS, where services communicate through events or messages.

## 6. Configuration Management

Each microservice may need environment-specific settings such as database URLs, API keys, feature flags, or service endpoints.

Configuration management keeps these settings externalized instead of hardcoding them in the source code. Tools such as Spring Cloud Config, Kubernetes ConfigMaps, and environment variables are commonly used.

## 7. Load Balancing

Load balancing distributes incoming requests across multiple instances of a service. This improves performance, availability, and fault tolerance.

For example, if the order service has three running instances, a load balancer can spread requests across all three instead of sending all traffic to one instance.

## 8. Centralized Logging

Since microservices run as separate applications, checking logs service by service can become difficult.

Centralized logging collects logs from all services into one place. This helps developers monitor behavior, debug errors, and trace requests across multiple services. Common tools include the ELK Stack, Grafana Loki, and cloud logging services.

## 9. Monitoring and Observability

Monitoring helps track system health, performance, and failures. Observability goes further by helping teams understand why something is happening inside the system.

Important observability components include metrics, logs, and distributed tracing. Tools such as Prometheus, Grafana, Jaeger, Zipkin, and OpenTelemetry are commonly used.

## 10. Security

Security is important because microservices communicate over a network and expose APIs.

Common security components include authentication, authorization, HTTPS, API tokens, OAuth2, JWT, secrets management, and network policies. Security should be applied both at the API Gateway and between internal services.

## 11. Containerization

Microservices are often packaged into containers. A container includes the application and everything it needs to run.

Docker is commonly used for containerization. Containers make services easier to deploy consistently across different environments.

## 12. Orchestration

When many containers are running, orchestration is needed to manage them.

Container orchestration tools such as Kubernetes can start services, stop services, restart failed services, scale services, manage networking, and handle deployments.

## 13. CI/CD Pipeline

CI/CD stands for Continuous Integration and Continuous Deployment or Delivery.

A CI/CD pipeline automates building, testing, and deploying microservices. This allows teams to release changes faster and with fewer manual errors.

## 14. Fault Tolerance and Resilience

In microservices architecture, one service failure should not bring down the whole system.

Resilience patterns such as circuit breakers, retries, timeouts, fallback responses, and bulkheads help the system continue working even when some services are slow or unavailable.

## 15. Synchronous and Asynchronous Communication

Microservices communicate with each other in two main ways: synchronous communication and asynchronous communication.

### Synchronous Communication

Synchronous communication means one service sends a request to another service and waits for an immediate response.

Common examples include REST API calls, GraphQL calls, and gRPC calls.

Example:

```text
Order Service -> Payment Service -> Response
```

If the order service needs to confirm payment immediately before placing an order, it may call the payment service and wait for the result.

Use synchronous communication when:

- An immediate response is required
- The client or service cannot continue without the result
- The operation is simple and fast
- The services are strongly connected in that workflow

Examples:

- Checking if a user login is valid
- Getting product details by ID
- Verifying payment before confirming an order
- Fetching current account balance

Advantages:

- Simple to understand and implement
- Easy request-response flow
- Good for real-time operations

Disadvantages:

- The caller must wait for the response
- If the called service is slow, the caller also becomes slow
- If the called service is down, the request may fail
- Too many synchronous calls can tightly couple services

### Asynchronous Communication

Asynchronous communication means one service sends a message or event and does not wait for an immediate response.

This is usually done using message brokers or event streaming platforms such as Kafka, RabbitMQ, AWS SQS, or ActiveMQ.

Example:

```text
Order Service -> Message Queue -> Notification Service
```

If an order is placed, the order service can publish an `OrderPlaced` event. The notification service can later consume that event and send an SMS or email.

Use asynchronous communication when:

- An immediate response is not required
- The task can happen in the background
- You want to reduce dependency between services
- The system needs to handle high traffic
- Reliability and retry handling are important

Examples:

- Sending order confirmation emails
- Sending SMS notifications
- Updating analytics
- Processing reports
- Inventory updates after order placement
- Background payment settlement

Advantages:

- Services are loosely coupled
- Better performance for background tasks
- Better fault tolerance because messages can be retried
- Helps handle high traffic using queues

Disadvantages:

- More complex than direct API calls
- Requires message broker setup
- Debugging and tracing can be harder
- Data may become eventually consistent instead of immediately consistent

### Key Difference

| Feature | Synchronous Communication | Asynchronous Communication |
| --- | --- | --- |
| Response | Caller waits for response | Caller does not wait |
| Speed | Good for immediate operations | Good for background processing |
| Coupling | More tightly coupled | More loosely coupled |
| Failure impact | Failure can directly affect caller | Failure can be retried later |
| Common tools | REST, GraphQL, gRPC | Kafka, RabbitMQ, SQS |
| Best for | Real-time request-response flows | Events, queues, background tasks |

### Simple Rule

Use synchronous communication when you need the answer immediately.

Use asynchronous communication when the work can happen later or in the background.

## 16. Exception Handling Strategies

Exception handling is very important in microservices because failure can happen in many places: inside a service, during database access, while calling another service, or while processing messages from a queue.

Good exception handling makes the system easier to debug, safer for users, and more reliable in production.

### Common Exception Handling Strategies

### 1. Use Global Exception Handling

Each service should have a common place to handle exceptions instead of writing repeated `try-catch` blocks everywhere.

In Spring Boot, this is usually done using `@ControllerAdvice` and `@ExceptionHandler`.

Example:

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}
```

This keeps controllers clean and makes error responses consistent.

### 2. Return Proper HTTP Status Codes

APIs should return correct status codes so clients understand what happened.

Common examples:

| Status Code | Meaning | Example |
| --- | --- | --- |
| `200 OK` | Request successful | Food item found |
| `201 Created` | New resource created | Order placed |
| `400 Bad Request` | Invalid input | Missing quantity |
| `401 Unauthorized` | User not logged in | Missing token |
| `403 Forbidden` | User has no permission | Access denied |
| `404 Not Found` | Resource not found | Food item ID does not exist |
| `500 Internal Server Error` | Server-side failure | Unexpected exception |

### 3. Create Custom Exceptions

Custom exceptions make business errors easier to understand.

Examples:

```java
public class FoodItemNotFoundException extends RuntimeException {
    public FoodItemNotFoundException(Long id) {
        super("Food item not found with id: " + id);
    }
}
```

Common custom exceptions:

- `ResourceNotFoundException`
- `InvalidRequestException`
- `PaymentFailedException`
- `OrderCannotBeCancelledException`
- `ServiceUnavailableException`

### 4. Validate Input Early

Invalid data should be caught before business logic runs.

For example, an order request should check:

- Customer name is present
- Phone number is valid
- Food item ID exists
- Quantity is greater than zero
- Price is not negative

In Spring Boot, validation is commonly handled using annotations like `@NotNull`, `@NotBlank`, `@Min`, and `@Valid`.

### 5. Use Meaningful Error Responses

Error responses should be clear but should not expose sensitive technical details.

Good response:

```json
{
  "timestamp": "2026-08-31T10:30:00",
  "status": 404,
  "error": "Not Found",
  "message": "Food item not found with id: 5",
  "path": "/api/food-items/5"
}
```

Avoid sending full stack traces to the client because they can expose internal implementation details.

### 6. Handle Service-to-Service Failures

In microservices, one service may call another service. That call can fail because the other service is slow, unavailable, or returning an error.

Strategies include:

- Timeout: Do not wait forever for another service
- Retry: Try again for temporary failures
- Circuit breaker: Stop calling a failing service for some time
- Fallback: Return a default response when a dependency fails
- Bulkhead: Isolate failures so one broken service does not affect the whole system

Example:

```text
Order Service calls Payment Service.
If Payment Service is down, Order Service should return a clear error or mark payment as pending.
```

### 7. Log Exceptions Properly

Logs should contain enough information to debug issues.

Useful log details:

- Error message
- Request ID or correlation ID
- Service name
- API endpoint
- User or order ID if safe to log
- Stack trace for server-side logs

Do not log sensitive data such as passwords, tokens, card numbers, or private customer information.

### 8. Use Correlation IDs

A single user request may pass through many services. A correlation ID helps trace the same request across all services.

Example flow:

```text
API Gateway -> Order Service -> Payment Service -> Notification Service
```

All logs for this request should include the same correlation ID.

### 9. Handle Asynchronous Processing Errors

When using queues or event-driven communication, failures should be handled carefully.

Common strategies:

- Retry failed messages
- Move repeatedly failing messages to a dead letter queue
- Make consumers idempotent so duplicate messages do not cause wrong results
- Log failed events for investigation

### Simple Rule

Handle expected business errors with clear custom exceptions and proper status codes.

Handle unexpected system errors with logging, safe error responses, retries, timeouts, and monitoring.

## 17. Overall Application Flow End to End

The end-to-end flow explains how a request moves through the application from the user interface to the database and back to the user.

Below is an example using a food ordering application.

### Step 1. User Sends a Request

The user performs an action from a frontend application, mobile app, or API client.

Examples:

- View available food items
- Add a new food item
- Place an order
- Update order status

Example request:

```text
GET /api/food-items
```

### Step 2. Request Reaches the API Gateway

In a microservices architecture, the request first reaches the API Gateway.

The API Gateway may handle:

- Authentication
- Authorization
- Request routing
- Rate limiting
- Logging

Then it forwards the request to the correct microservice.

Example:

```text
Client -> API Gateway -> Food Item Service
```

### Step 3. Controller Receives the Request

Inside the service, the controller receives the HTTP request.

The controller is responsible for exposing API endpoints and accepting request data. It should not contain complex business logic.

Example:

```text
FoodItemController receives GET /api/food-items
```

### Step 4. Service Layer Handles Business Logic

The controller calls the service layer.

The service layer contains business rules such as:

- Check whether a food item exists
- Calculate order total
- Validate order quantity
- Check item availability
- Update order status

Example:

```text
FoodItemController -> FoodItemService
```

### Step 5. Repository Layer Talks to the Database

The service layer calls the repository layer when it needs to read or write data.

The repository layer hides database operations from the rest of the application.

Example:

```text
FoodItemService -> FoodItemRepository -> Database
```

### Step 6. Database Returns Data

The database returns the requested data to the repository.

For example, when fetching food items, the database returns rows from the food item table.

Example:

```text
Database -> FoodItemRepository -> FoodItemService
```

### Step 7. Response Goes Back to the Client

After processing the request, the service returns a response to the controller. The controller sends the HTTP response back to the client.

Example:

```text
Database -> Repository -> Service -> Controller -> API Gateway -> Client
```

### Example Flow: Placing an Order

```text
1. User selects food items and quantity.
2. Client sends order request to API Gateway.
3. API Gateway forwards request to Order Service.
4. OrderController receives the request.
5. OrderService validates customer details and food items.
6. OrderService gets food item prices from the database.
7. OrderService calculates total amount.
8. OrderService saves the order and order items.
9. Order Service publishes an OrderPlaced event.
10. Notification Service consumes the event and sends confirmation.
11. Client receives order confirmation response.
```

### End-to-End Flow Diagram

```text
Client
  |
  v
API Gateway
  |
  v
Controller
  |
  v
Service Layer
  |
  v
Repository Layer
  |
  v
Database
  |
  v
Response back to Client
```

### In a Simple Spring Boot CRUD Application

If the application is not split into multiple microservices yet, the flow is simpler:

```text
Client -> Controller -> Service -> Repository -> Database
```

For your CRUD application, a request for food items follows this kind of flow:

```text
Client -> FoodItemController -> FoodItemService -> FoodItemRepository -> H2 Database
```

The response comes back in the reverse direction:

```text
H2 Database -> FoodItemRepository -> FoodItemService -> FoodItemController -> Client
```

## 18. Security: JWT Flow in This Project

JWT stands for JSON Web Token. It is commonly used to secure REST APIs by proving that a user is already authenticated.

### Current Status in This Project

In the current CRUD project, JWT security is not implemented yet.

The project currently has:

- Food item APIs
- Order APIs
- H2 database configuration
- JPA repositories
- Service and controller layers

The project currently does not have:

- Spring Security dependency
- Login or registration API
- User entity or user table
- Password encryption
- JWT generation logic
- JWT validation filter
- Protected API endpoints

So, in the current project, JWT is not generated, not used, and not stored anywhere.

### How JWT Flow Would Work If Added

If JWT security is added to this project, the flow would usually work like this:

```text
User Login -> Backend validates user -> JWT generated -> JWT returned to client
Client stores JWT -> Client sends JWT in future requests
Backend validates JWT -> Request is allowed or rejected
```

### Step 1. User Sends Login Request

The user sends login credentials from the frontend or API client.

Example:

```http
POST /api/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "password123"
}
```

### Step 2. Backend Validates Credentials

The backend checks whether the username and password are correct.

In a Spring Boot project, this is usually done using:

- `AuthController`
- `AuthService`
- `UserRepository`
- `PasswordEncoder`
- `AuthenticationManager`

The password stored in the database should not be plain text. It should be encrypted using a password encoder such as BCrypt.

### Step 3. JWT Is Generated

If the credentials are valid, the backend generates a JWT.

JWT generation is usually handled by a class such as `JwtService` or `JwtUtil`.

A JWT usually contains:

- Subject: username or user ID
- Claims: user role or permissions
- Issued time
- Expiry time
- Signature

Example JWT payload:

```json
{
  "sub": "admin",
  "role": "ADMIN",
  "iat": 1788143400,
  "exp": 1788147000
}
```

The JWT is signed using a secret key. The secret key stays on the backend server and should never be sent to the frontend.

Example concept:

```text
JWT = header + payload + signature
```

The signature proves that the token was created by the backend and was not modified by the client.

### Step 4. JWT Is Sent Back to the Client

After successful login, the backend returns the token to the client.

Example response:

```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9..."
}
```

### Step 5. JWT Is Stored on the Client Side

JWT is usually stored on the client side, not in the backend database.

Common storage options:

| Storage Location  |               Description |
| --- | --- |
| HttpOnly secure cookie |      Safer option for browser-based apps because JavaScript cannot read it |
| Memory |                      Safer against persistent theft, but token is lost on page refresh |
| Session storage | Stored only for the browser tab session |
| Local storage | Easy to use, but more vulnerable if the site has an XSS issue |

For browser applications, an HttpOnly secure cookie is usually preferred.

For simple learning projects or Postman testing, the JWT can be copied manually and sent in the `Authorization` header.

### Step 6. Client Sends JWT with Future Requests

For protected APIs, the client sends the JWT with every request.

Example:

```http
GET /api/food-items
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

The word `Bearer` means the request is carrying a token that proves authentication.

### Step 7. Backend Validates JWT

Before the request reaches the controller, a JWT filter checks the token.

In Spring Boot, this is usually done using a custom filter such as:

```java
JwtAuthenticationFilter extends OncePerRequestFilter
```

The filter checks:

- Is the token present?
- Is the token format valid?
- Is the signature valid?
- Has the token expired?
- Does the user still exist?
- What roles or permissions does the user have?

If the token is valid, Spring Security stores the authenticated user in the `SecurityContext`.

Example flow:

```text
Request -> JWT Filter -> SecurityContext -> Controller -> Service -> Repository -> Database
```

If the token is missing or invalid, the backend returns an error such as:

```text
401 Unauthorized
```

### Step 8. Protected APIs Are Accessed

After JWT validation, protected APIs can be accessed based on user role.

Example:

```text
ADMIN can add, update, and delete food items.
CUSTOMER can view food items and place orders.
```

Possible endpoint protection:

| Endpoint | Access |
| --- | --- |
| `POST /api/auth/login` | Public |
| `GET /api/food-items` | Public or authenticated users |
| `POST /api/food-items` | Admin only |
| `PUT /api/food-items/{id}` | Admin only |
| `DELETE /api/food-items/{id}` | Admin only |
| `POST /api/orders` | Authenticated customer |
| `GET /api/orders` | Admin only |

### Where JWT Is Stored

JWT access tokens are usually not stored in the backend database.

They are commonly stored:

- In the frontend as an HttpOnly secure cookie
- In memory for short-lived sessions
- In session storage or local storage for simple client apps
- In Postman manually during API testing

The backend stores only the secret key used to sign and verify JWTs.

In more advanced systems, refresh tokens may be stored in the database so users can get a new access token without logging in again.

### Simple JWT Flow for This CRUD App

If JWT is added to your current CRUD application, the flow can look like this:

```text
1. User logs in using /api/auth/login.
2. AuthController receives username and password.
3. AuthService validates user credentials.
4. JwtService generates a signed JWT.
5. Backend returns JWT to the client.
6. Client stores JWT.
7. Client sends JWT in Authorization header for protected APIs.
8. JwtAuthenticationFilter validates the token.
9. If valid, request reaches FoodItemController or OrderController.
10. Controller calls Service.
11. Service calls Repository.
12. Repository talks to H2 database.
13. Response is returned to the client.
```

### Important Point

JWT is not a replacement for database security or password encryption.

JWT only proves that the user has already logged in. Passwords should still be encrypted, APIs should still check roles, and sensitive endpoints should still be protected.

## 19. Schema Design: Schemas Used in This Project

Schema design means deciding how application data is stored in database tables and how those tables are related to each other.

### Current Database Used in This Project

This project uses an H2 database.

The database configuration is present in `application.properties`:

```properties
spring.datasource.url=jdbc:h2:mem:fooddb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=update
```

This means:

- The database name is `fooddb`
- The database is currently in-memory
- Data is lost when the application stops
- Hibernate creates or updates tables automatically
- Tables are generated from the Java `@Entity` classes

There is no separate `schema.sql` file in the project. The schema comes from the JPA entity classes.

### Entities Used for Schema Design

The project has three main entities:

- `FoodItem`
- `CustomerOrder`
- `OrderItem`

Each entity becomes a database table.

### 1. Food Item Table

The `FoodItem` entity stores menu item details.

Java class:

```text
FoodItem.java
```

Expected table:

```text
food_item
```

In the H2 console, it may appear as:

```text
FOOD_ITEM
```

Columns:

| Column | Type | Purpose |
| --- | --- | --- |
| `id` | Long / primary key | Unique food item ID |
| `name` | String | Food item name |
| `description` | String | Food item description |
| `price` | BigDecimal | Food item price |
| `available` | boolean | Whether the item is available |

Example data inserted by `DataSeeder`:

```text
Margherita Pizza
Veg Burger
Paneer Biryani
```

### 2. Customer Order Table

The `CustomerOrder` entity stores order-level information.

Java class:

```text
CustomerOrder.java
```

Expected table:

```text
customer_order
```

In the H2 console, it may appear as:

```text
CUSTOMER_ORDER
```

Columns:

| Column | Type | Purpose |
| --- | --- | --- |
| `id` | Long / primary key | Unique order ID |
| `customer_name` | String | Name of customer |
| `phone` | String | Customer phone number |
| `address` | String | Delivery address |
| `total_amount` | BigDecimal | Total order price |
| `created_at` | LocalDateTime | Order creation time |
| `status` | String enum | Current order status |

The `status` field uses the `OrderStatus` enum.

Possible values:

```text
PLACED
PREPARING
OUT_FOR_DELIVERY
DELIVERED
CANCELLED
```

Because the enum uses `@Enumerated(EnumType.STRING)`, the database stores the enum name as text.

### 3. Order Item Table

The `OrderItem` entity stores individual items inside an order.

Java class:

```text
OrderItem.java
```

Expected table:

```text
order_item
```

In the H2 console, it may appear as:

```text
ORDER_ITEM
```

Columns:

| Column | Type | Purpose |
| --- | --- | --- |
| `id` | Long / primary key | Unique order item ID |
| `order_id` | Foreign key | Links item to a customer order |
| `food_item_id` | Foreign key | Links item to a food item |
| `quantity` | int | Quantity ordered |
| `item_total` | BigDecimal | Price multiplied by quantity |

### Table Relationships

The project uses these relationships:

```text
CustomerOrder 1 -> many OrderItem
FoodItem      1 -> many OrderItem
```

Meaning:

- One customer order can contain many order items
- Each order item belongs to one customer order
- Each order item refers to one food item
- One food item can appear in many different orders

Relationship flow:

```text
customer_order
    |
    | one-to-many
    v
order_item
    |
    | many-to-one
    v
food_item
```

### Why OrderItem Is Needed

The `OrderItem` table is needed because one order can contain multiple food items.

Example:

```text
Order #1
- 2 x Veg Burger
- 1 x Paneer Biryani
```

This cannot be stored cleanly in only the `customer_order` table. So `order_item` acts as the child table that stores each selected food item and quantity.

### Approximate SQL Schema

Hibernate creates the schema automatically, but conceptually it looks like this:

```sql
CREATE TABLE food_item (
    id BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    name VARCHAR(255),
    description VARCHAR(255),
    price DECIMAL,
    available BOOLEAN
);

CREATE TABLE customer_order (
    id BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    customer_name VARCHAR(255),
    phone VARCHAR(255),
    address VARCHAR(255),
    total_amount DECIMAL,
    created_at TIMESTAMP,
    status VARCHAR(255)
);

CREATE TABLE order_item (
    id BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    order_id BIGINT,
    food_item_id BIGINT,
    quantity INTEGER,
    item_total DECIMAL,
    FOREIGN KEY (order_id) REFERENCES customer_order(id),
    FOREIGN KEY (food_item_id) REFERENCES food_item(id)
);
```

The exact SQL may differ slightly because Hibernate generates database-specific SQL for H2.

### Development vs Production Schema

For development, this setting is acceptable:

```properties
spring.jpa.hibernate.ddl-auto=update
```

It allows Hibernate to create or update tables automatically.

For production, it is better to use schema migration tools such as:

- Flyway
- Liquibase

In production, a safer setting is usually:

```properties
spring.jpa.hibernate.ddl-auto=validate
```

This makes the application validate that the database schema matches the entity classes instead of modifying tables automatically.

## 20. How a Spring Boot Application Gets Started

A Spring Boot application starts from the class that contains the `main` method.

In this project, the starting point is:

```text
CrudApplication.java
```

The main class looks like this:

```java
@SpringBootApplication
public class CrudApplication {

    public static void main(String[] args) {
        SpringApplication.run(CrudApplication.class, args);
    }
}
```

### Step 1. Main Method Runs

When the application starts, Java first executes the `main` method.

Example command:

```powershell
.\mvnw.cmd spring-boot:run
```

This starts the Java application and calls:

```java
SpringApplication.run(CrudApplication.class, args);
```

### Step 2. SpringApplication.run Starts Spring Boot

`SpringApplication.run()` starts the Spring Boot application.

It does several important things:

- Creates the Spring application context
- Reads configuration from `application.properties`
- Starts auto-configuration
- Scans for Spring components
- Creates required beans
- Starts the embedded web server
- Runs startup logic such as `CommandLineRunner`

### Step 3. @SpringBootApplication Enables Main Features

The `@SpringBootApplication` annotation is a combination of important Spring annotations.

It includes:

```text
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

Meaning:

- `@Configuration` tells Spring this class can provide configuration
- `@EnableAutoConfiguration` tells Spring Boot to configure things automatically based on dependencies
- `@ComponentScan` tells Spring to scan the current package and sub-packages for components

Because your main class is in this package:

```text
com.crudops.crud
```

Spring scans classes inside:

```text
com.crudops.crud.controller
com.crudops.crud.service
com.crudops.crud.repository
com.crudops.crud.model
```

### Step 4. Spring Reads application.properties

Spring Boot reads this file:

```text
src/main/resources/application.properties
```

In this project, it finds settings such as:

```properties
spring.datasource.url=jdbc:h2:mem:fooddb
spring.jpa.hibernate.ddl-auto=update
spring.h2.console.enabled=true
```

These settings tell Spring Boot:

- Which database to use
- How JPA/Hibernate should create tables
- Whether H2 console should be enabled
- What the application name is

### Step 5. Auto-Configuration Sets Up Framework Features

Spring Boot checks the dependencies in `pom.xml`.

For this project, important dependencies include:

- Spring Web MVC
- Spring Data JPA
- H2 database

Based on these dependencies, Spring Boot automatically configures:

- Embedded web server
- REST controller support
- JSON request and response handling
- DataSource for H2 database
- EntityManager for JPA
- Hibernate
- Spring Data repositories

### Step 6. Component Scanning Creates Beans

Spring scans the project and creates objects for classes marked with Spring annotations.

Examples:

| Annotation | Purpose |
| --- | --- |
| `@RestController` | Creates API controller beans |
| `@Service` | Creates service layer beans |
| `@Repository` | Creates repository beans |
| `@Component` | Creates general Spring-managed beans |
| `@Entity` | Marks a class as a database entity |

In your project, Spring creates beans for classes such as:

- `FoodItemController`
- `OrderController`
- `FoodItemService`
- `OrderService`
- `DataSeeder`
- `FoodItemRepository`
- `CustomerOrderRepository`

### Step 7. Dependency Injection Connects Classes

After creating beans, Spring injects dependencies into constructors.

Example:

```java
public FoodItemService(FoodItemRepository foodItemRepository) {
    this.foodItemRepository = foodItemRepository;
}
```

This means `FoodItemService` does not manually create `FoodItemRepository`. Spring provides it automatically.

The flow becomes:

```text
Controller -> Service -> Repository
```

### Step 8. Hibernate Creates or Updates Tables

Because the project has JPA and H2 configured, Hibernate checks the entity classes:

- `FoodItem`
- `CustomerOrder`
- `OrderItem`

Then it creates or updates the database tables because this setting is enabled:

```properties
spring.jpa.hibernate.ddl-auto=update
```

### Step 9. Embedded Web Server Starts

Spring Boot starts an embedded web server so the application can receive HTTP requests.

By default, the app runs on:

```text
http://localhost:8080
```

After this, APIs such as food item and order endpoints can receive requests.

### Step 10. CommandLineRunner Runs

After the application context is ready, Spring Boot runs any class that implements `CommandLineRunner`.

In this project:

```java
public class DataSeeder implements CommandLineRunner
```

So Spring Boot calls:

```java
run(String... args)
```

The `DataSeeder` checks whether food items already exist. If the table is empty, it inserts sample food items.

### Complete Startup Flow

```text
1. User runs the application.
2. Java executes the main method in CrudApplication.
3. SpringApplication.run() starts Spring Boot.
4. Spring creates the application context.
5. application.properties is loaded.
6. Auto-configuration sets up Web MVC, JPA, Hibernate, and H2.
7. Component scanning finds controllers, services, repositories, and components.
8. Spring creates beans and injects dependencies.
9. Hibernate creates or updates database tables.
10. Embedded server starts on localhost:8080.
11. DataSeeder runs and inserts sample food items if the table is empty.
12. The application is ready to handle API requests.
```

### Startup Flow in This Project

```text
CrudApplication
  |
  v
SpringApplication.run()
  |
  v
Load application.properties
  |
  v
Configure H2 + JPA + Web MVC
  |
  v
Scan controllers, services, repositories, entities
  |
  v
Create database tables
  |
  v
Run DataSeeder
  |
  v
Application ready on localhost:8080
```

## 21. JPA

JPA stands for Java Persistence API. In newer Spring Boot versions, it is part of Jakarta Persistence.

JPA is used to map Java classes to database tables. Instead of writing SQL manually for every basic database operation, we create Java classes called entities, and JPA handles the connection between those Java objects and database tables.

### Why JPA Is Used

JPA helps developers perform database operations using Java code.

Without JPA, we may need to write SQL manually:

```sql
SELECT * FROM food_item;
INSERT INTO food_item (...) VALUES (...);
UPDATE food_item SET ... WHERE id = ...;
DELETE FROM food_item WHERE id = ...;
```

With JPA, we can use repository methods:

```java
foodItemRepository.findAll();
foodItemRepository.findById(id);
foodItemRepository.save(foodItem);
foodItemRepository.deleteById(id);
```

JPA converts these Java method calls into SQL queries internally.


Spring Data JPA provides repository interfaces, JPA provides the persistence rules, Hibernate implements those rules, and H2 is the database used by the application.

### Entity Classes

An entity is a Java class that represents a database table.

In this project, these classes are JPA entities:

- `FoodItem`
- `CustomerOrder`
- `OrderItem`

Example:

```java
@Entity
public class FoodItem {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String description;
    private BigDecimal price;
    private boolean available;
}
```

Because `FoodItem` has `@Entity`, JPA creates a table for it.

Mapping:

```text
FoodItem class -> food_item table
id field       -> id column
name field     -> name column
description    -> description column
price          -> price column
available      -> available column
```

### Important JPA Annotations

| Annotation | Purpose |
| --- | --- |
| `@Entity` | Marks a Java class as a database table |
| `@Id` | Marks the primary key field |
| `@GeneratedValue` | Automatically generates primary key values |
| `@OneToMany` | Defines one-to-many relationship |
| `@ManyToOne` | Defines many-to-one relationship |
| `@JoinColumn` | Defines the foreign key column |
| `@Enumerated` | Stores enum values in the database |

### Repositories in JPA

This project uses repository interfaces to interact with the database.

Example:

```java
public interface FoodItemRepository extends JpaRepository<FoodItem, Long> {
    List<FoodItem> findByAvailableTrue();
}
```

`JpaRepository<FoodItem, Long>` means:

- `FoodItem` is the entity type
- `Long` is the primary key type

By extending `JpaRepository`, the project automatically gets common CRUD methods:

- `findAll()`
- `findById(id)`
- `save(entity)`
- `deleteById(id)`
- `existsById(id)`
- `count()`

### Hibernate and JPA

JPA is only a specification. It defines how persistence should work, but it does not directly execute database operations.

Hibernate is the actual implementation used behind the scenes.

In simple terms:

```text
JPA = rules and annotations
Hibernate = actual implementation
```
So when this project calls:

```java
foodItemRepository.save(foodItem);
```

Hibernate creates and runs the required SQL query.

### Table Creation

This project has the following setting:

```properties
spring.jpa.hibernate.ddl-auto=update
```

This tells Hibernate to automatically create or update database tables based on the entity classes.

That is why the project does not need a manual `schema.sql` file for development.

### Simple Definition

JPA is used to connect Java objects with database tables and perform database operations using Java code instead of writing SQL manually.

## 22. Lazy Loading and Eager Loading

Lazy loading and eager loading decide when related data should be loaded from the database.

They are mainly used with entity relationships such as:

- `@OneToMany`
- `@ManyToOne`
- `@OneToOne`
- `@ManyToMany`

### Lazy Loading

Lazy loading means related data is not loaded immediately. It is loaded only when it is actually needed.

Example from this project:

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "order_id")
@JsonIgnore
private CustomerOrder order;
```

Here, `OrderItem` has a relationship with `CustomerOrder`.

Because it uses `FetchType.LAZY`, when an `OrderItem` is loaded, the related `CustomerOrder` is not loaded immediately. Hibernate loads the order only if the code calls:

```java
orderItem.getOrder();
```

Lazy loading is useful when related data is large or not always needed.

Use lazy loading when:

- The related data is not always required
- You want better performance by avoiding unnecessary queries
- The relationship contains many records
- You want to load data only when needed

Example:

```text
OrderItem -> CustomerOrder
```

If the API response only needs item details, quantity, and price, there is no need to load the full order object every time.

Advantages:

- Improves performance when related data is not needed
- Reduces unnecessary database loading
- Useful for large relationships

Disadvantages:

- Can cause extra queries later
- Can cause `LazyInitializationException` if accessed outside an active Hibernate session
- Can be harder to debug because data loads later, not immediately

### Eager Loading

Eager loading means related data is loaded immediately along with the main entity.

Example from this project:

```java
@ManyToOne(fetch = FetchType.EAGER)
private FoodItem foodItem;
```

Here, when an `OrderItem` is loaded, the related `FoodItem` is also loaded immediately.

This makes sense because an order item usually needs food item details such as name, description, and price.

Use eager loading when:

- The related data is always required
- The relationship is small
- You want to avoid separate database calls later
- The API response needs both main and related entity data

Example:

```text
OrderItem -> FoodItem
```

If the user views an order, they should usually see which food item was ordered. So loading the `FoodItem` with the `OrderItem` is useful.

Advantages:

- Related data is available immediately
- Simple to use in API responses
- Avoids lazy loading errors for that relationship

Disadvantages:

- Can load unnecessary data
- Can reduce performance if many relationships are eager
- Can create large and complex SQL queries

### Key Difference

| Feature         | Lazy Loading        | Eager Loading |

| When data loads | Only when accessed | Immediately with parent entity |
| Performance | Better when related data is not always needed | Better when related data is always needed |
| Query behavior | May run extra queries later | Loads upfront |
| Risk | Lazy loading exception | Unnecessary heavy data loading |
| Best for | Large or optional relationships | Small and required relationships |

### Lazy and Eager Loading in This Project

In this project, `OrderItem` uses both lazy and eager loading:

```java
@ManyToOne(fetch = FetchType.LAZY)
private CustomerOrder order;

@ManyToOne(fetch = FetchType.EAGER)
private FoodItem foodItem;
```

Meaning:

- `CustomerOrder` is loaded lazily because every item does not always need the full order object
- `FoodItem` is loaded eagerly because item details are usually needed when showing an order item

### Simple Rule

Use lazy loading when the related data may not be needed.

Use eager loading when the related data is almost always needed.

In real projects, prefer lazy loading by default and load related data explicitly when required.

## Summary

The core components of microservices architecture include independent services, APIs, service discovery, separate databases, communication mechanisms, configuration management, logging, monitoring, security, containers, orchestration, CI/CD, and resilience patterns.

Together, these components help build applications that are scalable, maintainable, independently deployable, and fault tolerant.
