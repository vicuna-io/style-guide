
# Vicuna Style Guide

## Table of Contents
[1. Motivation](#Motivation)   

[2. Rules](#Rules)   

[3. Examples](#Examples)   

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

#### Continuation Indent

### Forbidden Language Constructs

#### Double brace initialization
Double brace initialization creates an anonymous class with an initializer. This doesn't only increase the possibility of memory leaks, but also adds an useless class definition. Furthermore this construct can only be applied to non-final classes. 
```java
new HashMap<String, Integer>() {{
  put("one", 1);
  put("two", 2);
}}; 
```

#### Public Fields
While public fields are easy ways to provide exported properties of a data structure, they can not 
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
If there is no strong reason for a public constructor (like framework constraints), don't create one.
There are several reasons not to use public constructors, Josh Bloch lists the following in his Book Effective Java (Item #1):
[Medium - Effective Java Item 1](https://medium.com/@biratkirat/learning-effective-java-item-1-57f85b93c254)

Create private constructors and use one of the following creators:
  
  - Static Factory Method - *When having a few (1-3) fields* 
  - Builder Class - *When having a more fields*
  - Factory Class - *When the construction of a class is compilated or requires reused dependencies.*

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
## Examples
