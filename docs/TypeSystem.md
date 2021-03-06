# Type System

## Everything is a type

Types are types, but so are concrete values, and so are functions. Objects are types, tuples are types, even parameters blocks are types.

What matters is the type hierarchy. `7` is an `integer`, an `integer` is a `number`, and a `number` is a `_`. Everything is a `_`.

## Comparing types

When checking for type equality or assignability, types are boiled-down to **3 core structures**: functions, objects and unions of objects.

A `string` is just a `[char]`, and all sequences `[]` are just objects with a `getIterator` function. A `char` is an object and so is a `number` too.

Intersection types create new object signatures and are therefore just objects themselves. The following lines are equivalent:

```
newType = { name = "Fred", age = 30 } & { email = "fred@example.com" }
newType = { name = "Fred", age = 30, email = "fred@example.com" }
```

Unions, however, cannot always create certain object signatures. When properties are the same, that is still certain, but some properties must remain unions.

```
newType = { name = "Fred", age = 30 } | { name = "Fred", age = 31 }
newType = { name = "Fred", age = 30 | 31 }
```

A similar thing happens with a function's parameter and return type. Thus, a 'union of functions' is not one of the 3 core structures.

```
newFunc = in => out | somethingElse => out
newFunc = in | somethingElse => out
```

## Scoping

Types can be overridden in a sub scope only to make them more specific.

```
name = string

{
    name = "Fred" // valid because "Fred" is a subtype of string

    print name // will print "Fred"
}

{
    name = 123 // invalid because 123 is not a subtype of string
}
```

## Refinement Types

Intersection types create more restricted types; the new type must be assignable to all the types in the intersection.

This idea can be combined with a predicate function to create a refinement type. To do this, we create an intersection of any type with an object containing an `isValid` function.

```
multipleOf5 = integer & { isValid = i => is modulo i 5 0 }
```

The type `multipleOf5` can only accept numbers that are multiples of 5. `10` is a `multipleOf5`.

## The concretion problem

> :warning: This is an unsolved problem.

Functions that serialise a type for printing to the console or sending via an API request cannot be sure if given parameters or return values are concrete values or just subtypes. Consider this example:

```
delayedPrint = { message = string, delay = timespan } => {
    sleep delay
    print message
}

possibleNames = "Fred" | "Jane" | "Sam"

delayedPrint { message = possibleNames, delay = seconds 200 }
```

The solution to this problem requires some concept of [concretion](ramblings/ConcretionProblem.md).

## Compile time

Compiler must be aware of whether a type is a function or not.

```
possibleNames = string & { isValid: s => is s.length greaterThan 5 }
```