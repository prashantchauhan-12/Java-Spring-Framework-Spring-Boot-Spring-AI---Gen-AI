# Core Java — Complete Revision Notes

> Based on the Core Java module (Telusko-style course). Covers Java basics through OOP fundamentals — everything needed before starting Spring.

---

## 1. Java Platform Basics

### 1.1 Tools you need
- **Editor/IDE**: Notepad works, but an IDE (VS Code, IntelliJ IDEA, Eclipse, NetBeans) gives you code + compile + run + debug in one place.
- **JDK (Java Development Kit)**: needed to compile and run code. Download the **LTS (Long-Term Support)** version (e.g. Java 17) rather than the latest short-term release — LTS versions get long-term support, and the core language stays backward compatible across versions.
- On Windows, after installing JDK you must add the `bin` folder (e.g. `C:\Program Files\Java\jdk-17\bin`) to the **Path** environment variable, then restart your terminal.
- Verify install: `java --version` and `javac --version`.

### 1.2 JShell
Introduced in **Java 9** — a REPL for quick experiments without writing a full class:
```
jshell> 2 + 3
$1 ==> 5
jshell> System.out.println("Hello World")
Hello World
```
JShell is for experimenting only — you can't build a real project in it.

### 1.3 Your first program
```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```
- File name **must** match the public class name: `Hello.java`.
- **Compile**: `javac Hello.java` → produces `Hello.class` (bytecode).
- **Run**: `java Hello` (class name, no `.class` extension).
- **Shortcut** (Java 11+): `java Hello.java` — compiles and runs in a single step, skipping the separate `.class` file. Fine for learning; never use it for real projects.
- `print` vs `println`: `println` = "print + new line".

### 1.4 JVM, JRE, JDK — how Java runs
```
Your .java code  --(javac, the compiler)-->  bytecode (.class file)
bytecode  --(runs inside)-->  JVM (Java Virtual Machine)
```
- **JVM** — executes bytecode. It's platform-*dependent* (a Windows JVM ≠ a Mac JVM), but because every OS has its own JVM, your **compiled bytecode** becomes platform-*independent* — "write once, run anywhere" (WORA).
- **JRE (Java Runtime Environment)** = JVM + the standard class libraries needed to actually run programs.
- **JDK (Java Development Kit)** = JRE + development tools like `javac`. Only developers need the JDK; a machine that merely *runs* a Java app only needs the JRE (which includes the JVM).
```
JDK  ⊃  JRE  ⊃  JVM
```

---

## 2. Variables & Data Types

### 2.1 What is a variable
A **named box** that stores a value of a specific type. Java is **statically / strongly typed** — every variable must have a declared type, and that type can't change later.
```java
int num = 5;
```
`type name = value;` — the `=` here is the **assignment operator**: it evaluates the right-hand side and assigns it to the left-hand side.

### 2.2 Primitive data types
| Type | Size | Notes |
|---|---|---|
| `byte` | 1 byte | range −128 to 127 |
| `short` | 2 bytes | |
| `int` | 4 bytes | default for whole numbers |
| `long` | 8 bytes | append `L` to literals: `long l = 123L;` |
| `float` | 4 bytes | append `f`: `float f = 5.6f;` — limited precision |
| `double` | 8 bytes | **default** for decimal literals; higher precision than float |
| `char` | 2 bytes | Java uses **Unicode** (not ASCII) — supports characters from every language, not just English. Use single quotes: `char c = 'K';` |
| `boolean` | — | only `true` / `false` — **not** `1`/`0` like some other languages |

```java
byte b = 127;                 // max value for byte
int a = 256;
byte b2 = (byte) a;            // needs explicit cast — see §2.4
double d = 5.6;                // decimal literal defaults to double
float f = 5.6f;                // 'f' suffix required for float
char c = 'a';
c++;                            // chars can be incremented like numbers -> 'b'
boolean flag = true;
```

### 2.3 Literals — different number formats
```java
int bin = 0b101;        // binary -> 5
int hex = 0x7E;          // hexadecimal -> 126
int big = 1_000_000;     // underscores for readability (Java 7+)
double sci = 12e10;      // scientific notation
```

### 2.4 Type Conversion, Casting, and Promotion
- **Conversion (implicit / automatic widening)**: assigning a smaller type into a bigger type happens automatically.
  ```java
  int i = 10;
  double d = i;   // auto-converts int -> double, no cast needed
  ```
- **Casting (explicit / narrowing)**: assigning a bigger type into a smaller type needs an explicit cast, and can **lose data**.
  ```java
  double f = 5.6;
  int t = (int) f;   // t = 5, the .6 is truncated (not rounded)
  ```
  If the value is out of range for the target type, Java effectively takes a **modulus** against the target type's range:
  ```java
  int a = 257;
  byte k = (byte) a;   // 257 % 256 = 1  ->  k = 1
  ```
- **Type promotion**: when an operation on two smaller types (e.g. `byte * byte`) produces a result that would overflow the smaller type, Java automatically promotes the result to a bigger type (`int`) when you store it:
  ```java
  byte a = 30, b = 30;
  int result = a * b;   // 900 -> too big for byte, auto-promoted to int
  ```

---

## 3. Operators

### 3.1 Arithmetic
```java
int num1 = 7, num2 = 5;
num1 + num2;   // 12
num1 - num2;   // 2
num1 * num2;   // 35
num1 / num2;   // 1  (integer division -> quotient only)
num1 % num2;   // 2  (modulus -> remainder only)
```

### 3.2 Compound assignment & increment/decrement
```java
num1 += 2;    // num1 = num1 + 2
num1 -= 2;
num1 *= 2;
num1 /= 2;

num1++;   // post-increment
++num1;   // pre-increment
num1--;   // post-decrement
--num1;   // pre-decrement
```
**Pre vs. post matters when you assign the result in the same statement:**
```java
int num = 7;
int result = num++;   // result = 7 (fetch THEN increment); num becomes 8
int result2 = ++num;  // increments FIRST, then fetches; result2 = num's new value
```
- `num++` → fetch old value, *then* increment.
- `++num` → increment *first*, then fetch new value.

### 3.3 Relational operators
`>`, `<`, `>=`, `<=`, `==`, `!=` — all return `boolean`.

### 3.4 Logical operators
| Operator | Meaning |
|---|---|
| `&&` | AND (short-circuit) |
| `\|\|` | OR (short-circuit) |
| `!` | NOT |

```java
boolean result = (x > y) && (a > b);   // true only if BOTH are true
boolean result2 = (x > y) || (a > b);  // true if EITHER is true
boolean result3 = !result;             // reverses true/false
```
**Short-circuit** means Java stops evaluating as soon as the result is determined:
- In `&&`, if the first condition is `false`, the second is never checked (result must be `false`).
- In `||`, if the first condition is `true`, the second is never checked (result must be `true`).
- (Single `&` / `|` also exist for booleans but always evaluate both sides — not preferred; use `&&`/`||`.)

---

## 4. Conditional Statements

### 4.1 if / else / else-if
```java
if (x > 10 && x <= 20) {
    System.out.println("Hello");
} else if (y > x) {
    System.out.println("y wins");
} else {
    System.out.println("Bye");
}
```
- Curly braces `{}` are optional for a **single statement**, but **required** the moment you have more than one statement inside the block.
- Java does **not** use indentation to define blocks (unlike Python) — only `{}` matters.

**Finding the max of 3 numbers (nested if-else pattern):**
```java
int x = 8, y = 17, z = 9;
if (x > y && x > z) {
    System.out.println(x);
} else if (y > z) {   // x already known to not be the max here
    System.out.println(y);
} else {
    System.out.println(z);
}
```

### 4.2 Ternary operator
Shorthand for a simple if-else that assigns a value:
```java
int result = (n % 2 == 0) ? 10 : 20;
// condition ? valueIfTrue : valueIfFalse
```
Only use it for short, simple assignments — don't force complex logic into a ternary.

### 4.3 switch statement
```java
int n = 2;
switch (n) {
    case 1:
        System.out.println("Monday");
        break;
    case 2:
        System.out.println("Tuesday");
        break;
    default:
        System.out.println("Enter a valid number");
}
```
- **`break` is critical.** Without it, execution "falls through" and runs every subsequent case until it hits a `break` or the end of the switch — it does NOT re-check the condition.
- `default` handles any value that doesn't match a case (like a catch-all `else`).
- (Note: newer Java versions have a switch-expression syntax that avoids `break` entirely — not covered here since this course targets Java 8-style syntax, still common in industry.)

---

## 5. Loops

### 5.1 while loop
```java
int i = 1;
while (i <= 4) {
    System.out.println("hi " + i);
    i++;
}
```
Checks the condition **before** each iteration — if false from the start, the body never executes.

### 5.2 do-while loop
```java
int i = 5;
do {
    System.out.println("hi " + i);
    i++;
} while (i <= 4);
```
Executes the body **at least once**, even if the condition is false from the start — the check happens *after* the body runs. Use this when you need guaranteed execution once (e.g., "try to send a message at least once even if the service seems down").

### 5.3 for loop
```java
for (int i = 0; i < 4; i++) {
    System.out.println("hi " + i);
}
```
Three parts in one line: `initialization; condition; increment/decrement`. Any part can be left empty (but the semicolons are still required) as long as you handle it elsewhere — otherwise you risk an infinite loop.

**Nested loops** (e.g. printing a grid):
```java
for (int i = 1; i <= 4; i++) {
    for (int j = 1; j <= 3; j++) {
        System.out.print("hi " + i + " " + j + " ");
    }
    System.out.println();
}
```

### 5.4 Which loop to use?
- **for** — when you know the number of iterations up front (most common).
- **while** — when you don't know how many iterations you'll need (reading a file, network, database until some condition changes).
- **do-while** — rare; use only when the body must run at least once regardless of the condition.

### 5.5 Enhanced for loop ("for-each")
```java
int[] nums = {3, 7, 2, 4};
for (int n : nums) {
    System.out.println(n);
}
```
No counter, no index, no `.length` check, no `ArrayIndexOutOfBoundsException` risk — Java iterates every element for you. Works with arrays and (later) collections. Also works with object arrays:
```java
for (Student stud : students) {
    System.out.println(stud.name + " : " + stud.marks);
}
```

---

## 6. Arrays

### 6.1 Why arrays
Instead of creating separate variables (`num1`, `num2`, `num3`...) for related values, store them together in one indexed structure.

### 6.2 Declaring & creating arrays
```java
int[] nums = {3, 7, 2, 4};          // literal initialization
int[] nums2 = new int[4];           // size-only; all values default to 0
```
- Indexing starts at **0**. For a 4-element array, valid indices are `0..3`.
- Once created, the size is **fixed** — you cannot resize an array; you'd have to create a new, bigger array and copy elements over.
- Use `.length` (property, no parentheses) to get the array's size safely instead of hardcoding numbers:
  ```java
  for (int i = 0; i < nums.length; i++) { ... }
  ```
- Accessing an index outside the valid range throws `ArrayIndexOutOfBoundsException` at **runtime** (not caught at compile time).

### 6.3 Arrays of objects
```java
class Student {
    int rollno;
    String name;
    int marks;
}

Student s1 = new Student();
s1.rollno = 1; s1.name = "Navin"; s1.marks = 88;
// ...s2, s3 similarly

Student[] students = new Student[3];
students[0] = s1;
students[1] = s2;
students[2] = s3;
```
Important: `new Student[3]` creates an array that can **hold** 3 `Student` references — it does **not** create 3 `Student` objects. You must create and assign each object yourself.

Printing an object directly (`System.out.println(s1)`) prints an address-like string, not the field values — you either print fields individually or override `toString()` (see §16).

### 6.4 Multi-dimensional & jagged arrays
```java
int[][] nums = new int[3][4];   // 3 rows, 4 columns each
nums[0][2] = 6;

for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 4; j++) {
        System.out.print(nums[i][j] + " ");
    }
    System.out.println();
}
```
- **Jagged array** — rows can have *different* lengths:
  ```java
  int[][] jagged = new int[3][];
  jagged[0] = new int[3];
  jagged[1] = new int[4];
  jagged[2] = new int[2];
  ```
- **3D array** works the same way, just with an extra `[]`:
  ```java
  int[][][] cube = new int[3][4][5];
  ```

### 6.5 Drawbacks of arrays
1. **Fixed size** — can't grow/shrink after creation.
2. **Costly search/insert** — no built-in efficient search; inserting in the middle means shifting elements.
3. **Single type only** — can't mix `int`, `String`, `double` in one primitive array (this is one of the motivations for the **Collections framework**, covered later in the course).

Arrays are still the right choice when the size is fixed and known in advance — they're fast and simple.

---

## 7. Classes, Objects & Methods

### 7.1 Class vs Object
- A **class** is a blueprint/design (like an architect's drawing).
- An **object** is the actual instance created from that blueprint, built by the **JVM** at runtime.
- Every real-world object has **properties** (what it *knows*) and **behavior** (what it *does*) — in Java, these become **instance variables** and **methods**.

### 7.2 Defining and using a class
```java
class Calculator {
    public int add(int n1, int n2) {
        return n1 + n2;
    }
}

public class Demo {
    public static void main(String[] args) {
        Calculator calc = new Calculator();   // create the object
        int result = calc.add(3, 4);          // call the method
        System.out.println(result);           // 7
    }
}
```
- `Calculator calc` — declares a **reference variable** of type `Calculator`.
- `new Calculator()` — this is where the actual **object** gets created in the **heap memory**.
- You can't call a method directly through the class design alone (`Calculator.add()` won't work for instance methods) — you need an object.

### 7.3 Method overloading
Same method name, **different parameter list** (different number and/or types of parameters) within the same class:
```java
public int add(int n1, int n2) { return n1 + n2; }
public int add(int n1, int n2, int n3) { return n1 + n2 + n3; }
public double add(double n1, double n2) { return n1 + n2; }
```
- Return type alone does **not** distinguish overloaded methods — the parameter list must differ.
- This is resolved at **compile time** → an example of **compile-time (static) polymorphism**.

### 7.4 Anonymous objects
```java
new A().show();   // object created and used inline, never assigned to a variable
```
Useful for one-off calls; the object can't be reused afterward since there's no reference to it.

---

## 8. Strings

### 8.1 String basics
```java
String name = "Navin";        // preferred — uses the String pool
String name2 = new String("Navin");   // creates a separate object, bypasses pool (rarely used)
```
- `+` is overloaded for `String` to mean **concatenation**: `"Hello " + "Navin"` → `"Hello Navin"`.
- Common methods: `charAt(index)`, `concat(other)`, `length()`.

### 8.2 Strings are immutable — and the String Constant Pool
```java
String s1 = "Navin";
String s2 = "Navin";
System.out.println(s1 == s2);   // true — same object!
```
- String literals live in a special area of the heap called the **String Constant Pool**.
- When you write a string literal, Java first checks the pool: if that exact value already exists, the new reference just points to the **existing** object — no new object is created.
- **Strings cannot be changed in place.** `name = name + " Reddy";` does **not** modify the original `"Navin"` object — it creates a **brand-new** string object (`"Navin Reddy"`) and re-points the `name` reference to it. The old `"Navin"` object still exists in the pool (unchanged) and may eventually be garbage collected if nothing else references it.
- This is why strings are called **immutable**.

### 8.3 Mutable alternatives: StringBuffer / StringBuilder
```java
StringBuffer sb = new StringBuffer("Navin");
sb.append(" Reddy");             // sb is now "Navin Reddy" — modified in place
sb.insert(0, "Java ");           // insert at a specific index
sb.delete(2, 3);                 // delete a range of characters
sb.capacity();                   // default buffer capacity is 16 chars extra, to reduce relocation
sb.length();                     // actual content length (different from capacity)
String str = sb.toString();      // convert back to a normal String
```
- **StringBuffer** is **thread-safe** (synchronized); **StringBuilder** has the same API but is **not** thread-safe (faster in single-threaded code).
- Use `StringBuffer`/`StringBuilder` whenever you need to build/modify a string repeatedly (e.g. in a loop) — using plain `String` concatenation in a loop creates a new object every time, which is wasteful.

---

## 9. Encapsulation

### 9.1 The idea
Bundle data (instance variables) with the methods that operate on them, and **hide** the data from direct outside access — the only sanctioned way in is through methods you control.

```java
class Human {
    private int age;
    private String name;

    public int getAge() { return age; }
    public void setAge(int a) { age = a; }

    public String getName() { return name; }
    public void setName(String n) { name = n; }
}
```
- Mark instance variables **`private`** — accessible only within the same class.
- Expose controlled access via **getters** (`getX()`) and **setters** (`setX()`) — the standard Java naming convention (not a hard requirement, but expected practice; IDEs can auto-generate these).
- You don't have to provide both — e.g., a read-only field can have only a getter.

### 9.2 The `this` keyword
`this` refers to the **current object** — the object on which the currently-executing method was called.
```java
public void setAge(int age) {
    this.age = age;   // this.age = the instance variable, age = the parameter
}
```
Essential when a parameter name shadows an instance variable name — `this.` disambiguates which one you mean.

---

## 10. Constructors

### 10.1 What is a constructor
A special "method" that runs automatically every time an object is created (`new ClassName(...)`), typically used to initialize instance variables.

Rules:
- Same name as the class.
- **No return type** (not even `void`).
- Called automatically — you never invoke it explicitly with `.constructorName()`.

```java
class Human {
    private int age;
    private String name;

    public Human() {              // default constructor
        age = 12;
        name = "John";
    }

    public Human(int a, String n) {   // parameterized constructor
        age = a;
        name = n;
    }
}

Human h1 = new Human();              // age=12, name="John"
Human h2 = new Human(18, "Navin");   // age=18, name="Navin"
```
- If you don't write **any** constructor, Java silently provides an empty **default constructor** for you.
- The moment you write **any** constructor yourself, Java stops auto-generating the default one — if you still want a no-arg constructor, you must write it explicitly.
- **Constructor overloading** — like method overloading, you can have multiple constructors with different parameter lists.
- Without an explicit constructor initializing them, instance fields get default values: numeric types → `0`, `boolean` → `false`, object references (including `String`) → `null`.

---

## 11. Static Keyword

### 11.1 Static variables
```java
class Mobile {
    String brand;
    int price;
    static String name = "SmartPhone";   // shared across ALL objects
}
```
- Instance variables (`brand`, `price`) are **per-object** — every object gets its own copy.
- `static` variables belong to the **class**, not to any individual object — there is only **one** copy, shared by everyone.
- Changing it through *any* object (or, better, through the class name) affects it for *all* objects: `Mobile.name = "Phone";`
- Access static members via the **class name** (`Mobile.name`), not an object reference — using an object reference works but triggers a compiler warning and is discouraged.

### 11.2 Static methods
```java
public static void show1() {
    System.out.println("in static method");
}
// called as: Mobile.show1();
```
- Callable directly via the class name — no object needed.
- **Cannot directly access non-static (instance) variables** — because a static method isn't tied to any particular object, so Java doesn't know *whose* instance variable you mean. You *can* access an instance variable indirectly if you pass in an object reference as a parameter.
- This is exactly why `main` is declared `static`: execution has to start *before* any object exists, so `main` can't require an object to be called.

### 11.3 Static blocks
```java
class Mobile {
    static String name;
    static {
        name = "Phone";
        System.out.println("in static block");
    }
    public Mobile() {
        System.out.println("in constructor");
    }
}
```
- Runs **once**, when the class is first **loaded** by the JVM — regardless of how many objects you later create.
- Order: class loads (static block runs) → *then* the constructor runs for each object.
- If you never create an object of the class (and never explicitly trigger loading), the static block never runs.
- You can force class loading without creating an object using `Class.forName("Mobile")` (throws a checked exception — handling covered later when exceptions are introduced).

---

## 12. Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Class / Interface | PascalCase (starts capital) | `Calculator`, `Runnable` |
| Variable / Method | camelCase (starts lowercase) | `marks`, `showMyMarks()` |
| Constant | ALL_CAPS with underscores | `MY_DATA`, `PI` |

Java uses **camelCase**, not snake_case (`show_my_marks` works syntactically but isn't idiomatic Java and will look out of place to other Java developers).

---

## 13. Inheritance

### 13.1 "is-a" relationship
```java
class Calculator {
    public int add(int n1, int n2) { return n1 + n2; }
    public int sub(int n1, int n2) { return n1 - n2; }
}

class AdvancedCalculator extends Calculator {
    public int multi(int n1, int n2) { return n1 * n2; }
    public int div(int n1, int n2) { return n1 / n2; }
}
```
- `AdvancedCalculator` (**subclass / child / derived class**) automatically gets everything from `Calculator` (**superclass / parent / base class**) via the `extends` keyword — no need to redefine `add`/`sub`.
- Avoids **redundancy** — don't repeat method definitions across classes.
- Inheritance only needs the **compiled `.class` file** of the parent, not the `.java` source.

### 13.2 Multilevel inheritance
```java
class VeryAdvancedCalculator extends AdvancedCalculator {
    public double power(int n1, int n2) { return Math.pow(n1, n2); }
}
```
`VeryAdvancedCalculator` → `AdvancedCalculator` → `Calculator` — a chain. (Two classes total = **single-level**; three or more in a chain = **multilevel**.)

### 13.3 No multiple inheritance
```java
class C extends A, B { }   // COMPILE ERROR — not allowed in Java
```
Java does **not** support a class inheriting from two classes directly. Reason: the **ambiguity problem** — if both `A` and `B` define a method `y()`, and `C extends A, B`, there's no way to know which `y()` a `C` object should use. Rather than solving this (like C++ does), Java simply disallows it. (Java later added a workaround via **interfaces** with default methods — covered when interfaces are introduced.)

---

## 14. `this` and `super`

### 14.1 Every class implicitly extends `Object`
Even if you don't write `extends Something`, every class in Java implicitly extends the built-in **`Object`** class.

### 14.2 `super` — call the superclass's constructor
```java
class A {
    public A() { System.out.println("in A"); }
    public A(int n) { System.out.println("in A int"); }
}

class B extends A {
    public B() {
        // super();  <-- implicit, added by the compiler if you don't write one
        System.out.println("in B");
    }
    public B(int n) {
        super(n);   // explicitly calls A's parameterized constructor instead of the default
        System.out.println("in B int");
    }
}
```
- **Every constructor's first statement is implicitly `super()`** (calling the parent's no-arg constructor) unless you explicitly write a different `super(...)` call yourself.
- This is why creating an object of a subclass always runs the superclass's constructor too — by default the **no-arg** one, unless you explicitly call a parameterized version.
- At the top of the chain, `super()` in the topmost user-defined class calls `Object`'s constructor.

### 14.3 `this(...)` — call another constructor in the same class
```java
public B() {
    this(5);   // calls B's own parameterized constructor, which then calls super(5)
    System.out.println("in B");
}
public B(int n) {
    super(n);
    System.out.println("in B int");
}
```
`this(...)` lets one constructor delegate to another constructor in the *same* class, avoiding duplicated initialization logic. `this(...)` and `super(...)` can't both be the first statement in the same constructor — you pick one chain or the other.

---

## 15. Method Overriding

```java
class A {
    public void show() { System.out.println("in A show"); }
    public void config() { System.out.println("in a config"); }
}

class B extends A {
    @Override
    public void show() { System.out.println("in B show"); }   // same name, same params -> overrides A's version
}
```
- **Same method name, same parameter types** in the subclass as in the superclass → the subclass version takes priority ("your own features first" — like preferring your own phone over your dad's).
- `config()` isn't overridden, so `B` still inherits `A`'s version unchanged.
- Contrast with **overloading**: overloading = same name, *different* parameters, resolved at **compile time**. Overriding = same name, *same* parameters, resolved at **runtime** — this is **runtime (dynamic) polymorphism**.

---

## 16. Packages

### 16.1 Why
Like folders organize files, **packages** organize related classes.
```java
package tools;

public class Calc {
    // ...
}
```
- The `package` statement must be the first line of the file, and the file's folder location must match the package name.
- Classes with a `main` method are conventionally kept **outside** any custom package (or in a clearly separate one) so other packages can be imported cleanly into them.

### 16.2 Importing
```java
import tools.Calc;
import tools.AdvancedCalculator;
import java.util.ArrayList;   // inbuilt Java classes live in packages too
```
- Every class you use that lives in a different package must be imported.
- Package names mirror folder structure: `java.util.ArrayList` = the `ArrayList` class inside the `java` → `util` folder path.
- `java.lang` (which contains `String`, `System`, etc.) is **automatically imported** into every Java file — that's why you never need `import java.lang.System;`.

---

## 17. Access Modifiers

| Modifier | Same class | Same package | Subclass (different package) | Everywhere |
|---|:---:|:---:|:---:|:---:|
| `private` | ✅ | ❌ | ❌ | ❌ |
| *default* (no modifier) | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

**General guidance:**
- Classes → `public`.
- Instance variables → `private` (always, to preserve encapsulation).
- Methods → usually `public`; use `protected` only when a member should be visible to subclasses in other packages but not the whole world.
- **Avoid the default (no-modifier) access** — it's vague; always choose `private`, `protected`, or `public` deliberately.
- Only one `public` class is allowed per `.java` file, and the filename must match that public class's name.

---

## 18. Polymorphism

"Poly" (many) + "morphism" (forms/behavior) — the same reference or method call behaves differently depending on context.

| Type | Also called | Resolved at | Example |
|---|---|---|---|
| Compile-time polymorphism | Early binding | Compile time | Method **overloading** |
| Runtime polymorphism | Late binding | Runtime | Method **overriding** |

### 18.1 Dynamic Method Dispatch (runtime polymorphism in action)
```java
class A { public void show() { System.out.println("in A show"); } }
class B extends A { public void show() { System.out.println("in B show"); } }
class C extends A { public void show() { System.out.println("in C show"); } }

A obj = new B();
obj.show();       // "in B show"

obj = new C();
obj.show();       // "in C show" -- same reference variable, different behavior!
```
- You can declare a reference of the **parent** type but assign it an object of a **child** type: `A obj = new B();` — valid because `B` **is-a** `A`.
- Which `show()` runs is decided at **runtime**, based on the actual object type, not the declared reference type. This only works because of inheritance — you can't assign an unrelated class's object to an unrelated reference type.
- This mechanism (deciding which overridden method to call at runtime) is called **dynamic method dispatch**.

---

## 19. `final` Keyword

`final` has three different effects depending on where it's used:

```java
final int num = 8;
num = 9;   // COMPILE ERROR — cannot reassign a final variable (it's now a constant)

final class Calc { ... }
class AdvancedCalc extends Calc { }   // COMPILE ERROR — cannot extend a final class

class Calc {
    public final void show() { ... }
}
class AdvancedCalc extends Calc {
    public void show() { ... }   // COMPILE ERROR — cannot override a final method
}
```
| Applied to | Effect |
|---|---|
| Variable | Becomes a constant — value can't be reassigned after initialization |
| Class | Cannot be **subclassed** (inheritance blocked) |
| Method | Cannot be **overridden** by a subclass |

Good practice: mark any variable that shouldn't change (e.g. `PI`, config values) as `final`.

---

## 20. The `Object` Class

Every class implicitly extends `Object`, which is why even an empty class gives you methods like `equals()`, `hashCode()`, `toString()`, `notify()`, `wait()` for free.

### 20.1 `toString()`
```java
System.out.println(obj);   // implicitly calls obj.toString()
```
Default `Object.toString()` returns something like `ClassName@hexHashCode` — not useful. Override it to control what gets printed:
```java
@Override
public String toString() {
    return model + " : $" + price;
}
```
Most IDEs can auto-generate a `toString()` from selected fields (Source Action → Generate toString).

### 20.2 `equals()`
```java
Laptop obj1 = new Laptop("Lenovo Yoga", 1000);
Laptop obj2 = new Laptop("Lenovo Yoga", 1000);

obj1 == obj2;          // false -- compares object references (memory addresses)
obj1.equals(obj2);      // also false by DEFAULT -- Object.equals() also just compares references
```
To compare by **value** instead of reference, override `equals()`:
```java
@Override
public boolean equals(Object o) {
    Laptop that = (Laptop) o;
    return this.model.equals(that.model) && this.price == that.price;
}
```
- Use `String.equals()` (not `==`) to compare string *contents* — `==` on strings compares references, which can misleadingly be `true` due to the String pool, or `false` for logically-equal strings created differently.
- Whenever you override `equals()`, you should also override `hashCode()` consistently (equal objects must produce equal hash codes) — IDEs offer a combined "Generate equals() and hashCode()" action that handles null-checks and edge cases properly; prefer that over writing it by hand.

---

## 21. Upcasting & Downcasting

Builds on general typecasting (§2.4), applied to object references in an inheritance hierarchy.

```java
class A { public void show1() { ... } }
class B extends A { public void show2() { ... } }

A obj = new B();     // UPCASTING — implicit; reference type A, object type B
obj.show1();          // fine -- A knows about show1
obj.show2();          // COMPILE ERROR -- A has no idea show2 exists

B obj1 = (B) obj;     // DOWNCASTING — explicit cast required
obj1.show2();          // now works -- obj1 is typed as B
```
- **Upcasting** (child object → parent-typed reference) happens **implicitly** — no cast needed, always safe since a `B` is guaranteed to have everything `A` has.
- **Downcasting** (parent-typed reference → child-typed reference) needs an **explicit cast**, because the compiler can't guarantee the object really is that subtype — it's only safe if the underlying object actually was created as that subtype (otherwise you get a `ClassCastException` at runtime).
- `A` never knows about members defined only in `B` — the parent has no visibility into the child's extra features.

---

## 22. Wrapper Classes, Autoboxing & Unboxing

### 22.1 Why wrapper classes
Primitives (`int`, `char`, `double`...) are fast but some Java frameworks/features (e.g. the Collections framework, covered later) only work with **objects**, not primitives. For every primitive type, Java provides a corresponding wrapper **class**:

| Primitive | Wrapper class |
|---|---|
| `int` | `Integer` |
| `char` | `Character` |
| `double` | `Double` |
| `boolean` | `Boolean` |
| `byte`, `short`, `long`, `float` | `Byte`, `Short`, `Long`, `Float` |

### 22.2 Boxing / Autoboxing
```java
int num = 7;
Integer num1 = new Integer(8);   // deprecated syntax, avoid
Integer num2 = num;              // AUTOBOXING -- primitive auto-converted to wrapper object
```
**Boxing** = storing a primitive value inside its wrapper object. When Java does this **automatically** (as in the `num2 = num` line above), it's called **autoboxing**.

### 22.3 Unboxing / Auto-unboxing
```java
int num3 = num2.intValue();   // manual UNBOXING
int num4 = num2;               // AUTO-UNBOXING -- happens automatically
```
**Unboxing** = extracting the primitive value back out of a wrapper object.

### 22.4 Handy wrapper methods
```java
String str = "12";
int num = Integer.parseInt(str);   // converts a numeric String into an int
```
`Integer.parseInt()` (and equivalents like `Double.parseDouble()`) are extremely common for converting user input (which typically arrives as `String`) into usable numeric types.

---

## 23. Extra — Things Worth Reviewing From the Web

These weren't fully covered in this module (some are flagged in the course as "we'll cover this later" — likely Collections, Exceptions, Threads, Interfaces modules) but are natural next steps:

1. **Exceptions** — this module only *mentions* exceptions in passing (e.g. `ArrayIndexOutOfBoundsException`, the checked exception thrown by `Class.forName()`). The full `try`/`catch`/`finally`, checked vs. unchecked exceptions, and custom exceptions are a separate topic — review Oracle's [Exceptions tutorial](https://docs.oracle.com/javase/tutorial/essential/exceptions/).
2. **Interfaces & abstract classes** — referenced multiple times ("we'll talk about interfaces later," and as the real-world workaround for Java's lack of multiple inheritance) but not defined in this transcript. Worth reviewing before moving to Spring, since Spring relies heavily on interfaces.
3. **The Collections Framework** (`ArrayList`, `HashMap`, `List` interface, etc.) — introduced only as "the solution to array's drawbacks" but not built out. This is essential before Spring / real-world Java.
4. **`@Override` annotation** — the IDE auto-generates this above overridden methods; it's optional but strongly recommended because it makes the compiler verify you're actually overriding a real superclass method (catches typos in method signatures).
5. **Modern switch expressions** (Java 14+) — this course intentionally teaches the classic `switch`/`break` syntax since "most companies are still on Java 8." Worth knowing the newer arrow-syntax `switch` (`case 1 -> ...;`) exists and avoids fall-through bugs entirely.
6. **Text blocks, `var`, records** and other modern Java features (Java 10+) aren't covered here since the course targets a Java 8-compatible baseline — useful to know they exist for reading modern codebases.
7. **`StringBuilder` vs `StringBuffer` performance** — the video mentions thread-safety as the only difference; in practice, prefer `StringBuilder` for single-threaded code since it's measurably faster (no synchronization overhead).
8. **Garbage Collection** — mentioned briefly ("this object becomes eligible for garbage collection") when discussing immutable strings, but not explained. Worth a light read on how the JVM's GC decides when to reclaim heap objects.

---

## 24. Quick-Reference Summary

- **JVM/JRE/JDK**: JDK (dev tools) ⊃ JRE (runtime + libraries) ⊃ JVM (executes bytecode).
- **Variables**: primitive types are fixed-size value types; strings/objects are references into the heap.
- **Casting**: widening = automatic; narrowing = explicit cast, may lose data.
- **Loops**: `for` for known iteration counts, `while`/`do-while` for condition-driven loops, enhanced-`for` for clean array/collection iteration.
- **Arrays**: fixed size, zero-indexed, `.length` for safe bounds checking.
- **Strings**: immutable + pooled; use `StringBuilder`/`StringBuffer` for repeated modification.
- **OOP core four**: Encapsulation (private + getters/setters), Inheritance (`extends`, is-a), Polymorphism (overloading = compile-time, overriding = runtime), Abstraction (interfaces/abstract classes — next module).
- **Constructors**: same name as class, no return type, called automatically on `new`; `this()`/`super()` chain to other constructors.
- **static**: belongs to the class, shared by all objects; static blocks run once at class-load time.
- **Access modifiers**: private < default < protected < public, in increasing visibility.
- **Object class**: root of every class; override `toString()`/`equals()`/`hashCode()` for meaningful comparisons/printing.
- **Upcasting** is implicit and safe; **downcasting** needs an explicit cast and can fail at runtime.
