# Object-Oriented Programming Basics

## Part 1: Object Relationships

Object relationships define how different classes and their instances interact with each other. Understanding these distinctions is crucial for designing flexible and maintainable software.

### Inheritance (is-a)

#### 1. Concept Definition

Inheritance is a mechanism that allows a new class (called a subclass or child class) to acquire the properties and behaviors (methods) of an existing class (called a superclass or parent class). This creates a hierarchical relationship where the subclass specializes the superclass.

#### 2. Simple Analogy

A Manager is-a type of Employee. A manager has all the common attributes of an employee, like a name and an employee ID. However, a manager also has specialized attributes and behaviors, such as the team they manage. The Manager class can inherit the general features from the Employee class and add its own specific ones.

#### 3. Practical Technical Example

Consider a web application with different user types. You could have a base User class that handles common functionality like logging in and managing a profile. Then, you can have specialized user types like AdminUser and ContentCreator that inherit from User. An AdminUser might have elevated permissions and override a method to bypass certain restrictions, while a ContentCreator might have an additional method to publish articles.

#### 4. Python Code Implementation

This example shows a base User and an AdminUser. The AdminUser inherits from User, uses super() to extend the parent's __init__, and overrides the can_access_resource() method to grant universal access.

```python
# Part 1: Inheritance Code Example
import datetime
from typing import List

class User:
    """A base class for a user in the system."""
    def __init__(self, username: str, email: str):
        self.username = username
        self.email = email
        self.last_login = None

    def login(self) -> None:
        """Updates the user's last login time."""
        self.last_login = datetime.datetime.now()
        print(f"User '{self.username}' logged in at {self.last_login}.")

    def can_access_resource(self, resource_id: str) -> bool:
        """Checks if a user can access a specific resource."""
        # In a real system, this would check a permissions list.
        # For this example, regular users can access only 'public' resources.
        print(f"Checking access for '{self.username}' on resource '{resource_id}'...")
        return resource_id.startswith("public")

class AdminUser(User):
    """A specialized user with administrative privileges."""
    def __init__(self, username: str, email: str, permission_level: int):
        # Use super() to call the parent's __init__ method
        # This ensures the parent's initialization logic is executed.
        super().__init__(username, email)
        self.permission_level = permission_level

    # Method Overriding: This method replaces the parent's version.
    def can_access_resource(self, resource_id: str) -> bool:
        """Admins have access to all resources, regardless of ID."""
        print(f"Admin '{self.username}' attempting access on '{resource_id}'...")
        return True # Admins can do anything!
    
    def view_system_logs(self) -> List[str]:
        """An admin-specific method."""
        print(f"Admin '{self.username}' is viewing system logs.")
        return ["Log entry 1: System started", "Log entry 2: User login failed"]

# --- Demonstration ---
regular_user = User("john_doe", "john.doe@example.com")
admin_user = AdminUser("super_admin", "admin@example.com", 99)

regular_user.login()
admin_user.login()

print("\n--- Testing Resource Access ---")
print(f"Regular user access to 'public_report_01': {regular_user.can_access_resource('public_report_01')}")
print(f"Regular user access to 'admin_panel_01': {regular_user.can_access_resource('admin_panel_01')}")

print("-" * 20)

print(f"Admin user access to 'public_report_01': {admin_user.can_access_resource('public_report_01')}")
print(f"Admin user access to 'admin_panel_01': {admin_user.can_access_resource('admin_panel_01')}")

# Call an admin-specific method
admin_user.view_system_logs()
```

#### 5. When to Use It (Trade-offs)

Pros ✅: Promotes code reuse and establishes a clear, logical hierarchy. It's the natural choice when you have a clear "is-a" relationship and want to share common behavior.  
Cons ❌: Creates tight coupling. Changes in the parent class can unexpectedly break child classes (this is known as the "fragile base class" problem). Overuse can lead to deep, complex inheritance chains that are difficult to debug and maintain.  
Best For: Modeling genuine is-a relationships where subclasses are true specializations of the parent class.

---

### Composition (has-a)

#### 1. Concept Definition

Composition is a relationship where one class (the "container" or "whole") is composed of one or more instances of other classes (the "components" or "parts"). The key characteristic is that the container manages the lifecycle of its components. When the container is destroyed, its components are destroyed as well.

#### 2. Simple Analogy

A Car has-an Engine. The Engine is an integral part of the Car. The Car object is responsible for creating and managing its own Engine object. The Engine doesn't exist independently; if the Car is scrapped, the Engine goes with it.

#### 3. Practical Technical Example

In a data engineering pipeline, a DataPipeline object is responsible for executing a series of steps to process data. The DataPipeline owns its steps, such as a DataLoader, a DataTransformer, and a DataExporter. The DataPipeline class instantiates these components in its constructor, and the components' existence is entirely managed by the pipeline itself.

#### 4. Python Code Implementation

This example shows a DataPipeline that is composed of a loader and an exporter. Notice how the DataPipeline's __init__ method creates the instances of CSVLoader and JSONExporter, managing their entire lifecycle.

```python
# Part 1: Composition Code Example
import pandas as pd
from io import StringIO
from typing import Dict, Any

# --- Component Classes ---
class CSVLoader:
    """A component to load data from a CSV source."""
    def load(self, csv_string: str) -> pd.DataFrame:
        print("CSVLoader: Loading data...")
        return pd.read_csv(StringIO(csv_string))

class JSONExporter:
    """A component to export data to a JSON format."""
    def export(self, data: pd.DataFrame) -> str:
        print("JSONExporter: Exporting data...")
        return data.to_json(orient='records')

# --- Container Class (composed of the components) ---
class DataPipeline:
    """
    A container that owns and manages its components (loader, exporter).
    This demonstrates Composition.
    """
    def __init__(self, loader_type: str, exporter_type: str):
        # The DataPipeline creates and owns these component objects.
        # Their lifecycle is tied to the pipeline's lifecycle.
        if loader_type == 'csv':
            self._loader = CSVLoader()
        else:
            raise ValueError("Unsupported loader type")
            
        if exporter_type == 'json':
            self._exporter = JSONExporter()
        else:
            raise ValueError("Unsupported exporter type")
        print("DataPipeline configured and ready.")
        
    def process(self, input_data: str) -> str:
        """Runs the full data processing pipeline."""
        print("\n--- Starting Pipeline Process ---")
        # 1. Delegate loading to the loader component
        loaded_data = self._loader.load(input_data)
        
        # 2. Perform some transformation (logic owned by the pipeline)
        print("DataPipeline: Applying transformation (adding a status column)...")
        loaded_data['status'] = 'processed'
        
        # 3. Delegate exporting to the exporter component
        exported_data = self._exporter.export(loaded_data)
        print("--- Pipeline Process Finished ---")
        return exported_data

# --- Demonstration ---
# The pipeline's components are created when the pipeline itself is created.
pipeline = DataPipeline(loader_type='csv', exporter_type='json')

csv_data = "name,age\nAlice,30\nBob,25"
json_output = pipeline.process(csv_data)

print("\nFinal JSON Output:")
print(json_output)
```

#### 5. When to Use It (Trade-offs)

Pros ✅: Very flexible, as components can be easily swapped (e.g., replace CSVLoader with ParquetLoader). It promotes loose coupling and follows the design principle "favor composition over inheritance."  
Cons ❌: Can lead to a larger number of smaller classes and more boilerplate code to wire everything together in the container.  
Best For: Modeling "part-of" or "owns-a" relationships where the container is responsible for the creation and destruction of its parts.

---

### Aggregation (has-a)

#### 1. Concept Definition

Aggregation is a weaker form of composition. It also models a "has-a" relationship, but the container and component have independent lifecycles. The container holds a reference to one or more external objects but does not own them. If the container is destroyed, the component objects continue to exist.

#### 2. Simple Analogy

A university Department has-a collection of Professors. The department has a list of professors who work in it. However, if the department is restructured or closed, the professors (as objects) still exist and can be assigned to other departments. The department doesn't "own" the professors.

#### 3. Practical Technical Example

In a game development context, a Team might aggregate multiple Player objects. The Player objects exist independently in the game world. They can be assigned to a Team, removed from it, or join another team. Destroying the Team object (e.g., if the team is disbanded) does not destroy the Player objects. The Team simply holds references to players that were created elsewhere.

#### 4. Python Code Implementation

Here, Player objects are created first. The Team class is then initialized with a list of these pre-existing Player objects. The Team doesn't create the players, it just aggregates them.

```python
# Part 1: Aggregation Code Example
from __future__ import annotations
from typing import List

# --- Component Class (with an independent lifecycle) ---
class Player:
    """Represents a player in the game."""
    def __init__(self, name: str, skill_level: int):
        self.name = name
        self.skill_level = skill_level

    def __repr__(self) -> str:
        return f"Player(name='{self.name}', skill={self.skill_level})"

# --- Container Class (aggregates external objects) ---
class Team:
    """
    Represents a team of players. It holds references to Player objects
    but does not control their lifecycle. This is Aggregation.
    """
    def __init__(self, team_name: str, members: List[Player]):
        self.name = team_name
        # The Team aggregates a list of players that were created elsewhere.
        self.members = members

    def get_average_skill(self) -> float:
        """Calculates the team's average skill level."""
        if not self.members:
            return 0.0
        total_skill = sum(player.skill_level for player in self.members)
        return total_skill / len(self.members)
        
    def __repr__(self) -> str:
        return f"Team(name='{self.name}', members={len(self.members)})"

# --- Demonstration ---
# 1. Create Player objects. They exist independently.
player1 = Player("Ryu", 95)
player2 = Player("Ken", 92)
player3 = Player("Chun-Li", 94)

all_players = [player1, player2, player3]
print(f"All players available in the game: {all_players}")

# 2. Create a Team and pass the existing Player objects to it.
# The team 'dragons' now aggregates player1 and player2.
dragons_team = Team("Dragons", [player1, player2])
print(f"\nCreated team: {dragons_team}")
print(f"Team skill average: {dragons_team.get_average_skill():.2f}")

# 3. The Player objects can be used elsewhere, e.g., in another team.
tigers_team = Team("Tigers", [player2, player3]) # Ken is on two teams!
print(f"\nCreated team: {tigers_team}")

# 4. Deleting a team does NOT delete the players.
print("\nDisbanding the Dragons team...")
del dragons_team

# player1 and player2 still exist and are accessible.
print(f"Player '{player1.name}' still exists after team was deleted.")
print(f"The Tigers team still has its members: {tigers_team.members}")
```

#### 5. When to Use It (Trade-offs)

Pros ✅: Offers the most flexibility and very loose coupling. Allows objects to be shared among multiple containers and managed independently.  
Cons ❌: The shared nature of components can sometimes make state management more complex. You need to be aware of which parts of your code hold references to which objects.  
Best For: Modeling "uses-a" or "has-a" relationships where the components must have a lifecycle independent of any single container, or where they need to be shared.

---

## Part 2: Advanced OOP Concepts

These concepts move beyond basic class structure to enable more robust, flexible, and scalable software designs.

### Abstract Base Classes (ABCs) & Abstract Methods

#### 1. Concept Definition

An Abstract Base Class (ABC) is a special type of class that is not meant to be instantiated directly. Instead, it serves as a blueprint or a contract for its subclasses. It can define abstract methods, which are methods declared without an implementation. Any concrete (i.e., non-abstract) subclass is required to provide an implementation for all inherited abstract methods.

#### 2. Simple Analogy

Think of an electrical wall socket as an ABC. The socket defines a standard interface (the shape and position of the holes). You cannot use the socket by itself; it's just a contract. Any appliance (a Lamp, a Toaster, a LaptopCharger) that wants to use it must have a plug that conforms to that interface. The ABC ensures that any "pluggable" device will work as expected. 🔌

#### 3. Practical Technical Example

Imagine you're building a notification system that can send alerts via different channels: email, SMS, and push notifications. To ensure consistency, you can define a NotificationService ABC with an abstract method called send(message). Then, concrete classes like EmailService, SMSService, and PushService must each implement their own version of the send method. This guarantees that any notification service you create will have the correct interface, preventing runtime errors.

#### 4. Python Code Implementation

This example uses Python's abc module to define a NotificationService contract. Notice that trying to instantiate the ABC directly raises a TypeError, while the concrete subclasses must implement send.

```python
# Part 2: ABC Code Example
from abc import ABC, abstractmethod
from typing import Dict

class NotificationService(ABC):
    """
    An Abstract Base Class that defines the contract for all
    notification services. It cannot be instantiated.
    """
    @abstractmethod
    def send(self, recipient: str, message: str) -> Dict[str, str]:
        """
        Sends a notification.
        Subclasses MUST implement this method.
        """
        pass

# --- Concrete Implementations ---

class EmailService(NotificationService):
    """A concrete implementation for sending emails."""
    def send(self, recipient: str, message: str) -> Dict[str, str]:
        # In a real app, this would use a library like smtplib.
        print(f"Sending EMAIL to '{recipient}': '{message}'")
        return {"status": "success", "channel": "email", "recipient": recipient}

class SMSService(NotificationService):
    """A concrete implementation for sending SMS messages."""
    def send(self, recipient: str, message: str) -> Dict[str, str]:
        # In a real app, this would call a service like Twilio.
        print(f"Sending SMS to '{recipient}': '{message}'")
        return {"status": "success", "channel": "sms", "recipient": recipient}

# --- Demonstration ---

# 1. You CANNOT instantiate an ABC.
try:
    service = NotificationService()
except TypeError as e:
    print(f"Error as expected: {e}")

# 2. Create instances of the concrete classes.
email_sender = EmailService()
sms_sender = SMSService()

# 3. Use them. Both have the .send() method, as guaranteed by the ABC.
email_result = email_sender.send("user@example.com", "Your order has shipped!")
sms_result = sms_sender.send("+1234567890", "Your package is out for delivery.")

print(email_result)
print(sms_result)

# This would fail if a subclass doesn't implement the abstract method.
# class IncompleteService(NotificationService):
#     def some_other_method(self): pass
# try:
#     incomplete = IncompleteService() # This would raise a TypeError
# except TypeError as e:
#     print(f"\nError for incomplete class: {e}")
```

#### 5. When to Use It (Trade-offs)

Pros ✅: Enforces a consistent API across multiple subclasses, making the system more predictable and easier to reason about. It acts as excellent documentation for developers.  
Cons ❌: Adds a layer of abstraction that can be overkill for simple class hierarchies. It is more rigid than informal interfaces (like duck typing).  
Best For: Frameworks, libraries, or large applications where you need to guarantee that different components conform to a specific interface. It's perfect for defining plugins or extensible systems.

---

### Polymorphism

#### 1. Concept Definition

Polymorphism (from Greek, meaning "many shapes") is the principle that allows objects of different classes to be treated as objects of a common superclass. It's the ability to use a single, common interface to represent different underlying forms (data types). In practice, this means you can call the same method on different objects, and each object will respond in its own specialized way.

#### 2. Simple Analogy

Think of the "speak" command for different animals. If you tell a Dog to "speak," it will bark. If you tell a Cat to "speak," it will meow. If you tell a Duck to "speak," it will quack. The interface (speak()) is the same, but the behavior is specific to each object type. 🗣️

#### 3. Practical Technical Example

Building on our previous NotificationService ABC, we can write a polymorphic function that sends a batch of notifications without needing to know the specific type of each service. The function can iterate through a list containing EmailService and SMSService objects and call the .send() method on each one. The correct implementation of send is automatically called based on the object's actual class. This is inheritance-based polymorphism.
Duck Typing is a more informal type of polymorphism common in Python. The idea is: "If it walks like a duck and it quacks like a duck, then it must be a duck." An object's suitability for an operation is determined by the presence of certain methods and properties, rather than its explicit type or inheritance.

#### 4. Python Code Implementation

This code demonstrates both inheritance-based polymorphism (with our NotificationService classes) and duck typing. The broadcast_notification function works with any object that has a .send() method.

```python
# Part 2: Polymorphism Code Example
from abc import ABC, abstractmethod
from typing import List, Dict, Any

# --- Using the ABC and classes from the previous example ---
class NotificationService(ABC):
    @abstractmethod
    def send(self, recipient: str, message: str) -> Dict[str, str]:
        pass

class EmailService(NotificationService):
    def send(self, recipient: str, message: str) -> Dict[str, str]:
        print(f"Sending EMAIL to '{recipient}': '{message}'")
        return {"status": "success", "channel": "email"}

class SMSService(NotificationService):
    def send(self, recipient: str, message: str) -> Dict[str, str]:
        print(f"Sending SMS to '{recipient}': '{message}'")
        return {"status": "success", "channel": "sms"}

# --- A class that does NOT inherit but has the right "shape" (for Duck Typing) ---
class SlackService:
    """This service does not inherit from NotificationService,
    but it implements a 'send' method with a compatible signature."""
    def send(self, channel_id: str, text: str) -> Dict[str, Any]:
        print(f"Posting to Slack channel '{channel_id}': '{text}'")
        return {"ok": True, "channel": channel_id}

# --- A Polymorphic Function ---
def broadcast_notification(services: List[Any], recipients: List[str], message: str):
    """
    This function leverages polymorphism. It can work with any object
    that has a 'send' method, regardless of its class or inheritance.
    """
    print("\n--- Broadcasting Notification ---")
    for service in services:
        for recipient in recipients:
            # The same .send() call works for different object types.
            # This is polymorphism in action!
            service.send(recipient, message)

# --- Demonstration ---
# 1. Inheritance-based Polymorphism
email_sender = EmailService()
sms_sender = SMSService()

# The 'notification_channels' list contains objects of different types,
# but they share the NotificationService interface.
notification_channels: List[NotificationService] = [email_sender, sms_sender]

# This function works because both objects are guaranteed to have .send()
broadcast_notification(notification_channels, ["user@example.com", "+12345"], "System maintenance tonight.")

# 2. Duck Typing Polymorphism
slack_sender = SlackService()
all_channels: List[Any] = [email_sender, sms_sender, slack_sender]

# The function STILL works, even with SlackService which doesn't inherit.
# As long as it "quacks" like a notification service (has a .send method),
# Python is happy. This is Duck Typing.
broadcast_notification(all_channels, ["#general", "alerts@dev.co"], "Deployment successful!")
```

#### 5. When to Use It (Trade-offs)

Pros ✅: Creates extremely flexible and decoupled code. It allows you to write generic algorithms and systems that can work with any new objects you create, as long as they adhere to the expected interface.  
Cons ❌: With duck typing, the lack of an explicit contract (like an ABC) can make code harder to reason about, and errors (like a missing method) will only appear at runtime. Static analysis tools may also struggle to catch these issues.  
Best For: Any situation where you want to process a collection of different objects in a uniform way. It's the foundation of many powerful software design patterns.

---

### Encapsulation

#### 1. Concept Definition

Encapsulation is the practice of bundling data (attributes) and the methods that operate on that data within a single unit, the class. A crucial part of this is information hiding: restricting direct access to an object's internal state and exposing only a controlled, public interface. This prevents external code from making arbitrary, and potentially invalid, changes to the object's state.

#### 2. Simple Analogy

Think of a bank account. Your account balance is a piece of data. You cannot just walk into the bank's server room and change the number in the database. Instead, you must use a public interface: the ATM. The ATM exposes controlled methods like deposit(amount) and withdraw(amount). These methods contain logic to validate the transaction (e.g., checking if you have sufficient funds). The internal balance is encapsulated and protected. 🏦

#### 3. Practical Technical Example

In a user account management system, a UserAccount class should encapsulate sensitive information like the user's password hash and account status (e.g., active, locked). Direct modification should be forbidden. Instead, the class provides public methods like change_password(old_password, new_password) and lock_account(). We can use Python's naming conventions (_protected and __private) to signal that attributes are internal and use the @property decorator to provide controlled, read-only access to certain state.

#### 4. Python Code Implementation

This UserAccount class demonstrates encapsulation:
_status: A "protected" member, accessible by subclasses but not intended for external use.
__password_hash: A "private" member, its name is mangled by Python to make it hard to access from outside.
status: A read-only @property that acts like an attribute but is really a controlled method call.
Public methods (change_password, lock) provide the only way to modify the internal state.

```python
# Part 2: Encapsulation Code Example
import hashlib

class UserAccount:
    """
    Manages a user's account, encapsulating sensitive data.
    """
    def __init__(self, username: str, password: str):
        self.username = username # Public attribute
        
        # "Protected" attribute: convention suggests this shouldn't be
        # accessed directly from outside the class or its subclasses.
        self._status = "active"
        
        # "Private" attribute: Python performs name mangling on this
        # (it becomes _UserAccount__password_hash) to make accidental
        # access very difficult.
        self.__password_hash = self._hash_password(password)
        
        print(f"Account for '{self.username}' created.")

    def _hash_password(self, password: str) -> str:
        """A helper method for internal use only."""
        return hashlib.sha256(password.encode()).hexdigest()

    # Using @property to create a read-only public interface for a protected attribute.
    # This is a "getter".
    @property
    def status(self) -> str:
        """Provides controlled read-only access to the account status."""
        return self._status

    def check_password(self, password: str) -> bool:
        """Public method to verify a password."""
        return self.__password_hash == self._hash_password(password)

    def lock_account(self) -> None:
        """Public method to safely change the internal state."""
        print(f"Locking account for '{self.username}'.")
        self._status = "locked"

# --- Demonstration ---
account = UserAccount("jane_doe", "MySecretPa$$w0rd")

# 1. Accessing public and read-only property
print(f"Username: {account.username}")
print(f"Initial Status: {account.status}") # Accessing the @property

# 2. Using public methods to interact with the object
print(f"Password check correct: {account.check_password('MySecretPa$$w0rd')}")
print(f"Password check incorrect: {account.check_password('wrongpassword')}")

# 3. We CANNOT easily change the status directly.
try:
    account.status = "hacked"
except AttributeError as e:
    print(f"\nError as expected: {e}") # Raises AttributeError: can't set attribute

# Instead, we must use the public interface.
account.lock_account()
print(f"New Status: {account.status}")

# 4. Attempting to access "private" attribute fails.
try:
    print(account.__password_hash)
except AttributeError as e:
    print(f"\nError accessing private attribute: {e}")

# We can still access the "mangled" name, but this is a strong signal to avoid it.
# This proves Python's privacy isn't absolute, but based on convention.
# print(account._UserAccount__password_hash)
```

#### 5. When to Use It (Trade-offs)

Pros ✅: Protects the integrity of an object's data by preventing uncontrolled access. It makes code safer and easier to maintain because the internal implementation can change without affecting external code, as long as the public interface remains the same.  
Cons ❌: Can sometimes lead to more boilerplate code (e.g., creating properties for many attributes). Python's privacy is not strictly enforced like in Java or C++, which relies on developer discipline.  
Best For: Virtually every class you write. Any class that holds important state should encapsulate it, exposing only the necessary methods to interact with that state in a controlled way.

---

## Bonus: Deep Dive into the `@property` Decorator

`@property` decorator is a fundamental and incredibly useful feature of Python's object-oriented system that promotes writing clean, maintainable, and "Pythonic" code.

### The Problem: Direct, Uncontrolled Attribute Access

Imagine we're creating a simple class to represent a temperature. We might start like this:

```python
class Temperature:
    def __init__(self, celsius: float):
        self.celsius = celsius

# We can access and change the attribute directly
temp = Temperature(25.0)
print(f"Temperature is {temp.celsius}°C") # Output: Temperature is 25.0°C

# But what happens if someone provides an invalid value?
temp.celsius = -300.0 # This is colder than absolute zero!
print(f"New temperature is {temp.celsius}°C") # Output: New temperature is -300.0°C
```

The problem here is that `self.celsius` is just a public number. We have **no control** over the values it can be set to. The state of our object can become invalid, which can lead to bugs later on.

-----

### The "Old School" Solution: Getter and Setter Methods

In many other programming languages (like Java or C++), the solution is to make the attribute "private" and create explicit methods to get and set its value.

```python
class Temperature:
    def __init__(self, celsius: float):
        self._celsius = celsius # Convention: '_' means "internal use"

    def get_celsius(self) -> float:
        return self._celsius

    def set_celsius(self, value: float) -> None:
        if value < -273.15:
            raise ValueError("Temperature cannot be below absolute zero (-273.15°C).")
        self._celsius = value

# This works, but it's clunky to use...
temp = Temperature(25.0)
print(f"Temperature is {temp.get_celsius()}°C") # Ugh, method call for reading
temp.set_celsius(30.0) # Method call for writing
# temp.set_celsius(-300.0) # This would correctly raise a ValueError
```

This approach, while functional, is not considered "Pythonic." It's verbose. Having to call `get_...()` and `set_...()` for what feels like a simple attribute makes the code less readable.

-----

### The Pythonic Solution: `@property`

The `@property` decorator is the elegant Pythonic solution. It lets you create methods that can be accessed **as if they were attributes**.

Think of it like the dashboard of a car. The speedometer looks like a simple gauge (an attribute), but behind it is a complex method constantly calculating your speed. `@property` gives us this magic.

It consists of three parts, which are implemented using three related decorators:

1.  **Getter (`@property`)**: For getting the attribute's value.
2.  **Setter (`@<name>.setter`)**: For setting the attribute's value (this is where we add validation\!).
3.  **Deleter (`@<name>.deleter`)**: For deleting the attribute (`del obj.attribute`).

#### 1\. The Getter: Creating a Read-Only Property

Let's start by just creating a read-only property. The `@property` decorator is placed above a method, turning that method into a getter for a property with the same name.

```python
class Temperature:
    def __init__(self, celsius: float):
        # We store the actual data in a "private" attribute
        if celsius < -273.15:
            raise ValueError("Temperature cannot be below absolute zero (-273.15°C).")
        self._celsius = celsius

    @property
    def celsius(self) -> float:
        """This is the 'getter' method for our property."""
        print("Getting value...")
        return self._celsius

# --- Usage ---
temp = Temperature(25.0)

# Notice we access it like an attribute, NOT a method! No () needed.
# The `celsius` method is automatically called.
print(f"Temperature is {temp.celsius}°C")
# Output:
# Getting value...
# Temperature is 25.0°C

# If we try to set it, we get an error because we haven't defined a setter.
try:
    temp.celsius = 30.0
except AttributeError as e:
    print(f"\nError as expected: {e}") # AttributeError: can't set attribute
```

We now have a **read-only property**. We've successfully controlled *getting* the value, but we can't change it yet.

#### 2\. The Setter: Enabling Controlled Writing

To allow writing to the property, we need to define a setter method. The decorator for this is special: it's named after the property itself.

```python
class Temperature:
    def __init__(self, celsius: float):
        # The setter is called even in the __init__!
        self.celsius = celsius

    @property
    def celsius(self) -> float:
        """The getter for the 'celsius' property."""
        print("Getting value...")
        return self._celsius

    @celsius.setter
    def celsius(self, value: float) -> None:
        """The setter for the 'celsius' property."""
        print(f"Setting value to {value}...")
        if value < -273.15:
            raise ValueError("Temperature cannot be below absolute zero (-273.15°C).")
        # The actual data is stored in a private attribute
        self._celsius = value

# --- Usage ---
temp = Temperature(25.0) # The setter is called here!
print(f"Temperature is {temp.celsius}°C")

# Now we can assign a new value, just like a regular attribute.
# The setter method is automatically called.
temp.celsius = 32.5
print(f"New temperature is {temp.celsius}°C")

# Let's try an invalid value. The validation logic in the setter will catch it.
try:
    temp.celsius = -300.0
except ValueError as e:
    print(f"\nError as expected: {e}")
```

This is the complete, elegant solution\!

  * The user of our class interacts with `temp.celsius` as a simple attribute.
  * Behind the scenes, our getter and setter methods are being called, allowing us to run validation or any other logic we need.

#### 3. The Deleter: Cleaning Up (Advanced/Less Common)

Finally, you can define what happens when someone uses the `del` keyword on your property. This is useful for cleanup or resetting a value.

```python
class Temperature:
    # ... (getter and setter from above) ...
    @property
    def celsius(self) -> float:
        return self._celsius
    @celsius.setter
    def celsius(self, value: float):
        if value < -273.15:
            raise ValueError("Cannot be below absolute zero.")
        self._celsius = value

    @celsius.deleter
    def celsius(self) -> None:
        """The deleter for the 'celsius' property."""
        print("Deleting value (resetting to 0)...")
        self._celsius = 0.0

# --- Usage ---
temp = Temperature(25.0)
print(f"Temperature is {temp.celsius}°C")

# This calls the @celsius.deleter method
del temp.celsius
print(f"Temperature after deletion is {temp.celsius}°C")
```

-----

### Key Benefits and "When to Use It"

1.  **Clean Public API**: Your class is easy to use (`obj.x = 10`). You hide the complex validation logic from the user, which is the core principle of **Encapsulation**.

2.  **Computed Properties**: A property doesn't have to just store and return a value. It can compute it on the fly. This is extremely powerful.

    ```python
    class Rectangle:
        def __init__(self, width: float, height: float):
            self.width = width
            self.height = height

        @property
        def area(self) -> float:
            """This property is calculated on the fly and is read-only."""
            return self.width * self.height

    rect = Rectangle(10, 5)
    print(f"The area is: {rect.area}") # Looks like an attribute, but is a calculation!
    # rect.area = 100 # This would raise an AttributeError
    ```

3.  **Maintainability (The BIGGEST Advantage)**: This is crucial. You can start your class with simple public attributes. Later, if you realize you need to add validation or logic, you can **change the attribute into a property without changing any of the code that uses your class**.

      * **Phase 1 (Simple):** `self.email = email`
      * **Phase 2 (Needs Validation):** Change `email` to a property with a setter that checks if the email address is valid. All the external code (`user.email = '...'`) continues to work perfectly. This is impossible with the `get/set` method approach without refactoring everything.

In summary, **`@property` gives you the simple syntax of direct attribute access while providing the power and control of method calls.** It is the standard, Pythonic way to manage an object's state.