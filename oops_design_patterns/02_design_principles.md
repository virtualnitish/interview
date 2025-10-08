# Design Principles & SOLID
Design principles are the foundational guidelines that help you write clean, maintainable, and scalable code. They are not specific to any programming language but are universally applicable across all object-oriented programming languages, including Python.

## Part 1: Foundational Principles

Before we dive into the famous SOLID principles, let's cover three foundational concepts that are more like guiding philosophies. They are simple to understand but have a profound impact on your day-to-day coding.

-----

### KISS (Keep It Simple, Stupid)

1.  **Principle & Mnemonic**
    **Keep It Simple, Stupid (KISS)**

2.  **Core Idea**
    Your system will work better and be more robust if you keep its design and implementation as simple as possible.

3.  **Problem It Prevents**
    This principle directly combats **over-engineering**. Violating KISS leads to unnecessarily complex abstractions, convoluted logic, and code that is difficult to understand, debug, and maintain. It's the root of solutions that are clever but not practical.

4.  **Simple Analogy**
    You need to hang a picture on a wall. The simple solution is a hammer and a nail. The over-engineered, KISS-violating solution is to design a robotic, laser-guided, automated picture-hanging machine. While technically impressive, it's far too complex for the problem at hand. 🔨

5.  **Python Backend Example (Violation vs. Solution)**

      * **Violation:** A function to determine if a new user's chosen username is available, written in a needlessly complex way.

        ```python
        # violation_kiss.py
        class UsernameStatus:
            def __init__(self, is_taken: bool, message: str):
                self.is_taken = is_taken
                self.message = message

        def check_username_availability_complex(username: str, users_in_db: list[str]) -> UsernameStatus:
            """Checks if a username is available using a complex structure."""
            is_found = False
            for u in users_in_db:
                if u.lower() == username.lower():
                    is_found = True
                    break
            
            if is_found:
                return UsernameStatus(is_taken=True, message=f"'{username}' is already taken.")
            else:
                return UsernameStatus(is_taken=False, message=f"'{username}' is available.")
        ```

      * **Explanation:** This code is verbose. It uses a custom class `UsernameStatus` where a simple boolean would do, a manual loop with a flag variable where a simple `in` check is more readable, and constructs complex messages.

      * **Solution:** A much simpler, more direct implementation.

        ```python
        # solution_kiss.py
        def check_username_availability_simple(username: str, users_in_db: list[str]) -> bool:
            """A simple, direct way to check for username availability."""
            return username.lower() in [u.lower() for u in users_in_db]
        ```

6.  **Benefits & Trade-offs**

      * **Benefits:**
          * **Increased Readability:** Simple code is easier for you and your team to understand.
          * **Easier Maintenance & Debugging:** Fewer moving parts means fewer places for bugs to hide.
          * **Faster Development:** You solve the problem directly instead of building unnecessary scaffolding.
      * **Trade-offs:**
          * **"Naive" Simplicity:** Being overly simplistic can sometimes ignore legitimate future requirements, leading to a rewrite later. The key is to be "as simple as possible, but no simpler."

7.  **How to Discuss in an Interview**
    "When designing a new API endpoint or service, my first question is always, 'What is the simplest solution that correctly solves the problem?' I apply the KISS principle to avoid over-engineering, which ensures my code is easy for the team to maintain and helps us deliver value more quickly."

-----

### DRY (Don't Repeat Yourself)

1.  **Principle & Mnemonic**
    **Don't Repeat Yourself (DRY)**

2.  **Core Idea**
    Every piece of logic or knowledge in your system should have one single, authoritative, and unambiguous representation.

3.  **Problem It Prevents**
    This principle prevents **code duplication**. When logic is copied and pasted, any future change to that logic must be manually applied to every single copy. This is highly error-prone, leads to inconsistencies, and is a maintenance nightmare (a code smell known as "shotgun surgery").

4.  **Simple Analogy**
    Imagine you're writing a cookbook. Instead of creating a single recipe for "Basic Tomato Sauce" and referring to it from other recipes like "Lasagna" and "Pizza," you write out the *entire* tomato sauce recipe inside both the lasagna and pizza chapters. If you later decide to improve your sauce recipe, you have to remember to update it in both places, and you might forget one. 🍅

5.  **Python Backend Example (Violation vs. Solution)**

      * **Violation:** Two different API endpoints for creating a user and updating a user's profile both contain the same email validation logic.

        ```python
        # violation_dry.py
        import re

        def create_user(username: str, email: str):
            # Email validation logic
            if not re.match(r"[^@]+@[^@]+\.[^@]+", email):
                raise ValueError("Invalid email format")
            # ... proceed to create user in database
            print(f"User '{username}' created with email '{email}'.")

        def update_user_profile(username: str, new_email: str):
            # --- DUPLICATED LOGIC ---
            if not re.match(r"[^@]+@[^@]+\.[^@]+", new_email):
                raise ValueError("Invalid email format")
            # ... proceed to update user's email
            print(f"User '{username}' email updated to '{new_email}'.")
        ```

      * **Explanation:** The regular expression and the validation check for the email format are duplicated. If the validation rule changes (e.g., to use a more robust library), you have to change it in two places.

      * **Solution:** Extract the duplicated logic into a single utility function.

        ```python
        # solution_dry.py
        import re

        def is_email_valid(email: str) -> bool:
            """A single, authoritative function for email validation."""
            return bool(re.match(r"[^@]+@[^@]+\.[^@]+", email))

        def create_user(username: str, email: str):
            if not is_email_valid(email):
                raise ValueError("Invalid email format")
            print(f"User '{username}' created with email '{email}'.")

        def update_user_profile(username: str, new_email: str):
            if not is_email_valid(new_email):
                raise ValueError("Invalid email format")
            print(f"User '{username}' email updated to '{new_email}'.")
        ```

6.  **Benefits & Trade-offs**

      * **Benefits:**
          * **High Maintainability:** Logic is defined in one place, making updates easy and reliable.
          * **Reduced Bugs:** Eliminates the risk of inconsistent logic across the application.
          * **Improved Readability:** Replaces blocks of repeated code with a call to a well-named function.
      * **Trade-offs:**
          * **Premature Abstraction:** Abstracting too early or incorrectly can be worse than duplication. If two pieces of code look similar but represent different business concepts, combining them can create a confusing and tightly coupled abstraction.

7.  **How to Discuss in an Interview**
    "During code reviews, I always look for opportunities to apply the DRY principle. If I see the same block of validation logic in two different services, I'll recommend extracting it into a shared utility or a domain model method. This makes our codebase much easier to maintain and reduces the chance of bugs when requirements change."

-----

### YAGNI (You Ain't Gonna Need It)

1.  **Principle & Mnemonic**
    **You Ain't Gonna Need It (YAGNI)**

2.  **Core Idea**
    Do not add functionality or code until it is actively required and necessary.

3.  **Problem It Prevents**
    YAGNI fights against **speculative generality** and **over-engineering**. It prevents you from wasting time and effort building features that are never used. Unused code (dead code) adds complexity, increases the surface area for bugs, and makes the system harder to understand and navigate.

4.  **Simple Analogy**
    You are packing for a weekend trip to the beach. YAGNI means you pack shorts, a swimsuit, and sunscreen because that's what you know you'll need. Violating YAGNI is packing snow boots and a parka "just in case" a freak blizzard hits the beach in summer. You've wasted effort and added clutter for a problem you don't have. 🏖️

5.  **Python Backend Example (Violation vs. Solution)**

      * **Violation:** A developer is building a simple blogging API. The current requirement is to export articles as JSON. The developer decides to add XML and PDF export functionality "because we might need it in the future."

        ```python
        # violation_yagni.py
        def export_article(article_data: dict, format: str = "json"):
            if format == "json":
                # ... implemented JSON export logic ...
                print("Exporting article to JSON.")
            elif format == "xml":
                # TODO: Implement this later
                print("XML export is not yet supported.")
                pass # Unnecessary code path
            elif format == "pdf":
                # TODO: Implement this later
                print("PDF export is not yet supported.")
                pass # Unnecessary code path
            else:
                raise ValueError("Unsupported format")
        ```

      * **Explanation:** The code includes branches for `xml` and `pdf` that do nothing. This is dead code based on a speculative future need. It clutters the function signature and the logic.

      * **Solution:** Implement only the functionality that is currently required.

        ```python
        # solution_yagni.py
        def export_article_to_json(article_data: dict):
            """Exports article data to JSON. Simple and direct."""
            # ... implemented JSON export logic ...
            print("Exporting article to JSON.")

        # When the requirement for XML arises, we can add a new function
        # or refactor using the Open/Closed Principle (more on that later!).
        ```

6.  **Benefits & Trade-offs**

      * **Benefits:**
          * **Maximizes Focus:** Keeps development focused on delivering current, required features.
          * **Reduces Code Bloat:** The codebase remains lean, clean, and relevant.
          * **Faster Delivery:** You're not spending time on features that provide no immediate value.
      * **Trade-offs:**
          * **Potential for Refactoring:** Can sometimes lead to more significant refactoring later if a correctly anticipated feature is harder to integrate than expected. This is often a good trade-off, as it's better to refactor for a *real* requirement than to build for a *speculative* one.

7.  **How to Discuss in an Interview**
    "I follow the YAGNI principle by focusing my implementation tightly on the user story's acceptance criteria. While I design my code to be extensible for the future, I avoid building concrete features or adding complex configurations until they are explicitly needed. This keeps our velocity high and our codebase simple."

-----

## Part 2: The SOLID Principles

SOLID is a mnemonic acronym for five design principles that are foundational to building understandable, maintainable, and flexible object-oriented systems. Mastering them is non-negotiable for a senior engineer.

-----

### S - Single Responsibility Principle (SRP)

1.  **Principle & Mnemonic**
    **Single Responsibility Principle (SRP)**

2.  **Core Idea**
    A class should have only one reason to change, meaning it should have only one primary job or responsibility.

3.  **Problem It Prevents**
    SRP fights against creating massive, tightly-coupled classes (often called **God Objects**). When a class does too many things (e.g., handles business logic, database access, and sending emails), a change in one of those responsibilities can inadvertently break the others. These classes have **low cohesion** and are extremely difficult to test in isolation.

4.  **Simple Analogy**
    A Swiss Army knife is a classic violation of SRP. It's a knife, a screwdriver, a can opener, and scissors all in one. If you want to redesign the scissors to be more ergonomic, you have to modify the entire tool, risking the functionality of the knife. The SRP-compliant solution is a toolbox containing a separate, dedicated screwdriver, a separate knife, etc. Each tool has a single responsibility. 🛠️

5.  **Python Backend Example (Violation vs. Solution)**

      * **Violation:** A single `OrderManager` class that processes the order, charges the customer's credit card, and sends a confirmation email.

        ```python
        # violation_srp.py
        class OrderManager:
            def process_order(self, order_data: dict, credit_card_info: dict):
                # Responsibility 1: Business Logic
                print("Processing order...")
                # ... logic to validate items, calculate total ...

                # Responsibility 2: Payment Gateway Integration
                print("Charging credit card...")
                # ... code to connect to Stripe API and charge card ...

                # Responsibility 3: Notification
                print("Sending confirmation email...")
                # ... code to format and send an email ...
                
                print("Order processed successfully.")
        ```

      * **Explanation:** This class has three distinct reasons to change: 1) The business rules for order processing might change. 2) The payment gateway might switch from Stripe to PayPal. 3) The email notification template or service might change.

      * **Solution:** Break the responsibilities into separate, focused classes.

        ```python
        # solution_srp.py
        class PaymentService:
            def charge_card(self, credit_card_info: dict, amount: float):
                print(f"Charging card for ${amount} via Stripe...")

        class NotificationService:
            def send_confirmation_email(self, email: str, order_id: str):
                print(f"Sending confirmation for order {order_id} to {email}...")

        class OrderProcessor:
            # This class now coordinates the other focused classes.
            def __init__(self, payment_service: PaymentService, notification_service: NotificationService):
                self.payment_service = payment_service
                self.notification_service = notification_service

            def process_order(self, order_data: dict, credit_card_info: dict):
                print("Processing order...")
                # ... business logic to validate order and calculate total ...
                total = 129.99
                
                self.payment_service.charge_card(credit_card_info, total)
                self.notification_service.send_confirmation_email("customer@email.com", "ORD-123")
                print("Order processed successfully.")
        ```

6.  **Benefits & Trade-offs**

      * **Benefits:**
          * **Improved Testability:** Each class can be unit-tested in isolation (e.g., you can test `PaymentService` without needing order data).
          * **Low Coupling:** Changes to the payment system don't affect the notification system.
          * **High Cohesion & Readability:** Classes are smaller, focused, and easier to understand.
      * **Trade-offs:**
          * **Increased Number of Classes:** Can lead to a larger number of small files and classes, which might seem like overkill for very simple applications.

7.  **How to Discuss in an Interview**
    "When designing a system, I apply SRP by identifying the core responsibilities or 'axes of change.' For instance, in a user registration service, I would separate the request handling (API layer), business logic validation (service layer), and database persistence (repository layer) into distinct classes. This makes the system far more maintainable and easier to test."

-----

### O - Open/Closed Principle (OCP)

1.  **Principle & Mnemonic**
    **Open/Closed Principle (OCP)**

2.  **Core Idea**
    Software entities should be **open for extension** (you can add new functionality) but **closed for modification** (you shouldn't have to change existing, working code).

3.  **Problem It Prevents**
    Violating OCP means that every time you need to add a new feature, you must go back and change existing, tested code. This is risky and often leads to long, brittle `if/elif/else` chains that grow over time and are prone to breaking existing functionality.

4.  **Simple Analogy**
    Think of your phone. It is "closed" for modification; you can't just open it up and solder a new camera onto the mainboard. However, it is "open" for extension through its App Store. You can add new functionality (apps) without changing the phone's core operating system. 📱

5.  **Python Backend Example (Violation vs. Solution)**

      * **Violation:** A reporting service that generates reports in different formats using an `if/elif` block.

        ```python
        # violation_ocp.py
        class ReportGenerator:
            def generate(self, data: list, format: str):
                if format == 'csv':
                    print("Generating CSV report...")
                    # ... CSV generation logic ...
                    return "csv_content"
                elif format == 'json':
                    print("Generating JSON report...")
                    # ... JSON generation logic ...
                    return "json_content"
                # To add PDF, I MUST MODIFY THIS METHOD!
                else:
                    raise ValueError("Unsupported format")
        ```

      * **Explanation:** This `generate` method must be modified every time a new report format is required. This violates the "closed for modification" rule.

      * **Solution:** Use abstraction (e.g., an abstract base class) and polymorphism. This is a classic application of the **Strategy design pattern**.

        ```python
        # solution_ocp.py
        from abc import ABC, abstractmethod

        class ReportExporter(ABC):
            @abstractmethod
            def export(self, data: list) -> str:
                pass

        class CsvExporter(ReportExporter):
            def export(self, data: list) -> str:
                print("Exporting data to CSV format...")
                return "csv_content"

        class JsonExporter(ReportExporter):
            def export(self, data: list) -> str:
                print("Exporting data to JSON format...")
                return "json_content"

        # Now, if we need a PDF exporter... we just add a new class!
        class PdfExporter(ReportExporter):
            def export(self, data: list) -> str:
                print("Exporting data to PDF format...")
                return "pdf_content"

        class ReportGenerator:
            def generate(self, data: list, exporter: ReportExporter):
                # This method no longer needs to be modified.
                return exporter.export(data)
        ```

6.  **Benefits & Trade-offs**

      * **Benefits:**
          * **Stability:** You can add new features without risking breaking existing, tested code.
          * **Maintainability:** Code is more organized and less prone to becoming a tangled mess of conditional logic.
          * **Flexibility:** Promotes a "plug-in" style architecture.
      * **Trade-offs:**
          * **Increased Complexity:** Requires adding a layer of abstraction (like an interface or ABC), which can be overkill for parts of the system that are very unlikely to change.

7.  **How to Discuss in an Interview**
    "I would use the Open/Closed Principle to handle something like different notification types (Email, SMS, Push). I'd define a `Notifier` interface with a `send` method. Then I'd create `EmailNotifier` and `SmsNotifier` classes. This way, adding a `PushNotifier` in the future only requires adding a new class, not changing the core notification-sending logic, making the system stable and extensible."

-----

### L - Liskov Substitution Principle (LSP)

1.  **Principle & Mnemonic**
    **Liskov Substitution Principle (LSP)**

2.  **Core Idea**
    If S is a subtype of T, then objects of type T may be replaced with objects of type S without altering any of the desirable properties of the program.

3.  **Problem It Prevents**
    LSP prevents broken or surprising inheritance hierarchies. A violation occurs when a subclass cannot be used interchangeably with its parent class. This forces you to write code that checks the type of an object (`if isinstance(...)`) before using it, which is a code smell and a violation of the Open/Closed Principle. It leads to subtle, hard-to-find bugs.

4.  **Simple Analogy**
    Imagine you have a remote control (the base type `T`) with a "volume up" button. This remote should work for any television (the subtype `S`). If you buy a new "WeirdTV" where the "volume up" button actually changes the channel, that `WeirdTV` violates LSP. It's not a substitutable replacement for a standard TV because it breaks the expected behavior. 📺

5.  **Python Backend Example (Violation vs. Solution)**

      * **Violation:** A classic example involves a `Rectangle` base class and a `Square` subclass.

        ```python
        # violation_lsp.py
        class Rectangle:
            def __init__(self, width: int, height: int):
                self.width = width
                self.height = height
            
            def set_width(self, width: int):
                self.width = width

            def set_height(self, height: int):
                self.height = height

            @property
            def area(self) -> int:
                return self.width * self.height

        class Square(Rectangle):
            def __init__(self, size: int):
                super().__init__(size, size)
            
            # A square must maintain its invariant: width == height
            def set_width(self, width: int):
                self.width = width
                self.height = width

            def set_height(self, height: int):
                self.width = height
                self.height = height

        def test_area(rect: Rectangle):
            rect.set_width(5)
            rect.set_height(4)
            # We EXPECT the area to be 5 * 4 = 20
            print(f"Expected area: 20, Actual area: {rect.area}")
            assert rect.area == 20
        ```

      * **Explanation:** The `test_area` function works perfectly for a `Rectangle`. However, if you pass a `Square` object to it, `set_height(4)` will also set the width to 4, making the area 16, not 20. The `Square` is not substitutable for the `Rectangle` because it changes the behavior of the parent's methods in a surprising way.

      * **Solution:** Re-evaluate the inheritance model. A square "is-a" rectangle mathematically, but it has different behavioral invariants. Inheritance might be the wrong tool here. A better approach might be to have a `Shape` protocol and treat them as distinct shapes, favoring composition or different abstractions.

        ```python
        # solution_lsp.py
        from abc import ABC, abstractmethod

        class Shape(ABC):
            @abstractmethod
            def area(self) -> int:
                pass

        class Rectangle(Shape):
            def __init__(self, width: int, height: int):
                self.width = width
                self.height = height
            
            @property
            def area(self) -> int:
                return self.width * self.height

        class Square(Shape):
            def __init__(self, size: int):
                self.size = size
            
            @property
            def area(self) -> int:
                return self.size ** 2
        # Now, you cannot substitute them in a way that breaks behavior,
        # because they don't share the problematic set_width/set_height methods.
        ```

6.  **Benefits & Trade-offs**

      * **Benefits:**
          * **Reliable Inheritance:** Guarantees that subclasses will behave predictably.
          * **Code Reusability:** Enables safe polymorphism, allowing you to write generic code that works with a family of types.
          * **Avoids Runtime Errors:** Prevents subtle bugs caused by unexpected subclass behavior.
      * **Trade-offs:**
          * **Stricter Design:** Requires more careful thought during the design phase to ensure that your inheritance hierarchies are behaviorally correct.

7.  **How to Discuss in an Interview**
    "For me, LSP is about ensuring behavioral consistency in an inheritance chain. A subclass must not just share an interface, but also honor the parent's 'contract.' For example, if a parent method guarantees it will never throw an exception, a subclass should not override it to throw one. This principle ensures that polymorphism in our system is safe and predictable."

-----

### I - Interface Segregation Principle (ISP)

1.  **Principle & Mnemonic**
    **Interface Segregation Principle (ISP)**

2.  **Core Idea**
    Clients should not be forced to depend on interfaces (or methods) that they do not use. It's better to have many small, specific interfaces than one large, general-purpose one.

3.  **Problem It Prevents**
    ISP combats **"fat" or "polluted" interfaces**. When a class implements a large interface with methods it doesn't need, it's forced to provide empty or error-raising implementations. This adds clutter and means that a change to an unused part of the interface can still force a change in the implementing class.

4.  **Simple Analogy**
    Imagine going to a restaurant where the only menu is a giant, 50-page book containing breakfast, lunch, dinner, drinks, and dessert. If you only want to order a coffee, you are still forced to deal with this massive, mostly irrelevant interface. The ISP solution is to have separate, smaller menus for drinks, for food, etc. ☕

5.  **Python Backend Example (Violation vs. Solution)**

      * **Violation:** A single, "fat" `IMachine` abstract class for all types of office machines.

        ```python
        # violation_isp.py
        from abc import ABC, abstractmethod

        class IMachine(ABC):
            @abstractmethod
            def print_document(self, doc: str): pass

            @abstractmethod
            def fax_document(self, doc: str): pass
            
            @abstractmethod
            def staple_document(self, doc: str): pass

        class ModernPrinter(IMachine):
            def print_document(self, doc: str): print(f"Printing {doc}")
            def fax_document(self, doc: str): raise NotImplementedError("Fax not supported")
            def staple_document(self, doc: str): raise NotImplementedError("Staple not supported")
        ```

      * **Explanation:** The `ModernPrinter` is a machine, but it only needs the `print_document` method. It is forced to implement `fax_document` and `staple_document`, methods it does not use.

      * **Solution:** Segregate the fat interface into smaller, more focused ones.

        ```python
        # solution_isp.py
        from abc import ABC, abstractmethod

        class IPrinter(ABC):
            @abstractmethod
            def print_document(self, doc: str): pass

        class IFax(ABC):
            @abstractmethod
            def fax_document(self, doc: str): pass

        class IStapler(ABC):
            @abstractmethod
            def staple_document(self, doc: str): pass

        # A simple printer only implements the interface it needs.
        class ModernPrinter(IPrinter):
            def print_document(self, doc: str): print(f"Printing {doc}")

        # A multi-function machine can implement several small interfaces.
        class AllInOneMachine(IPrinter, IFax, IStapler):
            def print_document(self, doc: str): print(f"Printing {doc}")
            def fax_document(self, doc: str): print(f"Faxing {doc}")
            def staple_document(self, doc: str): print(f"Stapling {doc}")
        ```

6.  **Benefits & Trade-offs**

      * **Benefits:**
          * **Decoupled Systems:** Clients only know about the methods that are relevant to them.
          * **High Cohesion:** Interfaces are small and focused on a single role.
          * **Easier Implementation:** Classes don't have to implement unnecessary methods.
      * **Trade-offs:**
          * **Interface Proliferation:** Can lead to a large number of very small interfaces, which might add some organizational overhead if taken to an extreme.

7.  **How to Discuss in an Interview**
    "I apply ISP by creating small, role-based interfaces for my services. For instance, instead of a single large `IUserService` with methods for authentication, profile updates, and password resets, I might create `IAuthenticator`, `IProfileManager`, etc. A component that only needs to log a user in would then only depend on `IAuthenticator`, minimizing its dependencies and making the system more modular."

-----

### D - Dependency Inversion Principle (DIP)

1.  **Principle & Mnemonic**
    **Dependency Inversion Principle (DIP)**

2.  **Core Idea**
    High-level modules (e.g., business logic) should not depend on low-level modules (e.g., database, file system); both should depend on abstractions (interfaces).

3.  **Problem It Prevents**
    This principle prevents building rigid, **tightly coupled** systems. When your core business logic directly depends on a concrete implementation detail like `PostgresDatabase`, you cannot easily change that detail (e.g., switch to MongoDB) without rewriting your business logic. It also makes your high-level code nearly impossible to unit-test without a real database.

4.  **Simple Analogy**
    When you plug a lamp into a wall, the lamp (high-level module) doesn't care about the specific electrical wiring inside the wall (low-level module). The lamp depends on a standard interface: the wall socket (the abstraction). This "inverts" the dependency—the high-level lamp and the low-level wiring both depend on the socket abstraction. This allows you to plug any compliant device into any compliant socket. 🔌

5.  **Python Backend Example (Violation vs. Solution)**

      * **Violation:** A high-level `DataAnalysisService` is directly coupled to a low-level `PostgresReportRepo`.

        ```python
        # violation_dip.py
        class PostgresReportRepo:
            def get_reports(self) -> list[str]:
                print("Getting reports from PostgreSQL...")
                return ["Postgres Report 1", "Postgres Report 2"]

        class DataAnalysisService:
            def __init__(self):
                # The high-level module directly instantiates the low-level one. TIGHT COUPLING!
                self.repo = PostgresReportRepo()

            def analyze_reports(self):
                reports = self.repo.get_reports()
                print("Analyzing reports...")
        ```

      * **Explanation:** `DataAnalysisService` is completely dependent on `PostgresReportRepo`. You cannot test this service without a running Postgres database, and you cannot use it to analyze reports from a different source.

      * **Solution:** Introduce an abstraction (an interface) that both modules depend on. Use **Dependency Injection** to provide the concrete implementation.

        ```python
        # solution_dip.py
        from abc import ABC, abstractmethod

        # 1. Define the Abstraction (Interface)
        class IReportRepo(ABC):
            @abstractmethod
            def get_reports(self) -> list[str]:
                pass

        # 2. Low-level module depends on the abstraction
        class PostgresReportRepo(IReportRepo):
            def get_reports(self) -> list[str]:
                print("Getting reports from PostgreSQL...")
                return ["Postgres Report 1", "Postgres Report 2"]

        class MongoReportRepo(IReportRepo):
            def get_reports(self) -> list[str]:
                print("Getting reports from MongoDB...")
                return ["Mongo Report A", "Mongo Report B"]

        # 3. High-level module depends on the abstraction
        class DataAnalysisService:
            def __init__(self, repo: IReportRepo): # The dependency is "injected"
                self.repo = repo

            def analyze_reports(self):
                reports = self.repo.get_reports()
                print(f"Analyzing {len(reports)} reports...")

        # --- At the application's entry point (the "composition root") ---
        postgres_repo = PostgresReportRepo()
        analysis_service = DataAnalysisService(repo=postgres_repo)
        analysis_service.analyze_reports()

        mongo_repo = MongoReportRepo()
        analysis_service_2 = DataAnalysisService(repo=mongo_repo)
        analysis_service_2.analyze_reports()
        ```

6.  **Benefits & Trade-offs**

      * **Benefits:**
          * **Maximum Decoupling:** Components are fully independent and interchangeable.
          * **Excellent Testability:** You can easily "inject" a mock repository (`MockReportRepo`) to unit-test the `DataAnalysisService` in complete isolation.
          * **High Flexibility:** The entire architecture becomes a set of plug-and-play components.
      * **Trade-offs:**
          * **Increased Boilerplate:** Requires defining abstract interfaces and wiring up dependencies (often done with a Dependency Injection framework), which can feel like over-engineering for very simple projects.

7.  **How to Discuss in an Interview**
    "Dependency Inversion is crucial for building testable and flexible microservices. I ensure my service layer (high-level logic) depends only on abstract interfaces for its dependencies, like an `IUserRepository`. We then use a dependency injection container to provide the concrete implementation, such as a `DynamoUserRepository`, at runtime. This allows us to unit-test our business logic with mocks and switch out our database technology without ever touching the service code itself."