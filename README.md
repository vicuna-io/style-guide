# Vicuna Style Guide

## Table of Contents
[1. Motivation](#Motivation)   

[2. Rules](#Rules)   

[3. Examples](#Examples)   

[4. Tools](#Tools)

## Motivation

## Rules

### Formatting

Indent using 2 Spaces (including continuous indents)

#### Use "One True Brace Style" aka. [1TBS](https://en.wikipedia.org/wiki/Indent_style#Variant:_1TBS)
Since we want a consistent code base and reduce the vertical size of our code.
```java
// BAD  
void attack(Creature target) 
{ 
  for (Warrior warrior : warriors) { 
    warrior.attack(target); 
  } 
}

// GOOD
void attack(Creature target) { 
  for (Warrior warrior : warriors) { 
    warrior.attack(target); 
  } 
}
```
#### Use Blocks whenever applicable
To improve clarity and prevent bugs like [Apple's goto bug](https://dwheeler.com/essays/apple-goto-fail.html#targetText=On%202014%2D02%2D21%20Apple,fail%20goto%20fail%E2%80%9D%20vulnerability).

```java
// BAD  
void defeatDragon(Dragon dragon) { 
  while (dragon.isAlive()) attack(dragon); 
}

// GOOD
void defeatDragon(Dragon dragon) { 
  while (dragon.isAlive()) { 
    attack(dragon); 
  } 
} 
```

#### Max line length
Prefer to keep the amount of characters below `80` and hard wrap lines at `90` characters.

#### White spaces
Avoid using whitespaces at the beginning and at the end of class and method definitions to reduce the vertical size of our code.

```java
// BAD
public final class Person {

  private final String name;

  public String getName() {
    
    return name;
  }
  
}

// GOOD
public final class Person {
  private final String name;

  public String getName() {
    return name;
  }
}
```

#### Continuation Indent

### Forbidden Language Constructs

#### Double brace initialization
Double brace initialization creates an anonymous class with an initializer. This increases the possibility 
of memory leaks and adds a useless class definition. Furthermore this construct can only be applied to non-final classes. 
```java
// BAD
new HashMap<String, Integer>() {{
  put("one", 1);
  put("two", 2);
}}; 
```

#### Public Fields
While public fields are an easy way to provide exported properties of a data structure, they can not 
be protected against illegal access and may not be removed from a class without breaking its API.
Prefer accessors to public fields.
```java
// BAD
public final class Person {
  public String name;
}

// GOOD
public final class Person {
  private String name;

  public String getName() {
    return name;
  }
}
```

#### Public constructors
If there is no strong reason for a public constructor (like framework constraints), 
don not create one. There are several reasons not to use public constructors, 
Josh Bloch lists the following in his Book Effective Java (Item #1):

[Medium - Effective Java Item 1](https://medium.com/@biratkirat/learning-effective-java-item-1-57f85b93c254)

Create private constructors and use one of the following creators:
  
  - Static Factory Method - *When having a few (1-3) fields* 
  - Builder Class - *When having a more fields*
  - Factory Class - *When the construction of a class is complicated or requires reused dependencies.*

```java
// BAD
public final class Address {
  private static final int DEFAULT_PORT = 8080;
  private static final String DEFAULT_HOST = "localhost";
  
  private final int port;
  private final String host;
  
  public Address(int port) {
    this(DEFAULT_HOST, port);
  }

  public Address(String host, int port) {
    this.host = host;
    this.port = port;
  }
}

// GOOD
public final class Address {
  private final int port;
  private final String host;
  
  private Address(String host, int port) {
    this.host = host;
    this.port = port;
  }
  
  public static Address create(String host, int port) {
    Objects.requireNonNull(host);
    validatePort(port);
    return new Address(host, port);
  }
  
  public static Address createLocalhost(int port) {
    validatePort(port);
    return new Address("localhost", port);
  }
  
  private static void validatePort(int port) { ... }
}
```

#### Don't use null
Using null to represent absent values is a bad design decision and leads to maintainability and safety problems.  
Instead of using null, favour returning `Optional` or throwing an exception if an error caused the value to be absent. 

Instead of taking null parameters, provide an extra method or overload.

Exception to this rule are internal/private methods. 
When allowing nullable parameters or returning nullable values, use the `@javax.annotation.Nullable`-Annotation. 

#### Don't declare boolean parameters or fields
Boolean fields and parameters should be used sparingly, as they are not self-documenting.   
   
Instead of having a field of type boolean, declare an Enum.   

```java
// BAD
public final class Session {
  private boolean active;
}

// GOOD
public final class Session {
  // Must not be public
  enum State {
    ACTIVE,
    CLOSED;
  }
  
  private State state;
}
```   
Instead of a method having a boolean parameter, split it up into two methods and mark their difference by descriptive names.   
```java
// BAD
void accept(Visitor visitor, boolean recursive);

// GOOD
void accept(Visitor visitor);
void acceptRecursive(Visitor visitor);
```   

Using the method from the bad example, one would write `accept(visitor, true)` and the reader, who may not know the signature of accept, can not infer what `true` causes.    
When using the method from the good example, the same code would be written as `acceptRecursive(visitor)`, which can be understood without looking at the methods documentation. 

### Object Oriented Programming Anti-Patterns to avoid
#### Job Names (Anemic Domain Model)
There are those class names that one comes across in almost 
every ~~object oriented~~ codebase, some examples are:

- `UserManager`
- `JsonParser`
- `RequestHandler`

A lot of these classes have one single exported method like:
- `parseJson`
- `handleRequest`

Whilst others have so many responsibilities, that their name becomes very generic 
like `Manager` or `Service`. But they have something in common, they consist of procedures,
operating on data, rather than objects exposing behaviour. And some just have a badly chosen name.

The real problem is not their name but their design. In order to avoid such classes,
move stop exposing data, move methods closer to the data they operate on and start 
thinking in objects rather than procedures.

## Tools

### Checkstyle

We created a simple [checkstyle config](checkstyle.xml) that enforces major rules of our style guide. The config is based on the google config and should be used as strict as possible (warnings = errors). 
