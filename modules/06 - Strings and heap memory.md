# 06 - Strings and heap memory

## Overview

* Learn how to receive C strings as arguments to library functions
* Learn how to return a C string from a library function

## Instructions


Let's add a function to `anvil` that accepts strings (text) as arguments.

We will add a function that calculates the [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance) between two strings. It has a fancy name but it's easy to see how it works. You give it two strings as input and it returns a number telling you how different they are:

"cat", "cat" -> 0  
"cat", "car" -> 1  
"cat", "dog" -> 3  
"cat", "cta" -> 2

We don't want to write the Levenshtein algorithm ourselves though - [there's a crate for that](https://crates.io/crates/levenshtein)!

Open your `Cargo.toml` and add it under the `[dependencies]` section:

```toml
[dependencies]
levenshtein = "1.0.4"
```

Look at the documentation for the provided function, [`levenshtein::levenshtein`](https://docs.rs/levenshtein/1.0.4/levenshtein/fn.levenshtein.html).

```rust
pub fn levenshtein(a: &str, b: &str) -> usize
```

It shows that we have to call it with two arguments which are _string slices_ `&str`. If you are new to Rust this is pretty weird. ([Here's the explanation](https://doc.rust-lang.org/book/ch04-03-slices.html) but don't worry about it right now.)

In your `src/lib.rs` in the `anvil` project, add a new function. (**Warning:** This won't compile yet!)

```rust
#[no_mangle]
pub extern "C" fn leven(s1: *const c_char, s2: *const c_char) -> c_ulong {
    let s1 = unsafe { CStr::from_ptr(s1) };
    let s1 = s1.to_str();
    let s2 = unsafe { CStr::from_ptr(s2) };
    let s2 = s2.to_str();
    levenshtein::levenshtein(s1, s2) as c_ulong
}
```

For each of the two string arguments, we create a `CStr`, a Rust helper object, to process the raw pointer that we received from C. Then we use its method [`to_str`](https://doc.rust-lang.org/std/ffi/struct.CStr.html#method.to_str) to turn it into a `&str`, and give that to the Rust library function.

But it doesn't compile. Let's look at why not.

```
81 |     levenshtein::levenshtein(s1, s2) as c_ulong
   |                              ^^ expected &str, found enum `std::result::Result`
   |
   = note: expected type `&str`
              found type `std::result::Result<&str, std::str::Utf8Error>`
```

In Rust, a string slice is guaranteed to be valid UTF-8 text. If we get a random bucket of bytes from C it may or may not be! We have three possible solutions:

1. Detect the `Utf8Error` and return some sort of error code, like -1. (Use a `match` statement on the result)
2. Assume it will always be valid, and crash if it isn't. (Use `unwrap()` on the result)
3. Use a different function that will strip out any weird stuff. (Use `to_string_lossy()` instead of `to_str()`)

We'll take the risk and use option 2. This is one of many examples where you have to carefully translate between the safety of Rust and the unsafety of C. Update your function to include some calls to `unwrap()`, to extract the `&str` from inside the `Result`.

```rust
#[no_mangle]
pub extern "C" fn leven(s1: *const c_char, s2: *const c_char) -> c_ulong {
    let s1 = unsafe { CStr::from_ptr(s1) };
    let s1 = s1.to_str().unwrap();
    let s2 = unsafe { CStr::from_ptr(s2) };
    let s2 = s2.to_str().unwrap();
    levenshtein::levenshtein(s1, s2) as c_ulong
}
```

Edit `ViewController.swift` to try out the new function. Notice that we don't have to do anything special at all. Converting a Swift String to a `const char*` C argument is such a common requirement that it does it automatically! Build and run the app to see the results.

```swift
let word1 = "agreeable"
let word2 = "affable"
let distance = leven(word1, word2)
print("Distance: \(distance)")
```

Next let's add a function that returns a string. We're going to supply it a number, and it will return that many copies of the letter `A`.

0 -> ""  
3 -> "AAA"  
10 -> "AAAAAAAAAA"

First, at the top of `lib.rs`, change the `use` line so it includes _both_ `CStr` and `CString`:

```rust
use std::ffi::{CStr, CString};
```

Now add a new function later in the file:

```rust
#[no_mangle]
pub extern "C" fn give_me_letter_a(count: c_ulong) -> *mut c_char {
    let string = "A".repeat(count as usize);
    let cstring = CString::new(string).unwrap();
    cstring.into_raw()
}
```

Use it from Swift:

```swift
if let five_a_cstr = give_me_letter_a(5) {
    let five_a = String.init(cString: five_a_cstr)
    print("5 of letter A: \(five_a)")
} else {
    print("Returned string was null!")
}
```

You should see this output in the console:

```
5 of letter A: AAAAA
```

Notice that Swift treats the returned pointer as an optional, and you must manually convert it back to a Swift String.

There is a loose end here! Rust created the string `"AAAAA"` on the heap and forgot about the allocation when we used `into_raw()`. Unless we free it again there will be a memory leak.

Add a new function to your Rust library:

```rust
#[no_mangle]
pub extern "C" fn free_string(s: *mut c_char) {
    let cstring = unsafe { CString::from_raw(s) };
    drop(cstring); // not technically required but shows what we're doing
}
```

Now we can clean up when we've finished using it in Swift.

```swift
if let five_a_cstr = give_me_letter_a(5) {
    let five_a = String.init(cString: five_a_cstr)
    free_string(five_a_cstr)
    print("5 of letter A: \(five_a)")
} else {
    print("Returned string was null!")
}
```

## Exercises

### 1. Unicode support

What is the Levenshtein distance between these emoji? 🐱🐶

### 2. Crash your app?

How many of the letter A can you ask for? (Check out the Memory Usage section in Xcode while it's printing.)

In Swift, come up with a way to supply invalid UTF-8 data to the `leven` function. Prove that it crashes your app when it calls `unwrap()`. ([Hint](https://developer.apple.com/documentation/swift/unsafemutablepointer/2295026-allocate))
