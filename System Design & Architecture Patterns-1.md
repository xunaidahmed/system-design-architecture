# System Design & Architecture Patterns #1

A practical guide to the most commonly used software architecture and system design patterns for building **maintainable, scalable, testable, and production-ready applications**.

Examples are primarily based on **PHP / Laravel**, but the architectural concepts can be applied to Node.js, Java, Python, .NET, Go, and other backend technologies.

---

## Table of Contents

1. [MVC Pattern](#1-mvc-pattern)
2. [Modular Architecture / Modular Monolith](#2-modular-architecture--modular-monolith)
3. [Repository Pattern](#3-repository-pattern)
4. [Service Layer Pattern](#4-service-layer-pattern)
5. [Factory Pattern](#5-factory-pattern)
6. [Strategy Pattern](#6-strategy-pattern)
7. [Observer / Event-Driven Pattern](#7-observer--event-driven-pattern)
8. [Dependency Injection Pattern](#8-dependency-injection-pattern)
9. [CQRS Pattern](#9-cqrs-pattern)
10. [Microservices + API Gateway Pattern](#10-microservices--api-gateway-pattern)
11. [Combining the Patterns](#combining-the-patterns)
12. [Recommended Production Architecture](#recommended-production-architecture)

---

# 1. MVC Pattern

## Overview

**MVC (Model-View-Controller)** separates an application into three major responsibilities:

* **Model** — Data and database interaction
* **View** — User interface / presentation
* **Controller** — Handles HTTP requests and coordinates application logic

### Architecture

```text
Client
  │
  ▼
Controller
  │
  ├──────────────► Model ──────────────► Database
  │
  ▼
View
  │
  ▼
Client
```

### Laravel Structure

```text
app/
├── Models/
│   └── User.php
│
└── Http/
    └── Controllers/
        └── UserController.php

resources/
└── views/
    └── users/
        └── index.blade.php
```

### Example

```php
// routes/web.php

Route::get('/users', [UserController::class, 'index']);
```

```php
// UserController.php

class UserController extends Controller
{
    public function index()
    {
        $users = User::latest()->get();

        return view('users.index', compact('users'));
    }
}
```

```php
// User.php

class User extends Model
{
    protected $fillable = [
        'name',
        'email',
    ];
}
```

### When to Use

Use MVC for:

* Web applications
* CRUD applications
* Admin panels
* Traditional Laravel applications
* Applications where presentation and business logic need separation

### Advantages

* Simple structure
* Easy to understand
* Good separation of responsibilities
* Supported by most web frameworks

### Disadvantages

* Controllers can become very large
* Business logic can leak into controllers
* Large applications may need additional architectural patterns

---

# 2. Modular Architecture / Modular Monolith

## Overview

A **Modular Monolith** keeps the application in a single deployable system but divides it into independent business modules.

Instead of organizing everything only by technical layers, organize the application around **business domains**.

For example:

```text
User
Order
Payment
Inventory
Notification
Reporting
```

### Architecture

```text
                    Application
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
     User Module     Order Module    Payment Module
        │                │                │
        ▼                ▼                ▼
   Controllers       Controllers       Controllers
   Services          Services          Services
   Repositories      Repositories      Repositories
   Models            Models            Models
```

### Recommended Laravel Structure

```text
app/
└── Modules/
    │
    ├── User/
    │   ├── Controllers/
    │   │   └── UserController.php
    │   ├── Models/
    │   │   └── User.php
    │   ├── Services/
    │   │   └── UserService.php
    │   ├── Repositories/
    │   │   └── UserRepository.php
    │   ├── Requests/
    │   ├── Events/
    │   ├── Jobs/
    │   └── Routes/
    │       └── api.php
    │
    ├── Order/
    │   ├── Controllers/
    │   ├── Models/
    │   ├── Services/
    │   ├── Repositories/
    │   ├── Events/
    │   └── Routes/
    │
    ├── Payment/
    │   ├── Controllers/
    │   ├── Models/
    │   ├── Services/
    │   ├── Repositories/
    │   ├── Events/
    │   └── Routes/
    │
    └── Notification/
        ├── Controllers/
        ├── Services/
        ├── Jobs/
        └── Routes/
```

### Example

```php
class OrderService
{
    public function createOrder(array $data)
    {
        // Validate business rules
        // Create order
        // Calculate totals
        // Trigger payment
        // Dispatch event

        return Order::create($data);
    }
}
```

Controller:

```php
class OrderController extends Controller
{
    public function store(
        Request $request,
        OrderService $orderService
    ) {
        return $orderService->createOrder(
            $request->validated()
        );
    }
}
```

### When to Use

Ideal for:

* SaaS applications
* ERP systems
* Marketplaces
* FinTech applications
* Large Laravel applications
* Applications with multiple business domains

### Advantages

* Better code organization
* Modules can evolve independently
* Easier team collaboration
* Easier testing
* Easier future migration to microservices

### Disadvantages

* Requires architectural discipline
* More initial structure
* Developers must understand module boundaries

### Key Idea

> **A Modular Monolith is often a practical step between a simple monolith and microservices.**

---

# 3. Repository Pattern

## Overview

The Repository Pattern separates **data access logic** from business logic.

Instead of directly querying the database from services:

```php
User::where('email', $email)->first();
```

Use:

```text
Service
   │
   ▼
Repository
   │
   ▼
Database
```

### Structure

```text
app/
├── Repositories/
│   ├── Contracts/
│   │   └── UserRepositoryInterface.php
│   │
│   └── UserRepository.php
│
├── Services/
│   └── UserService.php
│
└── Models/
    └── User.php
```

### Interface

```php
interface UserRepositoryInterface
{
    public function findById(int $id);

    public function findByEmail(string $email);

    public function create(array $data);
}
```

### Repository

```php
class UserRepository implements UserRepositoryInterface
{
    public function findById(int $id)
    {
        return User::find($id);
    }

    public function findByEmail(string $email)
    {
        return User::where('email', $email)->first();
    }

    public function create(array $data)
    {
        return User::create($data);
    }
}
```

### Service

```php
class UserService
{
    public function __construct(
        private UserRepositoryInterface $users
    ) {}

    public function register(array $data)
    {
        return $this->users->create($data);
    }
}
```

### Advantages

* Centralized data access
* Easier unit testing
* Database logic is isolated
* Easier to change data sources

### Disadvantages

* Additional abstraction
* Can become unnecessary for simple CRUD applications

---

# 4. Service Layer Pattern

## Overview

The Service Layer contains **business logic**.

Instead of putting complex logic inside controllers:

```text
Controller
    │
    ▼
Service
    │
    ├── Repository
    ├── Payment
    ├── Notification
    └── Event
```

### Example

```php
class OrderService
{
    public function __construct(
        private OrderRepository $orders,
        private PaymentService $payments
    ) {}

    public function create(array $data)
    {
        $order = $this->orders->create($data);

        $this->payments->authorize(
            $order->total
        );

        return $order;
    }
}
```

Controller:

```php
class OrderController extends Controller
{
    public function store(
        Request $request,
        OrderService $service
    ) {
        return $service->create(
            $request->validated()
        );
    }
}
```

### When to Use

Use Service Layer when:

* Business rules are complex
* Multiple controllers use the same logic
* Multiple external services are involved
* Transactions are required

### Advantages

* Thin controllers
* Reusable business logic
* Easier testing
* Better maintainability

---

# 5. Factory Pattern

## Overview

The Factory Pattern creates objects without exposing the object creation logic to the caller.

### Example

Imagine multiple payment providers:

```text
Payment
├── Stripe
├── PayPal
└── Checkout
```

### Structure

```text
PaymentFactory
      │
      ├── StripePayment
      ├── PayPalPayment
      └── CheckoutPayment
```

### Interface

```php
interface PaymentGateway
{
    public function charge(float $amount);
}
```

### Stripe

```php
class StripePayment implements PaymentGateway
{
    public function charge(float $amount)
    {
        // Stripe API
    }
}
```

### PayPal

```php
class PayPalPayment implements PaymentGateway
{
    public function charge(float $amount)
    {
        // PayPal API
    }
}
```

### Factory

```php
class PaymentFactory
{
    public static function make(
        string $gateway
    ): PaymentGateway {

        return match ($gateway) {
            'stripe' => new StripePayment(),
            'paypal' => new PayPalPayment(),
            default => throw new InvalidArgumentException(
                'Unsupported gateway'
            ),
        };
    }
}
```

### Usage

```php
$payment = PaymentFactory::make('stripe');

$payment->charge(100);
```

### Useful For

* Payment gateways
* Notification providers
* Storage providers
* Shipping providers
* AI providers
* Third-party API integrations

---

# 6. Strategy Pattern

## Overview

The Strategy Pattern allows an application to select different algorithms or business rules at runtime.

### Example

Different pricing strategies:

```text
PricingStrategy
       │
       ├── StandardPricing
       ├── PremiumPricing
       └── DiscountPricing
```

### Interface

```php
interface PricingStrategy
{
    public function calculate(float $amount): float;
}
```

### Standard

```php
class StandardPricing implements PricingStrategy
{
    public function calculate(float $amount): float
    {
        return $amount;
    }
}
```

### Discount

```php
class DiscountPricing implements PricingStrategy
{
    public function calculate(float $amount): float
    {
        return $amount * 0.90;
    }
}
```

### Context

```php
class PricingService
{
    public function __construct(
        private PricingStrategy $strategy
    ) {}

    public function calculate(float $amount): float
    {
        return $this->strategy->calculate($amount);
    }
}
```

### Useful For

* Pricing
* Tax calculation
* Payment methods
* Shipping methods
* Authentication methods
* AI provider selection
* Notification channels

---

# 7. Observer / Event-Driven Pattern

## Overview

The Observer Pattern allows one action to notify multiple listeners.

Instead of:

```text
Order Created
    │
    ├── Send Email
    ├── Send Notification
    ├── Update Analytics
    └── Generate Invoice
```

Use events:

```text
Order Created Event
        │
        ├── Email Listener
        ├── Notification Listener
        ├── Analytics Listener
        └── Invoice Listener
```

### Laravel Event

```php
class OrderCreated
{
    public function __construct(
        public Order $order
    ) {}
}
```

### Dispatch Event

```php
event(new OrderCreated($order));
```

### Listener

```php
class SendOrderConfirmation
{
    public function handle(OrderCreated $event)
    {
        Mail::to(
            $event->order->customer->email
        )->send(
            new OrderConfirmationMail($event->order)
        );
    }
}
```

### Architecture

```text
Application
     │
     ▼
Order Created
     │
     ▼
Event Bus
     │
 ┌───┼────────┬──────────┐
 ▼   ▼        ▼          ▼
Email SMS   Analytics  Invoice
```

### Advantages

* Loose coupling
* Async processing
* Easy extension
* Excellent for scalable systems

### Useful For

* Notifications
* Emails
* Analytics
* Audit logs
* Background jobs
* Webhooks
* Integrations

---

# 8. Dependency Injection Pattern

## Overview

Dependency Injection means a class receives its dependencies instead of creating them internally.

### Bad Example

```php
class OrderService
{
    public function process()
    {
        $payment = new StripePayment();

        $payment->charge(100);
    }
}
```

The class is tightly coupled to Stripe.

### Better Example

```php
class OrderService
{
    public function __construct(
        private PaymentGateway $payment
    ) {}

    public function process()
    {
        $this->payment->charge(100);
    }
}
```

Now we can inject:

```text
OrderService
     │
     ▼
PaymentGateway
     │
 ┌───┴────┐
 ▼        ▼
Stripe   PayPal
```

### Laravel Binding

```php
$this->app->bind(
    PaymentGateway::class,
    StripePayment::class
);
```

### Advantages

* Loose coupling
* Easy testing
* Easy mocking
* Flexible architecture
* Easier provider replacement

---

# 9. CQRS Pattern

## CQRS = Command Query Responsibility Segregation

CQRS separates:

```text
Commands = Change data

Queries = Read data
```

Instead of one service handling everything:

```text
Application
     │
     ├── Commands
     │
     └── Queries
```

### Architecture

```text
                 API
                  │
          ┌───────┴───────┐
          │               │
       Command          Query
          │               │
          ▼               ▼
    Write Model       Read Model
          │               │
          ▼               ▼
      Database        Read Database
```

### Command Example

```php
class CreateOrderCommand
{
    public function __construct(
        public int $customerId,
        public float $amount
    ) {}
}
```

### Command Handler

```php
class CreateOrderHandler
{
    public function handle(
        CreateOrderCommand $command
    ) {
        return Order::create([
            'customer_id' => $command->customerId,
            'amount' => $command->amount,
        ]);
    }
}
```

### Query

```php
class GetCustomerOrdersQuery
{
    public function __construct(
        public int $customerId
    ) {}
}
```

### Query Handler

```php
class GetCustomerOrdersHandler
{
    public function handle(
        GetCustomerOrdersQuery $query
    ) {
        return Order::where(
            'customer_id',
            $query->customerId
        )->get();
    }
}
```

### When to Use

CQRS is useful when:

* Read/write workloads are very different
* Reporting is complex
* Read performance is critical
* The domain has complex business rules
* Different read and write models are required

### Avoid CQRS when

* Application is small
* CRUD is simple
* The additional complexity provides no benefit

---

# 10. Microservices + API Gateway Pattern

## Overview

Microservices divide a large application into independently deployable services.

### Architecture

```text
                         Client
                           │
                           ▼
                     API Gateway
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
       ▼                   ▼                   ▼
   User Service       Order Service       Payment Service
       │                   │                   │
       ▼                   ▼                   ▼
   User DB             Order DB           Payment DB

                           │
                           ▼
                     Message Broker
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
           Email        Analytics     Notification
```

### Example Services

```text
services/
├── user-service/
│   ├── Controllers/
│   ├── Services/
│   ├── Models/
│   └── Routes/
│
├── order-service/
│   ├── Controllers/
│   ├── Services/
│   ├── Models/
│   └── Routes/
│
├── payment-service/
│   ├── Controllers/
│   ├── Services/
│   ├── Models/
│   └── Routes/
│
└── notification-service/
    ├── Services/
    └── Jobs/
```

### API Gateway

The API Gateway acts as the single entry point:

```text
Client
  │
  ▼
API Gateway
  │
  ├── /users      → User Service
  ├── /orders     → Order Service
  ├── /payments   → Payment Service
  └── /notifications → Notification Service
```

### Responsibilities

An API Gateway can handle:

* Authentication
* Authorization
* Rate limiting
* Routing
* Request validation
* Logging
* API versioning
* Load balancing

### Advantages

* Independent deployments
* Independent scaling
* Technology flexibility
* Team autonomy
* Fault isolation

### Disadvantages

* Distributed-system complexity
* Network failures
* Monitoring becomes harder
* Deployment becomes more complex
* Data consistency becomes more difficult

### When to Use

Use microservices when the system has:

* Large teams
* Multiple business domains
* Independent scaling requirements
* High traffic
* Independent deployment requirements

Do not split a small application into microservices simply because microservices are popular.

---

# Combining the Patterns

The patterns become more powerful when they are combined.

A production Laravel application might look like:

```text
                         Client
                           │
                           ▼
                         API
                           │
                           ▼
                    Module / Domain
                           │
                    ┌──────┴──────┐
                    │ Controller  │
                    └──────┬──────┘
                           │
                           ▼
                       Service
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
        Repository      Strategy      Factory
              │            │            │
              ▼            ▼            ▼
           Database    Business Rules  Provider
                           │
                           ▼
                         Event
                           │
                ┌──────────┼──────────┐
                ▼          ▼          ▼
              Email      Queue      Analytics
```

---

# Recommended Production Architecture

For a large Laravel SaaS, marketplace, FinTech, or enterprise application, a practical architecture can be:

```text
app/
│
├── Modules/
│   │
│   ├── User/
│   │   ├── Controllers/
│   │   ├── Requests/
│   │   ├── Models/
│   │   ├── Services/
│   │   ├── Repositories/
│   │   ├── Events/
│   │   ├── Listeners/
│   │   ├── Jobs/
│   │   └── Routes/
│   │
│   ├── Order/
│   │   ├── Controllers/
│   │   ├── Models/
│   │   ├── Services/
│   │   ├── Repositories/
│   │   ├── Events/
│   │   └── Routes/
│   │
│   ├── Payment/
│   │   ├── Controllers/
│   │   ├── Models/
│   │   ├── Services/
│   │   ├── Strategies/
│   │   ├── Factories/
│   │   ├── Repositories/
│   │   └── Routes/
│   │
│   └── Notification/
│       ├── Services/
│       ├── Jobs/
│       ├── Events/
│       └── Listeners/
│
├── Shared/
│   ├── Exceptions/
│   ├── Traits/
│   ├── Helpers/
│   └── Contracts/
│
└── Infrastructure/
    ├── Payments/
    ├── Storage/
    ├── Messaging/
    └── ExternalAPIs/
```

---

# Pattern Selection Guide

| Requirement                        | Recommended Pattern  |
| ---------------------------------- | -------------------- |
| Basic web application              | MVC                  |
| Large Laravel application          | Modular Monolith     |
| Separate database logic            | Repository           |
| Complex business logic             | Service Layer        |
| Multiple object creation types     | Factory              |
| Multiple algorithms/providers      | Strategy             |
| Notifications and async processing | Event-Driven         |
| Loose coupling                     | Dependency Injection |
| Heavy read/write separation        | CQRS                 |
| Independent scalable services      | Microservices        |

---

# Typical Request Flow

A well-structured Laravel application can process a request like this:

```text
HTTP Request
     │
     ▼
Route
     │
     ▼
Controller
     │
     ▼
Form Request / Validation
     │
     ▼
Service
     │
     ├──────────────► Strategy
     │
     ├──────────────► Factory
     │
     ▼
Repository
     │
     ▼
Database
     │
     ▼
Event
     │
     ├──────────────► Queue
     ├──────────────► Email
     ├──────────────► Notification
     └──────────────► Analytics
     │
     ▼
Response
```

---

# Monolith vs Modular Monolith vs Microservices

```text
Simple Monolith
      │
      ▼
Modular Monolith
      │
      ▼
Distributed Modules
      │
      ▼
Microservices
```

### Simple Monolith

```text
One Application
      │
 ┌────┼────┐
 ▼    ▼    ▼
User Order Payment
```

### Modular Monolith

```text
One Application
      │
 ┌────┼────────────┐
 ▼    ▼            ▼
User  Order      Payment
Module Module     Module
```

### Microservices

```text
API Gateway
     │
 ┌───┼───────────────┐
 ▼   ▼               ▼
User Order         Payment
Svc   Svc             Svc
 │     │               │
DB    DB              DB
```

---

# Architecture Evolution

A system does not need to start with microservices.

A practical evolution can be:

```text
Stage 1
Simple Laravel Application
        │
        ▼
Stage 2
Service Layer
        │
        ▼
Stage 3
Modular Monolith
        │
        ▼
Stage 4
Events + Queues
        │
        ▼
Stage 5
CQRS where required
        │
        ▼
Stage 6
Extract selected modules
        │
        ▼
Stage 7
Microservices
```

---

# Core Principles

Regardless of which pattern you choose, follow these principles:

### 1. Single Responsibility

Each class/module should have a clear responsibility.

### 2. Loose Coupling

Components should depend on abstractions rather than concrete implementations.

### 3. High Cohesion

Related business logic should stay together.

### 4. Separation of Concerns

Keep presentation, business logic, data access, and infrastructure concerns separated.

### 5. Don't Over Engineer

Do not introduce a pattern simply because it exists.

### 6. Design for Change

Use abstractions where future changes are reasonably expected.

### 7. Measure Before Scaling

Use real traffic, performance, and operational requirements to determine when architectural complexity is justified.

---

# Quick Reference

```text
MVC
│
├── Presentation separation
│
Modular Architecture
│
├── Business domain separation
│
Repository
│
├── Data access abstraction
│
Service Layer
│
├── Business logic
│
Factory
│
├── Object creation
│
Strategy
│
├── Replaceable algorithms
│
Observer / Events
│
├── Loose coupling
│
Dependency Injection
│
├── Dependency management
│
CQRS
│
├── Read / Write separation
│
Microservices
│
└── Independent distributed services
```

---

# Final Architecture

For a scalable application, the overall design can evolve toward:

```text
                         USERS
                           │
                           ▼
                    LOAD BALANCER
                           │
                           ▼
                      API GATEWAY
                           │
                           ▼
                 ┌───────────────────┐
                 │ Modular Application│
                 └───────────────────┘
                           │
       ┌───────────────────┼───────────────────┐
       │                   │                   │
       ▼                   ▼                   ▼
     USER                ORDER              PAYMENT
    MODULE               MODULE              MODULE
       │                   │                   │
       ▼                   ▼                   ▼
   Service              Service             Service
       │                   │                   │
       ▼                   ▼                   ▼
 Repository            Repository           Strategy
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
                           ▼
                         EVENTS
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
            QUEUE        EMAIL       ANALYTICS
              │
              ▼
           WORKERS
              │
              ▼
       External Services
```

---

# Conclusion

The goal of architecture is not to use every available pattern.

The goal is to choose the **simplest architecture that can reliably satisfy the application's current and reasonably expected future requirements**.

A strong progression for modern backend development is:

```text
MVC
 ↓
Service Layer
 ↓
Repository
 ↓
Dependency Injection
 ↓
Modular Monolith
 ↓
Factory + Strategy
 ↓
Events + Queues
 ↓
CQRS (when justified)
 ↓
Microservices (when justified)
```

These patterns provide a foundation for designing applications that are:

* Maintainable
* Testable
* Scalable
* Extensible
* Loosely coupled
* Easier to deploy
* Easier for teams to maintain

---

## Technology Examples

This architecture can be implemented using:

* PHP / Laravel
* Node.js / NestJS
* Python / Django / FastAPI
* Java / Spring Boot
* C# / .NET
* Go
* PostgreSQL / MySQL
* Redis
* RabbitMQ / Kafka
* Docker
* Kubernetes
* AWS / Google Cloud / Azure

---

## License

This documentation is provided for educational and architectural reference purposes.
