
# SOV Programming Language

Pretty much all programming languages are constructed in analogy to the SVO (Subject Verb Object) structure found in the English language.

`Subject.Verb(Object);`

Even mathematical expressions are effectively SVO:

`7 - 3;`, `counter += 1;`

Alternatively, an VSO-like order of terms is also sometimes employed:

`foldl (+) 0 [1,2,3]`

Generally speaking, I would call these styles of programming languages "data-last", whereas in this project, I would like to consider a programming language that is "data-first", inspired by the SOV word order found in the majority of natural languages.

## Why?

The core idea is to think in terms of data flowing through the program, going through various transformations, before being spat out the other end. All programs effectively work in that way, which, at least in imperative programming languages, is generally also represented in the macro structure of the language, i.e. with a series of statements that are executed from top to bottom. However, when it comes down to the syntax of applying functions/etc., the order tends to be reversed, with the input data to the right, and the result being to the left:

```rust
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

`[1,2,3] map(inverse,_) sum(_) ~> result`

Reading from left to right, we have a term defining some data, which then put through the map function, through the sum function, and finally assigned to the result. At each step, all we have to think about, is what data we have in our hand, and what we want to do next with it. If we split this up into multiple lines, we don't have to change anything:

```rust
[1,2,3]
map(inverse,_)
sum(_)
~> result
```

Which is analogous to how it would look in an imperative style:

```rust
x = [1,2,3];
y = map(inverse,x);
z = sum(y);
result = z;
```

In the data-first style we don't have to make the choice between either writing a nested chain of functions or writing multiple sequential statements with intermediate results. Instead we simply write a sequence of functions, independent of whether we want to keep it in a single line, or split it up across multiple lines.

Of course, as is already evident above with `map`, a question that immediately comes to mind, is how to handle functions that take multiple arguments. In the case of a function like `map`, it does make sense to have the operation that we want to apply (here: `inverse`) be in some way attached to it, since in the "flowing data" intuition we don't really think about the operation we apply until we want to map it over our current data. In a way, `inverse` is less viewed as data, and more as a parameter to the transformation itself.

Perhaps, we could even incorporate this differetiation between data and parameters into the language itself:

`[1,2,3] map(inverse) sum ~> result`

with `map` being defined to have one input data stream and one parameter, and `sum` being defined to have one input data stream and no parameters.

## Multiple inputs

However, in some cases we might simply have multiple independent streams of data, with each one coming itself out of a long sequence of transformations, which we want to put into a function. In an data-last style of programming, this is handled rather neatly:

`concat(map(f, list1), reverse(list2))`, `(1000 + 729) * (4096 + 8)`

Which, besides order, does not discriminate between the two inputs.

However, this again, does not necessarily agree with ones intutition about approaching the problem. In the example

`concat(map(f, list1), reverse(list2))`

given the lists `list1` and `list2`, we would first think, in arbitrary order, of applying `map(f,_)` to the former and `reverse` to the latter, and then, finally, to apply `concat` to the two results.

Putting this into syntax, we might want to write something like this:

`list1 map(f) ~ list2 reverse ~ concat`

The `~` here signifies, that we want to put the current data stream aside and start a new one. Finally, in calling concat at the beginning of a new datastream, the ones put aside are collected, and put through the transformation.

