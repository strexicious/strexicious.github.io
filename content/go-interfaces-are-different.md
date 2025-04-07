+++
date = '2025-04-07T15:13:17+02:00'
draft = true
title = 'Go Interfaces Are Different'
+++

# Go Interfaces Are Different

Different to what? Well, compared to most other languages. Although they are
named "interfaces", they are not to be (or rather, must not be) used in the same
manner as promoted in other languages.

## The manners of other languages

In other languages you if you want a cat to be an animal, the concept of animal
must be defined, and then definition of cat must explicitly declare that a cat
is an animal. In principle, this explicitness, sounds like a good thing to
require.

```
// in some not go language
// an animal-kingdom package

interface Animal {
  ...
}

class Cat extends Animal {
  ...
}
```

Before we move onto how Go interfaces are intended to be used, let's look at
how might a user of this package use these definitions, so that we have a point
of comparison for Go.

```
// in some not go language
// a pet-simulator package

void pet(Animal a) { ... }
```

## Go's manners

Let's quickly look at how the same code in Go would be separated.

```
package animal_kingdom

type Cat struct {
  ...
}
```

And then...

```
package pet_simulator

type Animal interface {
  ...
}

func pet(Animal a) { ... }
```

Do you see anything funny? (If not, take a closer look at how the code is
organised.) What's going on?

So basically, in Go, the implementer of `pet_simulator` defines the interface
that it wants to use. But the implementer of `pet_simulator` is a user of the
`animal_kingdom` package, and as such, the definition of cat can never explicitly
say that it is an animal since it does not know who the user will be a priori.

To summarise: in Go the _user_ of some code is suppose to provide an
abstraction for some other code, _not_ the _implementer_ of that other code.

But then how does Go know that something like `type Airplane struct { ...}` is
not an animal? Well let's get a bit technical now (but I am no expert)...

## Method sets

In Go, each type has a method set. To quickly illustrate, in the following,
`M1` and `M2` are in the method set of `T`.

```
type T { ... }

func (T) M1() { ... }
func (T) M2() { ... }
```

And when you define an interface in Go, you are basically defining a method
set that is not tied to any type. To give a quick example, let's look at a
standard library interface:

```
type Stringer interface {
  String() string
}
```

This is a method set not associated to any type. In this case, it just declares
a method named `String` with no parameters and return type `string`.

## How is `Airplane` not an `Animal`?

Well in the previous discussion we have been ignoring the definition of
`Animal`. Suppose it looks something like this:

```
type Animal interface {
  Walk(distance float64)
}
```

Now, if `Airplane` has a method named `Walk` that accepts a single `float64`
and does not return anything, then `Airplane` is considered an `Animal`
according to Go. But it is unlikely that `Airplane` has a method named `Walk`.

This is how Go interfaces work. They are implicitly "attached" to types. And so,
let us consider again the `pet_simulator` package, and examine a weird quirk of
how Go intends for us to use the interfaces. To clarify, the package looks like
this with our `Animal` interface better defined now:

```
package pet_simulator

type Animal interface {
  Walk(distance float64)
}

func pet(Animal a) { ... }
```

And we are going to consider another package now:

```
package human_simulator

type Human struct {
  ...
}

func (h *Human) Walk(distance float64) { ... }
```

And apparently now the following code is valid: `pet(&Human{})`. What the
flip-flops weeb stuff is going on here?? This is terrible! If only we could
let the implementer of cat explicitly declare itself as an animal from animal
kingdom then this would never happen! Go interfaces suck!

## Hold on... don't jump to conclusions yet

So what went wrong in the preceding discussion?

Well nothing and everything. It depends on the experiences of the reader. The
discussion up to this point has been to reel in the readers who have been taught
that software engineering is about designing systems from top to bottom,
starting with higher up abstractions that are closer to _user_'s notions and
intuitions rather than having you focusing on the solutions _you_ must implement.

What Go then emphasises for software engineers is that they should focus on the
problem at hand, giving them liberty in how to use plain-old data and making
them use their own judgement if a given code has interpretation that could be
considered valid. (Maybe, someone someday does find it useful for their code
to call `pet(&Human{})` in their problem domain.)

I have tried top to bottom approaches in the past different languages, from Java
and C++ to Rust and Haskell. But I find it more productive to worry about using
my data correctly because it lets me focus on solving the problem at hand, than
to think about how to create a world model in which I can then restate my
problem and finally find a nice solution. Unless a model for your problem
already exists (that is, a library), creating world models just adds an overhead
to the engineering process, especially when the requirements of the system keep
changing day to day.

## Problem solving and Go code

Instead of trying to explain what problem solving I am talking about, I will
just refer to George Pólya's amazing [problem solving list](https://sass.queensu.ca/sites/sasswww/files/uploaded_files/Resource%20PDFs/polya.pdf).
(One could say we are talking about pure logical, deductive problem solving here.)

Particularly, the resemblance between problem solving and coding is that solving
a problem equates to implementing a function. The function declaration
(including the properties asserted in the doc comment) is a statement of the
problem. The process of implementing the function is the process of finding
the solution.

The parameter data types of the function are the given data for a problem. More
often than not, when you are solving a problem for the first time, you will
use concrete data, more specialised input data forms to solve your problem.
After you have solved your problem, or in the process, you may realise that
you only need specific properties of the data, and you can solve a generic
problem by generalising the data in your problem statement, then you can obtain
a generic solution.

In conclusion, and to be less abstract, and finally make my point clearly, when
**implementing** a function, start with concrete types in parameters, and if you
find that you need only specific methods of the input data, then you can make an
interface to substitute the concrete type. And when **using** a function that
accepts an interface and you have a concrete type that satisfies such an
interface, verify yourself, as an engineer, if semantics of such code are valid.
