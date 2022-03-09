
# SOV Programming Language

Pretty much all programming languages are constructed in analogy to the SVO (Subject Verb Object) structure found in the English language.

`Subject.Verb(Object);`

Even mathematical expressions are effectively SVO:

`7 - 3;`, `counter += 1;`

Alternatively, an VSO-like order of terms is also sometimes employed:

`foldl (+) 0 [1,2,3]`

In this project however, I would like to consider a programming language that is inspired by the SOV word order, as is found in most natural languages.

## Why?

The core idea is to think in terms of data flowing through the program, going through various transformations, before being spat out the other end. All programs effectively work in that way, which, at least in imperative programming languages, is generally also represented in the macro structure of the language, i.e. with a series of statements that are executed from top to bottom. However, when it comes down to the syntax of applying functions/etc., the order tends to be reversed, with in input data to the right, and the result being to the left:

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

However, if we were to write this out as one line in an SVO/VSO programming language, we have to consider these steps backwards:

`result = sum(map(inverse, [1,2,3]))`

Which, at least to me, seems to go against the intuitive way we think about solving the problem.

## The basic structure

Taking the intuition above, a better order of terms would be something like this:

`[1,2,3] map(inverse,_) sum(_) ~> result`

Reading from left to right, we have a term defining some data, which then put through the map function, through the sum function, and finally assigned to the result.

