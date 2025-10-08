# Design Patterns in Python

Design patterns are proven solutions to common software design problems. They provide templates for how to structure your code to solve specific issues in a reusable way. Understanding design patterns is crucial for writing clean, maintainable, and scalable backend systems.

## Part 1: Creational Patterns

*Creational patterns provide various object creation mechanisms, which increase flexibility and reuse of existing code.*

-----

### Singleton

1.  **Pattern & Category**
    **Singleton** (Creational)

2.  **The Core Problem**
    How do we ensure a class has **only one instance** throughout the entire application lifecycle and provide a single, global point of access to it? This is crucial for managing shared resources like database connection pools, loggers, or application-wide configuration.

3.  **The Solution & Analogy**
    The Singleton pattern ensures that a class can only be instantiated once. The first time an instance is requested, it's created and stored. Every subsequent request returns this exact same stored instance.

    **Analogy:** The principal's office in a school. There is only one official office. No matter which student or teacher needs to see the principal, they are all directed to that same, single office. The school ensures no duplicate principal's offices are created. 🏫

4.  **Practical Backend Use Case**
    An application configuration manager. Your backend service needs to load settings (e.g., database URL, API keys, feature flags) from a `.env` file or environment variables at startup. These settings should be loaded only once and then be accessible from anywhere in the application without the overhead of re-reading the file or environment every time.

5.  **Python Code Implementation**
    Here's a thread-safe Singleton implementation using a metaclass, which is a robust and Pythonic way to control class instantiation.

    ```python
    # singleton_pattern.py
    import threading
    from typing import Any, Dict

    class SingletonMeta(type):
        """A thread-safe Singleton metaclass."""
        _instances: Dict[type, Any] = {}
        _lock: threading.Lock = threading.Lock()

        def __call__(cls, *args: Any, **kwargs: Any) -> Any:
            # Use double-checked locking for thread safety.
            if cls not in cls._instances:
                with cls._lock:
                    if cls not in cls._instances:
                        instance = super().__call__(*args, **kwargs)
                        cls._instances[cls] = instance
            return cls._instances[cls]

    class AppConfig(metaclass=SingletonMeta):
        """
        Singleton configuration manager for the application.
        Loads settings once and provides them globally.
        """
        def __init__(self) -> None:
            # In a real app, this would load from a file or env vars.
            print("Initializing application configuration...")
            self._config: Dict[str, Any] = {
                "DATABASE_URL": "postgresql://user:pass@host:5432/db",
                "API_KEY": "a1b2c3d4-e5f6-7890-g1h2-i3j4k5l6m7n8",
                "FEATURE_X_ENABLED": True,
            }
            print("Configuration loaded.")

        def get(self, key: str) -> Any:
            return self._config.get(key)

    # --- Demonstration ---
    print("Attempting to create first config instance...")
    config1 = AppConfig()
    print(f"API Key from config1: {config1.get('API_KEY')}")

    print("\nAttempting to create second config instance...")
    config2 = AppConfig() # __init__ will NOT be called again
    print(f"API Key from config2: {config2.get('API_KEY')}")

    print(f"\nAre config1 and config2 the same instance? {config1 is config2}")
    ```

6.  **Common Interview Questions & Answers**

      * **Q: When is Singleton an appropriate choice, and when is it considered an anti-pattern?**

          * **A:** "It's appropriate for managing truly global, stateless resources where having more than one instance would be problematic, like a logging service or an application configuration manager. However, it's often considered an anti-pattern when used to manage global state, as it effectively becomes a glorified global variable. This creates hidden dependencies between components, makes the system harder to reason about, and severely complicates unit testing. In many cases, **Dependency Injection** is a more flexible and testable alternative."

      * **Q: How does the Singleton pattern affect unit testing?**

          * **A:** "It makes unit testing significantly harder. Because the Singleton instance is global and persists throughout the test suite, tests are no longer isolated. A test that modifies the Singleton's state can affect the outcome of subsequent tests. To work around this, you either need to implement a special `reset()` method on the Singleton for test teardown or use mocking libraries to patch the Singleton's access point during tests to provide a controlled, isolated instance."

-----

### Factory Method

1.  **Pattern & Category**
    **Factory Method** (Creational)

2.  **The Core Problem**
    A class needs to create objects, but it cannot anticipate the exact class of objects it must create. It wants to defer the instantiation logic to its subclasses.

3.  **The Solution & Analogy**
    The pattern defines an interface (an abstract "factory method") for creating a single object but lets subclasses override this method to specify which exact class to instantiate. The superclass can then work with the objects created by its subclasses without knowing their concrete type.

    **Analogy:** A logistics company. The base `Logistics` class has a `plan_delivery()` method which schedules a delivery. This method calls a `create_transport()` factory method to get a vehicle. A `RoadLogistics` subclass implements `create_transport()` to return a `Truck`, while a `SeaLogistics` subclass implements it to return a `Ship`. The main `plan_delivery()` logic doesn't care if it's a truck or ship; it just knows it's a "transport" vehicle. 🚚

4.  **Practical Backend Use Case**
    A notification service that must send various types of notifications (e.g., Email, SMS, Push). A base `NotificationManager` class defines the high-level logic to send a notification (e.g., getting user data, formatting a message). It uses a factory method to create the actual `Notifier` object, allowing subclasses like `EmailManager` or `SmsManager` to provide the specific notifier implementation.

5.  **Python Code Implementation**

    ```python
    # factory_method_pattern.py
    from abc import ABC, abstractmethod

    # --- The Product Interface ---
    class Notifier(ABC):
        @abstractmethod
        def send(self, message: str) -> None:
            pass

    # --- Concrete Products ---
    class EmailNotifier(Notifier):
        def send(self, message: str) -> None:
            print(f"Sending email notification: '{message}'")

    class SmsNotifier(Notifier):
        def send(self, message: str) -> None:
            print(f"Sending SMS notification: '{message}'")

    # --- The Creator (Abstract) ---
    class NotificationManager(ABC):
        def send_notification(self, message: str) -> None:
            """The main logic that uses the factory method."""
            notifier = self.create_notifier() # The factory method call
            notifier.send(message)

        @abstractmethod
        def create_notifier(self) -> Notifier:
            """The Factory Method to be implemented by subclasses."""
            pass

    # --- Concrete Creators ---
    class EmailManager(NotificationManager):
        def create_notifier(self) -> Notifier:
            return EmailNotifier()

    class SmsManager(NotificationManager):
        def create_notifier(self) -> Notifier:
            return SmsNotifier()

    # --- Demonstration ---
    def client_code(manager: NotificationManager, message: str):
        """The client code works with a creator instance, not knowing the concrete product."""
        manager.send_notification(message)

    print("Client is using the EmailManager:")
    email_manager = EmailManager()
    client_code(email_manager, "Your order has been shipped!")

    print("\nClient is using the SmsManager:")
    sms_manager = SmsManager()
    client_code(sms_manager, "Your package is out for delivery.")
    ```

6.  **Common Interview Questions & Answers**

      * **Q: What is the key difference between Factory Method and Abstract Factory?**

          * **A:** "The Factory Method is a single method in a class that defers instantiation to subclasses, typically through **inheritance**. It's focused on creating one type of product. The Abstract Factory is an object that provides an interface for creating **families of related products**, typically through **composition**. You'd use Factory Method to let a `Document` subclass decide if it creates a `PdfPage` or `WordPage`, but you'd use Abstract Factory to get a `UIFactory` that can create a matched set of `DarkButton`, `DarkTextBox`, and `DarkScrollBar`."

      * **Q: How does the Factory Method pattern support the Open/Closed and Single Responsibility principles?**

          * **A:** "It strongly supports the **Open/Closed Principle**. The client code (like `send_notification`) is closed for modification. To add a new notification type, like Push Notifications, you just create a new `PushManager` and `PushNotifier` class; you don't touch the existing code. It also supports **SRP** by separating the responsibility of product creation from the business logic that uses the product. The `NotificationManager` focuses on the logic of sending, while the subclasses focus solely on which notifier to create."

-----

### Abstract Factory

1.  **Pattern & Category**
    **Abstract Factory** (Creational)

2.  **The Core Problem**
    How do you create **families of related objects** without being tightly coupled to their concrete classes? Your system needs to be configurable with one of several product families, ensuring that the created objects are compatible with each other.

3.  **The Solution & Analogy**
    The pattern provides an interface (the abstract factory) for creating a family of related products. Concrete factories implement this interface to produce objects from a specific family. The client code depends only on the abstract factory and the abstract product interfaces, ensuring consistency.

    **Analogy:** An IKEA furniture factory. You have an `AbstractFurnitureFactory` interface defining methods like `create_chair()` and `create_table()`. A `ModernFurnitureFactory` implements these to create a `ModernChair` and `ModernTable`. A `VictorianFurnitureFactory` creates a `VictorianChair` and `VictorianTable`. Your app is configured with one of these factories, ensuring that if you ask for a chair and a table, you get a matching set. 🛋️

4.  **Practical Backend Use Case**
    A data access layer that needs to support multiple database systems (e.g., PostgreSQL, MongoDB). An `AbstractRepositoryFactory` can define methods like `create_user_repository()` and `create_order_repository()`. A `PostgresRepositoryFactory` would create repositories that interact with PostgreSQL, while a `MongoRepositoryFactory` would create repositories that work with MongoDB. The business logic layer is configured with one of these factories and works with the repositories through a common interface, unaware of the underlying database.

5.  **Python Code Implementation**

    ```python
    # abstract_factory_pattern.py
    from abc import ABC, abstractmethod

    # --- Abstract Products ---
    class IUserRepo(ABC): @abstractmethod
    def get_user(self, user_id: int): pass
    class IOrderRepo(ABC): @abstractmethod
    def get_order(self, order_id: int): pass

    # --- Concrete Products for PostgreSQL ---
    class PostgresUserRepo(IUserRepo):
        def get_user(self, user_id: int): print(f"Postgres: Getting user {user_id}")
    class PostgresOrderRepo(IOrderRepo):
        def get_order(self, order_id: int): print(f"Postgres: Getting order {order_id}")

    # --- Concrete Products for MongoDB ---
    class MongoUserRepo(IUserRepo):
        def get_user(self, user_id: int): print(f"MongoDB: Getting user {user_id}")
    class MongoOrderRepo(IOrderRepo):
        def get_order(self, order_id: int): print(f"MongoDB: Getting order {order_id}")

    # --- Abstract Factory ---
    class RepositoryFactory(ABC):
        @abstractmethod
        def create_user_repository(self) -> IUserRepo: pass
        @abstractmethod
        def create_order_repository(self) -> IOrderRepo: pass

    # --- Concrete Factories ---
    class PostgresRepositoryFactory(RepositoryFactory):
        def create_user_repository(self) -> IUserRepo: return PostgresUserRepo()
        def create_order_repository(self) -> IOrderRepo: return PostgresOrderRepo()

    class MongoRepositoryFactory(RepositoryFactory):
        def create_user_repository(self) -> IUserRepo: return MongoUserRepo()
        def create_order_repository(self) -> IOrderRepo: return MongoOrderRepo()

    # --- Demonstration ---
    def get_factory(db_type: str) -> RepositoryFactory:
        """A helper to get the correct factory based on configuration."""
        if db_type == "postgres":
            return PostgresRepositoryFactory()
        elif db_type == "mongo":
            return MongoRepositoryFactory()
        raise ValueError("Unknown database type")

    # The application logic is configured with a factory.
    # It doesn't know or care about the concrete implementation.
    current_db_factory = get_factory("postgres")
    user_repo = current_db_factory.create_user_repository()
    order_repo = current_db_factory.create_order_repository()

    user_repo.get_user(123)
    order_repo.get_order(456)
    ```

6.  **Common Interview Questions & Answers**

      * **Q: When would you choose Abstract Factory over Factory Method?**

          * **A:** "I'd choose Abstract Factory when my system needs to create **multiple, related products that must be used together**, ensuring consistency. For example, creating a matched set of database repositories. I'd use Factory Method when the focus is on creating a **single product**, and I want to let subclasses control which specific type of that single product gets created."

      * **Q: How does this pattern facilitate testing and support the Dependency Inversion Principle?**

          * **A:** "It's fundamental for both. It directly implements the **Dependency Inversion Principle**: my high-level business logic depends on the abstract factory interface, not the concrete database implementations. For testing, this is a huge win. In my unit tests, I can create a `MockRepositoryFactory` that produces in-memory mock repositories. I can then inject this mock factory into my services, allowing me to test my business logic in complete isolation from any actual database."

-----

## Part 2: Structural Patterns

*Structural patterns explain how to assemble objects and classes into larger structures, while keeping these structures flexible and efficient.*

-----

### Decorator

1.  **Pattern & Category**
    **Decorator** (Structural)

2.  **The Core Problem**
    How can you add new behaviors or responsibilities to individual objects dynamically at runtime, without affecting other objects of the same class?

3.  **The Solution & Analogy**
    The pattern involves creating "wrapper" objects (decorators) that have the same interface as the object they wrap (the component). The decorator forwards requests to the component while adding its own logic either before or after the call. Multiple decorators can be stacked on top of each other.

    **Analogy:** Putting on clothes. You are the `Component`. You can dynamically add a `JacketDecorator` to add warmth, and then wrap that with a `RaincoatDecorator` for water resistance. Each layer adds functionality without changing your fundamental nature, and you can mix and-match layers as needed. 🧥

4.  **Practical Backend Use Case**
    Enhancing a simple API client. You have a base `ApiClient` that makes an HTTP request. You want to add functionality like request logging, response caching, and adding authentication headers. Instead of creating complex subclasses for every combination, you can dynamically wrap the base client in `LoggingDecorator`, `CachingDecorator`, and `AuthDecorator` as needed.

5.  **Python Code Implementation**

    ```python
    # decorator_pattern.py
    import time
    from abc import ABC, abstractmethod
    from typing import Any, Dict

    # --- The Component Interface ---
    class ApiClient(ABC):
        @abstractmethod
        def get(self, url: str) -> Any:
            pass

    # --- Concrete Component ---
    class SimpleApiClient(ApiClient):
        def get(self, url: str) -> Any:
            print(f"Making a real HTTP GET request to {url}")
            # Simulate a network call
            time.sleep(2)
            return {"data": "some payload from the server"}

    # --- Base Decorator ---
    class ApiClientDecorator(ApiClient):
        def __init__(self, wrapped_client: ApiClient):
            self._wrapped_client = wrapped_client

        def get(self, url: str) -> Any:
            return self._wrapped_client.get(url)

    # --- Concrete Decorators ---
    class LoggingDecorator(ApiClientDecorator):
        def get(self, url: str) -> Any:
            print(f"[LOG] Requesting URL: {url}")
            start_time = time.time()
            response = super().get(url)
            end_time = time.time()
            print(f"[LOG] Request took {end_time - start_time:.2f} seconds.")
            return response

    class CachingDecorator(ApiClientDecorator):
        def __init__(self, wrapped_client: ApiClient):
            super().__init__(wrapped_client)
            self._cache: Dict[str, Any] = {}

        def get(self, url: str) -> Any:
            if url in self._cache:
                print(f"[CACHE] Returning cached response for {url}")
                return self._cache[url]
            
            response = super().get(url)
            self._cache[url] = response
            print(f"[CACHE] Storing response for {url} in cache.")
            return response

    # --- Demonstration ---
    # Start with a simple client
    client = SimpleApiClient()

    # Wrap it with a logging decorator
    logging_client = LoggingDecorator(client)
    print("--- Making first call with logging only ---")
    logging_client.get("https://api.example.com/data")

    # Now wrap the logging client with a caching decorator
    cached_logging_client = CachingDecorator(logging_client)

    print("\n--- Making first call with caching and logging ---")
    cached_logging_client.get("https://api.example.com/products")

    print("\n--- Making second call to the same URL ---")
    # This time, it should hit the cache and be much faster
    cached_logging_client.get("https://api.example.com/products")
    ```

6.  **Common Interview Questions & Answers**

      * **Q: How is the Decorator pattern different from standard inheritance?**

          * **A:** "Inheritance adds responsibilities at compile time and applies them to all instances of a subclass. The Decorator pattern adds responsibilities to individual objects dynamically at runtime. This provides far more flexibility; you can mix and match decorators in any combination, avoiding the 'class explosion' problem where you'd need a separate subclass for every possible feature combination (e.g., `LoggingApiClient`, `CachingApiClient`, `LoggingAndCachingApiClient`)."

      * **Q: Python has built-in function and class decorators (the `@` syntax). How do they relate to this design pattern?**

          * **A:** "Python's built-in decorator syntax is a direct, language-level implementation of the Decorator design pattern's concept. When you use a decorator like `@app.route('/')` in Flask or `@dataclass` from the standard library, you are essentially wrapping your function or class to add new behavior without modifying its source code. It's a prime example of how deeply this pattern is integrated into the Python language itself."

-----

### Adapter

1.  **Pattern & Category**
    **Adapter** (Structural)

2.  **The Core Problem**
    How do you make two classes with incompatible interfaces work together without modifying their source code? This is a common issue when integrating with third-party libraries or legacy systems.

3.  **The Solution & Analogy**
    The pattern involves creating a wrapper class (the adapter) that implements the interface your client code expects. Internally, the adapter receives calls and translates them into calls that the incompatible object (the adaptee) can understand, handling any necessary data transformations along the way.

    **Analogy:** A universal travel power adapter. Your laptop charger (the client) has a Type A plug. The wall socket in Europe (the adaptee) is Type F. You use a power adapter that fits into the Type F socket and provides a Type A socket for your charger, seamlessly connecting the two incompatible interfaces.  🔌

4.  **Practical Backend Use Case**
    You are integrating a new third-party payment gateway into your e-commerce platform. Your existing system is built around an `IPaymentProcessor` interface with a `process_payment(amount, card_details)` method. The new gateway's SDK provides a `SuperGateway` class with a completely different method, `charge(charge_amount, credit_card)`, which also uses a different `credit_card` object structure. You create an `SuperGatewayAdapter` to make them compatible.

5.  **Python Code Implementation**

    ```python
    # adapter_pattern.py
    from abc import ABC, abstractmethod
    from typing import Dict

    # --- The Incompatible Third-Party Class (Adaptee) ---
    class SuperGateway:
        def charge(self, charge_amount: float, credit_card: Dict[str, str]) -> bool:
            print(f"Charging ${charge_amount:.2f} via SuperGateway...")
            # Some complex logic to charge the card
            print(f"Card Charged: {credit_card['number']}")
            return True

    # --- The Target Interface (what our system uses) ---
    class IPaymentProcessor(ABC):
        @abstractmethod
        def process_payment(self, amount: float, card_details: Dict[str, str]) -> bool:
            pass

    # --- The Adapter Class ---
    class SuperGatewayAdapter(IPaymentProcessor):
        def __init__(self, gateway: SuperGateway):
            self._gateway = gateway

        def process_payment(self, amount: float, card_details: Dict[str, str]) -> bool:
            print("--- Using Adapter to translate our call to the gateway's call ---")
            # Translation logic: map our card_details to their credit_card format
            gateway_card_format = {
                "number": card_details.get("card_number"),
                "exp": f"{card_details.get('exp_month')}/{card_details.get('exp_year')}"
            }
            return self._gateway.charge(charge_amount=amount, credit_card=gateway_card_format)

    # --- Demonstration ---
    our_payment_details = {
        "card_number": "1234-5678-9012-3456",
        "exp_month": "12",
        "exp_year": "2028",
        "cvv": "123"
    }

    # The client code only knows about our IPaymentProcessor interface.
    # It doesn't know anything about the SuperGateway.
    payment_processor: IPaymentProcessor = SuperGatewayAdapter(SuperGateway())
    payment_processor.process_payment(199.99, our_payment_details)
    ```

6.  **Common Interview Questions & Answers**

      * **Q: What's the key difference between the Adapter, Decorator, and Facade patterns? They all seem to be 'wrappers'.**

          * **A:** "While they are all wrappers, their **intent** is completely different. An **Adapter's** intent is to **convert** one interface into another to make them compatible. A **Decorator's** intent is to **add** responsibilities to an object without changing its interface. A **Facade's** intent is to **simplify** a complex subsystem by providing a single, high-level interface. The problem you're trying to solve dictates which one you use."

      * **Q: When is the Adapter pattern most useful in a backend context?**

          * **A:** "It's most valuable when you're integrating with systems you cannot change. This commonly includes third-party APIs or SDKs, legacy services within your own company, or different microservices that were developed with inconsistent interfaces. The Adapter acts as an anti-corruption layer, protecting your application's domain from being polluted by external, incompatible models and interfaces."

-----

### Facade

1.  **Pattern & Category**
    **Facade** (Structural)

2.  **The Core Problem**
    How do you provide a simple, unified interface to a large and complex subsystem of classes, making it much easier for client code to use?

3.  **The Solution & Analogy**
    The pattern provides a single class (the facade) that acts as a simplified entry point to the subsystem. Clients interact with the facade, which intelligently coordinates and delegates the requests to the appropriate objects within the complex system, hiding the complexity from the client.

    **Analogy:** The ignition of a car. When you turn the key or press the "Start" button (the facade's simple method), you are triggering a highly complex sequence of events in the car's subsystem: the battery is engaged, the fuel pump activates, the starter motor turns the engine, etc. As the driver, you don't need to know or manage any of that complexity; you just use the simple facade.  🔑

4.  **Practical Backend Use Case**
    An e-commerce order placement service. The process of placing an order is complex, involving multiple services: an `InventoryService` to check stock, a `PaymentService` to process payment, a `ShippingService` to arrange delivery, and an `OrderRepository` to save the order to the database. An `OrderPlacementFacade` can expose a single `place_order(cart, user)` method that orchestrates all these moving parts.

5.  **Python Code Implementation**

    ```python
    # facade_pattern.py

    # --- The Complex Subsystem ---
    class InventoryService:
        def check_stock(self, item_id: str, quantity: int) -> bool:
            print(f"Checking stock for {quantity} of item {item_id}...")
            return True

    class PaymentService:
        def process_payment(self, amount: float) -> bool:
            print(f"Processing payment of ${amount}...")
            return True

    class ShippingService:
        def arrange_shipping(self, order_id: str, address: str) -> str:
            print(f"Arranging shipping for order {order_id} to {address}...")
            return "tracking-ABC-123"

    # --- The Facade ---
    class OrderPlacementFacade:
        def __init__(self):
            self.inventory = InventoryService()
            self.payment = PaymentService()
            self.shipping = ShippingService()

        def place_order(self, item_id: str, quantity: int, amount: float, address: str) -> bool:
            """A single, simplified method to handle the complex order process."""
            print("--- Placing order using Facade ---")
            
            if not self.inventory.check_stock(item_id, quantity):
                print("Order failed: item out of stock.")
                return False
            
            if not self.payment.process_payment(amount):
                print("Order failed: payment was declined.")
                return False
            
            tracking_id = self.shipping.arrange_shipping("ORD-XYZ", address)
            print(f"Order placed successfully! Tracking ID: {tracking_id}")
            return True

    # --- Demonstration ---
    order_facade = OrderPlacementFacade()
    # The client code has a very simple interaction.
    success = order_facade.place_order(
        item_id="book-101",
        quantity=1,
        amount=29.99,
        address="123 Python Lane, Coder City"
    )
    ```

6.  **Common Interview Questions & Answers**

      * **Q: What is the primary benefit of using a Facade?**

          * **A:** "The primary benefit is **decoupling**. The facade decouples the client code from the intricate details of a subsystem. This is powerful because it means you can completely refactor or even replace the subsystem behind the facade without breaking any client code, as long as the facade's public API remains stable. A secondary benefit is a significant simplification of the client's logic."

      * **Q: Does a Facade prevent clients from accessing the underlying subsystem classes directly?**

          * **A:** "No, and that's an important point. A Facade provides a simplified, convenient entry point for common use cases, but it doesn't have to restrict access to the lower-level classes. If a client has a specialized need that requires more fine-grained control, it can still interact directly with the subsystem components. The facade is an optional simplification layer, not a mandatory prison."

-----

## Part 3: Behavioral Patterns

*Behavioral patterns are concerned with algorithms and the assignment of responsibilities between objects.*

-----

### Strategy

1.  **Pattern & Category**
    **Strategy** (Behavioral)

2.  **The Core Problem**
    You have a class (the "context") that needs to perform a certain action, but there are multiple different algorithms or ways ("strategies") to do it. How do you allow the algorithm to be selected and swapped at runtime?

3.  **The Solution & Analogy**
    The pattern defines a family of algorithms, encapsulates each one into a separate class with a common interface, and makes them interchangeable. The context class holds a reference to a strategy object and delegates the work to it, without knowing the details of the specific algorithm being used.

    **Analogy:** A map application for navigation. The app is the `Context`. It can be configured with different routing `Strategy` objects: a `DrivingStrategy` (prefers highways), a `WalkingStrategy` (prefers pedestrian paths), or a `PublicTransitStrategy` (uses bus and train routes). You can switch between these strategies on the fly to get a different route. 🗺️

4.  **Practical Backend Use Case**
    A user authentication service that needs to support multiple authentication methods, such as password-based, OAuth (e.g., Google/Facebook), and API Key. The `AuthService` (context) is configured with an `AuthStrategy`. You have concrete `PasswordStrategy`, `OauthStrategy`, and `ApiKeyStrategy` classes, each with an `authenticate` method that contains the specific logic for that method.

5.  **Python Code Implementation**

    ```python
    # strategy_pattern.py
    from abc import ABC, abstractmethod
    from typing import Dict, Any

    # --- The Strategy Interface ---
    class AuthStrategy(ABC):
        @abstractmethod
        def authenticate(self, credentials: Dict[str, Any]) -> bool:
            pass

    # --- Concrete Strategies ---
    class PasswordStrategy(AuthStrategy):
        def authenticate(self, credentials: Dict[str, Any]) -> bool:
            username = credentials.get("username")
            password = credentials.get("password")
            print(f"Authenticating '{username}' with a password...")
            # In a real app, hash password and compare with DB
            return password == "supersecret"

    class OauthStrategy(AuthStrategy):
        def authenticate(self, credentials: Dict[str, Any]) -> bool:
            token = credentials.get("oauth_token")
            print(f"Authenticating with OAuth token starting with '{token[:8]}...'")
            # In a real app, validate token with the OAuth provider
            return token is not None

    # --- The Context ---
    class AuthService:
        def __init__(self, strategy: AuthStrategy):
            self._strategy = strategy

        def set_strategy(self, strategy: AuthStrategy):
            self._strategy = strategy
        
        def login(self, credentials: Dict[str, Any]) -> None:
            if self._strategy.authenticate(credentials):
                print("Authentication successful.")
            else:
                print("Authentication failed.")

    # --- Demonstration ---
    password_creds = {"username": "admin", "password": "supersecret"}
    oauth_creds = {"oauth_token": "a1b2c3d4-e5f6-7890-g1h2-i3j4k5l6m7n8"}

    # Start with password authentication
    auth_service = AuthService(PasswordStrategy())
    print("--- Attempting password login ---")
    auth_service.login(password_creds)

    # Switch the strategy at runtime to OAuth
    auth_service.set_strategy(OauthStrategy())
    print("\n--- Attempting OAuth login ---")
    auth_service.login(oauth_creds)
    ```

6.  **Common Interview Questions & Answers**

      * **Q: How does the Strategy pattern differ from the Template Method pattern?**

          * **A:** "They solve similar problems but with different approaches. The key difference is **inheritance vs. composition**. **Template Method** uses **inheritance**; a base class defines a fixed algorithm skeleton, and subclasses override specific steps to provide variation. **Strategy** uses **composition**; the context class holds a reference to a separate strategy object, and the entire algorithm can be swapped out by providing a different object. I'd use Template Method for minor variations in a fixed algorithm, and Strategy for completely different algorithms that accomplish the same goal."

      * **Q: How does the Strategy pattern support the Open/Closed Principle?**

          * **A:** "It's a textbook example of OCP. The context class, our `AuthService`, is closed for modification. If we need to add a new authentication method like 'Biometric', we don't touch the `AuthService` at all. We simply create a new `BiometricStrategy` class that implements the `AuthStrategy` interface. The system is extended with new behavior without altering existing, tested code."

-----

### Observer

1.  **Pattern & Category**
    **Observer** (Behavioral)

2.  **The Core Problem**
    How do you define a one-to-many dependency between objects so that when one object (the "subject") changes its state, all its dependents ("observers") are notified and updated automatically without the objects being tightly coupled?

3.  **The Solution & Analogy**
    The pattern involves a "subject" object that maintains a list of its "observer" dependents. The subject provides methods to subscribe (attach) and unsubscribe (detach) observers. When the subject's state changes, it iterates through its list of observers and calls a notification method on each one.

    **Analogy:** Subscribing to a magazine. The publisher is the `Subject`. You and other subscribers are the `Observers`. When the publisher releases a new issue (a state change), they automatically mail a copy to every subscriber on their list. The publisher doesn't need to know who you are personally, only that you are on the subscription list. 📰

4.  **Practical Backend Use Case**
    An inventory management system. A `Product` object (the subject) tracks its stock level. When the stock level drops below a certain threshold, it needs to notify various other parts of the system. A `RestockService` (observer) might automatically create a purchase order, and a `MarketingService` (another observer) might remove the product from promotional ads.

5.  **Python Code Implementation**

    ```python
    # observer_pattern.py
    from __future__ import annotations
    from abc import ABC, abstractmethod
    from typing import List

    # --- The Subject (or Publisher) ---
    class Product:
        def __init__(self, name: str, initial_stock: int):
            self._name = name
            self._stock = initial_stock
            self._observers: List[Observer] = []

        def attach(self, observer: Observer) -> None:
            self._observers.append(observer)

        def detach(self, observer: Observer) -> None:
            self._observers.remove(observer)
        
        def _notify(self) -> None:
            print(f"Notifying {len(self._observers)} observers about '{self._name}'...")
            for observer in self._observers:
                observer.update(self)

        @property
        def stock(self) -> int: return self._stock
        
        @stock.setter
        def stock(self, new_stock: int) -> None:
            print(f"Updating stock for '{self._name}' to {new_stock}...")
            self._stock = new_stock
            if self._stock < 5: # Threshold for notification
                self._notify()

    # --- The Observer Interface ---
    class Observer(ABC):
        @abstractmethod
        def update(self, subject: Product) -> None:
            pass

    # --- Concrete Observers ---
    class RestockService(Observer):
        def update(self, subject: Product) -> None:
            if subject.stock < 5:
                print(f"RestockService: Stock for {subject._name} is low ({subject.stock})."
                      " Placing a new purchase order.")

    class MarketingService(Observer):
        def update(self, subject: Product) -> None:
            if subject.stock < 5:
                print(f"MarketingService: Stock for {subject._name} is low ({subject.stock})."
                      " Removing product from active promotions.")

    # --- Demonstration ---
    laptop = Product("SuperBook Laptop", 10)

    # Attach observers
    restocker = RestockService()
    marketer = MarketingService()
    laptop.attach(restocker)
    laptop.attach(marketer)

    print("\n--- Simulating a sale ---")
    laptop.stock = 8 # No notification, above threshold
    print("\n--- Simulating another sale, hitting the threshold ---")
    laptop.stock = 4 # This will trigger notifications
    ```

6.  **Common Interview Questions & Answers**

      * **Q: What is the main benefit of the Observer pattern for backend architectures?**

          * **A:** "The main benefit is achieving **loose coupling** between objects. The subject, for instance our `Product` class, knows nothing about the concrete classes of its observers. It only knows that they conform to the `Observer` interface. This is fantastic for building event-driven systems. You can add new listeners (like a `DashboardUpdateService`) to react to state changes without ever modifying the subject's code."

      * **Q: What are potential drawbacks or things to watch out for when using this pattern?**

          * **A:** "A potential drawback is that it can lead to complex and hard-to-debug update chains if not managed carefully. An observer, upon being updated, might trigger a change in another subject, which in turn notifies its own observers, and so on. Another consideration is that the basic pattern doesn't guarantee the order of notifications, which might be a requirement in some systems. Finally, if you forget to detach an observer, it can lead to memory leaks in some languages."

-----

### Template Method

1.  **Pattern & Category**
    **Template Method** (Behavioral)

2.  **The Core Problem**
    You have an algorithm that consists of a fixed series of steps, but some of those steps can have different implementations. How do you avoid duplicating the overall algorithm structure in multiple classes?

3.  **The Solution & Analogy**
    The pattern defines the skeleton of an algorithm in a base class method (the "template method"). This method calls a sequence of other methods in a fixed order. Some of these methods are abstract and *must* be implemented by subclasses, while others ("hooks") are concrete and can optionally be overridden.

    **Analogy:** The assembly line for building a car. The overall process (the template method) is fixed: 1) Build Chassis, 2) Install Engine, 3) Install Doors, 4) Paint Body. However, the `Install Engine` step can be implemented differently for a `GasolineCarBuilder` (installs a V8 engine) versus an `ElectricCarBuilder` (installs an electric motor and battery pack). The overall sequence remains the same. 🚗

4.  **Practical Backend Use Case**
    A generic data processing pipeline for an ETL (Extract, Transform, Load) job. A base `EtlPipeline` class defines a `run()` template method that orchestrates the process: it calls `extract()`, then `transform()`, then `load()`. The `extract()` and `load()` methods are abstract, as they depend on the specific data source and destination. Subclasses like `CsvToDatabasePipeline` or `ApiToDataWarehousePipeline` provide the concrete implementations for these steps.

5.  **Python Code Implementation**

    ```python
    # template_method_pattern.py
    from abc import ABC, abstractmethod

    class EtlPipeline(ABC):
        """The abstract base class defining the template method."""

        def run(self) -> None:
            """This is the Template Method."""
            print("--- Starting ETL Pipeline ---")
            data = self.extract()
            transformed_data = self.transform(data)
            self.load(transformed_data)
            self.after_load_hook() # A hook method
            print("--- ETL Pipeline Finished ---")

        @abstractmethod
        def extract(self) -> list:
            """Primitive operation: must be implemented by subclasses."""
            pass

        @abstractmethod
        def load(self, data: list) -> None:
            """Primitive operation: must be implemented by subclasses."""
            pass

        def transform(self, data: list) -> list:
            """A concrete method with a default implementation."""
            print("Transforming data (default: capitalizing strings)...")
            return [str(item).upper() for item in data]

        def after_load_hook(self) -> None:
            """A hook method: subclasses can override it, but don't have to."""
            print("Default hook: No post-load actions.")


    # --- Concrete Implementation ---
    class CsvToDatabasePipeline(EtlPipeline):
        def extract(self) -> list:
            print("Extracting data from a CSV file...")
            return ["first_record", "second_record"]
        
        def load(self, data: list) -> None:
            print(f"Loading transformed data into database: {data}")

        def after_load_hook(self) -> None:
            print("Hook: Sending a success notification email.")

    # --- Demonstration ---
    csv_pipeline = CsvToDatabasePipeline()
    csv_pipeline.run()
    ```

6.  **Common Interview Questions & Answers**

      * **Q: How is the Template Method pattern different from the Strategy pattern?**

          * **A:** "The core difference lies in their approach: **inheritance vs. composition**. **Template Method** uses **inheritance** to allow subclasses to redefine certain *parts* of a fixed algorithm. The overall structure is controlled by the parent class. **Strategy** uses **composition** to let a client replace the *entire* algorithm with a different one by providing a different strategy object. I'd use Template Method when the overall process is always the same, but specific steps vary. I'd use Strategy when the entire process can be done in fundamentally different ways."

      * **Q: What is a "hook" in the context of the Template Method pattern, and why is it useful?**

          * **A:** "A hook is a method in the base class that is given a default, often empty, implementation. This makes it optional for subclasses to override. It's useful because it provides an entry point for subclasses to add their own custom logic at specific points in the algorithm without being forced to. In our ETL example, the `after_load_hook` allows a subclass to optionally send a notification or run a cleanup task after the data is loaded, without cluttering the main algorithm."