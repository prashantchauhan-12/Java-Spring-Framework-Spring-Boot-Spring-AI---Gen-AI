# Advanced Java — Revision Notes

> Compiled from the *Advance Java* course transcript, expanded with full working code, explanations, and extra details useful for interview/revision prep.

---

## Table of Contents
1. [Mini Project – Console Quiz Application](#1-mini-project--console-quiz-application)
2. [Abstract Classes & the `abstract` Keyword](#2-abstract-classes--the-abstract-keyword)
3. [Anonymous Inner Classes](#3-anonymous-inner-classes)
4. [`@Override` Annotation](#4-override-annotation)
5. [Interfaces](#5-interfaces)
6. [Enums](#6-enums)
7. [Annotations](#7-annotations)
8. [Functional Interfaces & SAM](#8-functional-interfaces--sam)
9. [Lambda Expressions](#9-lambda-expressions)
10. [Exception Handling](#10-exception-handling)
11. [`throws` Keyword & Checked Exceptions (IOException)](#11-throws-keyword--checked-exceptions-ioexception)
12. [Multithreading](#12-multithreading)
13. [Thread Safety & `synchronized`](#13-thread-safety--synchronized)
14. [Collections Framework](#14-collections-framework)
15. [Generics](#15-generics)
16. [Comparable vs Comparator](#16-comparable-vs-comparator)
17. [Stream API](#17-stream-api)
18. [Predicate, Function, Consumer, Supplier, BinaryOperator](#18-predicate-function-consumer-supplier-binaryoperator)
19. [Optional Class](#19-optional-class)
20. [Method References & Constructor References](#20-method-references--constructor-references)
21. [Quick-Revision Cheat Sheet](#21-quick-revision-cheat-sheet)

---

## 1. Mini Project – Console Quiz Application

The course builds a **console-based Quiz Application** incrementally as each new concept is taught. This is the running example used throughout.

**Requirements gathered:**
- Two roles: **Admin/Trainer** (creates questions) and **Player/Student** (attempts the quiz).
- A quiz has a set of questions (e.g. 5). Each right answer = 1 point.
- Every question has: an `id`, the `question` text, 4 options (`opt1`–`opt4`), and the correct `answer`.
- Later enhancements (mentioned as stretch goals): real-time score, per-question timer (10–15 sec).
- No GUI — everything runs on the console (`Scanner` based input).

**Project skeleton:**

```java
// Main.java — entry point, kept minimal on purpose
public class Main {
    public static void main(String[] args) {
        QuestionService service = new QuestionService();
        service.startQuiz();
    }
}
```

**Representing a Question — why a class instead of parallel arrays?**

Naively you could keep `int[] ids`, `String[] questions`, `String[] opt1`, etc., but that scatters
related data across many arrays and is error-prone (index `i` in one array must always match index `i`
in every other array). The OOP fix: model **one Question as one object**, then keep an array/list of
those objects.

```java
public class Question {
    private int id;
    private String question;
    private String opt1;
    private String opt2;
    private String opt3;
    private String opt4;
    private String answer;   // correct option, e.g. "opt2" or the text itself

    public Question(int id, String question, String opt1, String opt2,
                     String opt3, String opt4, String answer) {
        this.id = id;
        this.question = question;
        this.opt1 = opt1;
        this.opt2 = opt2;
        this.opt3 = opt3;
        this.opt4 = opt4;
        this.answer = answer;
    }

    // getters (and setters if needed)
    public int getId() { return id; }
    public String getQuestion() { return question; }
    public String getOpt1() { return opt1; }
    public String getOpt2() { return opt2; }
    public String getOpt3() { return opt3; }
    public String getOpt4() { return opt4; }
    public String getAnswer() { return answer; }
}
```

**QuestionService — creating questions, displaying them, running the game:**

```java
import java.util.*;

public class QuestionService {

    private List<Question> questions = new ArrayList<>();

    public QuestionService() {
        loadQuestions();      // hardcoded / seeded questions
    }

    private void loadQuestions() {
        questions.add(new Question(1, "Capital of India?", "Mumbai", "Delhi", "Chennai", "Kolkata", "opt2"));
        questions.add(new Question(2, "2 + 2 * 2 = ?", "6", "8", "4", "2", "opt1"));
        // ... add more
    }

    public void startQuiz() {
        Scanner sc = new Scanner(System.in);
        System.out.println("Are you a (1) Student or (2) Trainer?");
        int role = sc.nextInt();

        if (role == 2) {
            // trainer flow -> create/add questions
            return;
        }

        int score = 0;
        for (Question q : questions) {
            System.out.println(q.getQuestion());
            System.out.println("A. " + q.getOpt1());
            System.out.println("B. " + q.getOpt2());
            System.out.println("C. " + q.getOpt3());
            System.out.println("D. " + q.getOpt4());
            System.out.print("Your answer: ");
            String userAns = sc.next();
            if (userAns.equalsIgnoreCase(q.getAnswer())) {
                score++;
            }
        }
        System.out.println("Your total score: " + score + "/" + questions.size());
    }
}
```

This project is revisited in later sections (e.g. `Comparator`/`Comparable` to rank players, `Stream API`
to filter/aggregate scores, `Optional` to safely fetch a question by id, etc.) — that's why it's worth
keeping the `Question` class in mind while reading the rest of the notes.

---

## 2. Abstract Classes & the `abstract` Keyword

### Why abstract?
Sometimes you know a class *must* have a certain method (because conceptually the "thing" isn't
complete without it), but you don't yet know **how** to implement it. Example used in the video: a
`Car` **must** be able to `drive()` (otherwise why call it a car?), but you may not know the internal
engine details yet, while `playMusic()` you already know how to implement.

- A **normal method with an empty body still compiles and runs** — but that's misleading: it silently
  does nothing instead of forcing the subclass to actually implement it.
- `abstract` solves this: it forces any concrete subclass to provide the implementation, and it
  **prevents object creation of the abstract class itself** (you can't say `new Car()` if `Car` is
  abstract).

### Rules
- Declared with the `abstract` keyword at class level and/or method level.
- An abstract method has **no body** (just a signature, ends with `;`).
- If a class has even **one** abstract method, the class itself **must** be declared `abstract`.
- An abstract class **can** also have concrete (fully implemented) methods, constructors, fields, static
  methods.
- You **cannot instantiate** an abstract class directly — only through a subclass that implements
  (overrides) all abstract methods.
- A subclass that doesn't implement all abstract methods must itself be declared `abstract`.

```java
public abstract class Car {

    // abstract method — no implementation, subclass MUST override it
    public abstract void drive();

    // concrete method — already implemented, subclasses can use or override it
    public void playMusic() {
        System.out.println("Playing music...");
    }
}

public class Honda extends Car {
    @Override
    public void drive() {
        System.out.println("Honda is driving smoothly.");
    }
}

public class Main {
    public static void main(String[] args) {
        // Car obj = new Car();      // ❌ compile error: Car is abstract
        Car obj = new Honda();       // ✅ upcasting, allowed
        obj.drive();
        obj.playMusic();
    }
}
```

### Abstract class vs a plain class with empty methods
| | Empty-body method in normal class | Abstract method in abstract class |
|---|---|---|
| Can you instantiate the class? | Yes | No |
| Is the subclass forced to implement it? | No (easy to forget) | Yes (compile-time enforced) |
| Communicates intent? | Not clearly | Clearly — "this MUST be implemented" |

---

## 3. Anonymous Inner Classes

An **anonymous inner class** lets you create a subclass (or an implementation of an interface/abstract
class) **and** instantiate it **in a single expression**, without giving it a name — useful when you
need a one-off implementation and don't want to create a separate named class file.

```java
abstract class Car {
    public abstract void drive();
}

public class Main {
    public static void main(String[] args) {
        // No named subclass created — implementation supplied inline
        Car obj = new Car() {
            @Override
            public void drive() {
                System.out.println("Driving anonymously!");
            }
        };
        obj.drive();
    }
}
```

Works the same way for interfaces:

```java
interface Greeting {
    void sayHello();
}

public class Main {
    public static void main(String[] args) {
        Greeting g = new Greeting() {
            @Override
            public void sayHello() {
                System.out.println("Hello from anonymous class!");
            }
        };
        g.sayHello();
    }
}
```

**Key points:**
- Compiles to a class like `Main$1.class` behind the scenes.
- Can access effectively-final local variables from the enclosing scope.
- Commonly used (pre-Java 8) for event listeners, `Runnable`, `Comparator`, etc. — largely replaced by
  **lambda expressions** for functional interfaces (see §9), but still needed for abstract classes or
  interfaces with more than one method.

---

## 4. `@Override` Annotation

`@Override` tells the compiler: *"I intend this method to override a method from the superclass/
interface."*

```java
class A {
    void show() {
        System.out.println("in A show");
    }
}

class B extends A {
    @Override
    void show() {
        System.out.println("in B show");
    }
}
```

- It is **optional** — overriding works even without it, as long as the signature matches exactly.
- Its real value: **compile-time safety**. If you mistype the method name/signature (e.g. rename
  `show()` in `A` but forget to rename it consistently), without `@Override` you'd accidentally create
  an **overloaded**/unrelated method instead of an override, and the bug would only show up at runtime
  as unexpected behaviour. With `@Override`, the compiler immediately flags:
  `method does not override a method from its superclass`.
- Best practice: **always** annotate intentional overrides.

```java
class A {
    void showTheDataWhichBelongsToThisClass() { System.out.println("A"); }
}
class B extends A {
    @Override
    void showTheDataWhichBelongsToThisClas() {   // typo! missing 's'
        System.out.println("B");
    }
    // ❌ compiler error thanks to @Override — without it, this would silently
    // become a brand-new method instead of overriding A's method.
}
```

---

## 5. Interfaces

### Why do we need interfaces?
Interfaces define a **contract** ("what must be done") without dictating **how**. Any class that
`implements` an interface promises to provide the implementation for all its methods. This enables
loose coupling, multiple inheritance of type, and polymorphism.

```java
interface Vehicle {
    void drive();
    void stop();
}

class Bike implements Vehicle {
    public void drive() { System.out.println("Bike is riding"); }
    public void stop()  { System.out.println("Bike stopped"); }
}
```

Rules:
- Methods in an interface are implicitly `public abstract` (unless `default`/`static`/`private`, Java 8+).
- Fields are implicitly `public static final` (constants).
- A class can `implement` **multiple** interfaces (solves the multiple-inheritance limitation of classes).
- From Java 8 onward interfaces can have `default` and `static` methods (with bodies); Java 9 added
  `private` interface methods.

```java
interface Vehicle {
    void drive();

    default void honk() {                 // default method, Java 8+
        System.out.println("Beep beep!");
    }

    static void info() {                   // static method, Java 8+
        System.out.println("Vehicles must be able to drive.");
    }
}
```

### Types of interfaces mentioned in the course

**1. Normal interface** — has two or more abstract methods.

**2. Marker interface** — a **completely empty** interface (no methods at all). It doesn't add
behaviour directly; it "marks"/tags a class so that the compiler/JVM/framework can treat it specially
at runtime via `instanceof` checks.
- Classic example: `java.io.Serializable`. By default, an object **cannot** be serialized (converted to
  bytes and saved, e.g. to disk, to later be restored — "deserialization"). Implementing the empty
  `Serializable` marker interface is literally *permission* to the JVM to allow that object to be
  serialized.
- Other marker interfaces: `Cloneable`, `Remote`.

```java
import java.io.Serializable;

class Player implements Serializable {   // marker interface — enables serialization
    int score;
    String name;
}
```

**3. Functional Interface / SAM (Single Abstract Method)** — an interface with **exactly one**
abstract method (it may still have `default`/`static` methods). This is the shape required for
**lambda expressions** and **method references** to work. See §8.

```java
@FunctionalInterface     // optional but recommended — compiler enforces "only 1 abstract method"
interface Calculator {
    int operate(int a, int b);
}
```

---

## 6. Enums

`enum` = a special class representing a fixed set of **named constants**. Useful whenever a value can
only be one of a known, limited set (status codes, days of week, directions, etc.) instead of using
raw `String`s or `int`s (which are error-prone and not type-safe).

```java
public enum Status {
    RUNNING, FAILED, PENDING, SUCCESS
}
```

```java
public class Main {
    public static void main(String[] args) {
        Status current = Status.RUNNING;

        switch (current) {
            case RUNNING -> System.out.println("Task is running");
            case FAILED  -> System.out.println("Task failed");
            case PENDING -> System.out.println("Task pending");
            case SUCCESS -> System.out.println("Task succeeded");
        }

        // Every enum constant has .name() and .ordinal()
        System.out.println(current.name());     // "RUNNING"
        System.out.println(current.ordinal());  // 0 (position in declaration)

        // Iterate all constants
        for (Status s : Status.values()) {
            System.out.println(s);
        }

        // String -> enum
        Status s2 = Status.valueOf("SUCCESS");
    }
}
```

**Enums can have fields, constructors, and methods** (they're really classes under the hood, each
constant is effectively a `static final` instance):

```java
public enum Status {
    RUNNING(1), FAILED(-1), PENDING(0), SUCCESS(2);

    private final int code;

    Status(int code) {          // constructor is implicitly private
        this.code = code;
    }

    public int getCode() { return code; }
}
```

Why not just a normal class with constant objects? Enums give you this **for free**: singleton-safe
constants, `switch` support, `values()`/`valueOf()`, `.equals()`/`==` safety, readable `toString()`.

---

## 7. Annotations

An **annotation** supplies **metadata** to the compiler/JVM/runtime tools. It doesn't change what your
code *does* at runtime by itself — it's information *about* the code that something else (compiler,
framework, reflection) can act on.

Built-in annotations covered:

| Annotation | Purpose |
|---|---|
| `@Override` | Tells compiler this method overrides a parent/interface method (compile-time check) |
| `@Deprecated` | Marks a method/class as outdated; compiler emits a warning if used |
| `@SuppressWarnings("...")` | Tells compiler to ignore specific warning types |
| `@FunctionalInterface` | Compiler enforces the interface has exactly one abstract method |

```java
class Utils {
    @Deprecated
    public void oldMethod() {
        System.out.println("don't use me anymore");
    }

    @SuppressWarnings("unchecked")
    public void rawTypeUsage() {
        List list = new ArrayList();   // would normally warn about raw type
        list.add("hello");
    }
}
```

### Creating a custom annotation

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)   // available at runtime via reflection
@Target(ElementType.METHOD)           // can only be applied to methods
@interface Test {
    String value() default "";
}

class MyClass {
    @Test(value = "smoke-test")
    public void run() {
        System.out.println("running...");
    }
}
```

- `@Retention` decides how long the annotation is kept: `SOURCE` (compiler only), `CLASS` (in .class
  file, not at runtime), `RUNTIME` (readable via reflection at runtime — used heavily by frameworks
  like Spring/JUnit).
- `@Target` restricts where the annotation can be applied (`TYPE`, `METHOD`, `FIELD`, etc.).

---

## 8. Functional Interfaces & SAM

**SAM = Single Abstract Method.** An interface with exactly one abstract method is a "functional
interface" — this is the shape Java 8's **lambda expressions** and **method references** can be
assigned to.

```java
@FunctionalInterface
interface A {
    void show();      // exactly ONE abstract method
}
```

- `@FunctionalInterface` is optional, but if present, the compiler throws an error if you add a second
  abstract method — good safety net.
- A functional interface **can** still have `default` and `static` methods (those don't count against
  the "single abstract method" rule).
- Java provides many ready-made functional interfaces in `java.util.function` — see §18.

```java
interface A {
    void show();
}

public class Main {
    public static void main(String[] args) {
        // Old way: anonymous inner class
        A obj1 = new A() {
            public void show() { System.out.println("Old way"); }
        };

        // New way (Java 8+): lambda expression — because A is a functional interface
        A obj2 = () -> System.out.println("Lambda way");

        obj1.show();
        obj2.show();
    }
}
```

---

## 9. Lambda Expressions

A **lambda expression** is a concise way to represent an implementation of a functional interface's
single method — essentially an anonymous function.

### Why lambdas ("verbose" problem)
Writing a full anonymous inner class for a one-line method body is verbose: you repeat the interface
name, method name, and braces just to say "do this one thing." Lambdas strip away the ceremony.

### Syntax evolution

```java
interface Calculator {
    int operate(int a, int b);
}
```

```java
// 1. Anonymous inner class (verbose)
Calculator c1 = new Calculator() {
    @Override
    public int operate(int a, int b) {
        return a + b;
    }
};

// 2. Lambda — full form
Calculator c2 = (int a, int b) -> {
    return a + b;
};

// 3. Lambda — types inferred
Calculator c3 = (a, b) -> {
    return a + b;
};

// 4. Lambda — single expression, no braces/return needed
Calculator c4 = (a, b) -> a + b;

// 5. Single parameter -> parentheses optional
interface Printer { void print(String s); }
Printer p = s -> System.out.println(s);

// 6. Zero parameters -> empty parentheses required
interface Greeter { void greet(); }
Greeter g = () -> System.out.println("Hi!");
```

**Key rules:**
- Lambdas can **only** be used where a **functional interface** type is expected (the compiler infers
  which method to implement because there's only one).
- Parameter types are optional (inferred from context).
- If the body is a single expression, `{}` and `return` are omitted; if it's a block (multiple
  statements), `{}` and explicit `return` are required.
- Lambdas can capture (read) **effectively final** variables from the enclosing scope.

```java
public class Main {
    public static void main(String[] args) {
        Calculator add = (a, b) -> a + b;
        Calculator mul = (a, b) -> a * b;

        System.out.println(add.operate(3, 4));  // 7
        System.out.println(mul.operate(3, 4));  // 12
    }
}
```

Applied to the quiz project — grading could be expressed as a lambda implementing a custom
`AnswerChecker` functional interface, or (more idiomatically) via `Predicate<Question>` — see §18.

---

## 10. Exception Handling

### Statements: normal vs critical
Ordinary statements (e.g. `int x = 5;`) rarely fail. **Critical statements** are ones that *can* fail
at runtime due to factors outside your control — file I/O, network calls, user input parsing, array
access, division, etc. Exception handling exists to deal gracefully with critical statements going
wrong instead of crashing the whole program.

### Exception hierarchy

```
                     Throwable
                    /         \
               Error          Exception
          (JVM/system,     /            \
           unrecoverable) RuntimeException   (all other) checked
                          (unchecked)          exceptions e.g. IOException,
                                                SQLException
```

- **Checked exceptions** — must be either caught or declared with `throws` (checked by the compiler
  at compile time). E.g. `IOException`, `SQLException`, `ClassNotFoundException`.
- **Unchecked exceptions (`RuntimeException` and subclasses)** — not enforced by the compiler; usually
  represent programming bugs. E.g. `NullPointerException`, `ArrayIndexOutOfBoundsException`,
  `ArithmeticException`, `NumberFormatException`, `ClassCastException`.
- **Errors** (`OutOfMemoryError`, `StackOverflowError`) — serious problems an app typically shouldn't
  try to catch/recover from.

### try / catch / finally

```java
public class Main {
    public static void main(String[] args) {
        try {
            int a = 10, b = 0;
            int result = a / b;              // throws ArithmeticException
            System.out.println(result);      // never reached
        } catch (ArithmeticException e) {
            System.out.println("Cannot divide by zero: " + e.getMessage());
        } finally {
            // ALWAYS runs — whether exception occurred or not, even if there's a return above
            System.out.println("Cleanup / finally block executed");
        }
    }
}
```

- `finally` is guaranteed to execute (used for cleanup: closing files, DB connections, etc.), regardless
  of whether an exception was thrown or a `return` happened in `try`/`catch`.
- You can chain **multiple `catch` blocks** — order matters: more specific exception types first, more
  general ones (like `Exception`) last, otherwise you get "unreachable catch block" compile errors.

```java
try {
    String s = null;
    System.out.println(s.length());
} catch (NullPointerException e) {
    System.out.println("Null pointer issue");
} catch (Exception e) {          // generic fallback, must come last
    System.out.println("Something else went wrong: " + e);
}
```

- **Multi-catch** (Java 7+) when you want the same handling for different exception types:

```java
try {
    // risky code
} catch (ArithmeticException | NumberFormatException e) {
    System.out.println("Bad input/math: " + e.getMessage());
}
```

### Custom exceptions

```java
class InvalidAnswerException extends Exception {   // checked
    public InvalidAnswerException(String message) {
        super(message);
    }
}

class QuizService {
    void submitAnswer(String option) throws InvalidAnswerException {
        if (!List.of("A", "B", "C", "D").contains(option)) {
            throw new InvalidAnswerException("Option must be A, B, C or D");
        }
    }
}
```

### try-with-resources (Java 7+)
Instead of manually closing resources in a `finally` block, declare them inside the `try(...)`
parentheses; anything implementing `AutoCloseable`/`Closeable` gets **closed automatically** when the
try block ends (success or failure) — no explicit `finally` needed.

```java
import java.io.*;

public class Main {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            System.out.println("Error reading file: " + e.getMessage());
        }
        // br.close() is called automatically here — no finally block needed
    }
}
```

---

## 11. `throws` Keyword & Checked Exceptions (IOException)

`throws` in a method signature **declares** that a method might throw a checked exception, pushing the
responsibility of handling it to the **caller** instead of handling it right there.

```java
import java.io.*;

public class FileUtil {
    // declares that IOException might occur — caller MUST handle or re-declare it
    public void readFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        System.out.println(br.readLine());
        br.close();
    }
}

public class Main {
    public static void main(String[] args) {
        FileUtil util = new FileUtil();
        try {
            util.readFile("notes.txt");
        } catch (IOException e) {
            System.out.println("Could not read the file: " + e.getMessage());
        }
    }
}
```

**`throw` vs `throws`:**
| | `throw` | `throws` |
|---|---|---|
| Used | Inside a method body | In a method signature |
| Purpose | Actually raises/creates an exception instance | Declares that a checked exception *might* propagate out |
| Example | `throw new IllegalArgumentException("bad");` | `void m() throws IOException { ... }` |

**`IOException`** is a classic **checked** exception — the compiler forces you to either:
1. `catch` it locally, or
2. re-declare it with `throws` and let the caller deal with it.

This is different from unchecked exceptions like `NullPointerException`, which you're never forced to
declare or catch (though you still can).

---

## 12. Multithreading

A **thread** is the smallest unit of execution within a program; multithreading lets a program do
multiple things "at once" (concurrently, and truly in parallel on multi-core CPUs).

### Two ways to create a thread

**1. Extending `Thread`:**

```java
class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.println(Thread.currentThread().getName() + " -> " + i);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        t1.start();     // starts a NEW thread, JVM calls run() on it internally
        // t1.run();    // ❌ this would just be a normal method call on the main thread!
    }
}
```

**2. Implementing `Runnable`** (preferred — Java doesn't support multiple inheritance, so implementing
an interface keeps the class free to extend something else; also decouples "the task" from "the thread
that runs it"):

```java
class MyTask implements Runnable {
    @Override
    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.println(Thread.currentThread().getName() + " -> " + i);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Thread t1 = new Thread(new MyTask());
        t1.start();

        // Java 8+: Runnable is a functional interface -> lambda works too
        Thread t2 = new Thread(() -> {
            for (int i = 1; i <= 5; i++) System.out.println("t2 -> " + i);
        });
        t2.start();
    }
}
```

**`start()` vs `run()`:** `start()` asks the JVM to allocate a new call stack and actually run `run()`
concurrently on a new thread. Calling `run()` directly just executes it like an ordinary method **on
the current thread** — no new thread is created.

### Thread lifecycle (states)
`NEW` → `RUNNABLE` → (`RUNNING`) → `BLOCKED` / `WAITING` / `TIMED_WAITING` → `TERMINATED`

- **NEW** — thread object created, `start()` not yet called.
- **RUNNABLE** — eligible to run; the OS/JVM **scheduler** decides which runnable thread actually gets
  CPU time next (you don't control the exact order).
- **BLOCKED / WAITING** — waiting for a lock or another thread's signal (e.g. inside a `synchronized`
  block waiting for the monitor, or after calling `wait()`/`join()`).
- **TERMINATED** — `run()` finished.

### Thread priority

```java
Thread t1 = new Thread(new MyTask());
t1.setPriority(Thread.MAX_PRIORITY);   // 10
t1.setPriority(Thread.MIN_PRIORITY);   // 1
t1.setPriority(Thread.NORM_PRIORITY);  // 5 (default)
System.out.println(t1.getPriority());
```

Priority is only a **hint** to the scheduler about relative importance — it does **not guarantee**
execution order; the actual behaviour is JVM/OS dependent.

### Useful thread methods
```java
t1.join();     // caller thread waits until t1 finishes before continuing
Thread.sleep(1000);  // pauses current thread for ~1000ms (throws InterruptedException — checked!)
t1.setName("Worker-1");
```

---

## 13. Thread Safety & `synchronized`

### The problem
When multiple threads read/write **shared mutable state** (e.g. a shared counter, a shared list)
without coordination, you get **race conditions** — the final result becomes unpredictable because
operations like `count++` are not atomic (they're really read → increment → write, and threads can
interleave mid-operation).

```java
class Counter {
    private int count = 0;
    public void increment() {
        count++;     // NOT thread-safe: read-modify-write, can be interleaved
    }
    public int getCount() { return count; }
}
```

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();
        Runnable task = () -> {
            for (int i = 0; i < 1000; i++) counter.increment();
        };
        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start(); t2.start();
        t1.join(); t2.join();
        System.out.println(counter.getCount());  // often NOT 2000 due to race condition!
    }
}
```

### The fix: `synchronized`
`synchronized` ensures only **one thread at a time** can execute a block/method on a given object's
monitor lock — others wait their turn.

```java
class Counter {
    private int count = 0;

    public synchronized void increment() {   // synchronized method
        count++;
    }

    public int getCount() { return count; }
}
```

Or a **synchronized block** (finer-grained — lock only what's necessary, better performance):

```java
class Counter {
    private int count = 0;
    private final Object lock = new Object();

    public void increment() {
        synchronized (lock) {
            count++;
        }
    }
}
```

- **Thread-safe** = a class/method behaves correctly even when accessed by multiple threads
  concurrently.
- Trade-off: synchronization adds overhead and can cause threads to block/wait, so only synchronize
  what truly needs protecting (shared mutable state), not everything.
- Other tools (beyond this course, good to know for revision): `java.util.concurrent.atomic.AtomicInteger`,
  `ReentrantLock`, `ConcurrentHashMap`, thread pools via `ExecutorService`.

---

## 14. Collections Framework

### Clearing up the naming confusion
| Term | What it is |
|---|---|
| **Collection API / Collections Framework** | The overall *concept* — the whole set of interfaces, classes and algorithms for storing and manipulating groups of objects |
| **`Collection`** (interface, capital C, singular) | The root **interface** — `List`, `Set`, `Queue` all extend it |
| **`Collections`** (class, capital C, plural) | A **utility class** full of `static` helper methods, e.g. `Collections.sort(list)`, `Collections.reverse(list)`, `Collections.max(list)` |

### Hierarchy (core parts)
```
Iterable
  └── Collection
        ├── List   (ordered, allows duplicates)      -> ArrayList, LinkedList, Vector
        ├── Set    (no duplicates)                    -> HashSet, LinkedHashSet, TreeSet
        └── Queue  (FIFO / priority processing)        -> LinkedList, PriorityQueue

Map (NOT a Collection, separate hierarchy — key/value pairs)
        -> HashMap, LinkedHashMap, TreeMap
```

### List

```java
import java.util.*;

List<String> names = new ArrayList<>();
names.add("Navin");
names.add("Harsh");
names.add("John");
names.add("Navin");          // duplicates allowed
System.out.println(names.get(0));   // "Navin"
names.remove("Harsh");
System.out.println(names);          // [Navin, John, Navin]
```
- `ArrayList` — backed by a dynamic array, fast random access (`get(i)` = O(1)), slower inserts/removals
  in the middle.
- `LinkedList` — doubly-linked list, fast insert/remove at ends, slower random access.

### Set

```java
Set<String> unique = new HashSet<>();
unique.add("A");
unique.add("B");
unique.add("A");             // ignored — duplicate
System.out.println(unique);  // size = 2, order not guaranteed

Set<String> ordered = new LinkedHashSet<>();  // keeps insertion order
Set<String> sorted = new TreeSet<>();         // keeps sorted (natural/comparator) order
```

### Map

```java
Map<Integer, String> map = new HashMap<>();
map.put(1, "Navin");
map.put(2, "Harsh");
map.put(1, "Updated");        // overwrites key 1's value

for (Map.Entry<Integer, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " -> " + entry.getValue());
}

System.out.println(map.getOrDefault(5, "Not found"));
map.computeIfAbsent(3, k -> "New Player");
```
- `HashMap` — no ordering guarantee, O(1) average `get`/`put`.
- `LinkedHashMap` — preserves insertion order.
- `TreeMap` — keeps keys sorted.

### `Collections` utility class

```java
List<Integer> nums = new ArrayList<>(List.of(5, 3, 8, 1));
Collections.sort(nums);                 // [1, 3, 5, 8]
Collections.reverse(nums);              // [8, 5, 3, 1]
System.out.println(Collections.max(nums));  // 8
System.out.println(Collections.min(nums));  // 1
List<Integer> readOnly = Collections.unmodifiableList(nums);
```

### Applied to the quiz project
```java
Map<String, Integer> leaderboard = new HashMap<>();
leaderboard.put("Navin", 4);
leaderboard.put("Harsh", 5);
Collections.max(leaderboard.entrySet(), Map.Entry.comparingByValue());
```

---

## 15. Generics

Generics let classes/interfaces/methods operate on a **type parameter** decided at usage time, giving
**compile-time type safety** and removing the need for manual casting.

### Why generics? (the `E` you see everywhere)
Before generics, collections stored `Object`, so you had to cast on every read and mistakes only
surfaced at **runtime** (`ClassCastException`). With generics (`List<E>`), the compiler enforces the
type at **compile time**.

```java
List list = new ArrayList();     // raw type — avoid this
list.add("hello");
list.add(42);                    // no compile error, but risky
String s = (String) list.get(1); // ❌ runtime ClassCastException

List<String> safeList = new ArrayList<>();
safeList.add("hello");
// safeList.add(42);             // ❌ compile-time error — caught immediately
String value = safeList.get(0);  // no cast needed
```

### Generic class

```java
class Box<T> {                 // T = placeholder type ("Type")
    private T value;
    public void set(T value) { this.value = value; }
    public T get() { return value; }
}

public class Main {
    public static void main(String[] args) {
        Box<String> stringBox = new Box<>();
        stringBox.set("Hello");
        System.out.println(stringBox.get());

        Box<Integer> intBox = new Box<>();
        intBox.set(100);
        System.out.println(intBox.get());
    }
}
```

### Generic method

```java
public class Utils {
    public static <T> void printArray(T[] arr) {
        for (T item : arr) {
            System.out.println(item);
        }
    }
}
```

### Bounded types & wildcards

```java
// only accepts Number or its subtypes (Integer, Double, ...)
class Calculator<T extends Number> {
    public double square(T num) {
        return num.doubleValue() * num.doubleValue();
    }
}

// wildcard: method that reads from any List of Number-or-subtype
public static void printNumbers(List<? extends Number> list) {
    for (Number n : list) System.out.println(n);
}
```

**Common type parameter conventions:** `T` (Type), `E` (Element — used a lot in `List<E>`),
`K`/`V` (Key/Value — used in `Map<K,V>`), `N` (Number), `R` (Return type).

---

## 16. Comparable vs Comparator

Both are used for **sorting custom objects**, but they answer different questions.

| | `Comparable` | `Comparator` |
|---|---|---|
| Package | `java.lang` | `java.util` |
| Method | `compareTo(T o)` | `compare(T o1, T o2)` |
| Implemented where | **Inside** the class itself — gives the class its own "natural ordering" | **Outside** the class — a separate strategy object, can define many different orderings |
| How many sort orders | Only one ("the" natural order) | As many as you like |
| Used by | `Collections.sort(list)`, `Arrays.sort(arr)` directly | `Collections.sort(list, comparator)`, `list.sort(comparator)` |

### Comparable — "compare myself to another of my own type"

```java
class Student implements Comparable<Student> {
    String name;
    int age;

    Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(Student other) {
        return this.age - other.age;    // natural order: ascending by age
    }

    @Override
    public String toString() { return name + "(" + age + ")"; }
}

public class Main {
    public static void main(String[] args) {
        List<Student> students = new ArrayList<>(List.of(
            new Student("Navin", 25), new Student("Harsh", 20), new Student("John", 30)
        ));
        Collections.sort(students);          // uses compareTo internally
        System.out.println(students);        // [Harsh(20), Navin(25), John(30)]
    }
}
```

### Comparator — "an external rule for comparing two objects of some type"

```java
class AgeComparator implements Comparator<Student> {
    @Override
    public int compare(Student s1, Student s2) {
        return s1.age - s2.age;
    }
}

class NameComparator implements Comparator<Student> {
    @Override
    public int compare(Student s1, Student s2) {
        return s1.name.compareTo(s2.name);
    }
}
```

```java
Collections.sort(students, new AgeComparator());
Collections.sort(students, new NameComparator());

// Java 8+: lambda / method reference / Comparator.comparing — no need for a separate class
Collections.sort(students, (s1, s2) -> s1.age - s2.age);
students.sort(Comparator.comparing(s -> s.name));
students.sort(Comparator.comparingInt((Student s) -> s.age).reversed());
```

**`compareTo`/`compare` contract:** return negative if first < second, `0` if equal, positive if
first > second.

### Applied to the quiz project
```java
// Rank players by score, highest first
List<Map.Entry<String, Integer>> ranking = new ArrayList<>(leaderboard.entrySet());
ranking.sort((a, b) -> b.getValue() - a.getValue());
```

---

## 17. Stream API

Introduced in **Java 8**. A `Stream` is a pipeline for processing **sequences of elements**
declaratively (what to do) instead of imperatively (how to loop) — filter, transform, and reduce
collections in a chainable, readable way.

### Motivation
Before streams, operating on a `List<Integer>` (e.g. "get all even numbers, doubled, summed") required
manual loops with temporary lists. Streams let you express that as a **pipeline**:

```java
List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4, 5, 6, 7, 8));

int result = nums.stream()
                  .filter(n -> n % 2 == 0)   // keep evens
                  .map(n -> n * 2)           // double each
                  .reduce(0, Integer::sum);  // sum them all

System.out.println(result);  // (2+4+6+8)*2 = 40
```

### Key characteristics
- A stream **does not store data** — it's a view/pipeline over a source (collection, array, etc.).
- Operations are **lazy**: intermediate operations (`filter`, `map`, `sorted`, ...) don't run until a
  **terminal operation** (`collect`, `forEach`, `reduce`, `count`, ...) is invoked.
- A stream can be consumed **only once**.
- Doesn't modify the original source.

### Common intermediate operations

```java
List<String> names = List.of("Navin", "Harsh", "John", "Alia", "Bob");

names.stream()
     .filter(n -> n.length() > 3)     // keep names longer than 3 chars
     .map(String::toUpperCase)        // transform each element
     .sorted()                        // sort naturally
     .forEach(System.out::println);   // terminal op: consume each

List<String> upper = names.stream()
                           .map(String::toUpperCase)
                           .collect(Collectors.toList());   // terminal op: gather into a List
```

### Common terminal operations

```java
long count = names.stream().filter(n -> n.startsWith("J")).count();
Optional<String> first = names.stream().filter(n -> n.startsWith("H")).findFirst();
boolean anyMatch = names.stream().anyMatch(n -> n.equals("Bob"));
boolean allMatch = names.stream().allMatch(n -> n.length() > 2);
int total = List.of(1,2,3,4).stream().reduce(0, Integer::sum);
```

### `map` vs `filter`
- `filter(Predicate<T>)` — keeps elements that satisfy a condition (returns `boolean`); stream size can
  shrink.
- `map(Function<T,R>)` — transforms each element into something else (1-to-1); stream size stays the
  same but the type can change.

### `collect` and Collectors

```java
import java.util.stream.Collectors;

List<Student> students = List.of(
    new Student("Navin", 25), new Student("Harsh", 20), new Student("John", 30)
);

List<String> namesOnly = students.stream()
                                  .map(s -> s.name)
                                  .collect(Collectors.toList());

Map<String, Integer> nameToAge = students.stream()
        .collect(Collectors.toMap(s -> s.name, s -> s.age));

double avgAge = students.stream().mapToInt(s -> s.age).average().orElse(0);
```

### Applied to the quiz project
```java
// Names of everyone who scored above 3
List<String> toppers = leaderboard.entrySet().stream()
        .filter(e -> e.getValue() > 3)
        .map(Map.Entry::getKey)
        .collect(Collectors.toList());
```

---

## 18. Predicate, Function, Consumer, Supplier, BinaryOperator

Java 8 ships a set of ready-made **functional interfaces** in `java.util.function` so you don't have to
declare your own for common shapes. These power `filter`, `map`, `forEach`, `reduce`, etc.

| Interface | Abstract method | Purpose |
|---|---|---|
| `Predicate<T>` | `boolean test(T t)` | A yes/no condition — used by `filter` |
| `Function<T,R>` | `R apply(T t)` | Transforms input of type T into output of type R — used by `map` |
| `Consumer<T>` | `void accept(T t)` | Takes a value, returns nothing (side effects) — used by `forEach` |
| `Supplier<T>` | `T get()` | Takes nothing, produces/supplies a value |
| `BinaryOperator<T>` | `T apply(T t1, T t2)` | Combines two values of the **same** type into one — used by `reduce` |
| `BiFunction<T,U,R>` | `R apply(T t, U u)` | Like `Function` but takes two different-typed inputs |

```java
import java.util.function.*;

Predicate<Integer> isEven = n -> n % 2 == 0;
System.out.println(isEven.test(4));   // true

Function<String, Integer> length = String::length;
System.out.println(length.apply("hello"));  // 5

Consumer<String> printer = s -> System.out.println("Value: " + s);
printer.accept("hi");

Supplier<Double> randomValue = Math::random;
System.out.println(randomValue.get());

BinaryOperator<Integer> add = (a, b) -> a + b;
System.out.println(add.apply(3, 4));  // 7

BiFunction<String, String, String> concat = (a, b) -> a + b;
System.out.println(concat.apply("Hello, ", "World!"));
```

**Predicate composition:**
```java
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> combined = isEven.and(isPositive);   // also: .or(), .negate()
System.out.println(combined.test(4));   // true
System.out.println(combined.test(-4));  // false
```

### Applied to the quiz project
```java
Predicate<Question> isHard = q -> q.getOptions().size() > 4; // example rule
Function<Question, String> toDisplay = q -> "Q: " + q.getQuestion();
```

---

## 19. Optional Class

`Optional<T>` is a **container object** that may or may not hold a non-null value. Its purpose is to
make the possibility of "no value" **explicit in the type system**, instead of quietly returning
`null` and risking a `NullPointerException` somewhere downstream.

```java
import java.util.Optional;

Optional<String> present = Optional.of("Hello");         // must be non-null
Optional<String> empty = Optional.empty();
Optional<String> maybeNull = Optional.ofNullable(getValueThatMightBeNull());

System.out.println(present.isPresent());   // true
System.out.println(empty.isPresent());     // false

present.ifPresent(v -> System.out.println("Got: " + v));

String result = maybeNull.orElse("default value");
String result2 = maybeNull.orElseGet(() -> computeDefault());
String result3 = maybeNull.orElseThrow(() -> new NoSuchElementException("missing!"));

Optional<Integer> lengthOpt = present.map(String::length);   // transform if present
```

**Why prefer `Optional` over returning `null`:**
- Forces the caller to consciously handle the "empty" case (via `isPresent`/`orElse`/`map`) rather than
  accidentally dereferencing `null`.
- Self-documenting return types — a method returning `Optional<Question>` clearly signals "you might
  not get a question back."

### Applied to the quiz project
```java
public Optional<Question> findById(int id) {
    return questions.stream()
                     .filter(q -> q.getId() == id)
                     .findFirst();       // findFirst() already returns Optional<Question>
}

// usage
findById(3).ifPresentOrElse(
    q -> System.out.println("Found: " + q.getQuestion()),
    () -> System.out.println("No question with that id")
);
```

---

## 20. Method References & Constructor References

A **method reference** is shorthand for a lambda that does nothing but **call an existing method**.
Syntax: `ClassOrObject::methodName`.

### Why
```java
List<String> names = List.of("navin", "harsh", "john");

// Lambda — explicitly names the parameter just to pass it along
names.stream().map(s -> s.toUpperCase()).forEach(s -> System.out.println(s));

// Method reference — same behaviour, less noise
names.stream().map(String::toUpperCase).forEach(System.out::println);
```

`System.out::println` reads as: *"the `println` method, which belongs to `System.out`, is responsible
for handling each value."* You just point at an existing method instead of writing a wrapper lambda for it.

### Four kinds of method references

```java
// 1. Static method reference:      ClassName::staticMethod
Function<String, Integer> parse = Integer::parseInt;

// 2. Instance method of a PARTICULAR object:   object::instanceMethod
String prefix = "Hello, ";
Function<String, String> greeter = prefix::concat;

// 3. Instance method of an ARBITRARY object of a type:  ClassName::instanceMethod
Function<String, Integer> len = String::length;   // equivalent to s -> s.length()

// 4. Constructor reference:        ClassName::new
Supplier<ArrayList<String>> listMaker = ArrayList::new;
```

### Constructor reference in detail

```java
class Student {
    private String name;
    private int age;

    public Student() {}                       // default constructor
    public Student(String name) {              // single-arg constructor
        this.name = name;
    }
    // getters/setters/toString omitted for brevity
}
```

```java
List<String> names = List.of("Navin", "Harsh", "John");

// Without constructor reference — using a regular loop
List<Student> students = new ArrayList<>();
for (String name : names) {
    students.add(new Student(name));
}

// With Stream + lambda
List<Student> students2 = names.stream()
                                .map(name -> new Student(name))
                                .collect(Collectors.toList());

// With Stream + constructor reference — cleanest
List<Student> students3 = names.stream()
                                .map(Student::new)     // calls Student(String name)
                                .collect(Collectors.toList());
```

`Student::new` automatically picks the constructor whose parameter list matches what the functional
interface expects (here, a `Function<String, Student>` needs a 1-arg constructor).

**Rule of thumb:** if your lambda's *only* job is "take the parameter(s) and pass them straight into an
existing method or constructor," it can almost always be rewritten as a method/constructor reference.

---

## 21. Quick-Revision Cheat Sheet

- **abstract class**: can't instantiate; may mix abstract + concrete methods; use when subclasses share
  some common code but must implement other parts themselves.
- **interface**: pure contract (+ `default`/`static` methods since Java 8); a class can implement many.
- **marker interface**: zero methods, just a tag (`Serializable`).
- **functional interface / SAM**: exactly one abstract method → target for lambdas/method references.
- **`@Override`**: always use it — free compile-time safety net against typo'd overrides.
- **enum**: type-safe fixed set of constants; can have fields/constructors/methods.
- **annotation**: metadata for compiler/runtime/frameworks; `@Retention` + `@Target` for custom ones.
- **checked exception**: must catch or `throws`; **unchecked (`RuntimeException`)**: optional.
- **`throw`** raises one exception instance; **`throws`** declares a method might propagate one.
- **try-with-resources**: auto-closes `AutoCloseable` resources, no `finally` needed.
- **Thread**: `start()` = new thread; `run()` = normal method call, no concurrency.
- **`synchronized`**: only one thread executes the block/method on a given lock at a time → prevents
  race conditions on shared mutable state.
- **Collection vs Collections**: interface root vs static-utility class.
- **Generics**: compile-time type safety, no manual casting, `<T>`/`<E>`/`<K,V>` conventions.
- **Comparable**: one natural order, defined inside the class (`compareTo`).
  **Comparator**: many external orders, defined outside the class (`compare`).
- **Stream**: lazy, one-time-use pipeline; `filter`/`map` are intermediate, `collect`/`forEach`/`reduce`
  are terminal.
- **Predicate/Function/Consumer/Supplier/BinaryOperator**: prebuilt functional interfaces for
  test/transform/consume/produce/combine.
- **Optional**: explicit "value may be absent" container — avoids stray `null`s.
- **Method reference (`Class::method`)**: shorthand for a lambda that just delegates to an existing
  method; **constructor reference (`Class::new`)** does the same for object creation.

---

### Further reading (official docs — good for filling in edge cases)
- Oracle Java Tutorials — Interfaces and Inheritance: https://docs.oracle.com/javase/tutorial/java/IandI/
- Oracle Java Tutorials — Lambda Expressions: https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html
- Java `java.util.function` package docs: https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/function/package-summary.html
- Java Concurrency (Threads): https://docs.oracle.com/javase/tutorial/essential/concurrency/
- `java.util.stream` package docs: https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/package-summary.html
