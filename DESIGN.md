# Design

Some programming languages are created to solve a certain type of problem, some are [esoteric](https://en.wikipedia.org/wiki/Esoteric_programming_language), and some are created out of frustration with existing tools

This document will serve and the living design document as this language takes shape

## Annoyances

This section will serve as a dumping ground for annoyances with existing programming languages, with the goal of finding cool problems to try and design against

### Various idioms for communicating failure

Functions fail. Sometimes these failures (we will prefer the term "errors") are unexpected, network failures, out of memory, etc.; but more often than this, failures are a normal part of using standard library APIs.

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

#### Integer.parse

I'm going to continue to pick on Java.  
There exists a method in Java to convert from an `Integer` to a `String`: `Integer.parse`.  
As one may expect, this method returns an integer from its' string representation.
```java
Integer.parse("1") // returns 1
Integer.parse("987562") // returns 987562
```

What happens when you pass in a string that cannot be converted to an integer?

In the first example, the return type was an Integer. When finding the index of a character in a string, valid indexes are in the set of integers greater than or equal to 0. In a failure case, where you don't find the character, it therefore is possible to return a negative number!

But in this case, strings can represent both positives and negative ints. Default values can't be used to communicate failure to parse a string.

#### Summary

Ways to communicate failure:
1. Return default values: `String.indexOf() => -1` or `User.name => ""`
2. Exceptions (Checked and Unchecked): `Integer.parse => BOOM` or `List.get() => BOOM`
3. Return null: `Map.get() => null`
4. Type safe containers: `List<T>.first(predicate) => Option<T>` or `GitHubClient.fetchRepos(username) => Either<TFailure, TSuccess>`

Alloy should prefer keeping things explicit, probably by using type safe containers to communicate the possibility of failure.

## Delights

This section will serve as a scrap paper for delights with existing programming languages, with the goal of finding cool ideas to try and replicate

### Type systems

In the C language, there is no boolean type. Instead, there are two constants: `TRUE = 1;` and `FALSE = 0;`. This pattern of `1` as true and `0` as false has become intuitive for many developers.  
Enums in C are actually just integers dressed up with a bowtie. `typedef enum {FALSE = 0, TRUE} boolean;` allows developers to specify their variable types as `boolean`. But what happens when you pass in value `2` into a function expecting `boolean`?

Many Java developers will know the pain of using a `switch/case` statement with an enum. If your enum is defined `enum Color { RED, BLUE, GREEN; }`, why is it necessary to have a `default` block on your case statement when you define a `case` for all of the possibilities?

Alloy aims to solve some of these problems. Using a mixture of structural typing and nominal typing, Alloy should have all the tools necessary to communicate any type of data using explicit types.

Strucutral typing is a classification of type system, where type compatibility and equivalence are based on the properties (structure) of the given type.
Unlike "duck typing", which depends on the characterics of a type at runtime, structural typing is based on compile time characterics.  
Typescript is probably the best example of a very popular language that uses a structural typing strategy.  

Nominal typing is a classification of type system, where type compatibility and equivalence are based on explicit declarations such as the name of a type, or the place of declaration.
Java, C#, and many other popular "static" programming languages use nominal typing.

Below, we go into specific examples of nominal typing, here used for types commonly known as union types, case classes, sealed classes, or algebraic data types.

### Unions

Most of the following examples are using Typescript. Typescript uses a structured type system, so assigning names to types needs to be part of the type structure.

Union types are really useful for explicitly defining possibilities. Alloy aims to also allow the number of possibilities to be limitted to exactly what is desired.

Lets look at an example:
```typescript
// Color has 3 possibilities
type Color = Red | Yellow | Green;

// Size has 2 possibilities
type Size = Small | Large;

// Shirt has 2 x 3 = 6 possibilities
type Shirt = [Color, Size];
```

What if we want to remove "Green" as a `FallColor`?
```alloy
data Color = Red | Yellow | Green

data FallColor = Red | Yellow

// Alternatively
data FallColor = Color where not Color.Green
```

What if we don't want to allow the combination of "Small" and "Red" for your shirts?
```alloy
data Color = Red | Yellow | Green

data Size = Small | Large

data Shirt = (Color, Size) where not (Color.Red, Shirt.Small)
```

Alloy aims to make union types easy to define and use, without a lot of boilerplate.

#### Simple unions

Alloy aims to provide unions that can act as enums, while using characteristics of nominal type systems to keep enums from being used in the wrong places. 

##### Typescript
Booleans:
```typescript
type False = { _type: "false" };
type True = { _type: "true" };

const False: Bool = { _type: "false" };
const True: Bool = { _type: "true" };

type Bool = False | True;
```

Colors:
```typescript
type Red = { _type: "red" };
type Yellow = { _type: "yellow" };
type Green = { _type: "green" }

const Red: Color = { _type: "red" };
const Yellow: Color = { _type: "yellow" };
const Green: Color = { _type: "green" };

type Color = Red | Yellow | Green;
```
##### Alloy
Booleans:
```
data Bool = False | True
```

Colors:
```
data Color = Red | Yellow | Green
```

#### Unions that hold data

It often makes sense to have your unions hold data, making them quite a bit more powerful than normal enums.
This is great for simple stuff, but it quickly becomes difficult to reason about.

##### Typescript
```typescript
type Circle = { _type: "circle", data: [number, number, number] };
type Rectangle = { _type: "rectangle", data: [number, number, number, number] };

type Shape = Circle | Rectangle;
```

##### Alloy
```
data Shape = Circle Float Float Float | Rectangle Float Float Float Float
```


### Data classes/data objects/records

Here, we go into specific examples of structural typing.

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
data Person = Person { firstName.  : String
                     , lastName    : String
                     , age         : Int
                     , height      : Float
                     , phoneNumber : String
                     , flavor      : String
                     }
                     

```

### Explicit nullability

Kotlin has a special way to communicate the possibility of a `null` value.

`val name: String?` has the possibility of being `null`. `val name: String` will never be `null`. 

Technically, `String?` is equivalent to `String | null`. 

While Alloy wants to keep types explicit, as well as avoiding the use of a `null` type, we will consider providing language level tooling (such as this) for making the `Option` type easier to work with.

### Expressions vs Statements

## Reference

### Syntax
- ReScript: https://rescript-lang.org/docs/manual/latest/overview
- Elm: https://elm-lang.org/docs/syntax

### Type systems
- Structural: https://en.wikipedia.org/wiki/Structural_type_system
- Types as sets: https://guide.elm-lang.org/appendix/types_as_sets.html
- Default/Zero values: https://tour.golang.org/basics/12
- Pattern Matching: https://rescript-lang.org/docs/manual/latest/pattern-matching-destructuring
- Types as Sets: https://guide.elm-lang.org/appendix/types_as_sets.html
- GADTs: https://en.m.wikibooks.org/wiki/Haskell/GADT


