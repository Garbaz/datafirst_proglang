
# Data-First Programming Language

Pretty much all programming languages are constructed in analogy to the SVO (Subject Verb Object) structure found in the English language.

`Subject.Verb(Object);`

Even mathematical expressions are effectively SVO:

`7 - 3;`, `counter += 1;`

Alternatively, an VSO-like order of terms is also sometimes employed:

`foldl (+) 0 [1,2,3]`

Generally speaking, I would call these styles of programming languages "data-last", whereas in this project, I would like to consider a programming language that is "data-first", inspired by the SOV word order found in the majority of natural languages.

## Why?

The core idea is to think in terms of data flowing through the program, going through various transformations, before being spat out the other end. All programs effectively work in that way, which, at least in imperative programming languages, is generally also represented in the macro structure of the language, i.e. with a series of statements that are executed from top to bottom. However, when it comes down to the syntax of applying functions/etc., the order tends to be reversed, with the input data to the right, and the result being to the left:

```
// Data flows from top to bottom (input ~> x ~> y ~> z ~> return)
foo(x) {
    y = bar(x); // But in applying functions
    z = x + y;  // or operators
    return z;   // the data is to the right.
}
```

So before we even consider what data we want to transform, and what function we want to apply, we already have to think about what we will want to do with the result. For example, say, we're given the data `[1,2,3]`, and want to take the inverse of numbers, and then sum the result up. To arrive at the final line of code that would achieve this, the thought process therefore would be:

- Given `[1,2,3]`
- Apply `map(inverse, _)`
- Apply `sum(_)`
- Save result

However, if we were to write this out as one line in a data-last programming language, we have to consider these steps backwards:

`result = sum(map(inverse, [1,2,3]))`

Which, at least to me, seems to go against the intuitive way we think about solving the problem.

## An intuitive order of terms

Taking the intuition above, a better order of terms would perhaps be something like this:

`[1,2,3] map(inverse,_) sum(_) |> result`

Reading from left to right, we have a term defining some data, which then put through the map function, through the sum function, and finally assigned to the result. At each step, all we have to think about, is what data we have in our hand, and what we want to do next with it. If we split this up into multiple lines, we don't have to change anything:

```
[1,2,3]
map(inverse,_)
sum(_)
~> result
```

Which is analogous to how it would look in an imperative style:

```
x = [1,2,3];
y = map(inverse,x);
z = sum(y);
result = z;
```

In the data-first style we don't have to make the choice between either writing a nested chain of functions or writing multiple sequential statements with intermediate results. Instead we simply write a sequence of functions, independent of whether we want to keep it in a single line, or split it up across multiple lines.

Of course, as is already evident above with `map`, a question that immediately comes to mind, is how to handle functions that take multiple arguments. In the case of a function like `map`, it does make sense to have the operation that we want to apply (here: `inverse`) be in some way attached to it, since in the "flowing data" intuition we don't really think about the operation we apply until we want to map it over our current data. In a way, `inverse` is less viewed as data, and more as a parameter to the transformation itself.

Perhaps, we could even incorporate this differetiation between data and parameters into the language itself:

`[1,2,3] map(inverse) sum |> result`

with `map` being defined to have one input data stream and one parameter, and `sum` being defined to have one input data stream and no parameters.

## Multiple inputs

However, in some cases we might simply have multiple independent streams of data, with each one coming itself out of a long sequence of transformations, which we want to put into a function. In an data-last style of programming, this is handled rather neatly:

`concat(map(f, list1), reverse(list2))`

Which, besides order, does not discriminate between the two inputs.

However, this again, does not necessarily agree with ones intutition about approaching the problem. In the example

`concat(map(f, list1), reverse(list2))`

given the lists `list1` and `list2`, we would first think, in arbitrary order, of applying `map(f,_)` to the former and `reverse` to the latter, and then, finally, to apply `concat` to the two results.

Putting this into syntax, we might want to write something like this:

`list1 map(f) | list2 reverse | concat`

The `|` here signifies, that we want to put the current data stream aside and start a new one. Finally, in calling concat at the beginning of a new datastream, the ones put aside are collected, and put through the final transformation. In a way, this `|` could be considered optional, since for example in writing `list2` in our statement above, which does not take any input, it already is clear that we have to leave the current data stream dangling and start a new one.

## Mathematical expressions

For better or worse, the use of infix operators in mathematical expressions is deeply ingrained into the way we think about calculations, even if that sometimes requires some juggling of symbols to achive the desired result. To take an example:

`(1000 + 729) * (4096 + 8)`

Writing this in the above syntax:

`1000 | 729 | + | 4096 | 8 | + | *`

or, taking `|` as optional, perhaps more clearly as:

`1000 729 + | 4096 8 + | *`

This effectively gives us our expression in Reverse Polish notation.

While perfectly readable with some rethinking, I still would consider it perhaps sensible to take a similar route to the one Haskell took, and allow symbolic infix operators, while also keeping open the option to turn an infix operator into a function when needed.

## Function definitions

Taking the above idea of separating a function's arguments into data and parameters, perhaps a syntax for defining a function as follows would make sense:

```
(List) \add_to_each(n:Number) -> List {
    map((n+))
}
```

With allowing for type inference as done in Rust or Haskell:

```
\add_to_each(n) {
    map((n+))
}
```

As with the idea of parameters, in writing the function name before the code, we do go against the idea of data-first for the sake of readability. Generally, when looking at a function again at a later point in time, we are primarily interested in it's signiature, so it makes sense to have it all in one place.

An alternative syntax that adheres more strictly to the data-first principle:

```
(List) \(n) -> List {
    map((n+))
} |> add_to_each
```

Or with inferred types and written into one line:

`\(n) { map((n+)) } |> add_to_each`

However, I would not consider these two styles mutually exclusive. In fact, if we consider the first syntax with the permission for a function to be anonymous:

`\add_to_each(n) { map((n+)) }` ==> `\(n) { map((n+))}`

Then it would make sense for this to be considered simply a statement of data to be transformed or assigned as usual, automatically giving us the option for the second style of syntax.

## Data to parameters

With the ability to treat a function like data, it follows that we perhaps would like to put it through a few transformations, before finally still taking it as a parameter. A way to allow for this, is to simply mark a parameter as such:

```
[1,2,3] |
inverse |
map(_) sum
```

With the rule here being, that the actual data streams come first, and the parameter data streams come last.

