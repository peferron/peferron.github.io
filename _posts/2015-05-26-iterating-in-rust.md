---
layout: post
title: "Iterating in Rust"
description: "How to use Rust Iterators, and why. Beginner-friendly (I'm one!) with a lot of examples."
---

I finished reading the [Rust book](https://doc.rust-lang.org/book/) a few days ago. [My first program](https://github.com/peferron/algo/tree/master/edge_connectivity) calculates the edge connectivity of a graph. It relies on iteration quite a lot, and I had the occasion to dig a bit deeper into Rust [iterators](https://doc.rust-lang.org/book/iterators.html).

{% include figure.html srcset="/images/silomem2/silomem2_768w.jpg 768w, /images/silomem2/silomem2_1536w.jpg 1536w, /images/silomem2/silomem2_2304w.jpg 2304w" src="/images/silomem2/silomem2_768w.jpg" caption="Silomem II. Huile sur toile, 2009, <a href='http://www.ferron-verron.com'>Odile Ferron-Verron</a>." href="http://www.ferron-verron.com" %}

Iterators are a simple, idiomatic way to iterate over lazily generated sequences. Rust didn't invent them, but they are absent from enough popular languages to warrant a discussion.

## Powers of 2

Let's pick a simple example: the powers of 2. The simplest way to iterate is a `for` loop:

```rust
let powers_first_five = [1, 2, 4, 8, 16];

println!("First 5 powers of 2");
for x in &powers_first_five {
    println!("{}", x);
}
```

What if we want more than five numbers? We can create a function that takes a `count` argument:

```rust
fn powers_count(count: usize) -> Vec<u32> {
    let mut x = 1;
    let mut v = Vec::with_capacity(count);
    for _ in 0..count {
        v.push(x);
        x *= 2;
    }
    v
}

println!("First 10 powers of 2");
for x in powers_count(10) {
    println!("{}", x);
}
```

What if we want all the powers of 2 up to 100? We can create a function that takes a `max` argument:

```rust
fn powers_max(max: u32) -> Vec<u32> {
    let mut x = 1;
    let mut v = vec![];
    while x <= max {
        v.push(x);
        x *= 2;
    }
    v
}

println!("Powers of 2 up to 100");
for x in powers_max(100) {
    println!("{}", x);
}
```

Alarms should start ringing: the code that generates powers of two is now duplicated. It's not a big deal in our example because generating powers of two is simple. But imagine if it was something more complicated, like a graph traversal. We really want to write, debug and maintain it only once.

Only one piece of code should generate numbers. This piece of code should not worry about how many numbers to generate. This responsibility is better left to the code that *uses* the numbers, because it knows best what it needs:

```rust
println!("First 10 powers of 2");
let mut count = 0;
for x in powers() {
    if count > 9 {
        break;
    }
    count += 1;
    println!("{}", x);
}

println!("Powers of 2 up to 100");
for x in powers() {
    if x > 100 {
        break;
    }
    println!("{}", x);
}
```

There is only one `powers()` function, reused everywhere. Replacing `count > 9` or `x > 100` with any other arbitrary condition is simple.

But what does `powers()` return? We can return a `Vec<u32>` containing all the powers of 2 that fit in a `u32`:

```rust
fn powers() -> Vec<u32> {
    let mut x = 1u32;
    let mut v = vec![x];
    while let Some(next) = x.checked_mul(2) {
        v.push(next);
        x = next;
    }
    v
}
```

It would work quite well in our example because there are only 32 such values. But there is room for improvement. We could avoid allocating a `Vec`. We could also generate lazily only as many numbers as we need—why generate 32 numbers if we only need 10?

## Iterators to the rescue

Let's go straight to the code:

```rust
struct Powers { next_value: Option<u32> }

impl Iterator for Powers {
    type Item = u32;

    fn next(&mut self) -> Option<u32> {
        let saved = self.next_value;
        if let Some(x) = self.next_value {
            self.next_value = x.checked_mul(2);
        }
        saved
    }
}

fn powers() -> Powers {
    Powers { next_value: Some(1) }
}

println!("Powers of 2 up to 100");
for x in powers() {
    if x > 100 {
        break;
    }
    println!("{}", x);
}
```

`Powers` contains the state of the iteration, and can be iterated over because it implements the `Iterator` trait. Each iteration calls `next`.

If the next value doesn't fit in a `u32`, `next` returns `None` and the `for` loop exits. Otherwise, `next` returns `Some(x: u32)` and the loop keeps going. If we `break` out of the loop, `next` stops being called and no more numbers are generated. Win!

This allows more complicated operations to be exposed in an familiar and performant way. For example, here's how to do a breadth-first-search in my little program (variables renamed for clarity):

```rust
for Edge { x, y } in graph.breadth_first_search(start) {
    // Do something with x and y. Break out of the loop
    // if y is the vertex we were looking for.
}
```

## Convenience methods

Iterators have [many convenience methods](https://doc.rust-lang.org/std/iter/). For example, printing the first 10 powers of 2 can be shortened to:

```rust
println!("First 10 powers of 2");
for x in powers().take(10) {
    println!("{}", x);
}
```

## Unfold

The unstable [Unfold](https://doc.rust-lang.org/std/iter/struct.Unfold.html) iterator helps shorten the code, but I find the type declarations overly verbose:

```rust
// Unfold is unstable as of 2015/05/25.
// Select the nightly channel to run in the Rust playground.

#![feature(core)]
use std::iter::Unfold;

fn powers() -> Unfold<Option<u32>,
                      fn(&mut Option<u32>) -> Option<u32>> {
    Unfold::new(Some(1), unfold_powers)
}

fn unfold_powers(next_value: &mut Option<u32>) -> Option<u32> {
    let saved = *next_value;
    if let Some(x) = *next_value {
        *next_value = x.checked_mul(2);
    }
    saved
}
```

The type declarations can be elided using a closure, but it makes `powers` difficult to reuse outside of the current scope. In most cases it's probably not a good choice:

```rust
// Unfold is unstable as of 2015/05/25.
// Select the nightly channel to run in the Rust playground.

#![feature(core)]
use std::iter::Unfold;

let powers = || {
    Unfold::new(Some(1u32), |next_value| {
        let saved = *next_value;
        if let Some(x) = *next_value {
            *next_value = x.checked_mul(2);
        }
        saved
    })
};
```

For reference and tinkering, here is the fully working code, ready to run in the [Rust playground](https://play.rust-lang.org/):

```rust
struct Powers { next_value: Option<u32> }

impl Iterator for Powers {
    type Item = u32;

    fn next(&mut self) -> Option<u32> {
        let saved = self.next_value;
        if let Some(x) = self.next_value {
            self.next_value = x.checked_mul(2);
        }
        saved
    }
}

fn powers() -> Powers {
    Powers { next_value: Some(1) }
}

fn main() {
    println!("First 10 powers of 2");
    for x in powers().take(10) {
        println!("{}", x);
    }

    println!("Powers of 2 up to 100");
    for x in powers() {
        if x > 100 {
            break;
        }
        println!("{}", x);
    }
}
```
