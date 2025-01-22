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
a method named `String` with no paramters and return type `string`.

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
let us consider again the `pet_adoption` package, and examine a weird quirk of
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
intuitions rather than having you focusing on the solution _you_ must implement.
(The people who teach and have been taught to use UML diagrams to "engineer"
systems, for example.) But in my experience that is the wrong approach to
problem solve, and in my opinion, especially in engineering fields. (I have
tried that top-down approach many times. From Java and C++ to Rust and Haskell.)

## Problem solving and Go code

Instead of trying to explain what problem solving I am talking about, I will
just refer to George Pólya's amazing [problem solving list](https://sass.queensu.ca/sites/sasswww/files/uploaded_files/Resource%20PDFs/polya.pdf).
(One could say we are talking about pure logical, deductive problem solving here.)

Particularly, the resemblence between problem solving and coding is that solving
a problem equates to implementing a function. The function declaration
(including the properties asserted in the doc comment) is a statement of the
problem. The process of implementing the function is the process of finding
the solution.

The paramater data types of the function are the given data for a problem. More
often than not, when you are solving a problem for the first time, you will
use concrete data, more specialised input data forms to solve your problem.
After you have solved your problem, or in the process, you may realise that
you only need specific properties of the data, and you can solve a generic
problem by generalising the data in your problem statement, then you can obtain
a generic solution.

To be less abstract, and finally make my point clearly, when implementing a
function, start with concrete types in parameters, and if you find that you
need only specific methods of the input data, then you can make an interface
to substitute the concrete type.

So how does this help with `pet(&Human{})`? Well, it does not. In Go, it is the
responsibility of the caller of `pet` to not call it with some variable for
which it does not make much sense. And I find it to be more productive to worry
about using my data correctly because it lets me focus on solving the problem
at hand, than to think about how to create a perfect world model in which I can
then restate my problem and find a nice solution. It is a good thing to try to
find an existing world model in which your problem can solved more simply. (Basically looking for an existing library.) But if that world model has not
been implemented, just use the tools at your disposal. And at the minimum, functions and plain-old-data types are always there, and those are the tools
that any new programmer and software engineer is taught from the get go.