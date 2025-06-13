# Clean Code: Minimized

This document contains a condensed version of notes on Clean Code, with redundant principles removed and examples preserved.

## Core Tenets
- **Leave it cleaner than you found it**: Continuously improve the code you work on.
- **Write dirty, then clean**: It's a process. No one writes perfect code from the start.
- **Root of all evil**: Don't prematurely optimize.
- **The Boy Scout Rule**: Leave the campground cleaner than you found it.

---

## Naming
A name should reveal its intent, explaining why it exists, what it does, and how it is used. If it needs a comment, the name is not good enough.

- **Use Intention-Revealing Names**: The name should be descriptive and answer big questions.
- **Avoid Disinformation**: Don't use inconsistent spellings or misleading names.
- **Avoid Noise Words**: Words like `info`, `data`, `a`, `an`, `the` add no value. `moneyAmount` is the same as `money`.
- **Make Names Pronounceable**: If you can't say it, you can't discuss it.
- **Class Names**: Should be nouns or noun phrases (e.g., `Customer`, `AddressParser`). Avoid verbs or manager-type words.
- **Method Names**: Should be verbs or verb phrases (e.g., `deletePage`, `save`).
- **Use Standard Conventions**: Use `get`, `set`, and `is` for accessors, mutators, and predicates. Use standard nomenclature agreed upon by the team.

    ```
    // Good accessor (getter), mutator (setter), and predicate (is) names
    public class Customer {
        private String name;
        private boolean active;

        // Accessor (getter)
        public String getName() { return name; }

        // Mutator (setter)
        public void setName(String name) { this.name = name; }

        // Predicate (boolean check)
        public boolean isActive() { return active; }
    }
    ```
- **Static Factory Methods**: For overloaded constructors, use static factory methods with descriptive names.
    ```
    // Confusing
    Rectangle rect = new Rectangle(50, 30);

    // Clear and self-documenting
    Rectangle rect = Rectangle.withWidthAndHeight(50, 30);
    ```
- **Name Length and Scope**: Use long, descriptive names for long scopes. Short, simple names (like `i`) are fine for very short scopes.
- **Avoid Encodings**: Don't use prefixes or type encodings in names (e.g., `m_`, `f_`, `vis_`). Modern tools make this unnecessary.
- **Describe Side-Effects**: Names should encompass any side-effects a function might have.

---

## Functions
Functions should be small, and even smaller than that. They should do one thing, do it well, and do it only.

- **Small Size**: Functions should hardly ever be 20 lines long.
- **Single Responsibility Principle (SRP)**: A function should have only one reason to change.
- **One Level of Abstraction**: Code within a function should all be at the same level of abstraction, one level below the function's name.

    ```
    // BAD: Mixed levels of abstraction
    function renderUserProfile(user) {
        let html = '<div class="profile">';
        html += `<img src="${user.avatar}" width="50" height="50">`; // HTML details
        html += formatUserName(user.name); // Higher-level concept
        html += '</div>';
        return html;
    }

    // GOOD: Consistent abstraction levels
    function renderUserProfile(user) {
        const profileElements = [
            renderAvatar(user),
            renderUserName(user)
        ];
        return wrapInProfileContainer(profileElements);
    }

    // Lower-level implementation details
    function renderAvatar(user) { /* ... */ }
    function renderUserName(user) { /* ... */ }
    function wrapInProfileContainer(elements) { /* ... */ }
    ```

### Arguments
- **Argument Count**: The ideal number of arguments is zero (niladic), followed by one (monadic) and two (dyadic). Avoid three or more arguments where possible. If a function needs many arguments, consider wrapping them into a dedicated parameter object or class.
- **No Flag Arguments**: Passing a boolean into a function is a terrible practice. It implies the function does more than one thing. Create separate functions instead.
    ```
    // BAD - flag argument
    function notify(user, isUrgent) {
        if (isUrgent) { /* ... */ }
    }
    notify(user, true); // unclear what true means

    // GOOD - separate functions
    function sendUrgentNotification(user) { /* ... */ }
    function sendNotification(user) { /* ... */ }
    ```
- **Avoid Output Arguments**: Arguments should be inputs. If a function needs to change state, it should change the state of its owning object.
    ```
    // BAD - modifying input argument
    function appendGreeting(name, result) {
        result.push(`Hello ${name}!`);
    }

    // GOOD - function returns new value
    function getGreeting(name) {
        return `Hello ${name}!`;
    }
    ```

### Function Behavior
- **No Side Effects**: A function should not have hidden behaviors that change the state of variables outside its own scope.
- **Command Query Separation**: A function should either *do* something (change state) or *answer* something (return information), but not both.
    ```
    // BAD - both changes state and returns value
    class UserManager {
        addUser(user) {
            this.users.push(user);
            return this.users.length; // Why return this?
        }
    }

    // GOOD - each function does one thing
    class UserManager {
        addUser(user) { this.users.push(user); }
        getUserCount() { return this.users.length; }
    }
    ```
- **Prefer Exceptions to Error Codes**: Returning error codes leads to nested checking and couples the caller to the error codes. Exceptions are cleaner.
    ```
    // BAD - returning error codes
    function connectToDatabase() {
        if (!config) return -1;
        if (!network) return -2;
        return 0;
    }

    // GOOD - using exceptions
    function connectToDatabase() {
        if (!config) throw new ConfigError('Database config missing');
        if (!network) throw new NetworkError('Network unavailable');
    }
    ```
- **Dead Functions**: Methods that are never called should be deleted.

---

## Comments
The only truly good comment is the comment you found a way not to write. Comments are often used to compensate for a failure to express something in the code itself.

- **Avoid Redundant Comments**: Don't comment on things that the code already explains clearly.
- **Maintain Your Comments**: Inaccurate or obsolete comments are far worse than no comments at all.
- **Don't Use Comments for Metadata**: Use version control for history, author names, etc.
- **Write Good Comments**: If a comment is necessary, make it concise, well-written, and unambiguous.
- **Don't Leave Commented-Out Code**: Delete it. Version control will remember it if you need it back.
- **Use TODO for Future Work**: `//TODO` comments are acceptable for explaining future intentions or degenerate implementations.
- **Use for Warnings**: Comments can be useful to warn other programmers about certain consequences.

---

## Formatting
- **Vertical Separation**: Keep related concepts vertically close to each other. Declare variables near their usage. Private functions should be defined just below their first call.
- **Line Length**: Keep lines reasonably short. 80-120 characters is a good guideline.
- **Organized Classes**: A class should be organized with a standard convention:
    1. Public static constants
    2. Private static variables
    3. Private instance variables
    4. Public functions
    5. Private functions (close to the public functions that call them).

---

## Objects and Data Structures
- **Symmetry**: Objects hide their data behind abstractions and expose functions that operate on that data. Data structures expose their data and have no meaningful functions.
- **Law of Demeter**: A module should not know about the internals of the objects it manipulates. Talk to your immediate friends, not to strangers. Avoid long chains of method calls.
    ```
    // BAD - chain of calls
    user.getWallet().getPaymentCard().charge(100);

    // GOOD - hiding internal structure
    class User {
        chargeAmount(amount) {
            this.wallet.processPayment(amount);
        }
    }
    ```
- **Data Transfer Objects (DTOs)**: A DTO is a class with public variables and no functions, useful for communicating with databases or parsing messages.
- **Procedural vs. OO Trade-off**: Procedural code (using data structures) makes it easy to add new functions without changing data structures. OO code makes it easy to add new classes without changing existing functions. Choose the right tool for the job based on the kind of flexibility you need.
    ```
    // OO Approach - Easy to add new shapes (classes)
    class Circle { area() { /*...*/ } }
    class Square { area() { /*...*/ } }
    // but hard to add new functions (e.g., perimeter()) without changing all classes.

    // Procedural Approach - Easy to add new functions
    function area(shape) { /*...*/ }
    function perimeter(shape) { /*...*/ }
    // but hard to add new shapes without changing all functions.
    ```

---

## Boundaries
- **Keep Boundary Interfaces Internal**: Avoid passing boundary interfaces like `Map` around your system. If you use a `Map`, keep it inside the class where it is used. Avoid returning it from, or accepting it as an argument to, public APIs. This reduces coupling to a specific implementation.
    ```
    // GOOD - keep Map internal
    class UserManager {
        constructor() {
            this._users = new Map();
        }
        // Good: return plain array/object
        getAllUsers() {
            return Array.from(this._users.values());
        }
        // Good: accept simple parameters
        addUser(id, user) {
            this._users.set(id, user);
        }
    }
    ```
- **Wrap Third-Party APIs**: Wrapping third-party APIs is a best practice. It minimizes your dependencies, makes it easier to mock for testing, and allows you to switch libraries in the future with less effort.
    ```
    // GOOD - wrapped third-party API
    class HttpClient {
        async get(path) {
            // abstracts away the specific library (e.g., axios, fetch)
        }
    }

    class UserService {
        constructor(httpClient) { this.client = httpClient; }
        async getUser(id) { return this.client.get(`/users/${id}`); }
    }
    ```

---

## Error Handling
- **Use Exceptions**: Prefer exceptions over returning error codes.
- **Provide Context with Exceptions**: Create informative error messages and pass them with your exceptions.
    ```
    // BAD - uninformative errors
    throw new Error('Invalid');

    // GOOD - descriptive errors with context
    throw new ValidationError('User name is required', { field: 'name', userId: user.id });
    ```
- **Define Exceptions by Caller's Needs**: Create specific exception classes for different failure cases so the caller can handle them appropriately.
    ```
    // GOOD - specific exceptions for different failure cases
    class ConnectionError extends DatabaseError {}
    class AuthenticationError extends DatabaseError {}

    try {
        connectToDatabase();
    } catch (error) {
        if (error instanceof ConnectionError) {
            retryConnection();
        } else if (error instanceof AuthenticationError) {
            alertAdmin();
        }
    }
    ```
- **Extract try/catch Blocks**: Error handling is "one thing," so a function that handles errors should do nothing else. Extract the bodies of `try` and `catch` blocks into their own functions.
    ```
    // BAD - mixing logic with error handling
    function saveUser(user) {
        try {
            validateUser(user);
            updateDatabase(user);
            sendEmail(user);
        } catch (error) {
            logError(error);
        }
    }

    // GOOD - separated concerns
    function saveUser(user) {
        try {
            performSave(user);
        } catch (error) {
            handleSaveError(error, user);
        }
    }
    ```
- **Don't Return Null**: Returning `null` leads to a cascade of `null` checks.
- **Don't Pass Null**: Avoid passing `null` into methods unless an API specifically requires it.
- **Use the Special Case Pattern**: Instead of returning `null`, return a special case object that handles the exceptional behavior (e.g., a `NullUser` object).
    ```
    // GOOD - Special Case Pattern (Null Object Pattern)
    class NullCustomer {
        get name() { return 'Guest'; }
        get discount() { return 0; }
    }
    function getCustomer(id) {
        const customer = database.find(id);
        return customer || new NullCustomer();
    }
    ```
- **Transactional Error Handling**: When an operation can fail, ensure your `catch` block leaves the system in a consistent state. Don't leave operations half-done.
    ```
    // GOOD - maintains consistent state
    function transferMoney(from, to, amount) {
        try {
            from.deduct(amount);
            to.add(amount);
        } catch (error) {
            // Rollback the deduction if transfer fails
            from.add(amount);
            throw new Error('Transfer failed');
        }
    }
    ```

---

## Classes
- **Small Size & Single Responsibility**: Classes should be small and have only one responsibility (one reason to change).
    ```
    // BAD - class has multiple responsibilities
    class User {
        validateEmail() {}
        saveToDatabase() {}
        generateReport() {}
    }

    // GOOD - single responsibility per class
    class User { /* ... */ }
    class UserRepository { /* ... */ }
    class UserReporter { /* ... */ }
    ```
- **Cohesion**: When classes lose cohesion (methods work with only a subset of instance variables), split them into smaller, more focused classes.
- **Open/Closed Principle**: Classes should be open for extension but closed for modification. You should be able to add new functionality without changing existing code.
- **Dependency Inversion Principle (DIP)**: Depend on abstractions, not on concrete details. This isolates your code from changes.
    ```
    // BAD - dependent on concrete details
    class ReportGenerator {
        generate() {
            const data = MySQL.query('...'); // Concrete
            const pdf = PDFWriter.write(data); // Concrete
        }
    }

    // GOOD - isolated via interfaces/abstractions
    class ReportGenerator {
        constructor(dataSource, formatter) { /* ... */ }
        generate() {
            const data = this.dataSource.getData();
            const report = this.formatter.format(data);
        }
    }
    ```
- **Duplication (DRY - Don't Repeat Yourself)**: The root of most evil in software. Eliminate it wherever you find it.
- **Clutter**: Remove unused variables, dead code, and anything that adds no value.
- **Inconsistency**: Be consistent in naming, patterns, and structure.
- **Misplaced Responsibility**: Put code where a reader would naturally expect to find it.
- **Prefer Polymorphism to If/Else or Switch/Case**: Use polymorphism to handle type-based behavior. Avoid long `if/else` or `switch` chains that check an object's type.
- **Base Classes Shouldn't Know About Derivatives**: A base class should be independent of the classes that inherit from it. This allows for modularity and easier maintenance.
- **Keep Interfaces Small**: Good modules have minimal interfaces that do a lot. Hide implementation details and limit the number of exposed methods and variables to reduce coupling.
- **Artificial Coupling**: Don't force unrelated concepts together. General-purpose utilities or enums should not be placed inside specific classes, as this creates unnecessary dependencies.
- **Understand the Algorithm**: Don't just write code that "works." Strive to deeply understand the problem so you can write a solution that is clean, obvious, and provably correct.
- **Make Logical Dependencies Physical**: If one module relies on another, that dependency should be made explicit. Don't rely on implicit knowledge of another module's internals; ask for what you need.
- **Structure Over Convention**: Enforcing rules with structure (e.g., abstract methods in a base class) is more robust than relying on conventions (e.g., naming patterns).
- **Encapsulate Conditionals**: Extract complex boolean logic into well-named functions.
- **Avoid Negative Conditionals**: `if (isOpen)` is better than `if (!isClosed)`.
- **Use Explanatory Variables**: Break down complex calculations into intermediate variables with descriptive names.
- **Replace Magic Numbers with Named Constants**: Avoid unexplained literal values in your code.
- **Be Precise**: Don't make naive assumptions. Handle edge cases, use appropriate data types (e.g., no floats for money), and check for nulls.
- **Structure Over Convention**: Where possible, use language features to enforce rules rather than relying on naming conventions.
- **Encapsulate Boundary Conditions**: Don't let boundary conditions leak all over your code. Wrap them in a single place.
- **Feature Envy**: A method is "envious" of the data of another class, accessing it frequently. This often means the method belongs in that other class.
- **Inappropriate Static**: Use `static` only for functions that have no instance-level dependencies and will never need polymorphic behavior. When in doubt, prefer instance methods.

---

## Systems
- **Separate Construction and Use**: Separate the startup process (object construction and wiring) from the runtime logic. Use Dependency Injection to provide dependencies from the outside.
- **Lazy Initialization**: Defer the creation of an object until the first time it is needed. This can improve performance for resource-intensive objects.
    ```
    // GOOD - initializes only once when needed
    class ExpensiveService {
        getData() {
            if (!this._database) {
                // Creates connection only the first time
                this._database = new Database();
            }
            return this._database.query();
        }
    }
    ```
- **Aspect-Oriented Programming (AOP)**: Use AOP-like patterns (such as decorators) to handle cross-cutting concerns (e.g., logging, authentication, transactions) in a modular way, keeping the core business logic clean (in POJOs).
- **Use Plain Old Objects (POJOs)**: Write domain logic using simple objects, decoupled from architectural concerns. This makes the code easier to test and maintain.
- **Big Design Up Front (BDUF) is Harmful**: Don't design everything before implementing. Let the system evolve organically based on real needs.
- **Keep Configurable Data at High Levels**: Pass configuration data down from high-level modules instead of hardcoding it in low-level functions.

---

## Tests
- **The Three Laws of TDD**:
    1. Write a failing unit test before production code.
    2. Write only enough of a test to fail.
    3. Write only enough production code to pass the failing test.
- **Tests must be F.I.R.S.T.**:
    - **Fast**: Run quickly.
    - **Independent**: Don't depend on each other.
    - **Repeatable**: Run in any environment.
    - **Self-Validating**: Return a boolean pass/fail.
    - **Timely**: Written just before the code that makes them pass.
- **One Assert per Test**: As a guideline, each test function should have only one assert statement. This forces you to test a single concept at a time. Multiple asserts are acceptable if they test closely related aspects of a single concept.
- **Given-When-Then**: Structure tests using the Given-When-Then convention for clarity.
    ```
    // GOOD - Given-When-Then format
    test('processed order should apply discount', () => {
        // Given
        const order = new Order(100);
        order.addDiscount(10);

        // When
        order.process();

        // Then
        expect(order.total).toBe(90);
    })
    ```
- **One Concept per Test**: A test function should test a single concept.
- **Use a Coverage Tool**: Identify gaps in your testing strategy.
- **Don't Skip Trivial Tests**: Their documentary value is higher than their cost.
- **Test Boundary Conditions**: Pay special attention to edge cases.
- **Encapsulate Conditionals**: Extract complex boolean logic into well-named functions.
- **Avoid Negative Conditionals**: `if (isOpen)` is better than `if (!isClosed)`.
- **Hidden Temporal Couplings**: If functions must be called in a specific order, make that dependency explicit. Pass the result of one function as an argument to the next to create a clear flow.
    ```
    // BAD: Hidden temporal coupling
    class DataProcessor {
        process(data) {
            this.prepareGradient();
            this.calculateSplines();
            this.processData(data);
        }
    }

    // GOOD: Explicit temporal coupling
    class DataProcessor {
        process(data) {
            const gradient = this.prepareGradient();
            const splines = this.calculateSplines(gradient);
            return this.processData(splines, data);
        }
    }
    ```
- **Use Explanatory Variables**: Break down complex calculations into intermediate variables with descriptive names.
- **Replace Magic Numbers with Named Constants**: Avoid unexplained literal values in your code.
- **Be Precise**: Don't make naive assumptions. Handle edge cases, use appropriate data types (e.g., no floats for money), and check for nulls.
- **Domain-Specific Testing Language (DSL)**: For complex scenarios, consider creating a set of functions and utilities that act as a DSL for your tests, making them more readable and expressive.
- **Failure Patterns are Revealing**: Look for patterns in failing tests. They often point to the root cause of the problem.

---

## Code Smells & Heuristics (Summary of Key Principles)
This section summarizes key principles, some of which were also mentioned in the main chapters.

- **Duplication (DRY - Don't Repeat Yourself)**: The root of most evil in software. Eliminate it wherever you find it.
- **Clutter**: Remove unused variables, dead code, and anything that adds no value.
- **Inconsistency**: Be consistent in naming, patterns, and structure.
- **Misplaced Responsibility**: Put code where a reader would naturally expect to find it.
- **Prefer Polymorphism to If/Else or Switch/Case**: Use polymorphism to handle type-based behavior. Avoid long `if/else` or `switch` chains that check an object's type.
- **Base Classes Shouldn't Know About Derivatives**: A base class should be independent of the classes that inherit from it. This allows for modularity and easier maintenance.
- **Keep Interfaces Small**: Good modules have minimal interfaces that do a lot. Hide implementation details and limit the number of exposed methods and variables to reduce coupling.
- **Artificial Coupling**: Don't force unrelated concepts together. General-purpose utilities or enums should not be placed inside specific classes, as this creates unnecessary dependencies.
- **Understand the Algorithm**: Don't just write code that "works." Strive to deeply understand the problem so you can write a solution that is clean, obvious, and provably correct.
- **Make Logical Dependencies Physical**: If one module relies on another, that dependency should be made explicit. Don't rely on implicit knowledge of another module's internals; ask for what you need.
- **Structure Over Convention**: Enforcing rules with structure (e.g., abstract methods in a base class) is more robust than relying on conventions (e.g., naming patterns).
- **Encapsulate Conditionals**: Extract complex boolean logic into well-named functions.
- **Avoid Negative Conditionals**: `if (isOpen)` is better than `if (!isClosed)`.
- **Use Explanatory Variables**: Break down complex calculations into intermediate variables with descriptive names.
- **Replace Magic Numbers with Named Constants**: Avoid unexplained literal values in your code.
- **Be Precise**: Don't make naive assumptions. Handle edge cases, use appropriate data types (e.g., no floats for money), and check for nulls.
- **Encapsulate Boundary Conditions**: Don't let boundary conditions leak all over your code. Wrap them in a single place.
- **Feature Envy**: A method is "envious" of the data of another class, accessing it frequently. This often means the method belongs in that other class.
- **Inappropriate Static**: Use `static` only for functions that have no instance-level dependencies and will never need polymorphic behavior. When in doubt, prefer instance methods.
