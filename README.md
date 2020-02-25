# GommeHD.net Style Guide

## Formatting

Indent using 2 Spaces (including continuous indents)

### Use "One True Brace Style" aka. [1TBS](https://en.wikipedia.org/wiki/Indent_style#Variant:_1TBS)
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

### Use Blocks whenever applicable
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

### Max line length
Prefer to keep the amount of characters below `80` and hard wrap lines at `90` characters.

### Blank Lines
Avoid using blank line at the beginning and at the end of class and method definitions to reduce the code's vertical size.

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

### Whitespaces
Don't put whitespaces after parentheses and generally use them sparingly. Horizontal length is one of the most important properties of clean formatting. You should not waste your horizontal space with whitespaces. They can also make code harder to read.

You should however put whitespaces after control structure keywords like `if` and `for`.

```java
// BAD
Sandwich prepareSandwich( Ingredients ingredients, SandwichRecipe recipe ) {
  synchronized ( kitchen ) {
    return recipe.prepare( ingredients ); 
  }
}

// GOOD
Sandwich prepareSandwich(Ingredients ingredients, SandwichRecipe recipe) {
  synchronized (kitchen) {
    return recipe.prepare(ingredients);
  }
}
```

## Naming



## Forbidden Language Constructs

### Double brace initialization
Double brace initialization creates an anonymous class with an initializer. This increases the possibility of memory leaks and adds a useless class definition. Furthermore this construct can only be applied to non-final classes.

```java
// BAD
new HashMap<String, Integer>() {{
  put("one", 1);
  put("two", 2);
}}; 
```

### Public fields
While public fields are an easy way to provide exported properties of a data structure, they can not be protected against illegal access and may not be removed from a class without breaking its API. Prefer accessors to public fields.

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

### Public constructors
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

### Don't use null
Using null to represent absent values is a bad design decision and leads to maintainability and safety problems.  Instead of using null, favour returning `Optional` or throwing an exception if an error caused the value to be absent.

Instead of taking null parameters, provide an extra method or overload.

Exception to this rule are internal/private methods. 
When allowing nullable parameters or returning nullable values, use the `@javax.annotation.Nullable`-Annotation. 

### Don't declare boolean parameters or fields
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

## Object Oriented Programming Anti-Patterns to avoid

### God Object
God objects know too much and are known to well. They often store 'global variables' or
dependencies and are referenced to access them in other classes. Sometimes they also
contain methods that can be thought of as procedures (procedural programming). Those
objects result in code being harder to test, extend and maintain. Suddenly every class
will be coupled to this God object. 

Instead use dependency injection and apply the [information expert principle](https://en.wikipedia.org/wiki/GRASP_(object-oriented_design)#Information_expert).

### Job Names (Anemic Domain Model)
There are those class names that one comes across in almost every ~~object oriented~~ codebase, some examples are:

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

## Other bad practices

### Anonymous classes instead of lambdas
Some developers think using anonymous classes instead of lambdas would give them some performance benefits. This isn't true, at least not for the OpenJDK. While linking time may be affected by lambdas, their implementation doesn't really differ from that of anonymous classes. They actually define an anonymous class at link-time. One difference is, that lambdas, defined in instance methods, won't capture the instance variable unless required. Anonymous classes always capture it, resulting in a higher chance of memory leaks. Lambdas and especially stateless lambdas can be optimized quite well by the JVM. 

One major difference is, that lambdas are much easier to read and don't consume the amount of space that anonymous classes do. 

### Lambdas with blocks
Lambdas should be small. Try not to define lambdas with more than one statement.
Instead create a new method and call it from within the lambda.

```java
// BAD
shoppingCart.removeIf(item -> {
  var owned = inventory.lookup(item.id());
  return owned != null && !owned.isBroken();
});

// BETTER
shoppingCart.removeIf(item -> isAlreadyOwned(item));

// BEST
shoppingCart.removeIf(this::isAlreadyOwned)
```

### Magic numbers
When looking at someone else's code, one often comes across magic numbers. They have a meaning which is often only known by the author. Instead of using magic numbers, use number constants.

The only number literals allowed in code are `-1`, `0`, and `1`. Which doesn't mean that you shouldn't put them into constants.

```java 
// BAD
int reward = ((60 / time) * 100) + (kills * 2);

// GOOD
int reward = ((TIME_LIMIT / time) * TIME_REWARD) + (kills * KILL_REWARD));
```

## Principles to follow
- Design By Contract
- Law Of Demeter
- Law of Least Astonishment
- Command Query Separation (if applicable)

## Checkstyle

We created a simple [checkstyle config](checkstyle.xml) that enforces major rules 
of our style guide. The config is based on the 
[google config](https://github.com/checkstyle/checkstyle/blob/master/src/main/resources/google_checks.xml) 
and should be used as strict as possible (warnings = errors). 