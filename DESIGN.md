# Design

Some programming languages are created to solve a certain type of problem, some are [esoteric](https://en.wikipedia.org/wiki/Esoteric_programming_language), and some are created out of frustration with existing tools

This document will serve and the living design document as this language takes shape

## Annoyances

This section will serve as a dumping ground for annoyances with existing programming languages, with the goal of finding cool problems to try and design against

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


#### Effects
##### Typescript
```typescript
export type Effect = Timing | Timed;

export type Timing = { type: "Timing", timeMs: number, label: string };
export type Timed = { type: "Timed", timeMs: number, label: string };

export function Timing(timeMs: number, label: string): Timing {
    return {type: "Timing", timeMs: timeMs, label: label};
}

export function Timed(timeMs: number, label: string): Timed {
    return {type: "Timed", timeMs: timeMs, label: label};
}
```

##### Alloy
```
data Timing = { timeMs: number, label: string }
data Timed = { timeMs: number, label: string }

union Effect = Timing | Timed
