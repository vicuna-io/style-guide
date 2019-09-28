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

## Examples
