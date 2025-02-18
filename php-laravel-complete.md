

# **PSR Standards Compliance Guide (PSR-1, PSR-4, PSR-12)**

## **What are PSR Standards?**

PHP-FIG (**PHP Framework Interoperability Group**) defines **PHP Standard Recommendations (PSRs)** to ensure **consistent coding standards** and **interoperability** between PHP frameworks and libraries.

The most commonly used PSRs are:

1. **PSR-1** – Basic Coding Standard
2. **PSR-4** – Autoloading Standard
3. **PSR-12** – Extended Coding Style Guide (an improvement over PSR-2)

* * *

## **1. PSR-1: Basic Coding Standard**

PSR-1 defines **basic coding standards** to ensure uniformity across PHP projects.

###  **PSR-1 Rules**

-   **Files must use `<?php` or `<?=`** and use UTF-8 encoding (without BOM).  
-   **Class names must use `StudlyCaps`** (PascalCase).  
-   **Method names must use `camelCase`**.  
-   **Constants must be in `UPPER_CASE`**.  
-   **Classes must be in `namespace`** and use `use` for importing.  
-   **One class per file**.

###  **Example: PSR-1 Compliance**

```php
<?php

namespace App\Services;

class UserService
{
    public const MAX_USERS = 100; // UPPER_CASE constant

    public function getUserData(): array // camelCase method name
    {
        return ['name' => 'John Doe'];
    }
}
```

* * *

##  **2. PSR-4: Autoloading Standard**

PSR-4 defines how **class files** should be autoloaded, replacing PSR-0.

### **PSR-4 Rules**

-   **Namespace matches directory structure**.  
-  **Class names use StudlyCaps**.  
-  **Autoloading is done via `composer.json`**.

###  **Example: PSR-4 Directory Structure**

```
project-root/
 ├── src/
 │    ├── Services/
 │    │    ├── UserService.php
 ├── composer.json
```

### **Example: PSR-4 Class**

```php
<?php

namespace App\Services;

class UserService
{
    public function getUserById(int $id)
    {
        return "User #$id";
    }
}
```

### **Step 1: Define Autoloading in `composer.json`**

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

### **Step 2: Run Composer Autoload**

```sh
composer dump-autoload
```

Now, you can **use the class without requiring files manually**:

```php
use App\Services\UserService;

$userService = new UserService();
echo $userService->getUserById(1);
```

* * *

##  **3. PSR-12: Extended Coding Style Guide**

PSR-12 builds upon **PSR-1 & PSR-2** with **stricter formatting** rules.

###  **PSR-12 Rules**

-  **Declare strict types**.  
-  **One class per file**.  
-  **Use `camelCase` for variables and methods**.  
-  **Opening braces `{` must be on the next line**.  
-  **Method arguments must have a space after commas**.  
-  **Namespace declarations must be on separate lines**.  
-  **Use fully qualified class names in type hints**.  
-  **No trailing spaces**.

###  **Example: PSR-12 Compliance**

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Models\User;
use App\Repositories\UserRepository;

class UserService
{
    private UserRepository $repository;

    public function __construct(UserRepository $repository)
    {
        $this->repository = $repository;
    }

    public function getUserById(int $id): ?User
    {
        return $this->repository->find($id);
    }
}
```

* * *

###  **PSR-1, PSR-4, PSR-12 Summary**

| PSR        | Purpose               | Key Rules                                                             |
| ---------- | --------------------- | --------------------------------------------------------------------- |
| **PSR-1**  | Basic coding standard | `StudlyCaps` class names, `camelCase` methods, `UPPER_CASE` constants |
| **PSR-4**  | Autoloading standard  | Namespace follows folder structure, composer autoload                 |
| **PSR-12** | Extended coding style | Strict types, spacing, method braces on the next line                 |

* * *

###  **Tools to Enforce PSR Standards**

**PHP CodeSniffer** (`phpcs`) – Detects PSR violations  
**PHP-CS-Fixer** – Automatically fixes code style  
**Laravel Pint**: Pre-configured PSR-12 fixer for Laravel.

---
# Design Principles

### **Single Responsibility Principle (SRP)**

* Every class or module should have **only one reason to change**.
* It should focus on a **single responsibility** and avoid handling multiple concerns.
* Helps in better **maintainability, scalability, and testability**.

**Example (Bad Practice - Violating SRP):**

```php
class ReportGenerator {
    public function generateReport($data) {
        // Generate report logic
    }

    public function saveToFile($filename) {
        // File saving logic (violates SRP)
    }

    public function sendEmail($recipient) {
        // Email sending logic (violates SRP)
    }
}
```

**Refactored Example (Following SRP):**

```php
class ReportGenerator {
    public function generate($data) {
        // Generate report logic
    }
}

class FileStorage {
    public function save($filename, $content) {
        // File saving logic
    }
}

class EmailService {
    public function send($recipient, $message) {
        // Email sending logic
    }
}
```

* * *

### **DRY (Don't Repeat Yourself) Principle**

* Avoid **code duplication** by ensuring that each piece of knowledge exists in **a single place**.
* Improves **code reusability, maintainability, and reduces bugs**.

**Example (Bad Practice - Violating DRY):**

```php
class Invoice {
    public function calculateTotal($items) {
        $total = 0;
        foreach ($items as $item) {
            $total += $item['price'] * $item['quantity'];
        }
        return $total;
    }

    public function calculateDiscountedTotal($items, $discount) {
        $total = 0;
        foreach ($items as $item) {
            $total += $item['price'] * $item['quantity'];
        }
        return $total - $discount;
    }
}
```

**Refactored Example (Following DRY):**

```php
class Invoice {
    private function getTotal($items) {
        $total = 0;
        foreach ($items as $item) {
            $total += $item['price'] * $item['quantity'];
        }
        return $total;
    }

    public function calculateTotal($items) {
        return $this->getTotal($items);
    }

    public function calculateDiscountedTotal($items, $discount) {
        return $this->getTotal($items) - $discount;
    }
}
```

   

By following **SRP**, you ensure that classes and modules are **focused and easier to maintain**.  
By applying **DRY**, you **reduce redundancy** and make the code more **efficient and maintainable**. 

### **Dependency Inversion Principle (DIP)**

The **Dependency Inversion Principle (DIP)** states that:

1. **High-level modules** should not depend on **low-level modules**. Both should depend on abstractions (interfaces).
2. **Abstractions** should not depend on **details** (concrete implementations). Instead, **details should depend on abstractions**.

DIP is one of the five **SOLID** principles and plays a crucial role in creating a **loosely coupled**, **scalable**, and **maintainable** system

 **Why Use DIP?**

Without DIP, a high-level module (like a service class) directly depends on a low-level module (like a database class). This creates a **tight coupling**, making the system:

- **Hard to modify** – Any change in the low-level module (e.g., switching from MySQL to PostgreSQL) requires changes in the high-level module.
- **Difficult to test** – Because the high-level module directly depends on a specific class, we cannot easily replace it with a mock for unit testing.
- **Less scalable** – New features or integrations require modifying multiple places in the code.

By applying **DIP**, we introduce **abstraction** (interfaces) between the high-level and low-level modules. This makes the system **flexible**, **testable**, and **future-proof**.

#### **Example Without DIP (Tightly Coupled Code)**

Let's say we have an `OrderService` class that interacts with a database class `MySQLDatabase` to retrieve order details.

``` php
class MySQLDatabase {
    public function getOrderById($id) {
        // Fetch order from MySQL database
    }
}

class OrderService {
    private $database;

    public function __construct() {
        $this->database = new MySQLDatabase(); // Directly depending on a concrete class
    }

    public function getOrder($id) {
        return $this->database->getOrderById($id);
    }
}

```

#### **Problems with this Approach**

- **Tightly Coupled:** `OrderService` is directly dependent on `MySQLDatabase`. If we want to switch to PostgreSQL, MongoDB, or an API, we must modify `OrderService`, violating **Open/Closed Principle (OCP)**.
- **Hard to Test:** To test `OrderService`, we need a real database connection. We cannot easily inject a mock or stub for testing.

#### **Applying DIP (Using Abstraction)**

To follow **DIP**, we introduce an **interface** that `OrderService` depends on instead of directly using `MySQLDatabase`.

#### Step 1: Create an Interface
``` php
interface DatabaseInterface {
    public function getOrderById($id);
}

```

#### Step 2: Implement the Interface in the Low-Level Module
``` php
class MySQLDatabase implements DatabaseInterface {
    public function getOrderById($id) {
        // Fetch order from MySQL database
    }
}

```

#### Modify the High-Level Module to Depend on the Interface
``` php
class OrderService {
    private $database;

    public function __construct(DatabaseInterface $database) {
        $this->database = $database; // Inject dependency through constructor
    }

    public function getOrder($id) {
        return $this->database->getOrderById($id);
    }
}

```

#### Step 4: Use Dependency Injection to Provide the Implementation

```php
$database = new MySQLDatabase();
$orderService = new OrderService($database); // Injecting dependency
```

#### Benefits of Applying DIP

#### Loosely Coupled Code:

- The high-level `OrderService` no longer depends on a concrete database implementation.
- If we want to change the database, we only need to modify the implementation of `DatabaseInterface`, not `OrderService`.

#### Easier to Extend and Maintain:

- If we need to add a `PostgreSQLDatabase` class, we don’t modify `OrderService`. We just create a new class:

```php
class PostgreSQLDatabase implements DatabaseInterface {
    public function getOrderById($id) {
        // Fetch order from PostgreSQL database
    }
}
```

- Now we can swap database implementations without modifying the service layer.

#### Better Unit Testing:

- We can inject a mock database for testing instead of a real database connection:

```php
class MockDatabase implements DatabaseInterface {
    public function getOrderById($id) {
        return ["id" => $id, "item" => "Test Product"];
    }
}

$mockDb = new MockDatabase();
$orderService = new OrderService($mockDb);
var_dump($orderService->getOrder(1)); // Returns test data
```

#### DIP in Laravel (Using Service Providers and Dependency Injection)

Laravel already encourages DIP by allowing services and repositories to depend on interfaces, which are then resolved through service providers.

#### Step 1: Define an Interface

```php
namespace App\Repositories;

interface OrderRepositoryInterface {
    public function find($id);
}
```

#### Step 2: Implement the Repository

```php
namespace App\Repositories;

use App\Models\Order;

class OrderRepository implements OrderRepositoryInterface {
    public function find($id) {
        return Order::find($id);
    }
}
```

#### Step 3: Inject Dependency into a Service

```php
namespace App\Services;

use App\Repositories\OrderRepositoryInterface;

class OrderService {
    private $orderRepository;

    public function __construct(OrderRepositoryInterface $orderRepository) {
        $this->orderRepository = $orderRepository;
    }

    public function getOrderById($id) {
        return $this->orderRepository->find($id);
    }
}
```

#### Step 4: Bind Interface to Implementation in `AppServiceProvider.php`

```php
use App\Repositories\OrderRepository;
use App\Repositories\OrderRepositoryInterface;

public function register() {
    $this->app->bind(OrderRepositoryInterface::class, OrderRepository::class);
}
```

#### Step 5: Use Dependency Injection in Controller

```php
namespace App\Http\Controllers;

use App\Services\OrderService;
use Illuminate\Http\JsonResponse;

class OrderController extends Controller {
    private $orderService;

    public function __construct(OrderService $orderService) {
        $this->orderService = $orderService;
    }

    public function show($id): JsonResponse {
        return response()->json($this->orderService->getOrderById($id));
    }
}
```


1. **Loosely Coupled Code**
    
    - `OrderService` does not depend on a specific database implementation but an abstraction (`OrderRepositoryInterface`).
    - Easily switch databases without modifying business logic.
2. **Easier to Extend and Maintain**
    
    - New database implementations can be added (e.g., `CachedOrderRepository`).
    - Follows the Open-Closed Principle (OCP).
3. **Better Unit Testing**
    
    - Replace dependencies with mock objects during testing.
    - Faster, isolated unit tests.
4. **Laravel’s Built-in Support for DIP**
    
    - Uses **Service Providers** and **dependency injection** via the container.
    - Keeps business logic (`OrderService`) separate from data access logic (`OrderRepository`).
5. **Flexibility and Scalability**
    
    - Additional repository implementations like `CachedOrderRepository` can enhance performance.
    - Enables database strategies like **sharding**, **partitioning**, or **caching** without modifying the service layer.

---
## Interface Segregation Principle (ISP)

The Interface Segregation Principle (ISP) states that a class should not be forced to depend on interfaces it does not use. Instead of having one large interface, we should create smaller, more specific interfaces.

### Violating ISP

Consider a large interface that forces all implementations to define methods they might not need:

```php
interface WorkerInterface {
    public function work();
    public function eat();
}
```

Now, implementing this interface forces all classes to define `eat()`, even if it doesn’t make sense for them:

```php
class RobotWorker implements WorkerInterface {
    public function work() {
        echo "Robot is working";
    }
    
    public function eat() {
        // Robots don’t eat, but we must define this method
        throw new Exception("Robots don’t eat");
    }
}
```

### Applying ISP Correctly

To follow ISP, we split the interface into smaller, more specific ones:

```php
interface Workable {
    public function work();
}

interface Eatable {
    public function eat();
}
```

Now, classes only implement the interfaces they need:

```php
class HumanWorker implements Workable, Eatable {
    public function work() {
        echo "Human is working";
    }
    
    public function eat() {
        echo "Human is eating";
    }
}
```

```php
class RobotWorker implements Workable {
    public function work() {
        echo "Robot is working";
    }
}
```

### ISP in Laravel

Laravel follows ISP by providing multiple small contracts instead of large monolithic interfaces.

Example: Instead of a single repository interface with multiple responsibilities, Laravel uses specific contracts such as `Authenticatable`, `Authorizable`, and `CanResetPassword`.

#### Example: Segregating Repositories

Instead of:

```php
interface OrderRepositoryInterface {
    public function find($id);
    public function save($order);
    public function exportOrdersToCSV();
}
```

We create multiple interfaces:

```php
interface OrderRetrievalInterface {
    public function find($id);
}

interface OrderPersistenceInterface {
    public function save($order);
}

interface OrderExportInterface {
    public function exportOrdersToCSV();
}
```


1. **Avoid large, all-encompassing interfaces**
    
    - Classes should not be forced to implement methods they don’t need.
2. **Create smaller, role-specific interfaces**
    
    - Keeps implementations clean and focused.
3. **Easier to extend and maintain**
    
    - New functionality can be added without affecting unrelated implementations.
4. **Laravel follows ISP**
    
    - Uses multiple contracts (`Authenticatable`, `CanResetPassword`) instead of large interfaces.
5. **Enhances Testability**
    
    - Mocking becomes easier when interfaces are specific and limited in scope.




---

# Dependency Injection (DI) in Laravel

### **What is Dependency Injection?**

Dependency Injection (DI) is a design pattern that **injects dependencies instead of instantiating them within a class**.

### **Types of Dependency Injection**

1️⃣ **Constructor Injection (Recommended)**  
2️⃣ **Setter Injection**  
3️⃣ **Method Injection**

* * *

### **Constructor Injection (Best Practice)**

* **Dependencies are injected in the constructor** and cannot be changed after instantiation.
* **Best for required dependencies**.

#### **Example: Constructor Injection in a Service**

```php
declare(strict_types=1);

namespace App\Services;

use App\Repositories\UserRepository;

class UserService
{
    private UserRepository $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }
}
```

```php
$userService = new UserService(new UserRepository());
```

* * *

### **Setter Injection (For Optional Dependencies)**

* **The dependency can be injected after the object is created**.
* **Best for optional dependencies**.

#### **Example: Setter Injection**

```php
declare(strict_types=1);

namespace App\Services;

use App\Repositories\UserRepository;

class UserService
{
    private ?UserRepository $userRepository = null;

    public function setUserRepository(UserRepository $userRepository): void
    {
        $this->userRepository = $userRepository;
    }
}
```

```php
$userService = new UserService();
$userService->setUserRepository(new UserRepository());
```

* * *

### **Method Injection (For Temporary Dependencies)**

* **Use when a dependency is only needed in one method**.
* **Prevents unnecessary dependencies in the constructor**.

#### **Example: Method Injection in a Controller**

```php
declare(strict_types=1);

namespace App\Http\Controllers;

use App\Services\UserService;
use Illuminate\Http\JsonResponse;

class UserController extends Controller
{
    public function show(int $id, UserService $userService): JsonResponse
    {
        $user = $userService->getUser($id);
        return response()->json($user);
    }
}
```

 **Use Constructor Injection** for required dependencies.  
 **Use Setter Injection** for optional dependencies.  
 **Use Method Injection** for temporary dependencies.

* * *

# Service and Repository Patterns
##  **Service-Based Pattern?**

The **Service-Based Pattern** (also called **Service Layer Pattern**) is a software design pattern that organizes **business logic** into reusable, centralized services.

Instead of placing logic directly in **controllers** or **models**, a **service layer** manages:

* **Business operations** (e.g., payments, notifications, logging).
* **Code reusability** (shared across different parts of the application).
* **Decoupling** (separates business logic from controllers).

* * *

###  **Key Benefits of Service-Based Pattern**

-  **Separation of Concerns** – Keeps controllers clean and focused.  
-  **Reusability** – One service can be used across multiple controllers.  
-  **Scalability** – Easy to modify or extend without affecting other parts.  
-  **Unit Test Friendly** – Easier to test services independently.

* * *

###  **Example: Logging Service Using Service-Based Pattern**

We'll create a **LoggingService** that logs messages using Laravel's **Monolog**.

#### **Step 1: Create `LoggingService`**

Create a new service class in `app/Services/LoggingService.php`.

```php
declare(strict_types=1);

namespace App\Services;

use Monolog\Logger;
use Monolog\Handler\StreamHandler;

class LoggingService
{
    private Logger $logger;

    public function __construct()
    {
        $this->logger = new Logger('app_logs');
        $this->logger->pushHandler(new StreamHandler(storage_path('logs/custom.log'), Logger::DEBUG));
    }

    public function logError(string $message, array $context = []): void
    {
        $this->logger->error($message, $context);
    }

    public function logInfo(string $message, array $context = []): void
    {
        $this->logger->info($message, $context);
    }
}
```

* * *

#### **Step 2: Register `LoggingService` as a Singleton**

Bind this service in `AppServiceProvider.php` inside `register()`.

```php
use App\Services\LoggingService;

public function register()
{
    $this->app->singleton(LoggingService::class, function ($app) {
        return new LoggingService();
    });
}
```

Now, Laravel will always return **the same instance** of `LoggingService`.

* * *

#### **Step 3: Usage in Controllers**

Now you can use `LoggingService` in any controller by injecting it.

```php
use App\Services\LoggingService;
use Illuminate\Http\Request;

class UserController extends Controller
{
    private LoggingService $loggingService;

    public function __construct(LoggingService $loggingService)
    {
        $this->loggingService = $loggingService;
    }

    public function store(Request $request)
    {
        try {
            // Simulating user creation
            $user = ['name' => $request->input('name')];

            $this->loggingService->logInfo("User created successfully", $user);

            return response()->json(['message' => 'User created', 'user' => $user], 201);
        } catch (\Exception $e) {
            $this->loggingService->logError("User creation failed", ['error' => $e->getMessage()]);

            return response()->json(['error' => 'User creation failed'], 500);
        }
    }
}
```

* * *

### **Step 4: Alternative Usage (Without Dependency Injection)**

If you don't want to inject the service in the constructor, you can use Laravel’s **Facade Pattern**.

#### **Create `LoggingServiceFacade.php`**

```php
namespace App\Facades;

use Illuminate\Support\Facades\Facade;

class LoggingServiceFacade extends Facade
{
    protected static function getFacadeAccessor()
    {
        return 'App\Services\LoggingService';
    }
}
```

#### **Register the Facade in `config/app.php`**

```php
'aliases' => [
    'LoggingService' => App\Facades\LoggingServiceFacade::class,
],
```

#### **Use It Like a Laravel Helper**

```php
use LoggingService;

LoggingService::logInfo('User created successfully.');
LoggingService::logError('Something went wrong.', ['code' => 500]);
```

* * *



## **Repository Pattern**

### **Why Use a Repository Pattern?**

 **Keeps database logic separate from business logic.**  
 **Easier to switch databases (e.g., MySQL to MongoDB).**  
 **Improves testability and maintainability.**

### Using Repository Pattern

```php
declare(strict_types=1);

namespace App\Repositories;

use App\Models\User;

class UserRepository
{
    public function find(int $id): ?User
    {
        return User::find($id);
    }
}
```

```php
declare(strict_types=1);

namespace App\Services;

use App\Repositories\UserRepository;

class UserService
{
    private UserRepository $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function getUser(int $id): ?User
    {
        return $this->userRepository->find($id);
    }
}
```

---
## **Service & Repo pattern**

In Laravel, we can effectively use both patterns together by following this approach:

- **Controllers:** Should handle HTTP requests, validate input, and delegate complex business operations to the service layer.
- **Services:** Should execute business logic, rules, and calculations. They act as the bridge between controllers and repositories.
- **Repositories:** Should be called by the service layer to retrieve data from and persist data to the database. They should not contain any business logic.

### **Normal UserController**

* The `UserController` handles both **business logic** and **database operations**, violating SRP.

```php
declare(strict_types=1);

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\JsonResponse;

class UserController extends Controller
{
    public function getUser(int $id): JsonResponse
    {
        $user = User::where('id', $id)->first();

        if (!$user) {
            return response()->json(['error' => 'User not found'], 404);
        }

        return response()->json($user);
    }
}
```

* * *

### Using Service & Repository Layers

 **Separating Concerns:**

* **UserRepository** handles **database queries**.
* **UserService** manages **business logic**.
* **UserController** only handles **HTTP requests**.

#### **Step 1: Create a Repository for Data Access**

```php
declare(strict_types=1);

namespace App\Repositories;

use App\Models\User;

class UserRepository
{
    public function find(int $id): ?User
    {
        return User::find($id);
    }
}
```

#### **Step 2: Create a Service for Business Logic**

```php
declare(strict_types=1);

namespace App\Services;

use App\Repositories\UserRepository;

class UserService
{
    private UserRepository $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function getUser(int $id): ?User
    {
        return $this->userRepository->find($id);
    }
}
```

#### **Step 3: Use Service in Controller**

```php
declare(strict_types=1);

namespace App\Http\Controllers;

use App\Services\UserService;
use Illuminate\Http\JsonResponse;

class UserController extends Controller
{
    private UserService $userService;

    public function __construct(UserService $userService)
    {
        $this->userService = $userService;
    }

    public function show(int $id): JsonResponse
    {
        $user = $this->userService->getUser($id);

        if (!$user) {
            return response()->json(['error' => 'User not found'], 404);
        }

        return response()->json($user);
    }
}
```


* * *
# Additional Best Practices and Patterns

## **Early Returns 

**Early returns** refer to the practice of returning from a function as soon as a certain condition is met, rather than using nested `if` statements or complex logic. This helps improve **readability, maintainability, and reduces nesting depth**.

####  **Bad Example (Nested If Conditions)**

```php
public function isValidUser(int $id): bool
{
    $user = User::find($id);
    if ($user) {
        if ($user->isActive()) {
            return true;
        }
    }
    return false;
}
```

####  **Good Example (Using Early Return)**

```php
public function isValidUser(int $id): bool
{
    $user = User::find($id);

    if (!$user || !$user->isActive()) {
        return false;
    }

    return true;
}
```

* * *

### **Using a Predefined Hashed Timestamp for Exception Tracking**

generate a **hashed timestamp once (in the terminal) and hardcode it in your code**, this can help make tracking issues easier across logs. This method ensures that:

6. **The trace key remains the same** across different environments (for debugging consistency).
7. **You can search logs easily** using a predefined key.
8. **Unique tracking per deployment** can be achieved by generating a new hash before each deployment **(Optional)**

## **Hardcoded Hash Key + Optional Deployment Key** 

### Custom Exception

``` php
declare(strict_types=1);

use Exception;

class CustomException extends Exception
{
    private const HASH_KEY = 'Xyz1234DefGhIjKl'; // Hardcoded Unique Hash Key
    private ?string $deploymentKey;

    public function __construct(string $message)
    {
        // Retrieve optional deployment key from .env (or fallback to null)
        $this->deploymentKey = getenv('DEPLOYMENT_TRACK_KEY') ?: null;

        // Format the exception message using the hardcoded hash key + optional deployment key
        $prefix = $this->deploymentKey 
            ? "[" . self::HASH_KEY . "-" . $this->deploymentKey . "]"
            : "[" . self::HASH_KEY . "]";

        parent::__construct("$prefix $message");
    }

    public function getDeploymentKey(): ?string
    {
        return $this->deploymentKey;
    }
}

```


#### Example Usage
``` php
// Example Usage
try {
    throw new CustomException("Something went wrong.");
} catch (CustomException $e) {
    echo "Exception: " . $e->getMessage() . PHP_EOL;
}
```



### Normal Exception with Hardcoded Hash Key + Optional Deployment Key

``` php
try {
    // Hardcoded unique hash key (pre-generated and stored in code)
    $hashKey = 'Xyz1234DefGhIjKl';

    // Optional deployment key from .env
    $deploymentKey = getenv('DEPLOYMENT_TRACK_KEY') ?: null;

    // Build the prefix for the exception message
    $prefix = $deploymentKey ? "[$hashKey-$deploymentKey]" : "[$hashKey]";

    // Throw a standard exception with the formatted message
    throw new Exception("$prefix Something went wrong.");
} catch (Exception $e) {
    echo "Exception: " . $e->getMessage() . PHP_EOL;
}

```


### **How This Works:**

9. **Hardcoded Hash Key (`HASH_KEY`)**
    
    - Pre-generated and stored in code.
    - Ensures a fixed identifier for tracking across different instances.
    - Example: `"Xyz1234DefGhIjKl"`
10. **Optional Deployment Key (`DEPLOYMENT_TRACK_KEY` from `.env`)**
    
    - Pulled dynamically from the environment.
    - Used for tracking **errors by deployment**.
    - Example: `"Deployment2025A"`
11. **Final Exception Message Format**
    
    * If the deployment key exists:
        
        ```
        [Xyz1234DefGhIjKl-Deployment2025A] Something went wrong.
        ```
        
    * If the deployment key is missing:
        
        ```
        [Xyz1234DefGhIjKl] Something went wrong.
        ```



* * *

## Using Form Request Validation?**

Form Request Validation is a **dedicated request class** in Laravel that **centralizes validation logic**, making controllers **cleaner and more maintainable**.

Instead of writing validation in controllers, Laravel provides **Form Request Classes** to handle validation separately.

* * *

## **1. Creating a Form Request Class**

Run the following command to create a **Form Request Class**:

```sh
php artisan make:request StoreUserRequest
```

This creates a file at:  
`app/Http/Requests/StoreUserRequest.php`

* * *

## **2. Defining Validation Rules**

Open `StoreUserRequest.php` and define rules inside the `rules()` method:

```php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true; // Set to false if authorization is required
    }

    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'unique:users,email'],
            'password' => ['required', 'min:8', 'confirmed'], // `password_confirmation` required in request
        ];
    }
}
```

* * *

## **3. Using the Form Request in Controllers**

Instead of using `request()->validate()`, inject `StoreUserRequest` into your controller method:

```php
use App\Http\Requests\StoreUserRequest;
use App\Models\User;
use Illuminate\Http\JsonResponse;

class UserController extends Controller
{
    public function store(StoreUserRequest $request): JsonResponse
    {
        $user = User::create($request->validated()); // Auto-validation
        
        return response()->json([
            'message' => 'User created successfully!',
            'user' => $user
        ], 201);
    }
}
```

* * *

##  **4. Custom Validation Messages**

Define custom error messages using the `messages()` method inside the Form Request:

```php
public function messages(): array
{
    return [
        'name.required' => 'The name field is required.',
        'email.unique' => 'This email is already registered.',
        'password.confirmed' => 'Passwords do not match.',
    ];
}
```

* * *

##  **5. Custom Validation Logic (Before Validation)**

You can **modify the request data** before validation using `prepareForValidation()`:

```php
protected function prepareForValidation()
{
    $this->merge([
        'email' => strtolower($this->email), // Convert email to lowercase before validation
    ]);
}
```

* * *

##  **6. Custom Authorization Logic**

If you need to **restrict validation to specific users**, update the `authorize()` method:

```php
public function authorize(): bool
{
    return auth()->user()->isAdmin(); // Allow only admins
}
```

* * *

## **7. Advanced: Custom Rule Objects**

For complex validation, create a **custom rule**:

```sh
php artisan make:rule StrongPassword
```

This generates: `app/Rules/StrongPassword.php`

```php
namespace App\Rules;

use Closure;
use Illuminate\Contracts\Validation\ValidationRule;

class StrongPassword implements ValidationRule
{
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if (!preg_match('/[A-Z]/', $value) || !preg_match('/\d/', $value)) {
            $fail('The :attribute must contain at least one uppercase letter and one number.');
        }
    }
}
```

### **Use the Rule in Form Request**

```php
use App\Rules\StrongPassword;

public function rules(): array
{
    return [
        'password' => ['required', 'min:8', new StrongPassword],
    ];
}
```

* * *

##  **8. Handling Validation Errors**

If validation fails, Laravel automatically **returns a JSON response** for API requests:

```json
{
    "message": "The given data was invalid.",
    "errors": {
        "email": ["The email field is required."]
    }
}
```

If handling errors manually, override the `failedValidation()` method:

```php
use Illuminate\Contracts\Validation\Validator;
use Illuminate\Http\Exceptions\HttpResponseException;

protected function failedValidation(Validator $validator)
{
    throw new HttpResponseException(response()->json([
        'message' => 'Validation failed',
        'errors' => $validator->errors(),
    ], 422));
}
```

* * *

## **9. Best Practices**

-  **Use Form Request for all CRUD operations**  
-  **Keep rules simple; use Custom Rules for complex logic**  
-  **Use `validated()` to get only valid input**  
-  **Modify data using `prepareForValidation()` if needed**  
-  **Authorize requests with `authorize()` if required**

* * *