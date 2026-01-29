---
date: 2026-01-24
tags:
  - jpa
---


[Spring Data JPA](https://www.javaguides.net/p/spring-data-jpa-tutorial.html) **simplifies database interactions** by reducing boilerplate code and enabling **seamless ORM (Object-Relational Mapping)** with **Hibernate**.

However, improper usage can lead to **performance issues, unnecessary queries, and resource leaks**.

In this article, we will discuss **12 best practices** to optimize your **Spring Data JPA applications** for performance, scalability, and maintainability. 🚀

Read this article for free on my blog: [Best Practices for Spring Data JPA — The ULTIMATE Guide.](https://www.javaguides.net/2025/02/top-10-best-practices-for-spring-data.html)

> My Udemy Course: [Master Spring Data JPA with Hibernate](https://www.udemy.com/course/master-spring-data-jpa-with-hibernate/?referralCode=83ABBA51A8BB3E692F8F) Learn Spring Data JPA Features and Build Domain Model Relationships for E-Commerce Projects Using Hibernate ORM.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:1400/1*9_YfJwpwvawsytDc7dogFg.png)

Credit: [https://www.javaguides.net/2025/02/top-10-best-practices-for-spring-data.html](https://www.javaguides.net/2025/02/top-10-best-practices-for-spring-data.html)

## 1️⃣ Use DTOs (Data Transfer Objects) Instead of Entities in Responses

Instead of returning JPA entities directly from the repository, a **better approach** is to **convert the entity to a DTO in the service layer** and return the DTO from the REST API. This ensures:  
✅ **Decoupling** between the persistence layer and the API layer.  
✅ **Security** by preventing direct exposure of sensitive entity fields.  
✅ **Flexibility** to customize API responses without modifying the database schema.

### 🔹 Example: Best Practice for Using DTOs in a Spring Boot Application

✅ Step 1: Define DTO Class (Only Required Fields)

public record UserDTO(Long id, String name, String email) {}

✅ Step 2: Convert Entity to DTO in the Service Layer

@Service  
public class UserService {  
  
    private final UserRepository userRepository;  
  
    public UserService(UserRepository userRepository) {  
        this.userRepository = userRepository;  
    }  
  
    public UserDTO getUserById(Long id) {  
        User user = userRepository.findById(id)  
                .orElseThrow(() -> new RuntimeException("User not found"));  
        return new UserDTO(user.getId(), user.getName(), user.getEmail());  
    }  
}

✅ Step 3: Return DTO from REST API (Controller Layer)

@RestController  
@RequestMapping("/api/users")  
public class UserController {  
  
    private final UserService userService;  
  
    public UserController(UserService userService) {  
        this.userService = userService;  
    }  
  
    @GetMapping("/{id}")  
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {  
        return ResponseEntity.ok(userService.getUserById(id));  
    }  
}

💡 **Pro Tip:** Use **MapStruct** or **ModelMapper** for automated DTO conversion instead of manual mapping.

### Alternative: Use Projection in the Repository (For Read-Only Queries)

If you **only need to fetch DTOs without modifying entities**, you can use Spring **projection** in the repository:

public interface UserRepository extends JpaRepository<User, Long> {  
    @Query("SELECT new com.example.dto.UserDTO(u.id, u.name, u.email) FROM User u WHERE u.id = :id")  
    UserDTO findUserById(Long id);  
}

Use this approach **only for read-only queries** to avoid unnecessary entity fetching when updating data.

### Summary

✅ **Best Approach**: Convert **Entity to DTO in Service Layer** and return DTO from the API.  
✅ **Alternative for Read-Only Queries**: Use **DTO projection in repository**.  
✅ **Pro Tip**: Use **MapStruct** for clean, maintainable DTO conversions.

Following this approach will make your Spring Boot application **secure, scalable, and maintainable**! 🚀

## 2️⃣ Use `@Modifying` and `@Transactional` for Bulk Updates & Deletes

Spring Data JPA **does not automatically apply transactions** to custom `UPDATE` and `DELETE` queries. Without proper transaction management, updates may **fail or be inconsistent**.

### ✅ Best Practice: Use `@Modifying` with `@Transactional` for Bulk Updates & Deletes

This ensures that bulk operations:  
✔ Run **efficiently** in a **single transaction**  
✔ Avoid **unexpected behavior** due to lack of transaction handling  
✔ Prevent **auto-flushing issues** with JPA

### 🔹 Example: Using `@Modifying` for Bulk Updates

@Transactional  
@Modifying  
@Query("UPDATE User u SET u.status = 'INACTIVE' WHERE u.lastLogin < :date")  
int deactivateInactiveUsers(@Param("date") LocalDate date);

✅ **Key Benefits:**

- Executes the bulk update in **one transaction**, ensuring data consistency.
- Improves **performance** by avoiding loading and updating entities one by one.
- Uses **JPQL** instead of retrieving & modifying entities in memory.

### 🔹 Example: Using `@Modifying` for Bulk Deletes

@Transactional  
@Modifying  
@Query("DELETE FROM User u WHERE u.status = 'INACTIVE'")  
int deleteInactiveUsers();

💡 **Why Use This?**  
✔ Deletes inactive users **in bulk**, reducing unnecessary entity loading.  
✔ Runs efficiently in a **single database operation**.

### 🚀 Additional Best Practices for Bulk Operations

1️⃣ **Always use** `**@Transactional**` to ensure data consistency.  
2️⃣ **Use** `**@Modifying(clearAutomatically = true)**` if you need the persistence context to be cleared after execution.  
3️⃣ **Avoid** `**@Transactional**` **in the Controller Layer** – Keep transaction logic in the **Service Layer**.

**💡 Pro Tip:** If the bulk operation is complex, consider using **native queries** or **batch processing** for even better performance. 🚀

## 3️⃣ Use `fetch = FetchType.LAZY` for Better Performance

By default, JPA **fetches relationships eagerly**, which can **cause performance issues**.

✅ **Best Practice:** Always use **lazy loading (**`**FetchType.LAZY**`**)** unless eager fetching is explicitly needed.

🔹 **Example: Using Lazy Loading to Improve Performance**

@Entity  
public class Order {  
    @ManyToOne(fetch = FetchType.LAZY)  
    private Customer customer; // ✅ Loads only when needed  
}

💡 **Lazy loading reduces unnecessary data fetching.**

## 4️⃣ Use `@EntityGraph` for Fetching Related Data Efficiently

Spring Data JPA uses **lazy loading** by default, which can lead to **N+1 query issues** — where a single query retrieves the main entity, but each related entity triggers additional queries.

### ✅ Best Practice: Use `@EntityGraph` to Fetch Related Data Efficiently

`@EntityGraph` allows **loading related entities in a single optimized query**, avoiding multiple database hits and improving performance.

### 🔹 Example: Using `@EntityGraph` to Fetch Related Data Efficiently

public interface CustomerRepository extends JpaRepository<Customer, Long> {  
  
    @EntityGraph(attributePaths = {"orders"})  
    @Query("SELECT c FROM Customer c WHERE c.id = :id")  
    Customer findCustomerWithOrders(@Param("id") Long id);  
}

✅ **Why This Works?**  
✔ **Loads** `**Customer**` **and related** `**orders**` **in a single query**, avoiding multiple SELECT statements.  
✔ **Prevents N+1 query issues**, improving query efficiency.  
✔ **Ensures optimal performance** for fetching relationships.

### 🔹 Alternative: Fetching Multiple Relationships

If multiple related entities need to be loaded together, specify them in `attributePaths`:

@EntityGraph(attributePaths = {"orders", "addresses"})  
@Query("SELECT c FROM Customer c WHERE c.id = :id")  
Customer findCustomerWithOrdersAndAddresses(@Param("id") Long id);

💡 **Loads** `**Customer**`**,** `**orders**`**, and** `**addresses**` **in a single optimized query!**

### 🚀 Best Practices for `@EntityGraph`

1️⃣ **Use** `**attributePaths**` **for specifying relationships to load eagerly.**  
2️⃣ **Apply** `**@EntityGraph**` **only when necessary** to avoid loading excess data.  
3️⃣ **Combine with pagination (**`**Pageable**`**)** for better performance on large datasets.

**💡 Pro Tip:** If you need **dynamic fetching**, use **fetch joins (**`**JOIN FETCH**`**) in JPQL queries** for more control over query execution. 🚀

## 5️⃣ Use Pagination for Large Datasets

Fetching large datasets **without pagination** can overload memory and degrade performance, leading to **OutOfMemoryErrors** or slow response times.

### ✅ Best Practice: Use Spring Data JPA Pagination to Fetch Data in Chunks

Pagination improves efficiency by **retrieving only a subset of records at a time**, reducing memory usage and query execution time.

### 🔹 Example: Implementing Pagination in Repository

public interface UserRepository extends JpaRepository<User, Long> {  
    Page<User> findByStatus(String status, Pageable pageable);  
}

✅ **Why This Works?**  
✔ Returns **a paginated list** instead of loading all users at once.  
✔ Supports **sorting, filtering, and dynamic page sizes**.

### 🔹 Service Layer: Implement Pagination & Sorting

A **service layer** helps keep business logic separate from the controller.

@Service  
public class UserService {  
    private final UserRepository userRepository;  
  
    public UserService(UserRepository userRepository) {  
        this.userRepository = userRepository;  
    }  
  
    public Page<User> getActiveUsers(int page, int size, String sortBy, String sortDir) {  
        Sort sort = Sort.by(Sort.Direction.fromString(sortDir), sortBy);  
        Pageable pageable = PageRequest.of(page, size, sort);  
        return userRepository.findByStatus("ACTIVE", pageable);  
    }  
}

✅ **Why Use a Service Layer?**  
✔ Keeps **business logic separate** from the controller.  
✔ Supports **sorting and pagination dynamically**.

### 🔹 Controller: Implement Paginated API with Sorting

@RestController  
@RequestMapping("/api/users")  
public class UserController {  
  
    private final UserService userService;  
  
    public UserController(UserService userService) {  
        this.userService = userService;  
    }  
  
    @GetMapping  
    public ResponseEntity<Page<User>> getUsers(  
            @RequestParam(defaultValue = "0") int page,  
            @RequestParam(defaultValue = "10") int size,  
            @RequestParam(defaultValue = "id") String sortBy,  
            @RequestParam(defaultValue = "asc") String sortDir) {  
  
        Page<User> users = userService.getActiveUsers(page, size, sortBy, sortDir);  
        return ResponseEntity.ok(users);  
    }  
}

✅ **Why This Works?**  
✔ Supports **dynamic pagination & sorting** via query parameters.  
✔ Returns **structured responses**, making API consumption easier.  
✔ Uses **best practices** by handling logic in the **service layer**.

### 🚀 Additional Best Practices for Pagination

1️⃣ **Always use pagination when dealing with large datasets.**  
2️⃣ **Allow dynamic sorting and filtering** via query parameters.  
3️⃣ **Use DTOs for API responses** instead of exposing entities.  
4️⃣ **Use caching (e.g., Redis) for frequently accessed pages**.

**💡 Pro Tip:** If dealing with **millions of records**, consider **keyset pagination** (using indexed queries) for faster performance! 🚀

## 6️⃣ Prefer `@Query` for Complex Queries Instead of Query Methods

Spring Data JPA **query methods** (`findByFieldName()`) work well for **simple queries**, but they become **long, unreadable, and inefficient** for **complex queries** involving multiple conditions, joins, or aggregations.

### ✅ Best Practice: Use JPQL or Native Queries for Better Control

✅ Using JPQL for Complex Queries

JPQL (**Java Persistence Query Language**) is recommended when querying entities in a **database-independent way**.

@Query("SELECT u FROM User u WHERE u.status = 'ACTIVE' AND u.role = :role")  
List<User> findActiveUsersByRole(@Param("role") String role);

✔ **Database-agnostic** (works across different databases).  
✔ Supports **object-oriented queries** using entity attributes.

✅ Using Native Query for Performance Optimization

If a query requires **database-specific optimizations** (e.g., using indexes or custom SQL functions), a **native query** is more efficient.

@Query(value = "SELECT * FROM users u WHERE u.status = 'ACTIVE' AND u.role = :role", nativeQuery = true)  
List<User> findActiveUsersByRoleNative(@Param("role") String role);

✔ Best for **raw SQL performance optimizations** (index hints, stored procedures).  
✔ Required when working with **non-mapped database columns**.

### 🚀 Best Practices

✅ **Use JPQL for complex queries involving multiple conditions & joins.**  
✅ **Use Native Queries only when needed for performance tuning.**  
✅ **Always use named parameters (**`**:paramName**`**)** instead of positional parameters (`?1`) for readability.  
✅ **Optimize indexes and batch processing** for better query performance.

💡 **Pro Tip:** Use **Spring Data Projections** for fetching only required fields instead of entire entities! 🚀

## 7️⃣ Use `Specification` for Dynamic Filtering

For **flexible search filters**, using multiple `@Query` methods **is inefficient**.

✅ **Best Practice:** Use `Specification` for **dynamic search criteria**.

🔹 **Example: Using** `**Specification**` **for Dynamic Queries**

public class UserSpecification {  
    public static Specification<User> hasRole(String role) {  
        return (root, query, builder) -> builder.equal(root.get("role"), role);  
    }  
  
    public static Specification<User> isActive() {  
        return (root, query, builder) -> builder.equal(root.get("status"), "ACTIVE");  
    }  
}

💡 `**Specification**` **allows flexible query building.**

## 8️⃣ Use Database Connection Pooling for Better Performance

Creating a **new database connection** for every request is **expensive**.

✅ **Best Practice:** Use **HikariCP** (default in Spring Boot) for **efficient connection pooling**.

🔹 **Example: Configuring HikariCP in** `**application.properties**`

spring.datasource.hikari.maximum-pool-size=10  
spring.datasource.hikari.minimum-idle=5  
spring.datasource.hikari.idle-timeout=30000  
spring.datasource.hikari.max-lifetime=60000

💡 **Connection pooling reduces database load and improves query performance.**

## 9️⃣ Use `@Version` for Optimistic Locking in Concurrent Updates

When multiple users **update the same record**, data inconsistency can occur.

✅ **Best Practice:** Use `**@Version**` for **optimistic locking** to prevent overwriting changes.

🔹 **Example: Using** `**@Version**` **for Optimistic Locking**

@Entity  
public class Product {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    private String name;  
    private double price;  
  
    @Version  
    private int version; // ✅ Ensures safe concurrent updates  
}

💡 **Optimistic locking prevents accidental overwrites in concurrent updates.**

## 🔟 Enable SQL Logging for Debugging Queries

To **debug slow queries or optimize performance**, enable SQL logging.

✅ **Best Practice:** Use `**spring.jpa.show-sql**` and **Hibernate format logging**.

🔹 **Example: Enabling SQL Logging in** `**application.properties**`

spring.jpa.show-sql=true  
spring.jpa.properties.hibernate.format_sql=true

## 1️⃣1️⃣ Avoid `CascadeType.REMOVE` on Large Relationships

Using `CascadeType.REMOVE` on **large relationships** can trigger **massive delete operations**, potentially leading to **performance bottlenecks** and **database locks**.

### ✅ Best Practice: Handle Deletions Manually or Use `orphanRemoval` Selectively

### ❌ Bad Practice: Using `CascadeType.REMOVE` on Large Collections

@OneToMany(mappedBy = "customer", cascade = CascadeType.REMOVE)  
private List<Order> orders;

💡 **Issue:**

- Deletes **all associated** `**orders**` when a `customer` is deleted.
- Can **block database transactions** if many orders exist.
- Risk of **accidental data loss**.

### ✅ Better Approach: Use `orphanRemoval = true` for Child Entity Cleanup

@OneToMany(mappedBy = "customer", cascade = CascadeType.ALL, orphanRemoval = true)  
private List<Order> orders;

✔ Automatically removes **child entities** (`orders`) when they are removed from the list.  
✔ Does **not** trigger unnecessary bulk delete queries when deleting `customer`.

### ✅ Best Approach: Handle Deletions Manually for More Control

Instead of using `CascadeType.REMOVE`, delete child entities **explicitly** in the service layer.

@Service  
public class CustomerService {  
    private final OrderRepository orderRepository;  
    private final CustomerRepository customerRepository;  
  
    public CustomerService(OrderRepository orderRepository, CustomerRepository customerRepository) {  
        this.orderRepository = orderRepository;  
        this.customerRepository = customerRepository;  
    }  
  
    @Transactional  
    public void deleteCustomer(Long customerId) {  
        orderRepository.deleteByCustomerId(customerId); // Delete orders first  
        customerRepository.deleteById(customerId); // Then delete customer  
    }  
}

✔ **Ensures deletion order is optimized** (e.g., deletes child entities before the parent).  
✔ **Prevents unnecessary cascading deletes** that could lock the database.  
✔ **Provides better control over transactional operations**.

### 🚀 Why This Approach?

✔ **Prevents unintended cascading deletions** of large datasets.  
✔ **Reduces database load** by executing deletions in an optimal order.  
✔ **Provides more control** over the deletion process.

💡 **Pro Tip:** Use **batch deletes** or **native queries** (`DELETE FROM orders WHERE customer_id = ?`) for better performance in large datasets! 🚀

💡 **Always enable SQL logging in development mode for query optimization.**

## 1️⃣2️⃣ Avoid Annotating Repository Interfaces with `@Repository`

Spring Data JPA automatically detects repository interfaces **without requiring the** `**@Repository**` **annotation**. Adding `@Repository` manually is **redundant** and unnecessary.

### ✅ Best Practice: Let Spring Data JPA Detect Repositories Automatically

### ❌ Bad Practice: Explicitly Annotating with `@Repository`

@Repository // ❌ Not needed  
public interface UserRepository extends JpaRepository<User, Long> {  
    List<User> findByStatus(String status);  
}

💡 **Issue:**

- `@Repository` is redundant because Spring Data JPA **automatically registers repositories** under `@SpringBootApplication`.
- Unnecessary annotation clutter.

### ✅ Best Practice: Omit `@Repository` for Simplicity

public interface UserRepository extends JpaRepository<User, Long> {  
    List<User> findByStatus(String status);  
}

✔ **Spring automatically registers JPA repositories** if `@EnableJpaRepositories` is present (default in Spring Boot).  
✔ **Less clutter** in the codebase.

### 🔹 When Should You Use `@Repository`?

Use `@Repository` **only in manually implemented repository classes**, where you define custom queries outside of Spring Data JPA.

@Repository  
public class CustomUserRepository {  
      
    private final EntityManager entityManager;  
  
    public CustomUserRepository(EntityManager entityManager) {  
        this.entityManager = entityManager;  
    }  
  
    public List<User> findCustomUsers() {  
        return entityManager.createQuery("SELECT u FROM User u", User.class).getResultList();  
    }  
}

✔ **Informs Spring to handle exceptions as** `**DataAccessException**`  
✔ Required **only for manually implemented repositories**

## 🚀 Summary: Best Practices for Spring Data JPA

✅ **Use DTOs** instead of returning entities directly.

✅ **Use** `**@Modifying**` **&** `**@Transactional**` for bulk updates & deletes.

✅ **Use** `**FetchType.LAZY**` to improve performance.

✅ **Use** `**@EntityGraph**` to avoid N+1 query problems.

✅ **Use Pagination** for handling large datasets.

✅ **Use** `**@Query**` for complex queries.

✅ **Use** `**Specification**` for dynamic filtering.

✅ **Use Connection Pooling (HikariCP)** for better performance.

✅ **Use** `**@Version**` **for Optimistic Locking** in concurrent updates.

✅ **Enable SQL Logging** to monitor and optimize queries.

✅ **Avoid** `**CascadeType.REMOVE**` **on Large Relationships**

✅ **Avoid Annotating Repository Interfaces with** `**@Repository**`

Following these **best practices** ensures your **Spring Data JPA applications** are **efficient, scalable, and maintainable**. 🚀🔥

Do you follow these practices in your Spring Boot projects? Let me know! 😊

[  
](https://medium.com/tag/spring-data-jpa?source=post_page-----c2a84a4cd45e---------------------------------------)

[^1]: 
