---
layout: post
title: What’s Stopping Me From Using Rust?
---

Let me start out by saying that I've been using C++ for over a decade and I consider myself an advanced C++ user&mdash;that clearly biases my perspective. If there is something stupid that C++ lets me do, I've probably tried it at some point (and yes, that includes building a compile-time brainfuck interpreter before `constexpr` was the new shiny thing and using template template template parameters). Rust is a great language and has without a doubt a bright future ahead of it. The memory safety it provides is a killer feature that I've only realized I've been missing in C++ after trying Rust. There are a lot of things I like about the language like nice stack traces by default, the borrow checker, a proper build process, the package manager, ... and the list goes on.

That said, there are a few reasons why I still cannot or prefer not to use Rust. Some of them are just minor inconveniences, some are major annoyances and some are complete dealbreakers. With this post I'm going to try to list things that stop me from using the language as I remember it. Please note that this is not meant to be a complete list. In particular, it's been a few months since I've last tried Rust and I might be forgetting about some things or not remembering them 100% accurately. I'm also sure the language has made progress in the meantime. Nonetheless, here goes ...

#### Ugly Generic Syntax
Let's start out with the easy one, the so-called [turbofish](https://matematikaadit.github.io/posts/rust-turbofish.html). I find the examples that create container objects especially bad, like `Vec::<u8>::new()`. I'm not quite sure why this specific syntax was chosen&mdash;I assume it has something to do with parsing ambiguities that would arise without the `::` regarding angle brackets being confused for comparison operators. I get not wanting to deal with this context-sensitivity, but are there really no better alternatives? [D uses the syntax](https://dlang.org/spec/template.html) `X!(T)`, which has the nice property that parentheses can be omitted if only a single type is passed.

#### Naming
This one I find really weird. Mostly because it seems so unnecessary for a relatively new language liek Rust that is able to learn from many older languages. I'm talking about how things in the Rust standard library tend to be named. There are the names that are way too short like `Vec`, `Eq`, `Ord`, `Arc` and so on. This feels like [strstr](https://en.cppreference.com/w/c/string/byte/strstr) all over again. And then there are the names that are actual words, but way too generic to infer the actual meaning without looking at the source or documentation. An example of this is the `Debug` trait. Couldn't this be named `DebugFormattable`, `DebugStringifyable` or something like that? Another one of these is `Into`. Take a look at [this example from the docs](https://doc.rust-lang.org/std/convert/trait.Into.html#examples), in particular this line:
```rust
assert_eq!(bytes, s.into());
```

What? I can't be the only one who finds the use of a preposition as a function name very confusing.

#### No Variadic Templates
This is a major dealbreaker for me. I see the same problems in every programming language I've used that supports some form of generics but no variadics: Java, Kotlin and now Rust. They end up duplicating code for functions taking `0`, `1`, ... , `k-1`, `k` arguments for some hardcoded `k`. It means you cannot write a generic `printf` without wrapping the arguments in an untyped array like Java's `Object[]` or using macros to emulate them like Rust does. And these are just the most basic use cases, you can do much more interesting things with variadic generic types.
Having variadic templates means that you don't need duplication, you can support any number of parameters and there is no overhead. They are an essential building block for truly generic abstractions.

#### No Value Template Parameters
While we're on the topic of generics, Rust does not let me pass compile-time constants as generic parameters. This means I cannot have a statically-sized vector or matrix class that is generic over the number of elements. You might be able to do it by requiring the user to pass an array type, but that is not pretty and means they could pass an invalid type. It is my understanding though that this issue is being worked on, so I have hope that this will be supported in the not too distant future.

#### No Template Specializations and Partial Template Specializations
To be honest, I feel like this is not as much of an immediate blocker as something that I know I'm going to need for some very specific thing at some point in the future. I know because I use it in C++ too, although very infrequently.
Partial specializations open the door to type functions, like `std::remove_pointer` in C++. That template is simply specialized on `T*` and returns `T` in that case.

#### Error Handling is a Mess
I've saved this one for last and it's going to be controversial so buckle up. If I could choose just one thing to be changed about Rust then forget about everything else because it would be this one.
The way "idiomatic" error handling is currently done in Rust is just untenable to me, it infects everything. Every return is wrapped in an `Ok()` or `Err()` which just adds unnecessary noise, every return type must be wrapped in `Result<>`. It's awful.
When I'm coding I usually start out by writing functions that cannot fail and then add error conditions in as I discover them. But what this means is that every single function in the entire call chain needs to have its return type updated and all return statements wrapped in `Ok()`. This is so tedious, I genuinely cannot comprehend how it does not come up more often. Am I doing something wrong here or has simply nobody dared to question the design?
Regardless, let me make a suggestion to improve the situation: Introduce syntax that separates the error from the return value, for example:
```rust
fn can_fail() -> f32 | MyError
```
This would be equivalent to `Result<f32, MyError>`. The compiler can then translate every `return val;` to `return Ok(val);` if the function declares an error. For the failure return case we're also going to need new syntax that desugars to `return Err(val);`, I suggest `fail val;`.
The result is that the code is semantically equivalent to the manual version, but returns are now always the same, regardless of whether the function can fail or not. All the noise is gone. You would still need to manually propagate errors, but I consider this a good thing&mdash;it is a hidden code path after all.

## Conclusion
I wrote this post because I've been noticing an uptick in the amount of positive coverage Rust has been getting lately and I felt that my view was underrepresented. By no means am I saying that Rust is a bad language, just that it leaves a lot to be desired. In the end, it is all about trade-offs: Is the gain in memory safety worth the loss of flexibility? Presently I can answer this question for myself and my hobbyist projects with no. But Rust is clearly the future and I will hopefully be able to switch one day.
