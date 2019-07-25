# Design

Some programming languages are created to solve a certain type of problem, some are [esoteric](https://en.wikipedia.org/wiki/Esoteric_programming_language), and some are created out of frustration with existing tools

This document will serve and the living design document as this language takes shape

## Annoyances

This section will serve as a dumping ground for annoyances with existing programming languages, with the goal of finding cool problems to try and design against

### Various idioms for communicating failure

Functions fail. Sometimes these failures (we will prefer the term "errors") are unexpected, network failures, out of memory, etc.; but more often than not failures is a normal part of using standard library APIs.

One annoyance with various languages is the inconsistant and seemily random ways of communicating failure.

#### String.indexOf

Let's look at the example `int String.indexOf(char)` from Java (and many, many other languages as well).
As one may expect, this method returns an integer that represents the index of the character that's passed as an argument.
```java
"Hello".indexOf('e') // returns 1
"world".indexOf('d') // return 4
```
But, what happens when you pass in a character that is not present? How is failure communicated?

In the case of `indexOf`, the lowest possible integer than can be returned in a success case is a `0`. In Java, `indexOf` returns `-1` when the character isn't present in the String!
```java
"Hello".indexOf('q') // returns -1
```
The function returns a value that would be impossible to receive during a success.

This return value does communicate failure, but it also requires some specific knowledge and expectation of what values will be returned in order to use this function correctly.

#### Summary

Ways to communicate failure:
1. Return default values: `String.indexOf() => -1`
2. Exceptions: `Integer.parse => BOOM` or `List.get() => BOOM`
3. Return null: `Map.get() => null`
4. ???
5. Type safe containers: `List<T>.first(predicate) => Option<T>` or ``

Alloy should prefer keeping things explicit, probably by using type safe containers to communicate the possibility of failure.

## Delights

This section will serve as a scrap paper for delights with existing programming languages, with the goal of finding cool ideas to try and replicate

### Data classes/data objects/records

#### Simple unions
##### Typescript
```typescript
type False = "false";
type True = "true";

const False: Bool = "false";
const True: Bool = "true";

type Bool = False | True  
```

##### Alloy
```typescript
data Bool = False | True  
```

#### Unions that hold data
##### Typescript
```typescript
type Circle = [number, number, number];
type Rectangle = [number, number, number, number];

type Shape = Circle | Rectangle;
```

##### Alloy
```
data Shape = Circle Float Float Float | Rectangle Float Float Float Float
```

#### Records
##### Typescript
```typescript
type Person = {
  firstName: string,
  lastName: string,
  age: number,
  height: number,
  phoneNumber: string,
  flavor: string,
};
```
##### Alloy

Just like many modern languages, Alloy has variable types on the left side of the name.  
Those familiar with Haskell or Elm might recognize the style of having the `,` on the front.  

The `:` is allowed to have any amount of whitespace between the variable and the Type.

```
data Person = Person { firstName:  String
                     , lastName :  String
                     , age:        Int
                     , height   :  Float
                     , phoneNumber:String
                     , flavor : String
                     }
```

#### Type Constructors
##### Typescript
```typescript
type Circle = [number, number, number];
type Rectangle = [number, number, number, number];

type Shape = Circle | Rectangle;

function circle(x: number, y: number, radius: number): Circle {
  return [x, y, radius];
}

function rectangle(upper_right_x: number, upper_right_y: number, lower_left_x: number, lower_left_y: number): Rectangle {
  return [upper_right_x, upper_right_y, lower_left_x, lower_left_y];
}

```

##### Alloy
```
data Shape = Circle Float Float Float | Rectangle Float Float Float Float
```

#### Type Constructors - Records
##### Typescript
```typescript
type Person = {
  firstName: string,
  lastName: string,
  age: number,
  height: number,
  phoneNumber: string,
  flavor: string,
};
```

##### Alloy

```
data Person = Person { firstName:  String
                     , lastName :  String
                     , age:        Int
                     , height   :  Float
                     , phoneNumber:String
                     , flavor : String
                     }
                     

```

## Reference

### Structural type systems
- https://en.wikipedia.org/wiki/Structural_type_system
