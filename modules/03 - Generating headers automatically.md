# 03 - Generating headers automatically

## Overview

* Add functions to `libanvil` that take an argument and have a return value
* Install cbindgen and use it to generate a C header

## Instructions

So far `anvil` only has a very boring function that has no arguments or return value. In `src/lib.rs`, add some imports to the top of the file:

```rust
use std::ffi::CStr;
use std::os::raw::{c_char, c_int, c_longlong, c_ulong};
```

Further down, add two more functions. It is possible to use Rust primitives like `i32` and `i64` but you run the risk that the C types in the header file will vary between platforms, which can cause downstream build problems. The `c_*` types will always match the C size for the current platform.

This code is not normal Rust at all - it's written this way to facilitate communication with Swift. If you're new to Rust, please read ["the book"](https://doc.rust-lang.org/book/) to see what regular data types and functions look like!

```rust
/// Add two signed integers.
/// 
/// On a 64-bit system, arguments are 32 bit and return type is 64 bit.
#[no_mangle]
pub extern "C" fn add_numbers(x: c_int, y: c_int) -> c_longlong {
    x as c_longlong + y as c_longlong
}

/// Take a zero-terminated C string and return its length as a
/// machine-size integer.
#[no_mangle]
pub extern "C" fn string_length(sz_msg: *const c_char) -> c_ulong {
    let slice = unsafe { CStr::from_ptr(sz_msg) };
    slice.to_bytes().len() as c_ulong
}
```

**Note:** There is some weird stuff going on in the `string_length` function. Don't worry about it right now. We'll be looking at strings later!

From now on we will use a tool called `cbindgen` to create our header files automatically. Install it as a command for your user using cargo:

<pre>$ <b>cargo install --force cbindgen</b></pre>

After it finishes installing, inside the `anvil` directory, ask it to generate a header:

<pre>$ <b>cbindgen -l C -o target/anvil.h</b></pre>

Open that target file in your editor. It should look like this.

```c
#include <stdarg.h>
#include <stdbool.h>
#include <stdint.h>
#include <stdlib.h>

/**
 * Add two signed integers.
 * On a 64-bit system, arguments are 32 bit and return type is 64 bit.
 */
long long add_numbers(int x, int y);

void hello_devworld(void);

/**
 * Take a zero-terminated C string and return its length as a
 * machine-size integer.
 */
unsigned long string_length(const char *sz_msg);
```

Repeat the build steps for your new library (we will automate this in the next module):
<pre>$ <b>cargo build --target aarch64-apple-ios</b>
$ <b>cargo build --target x86_64-apple-ios</b>
$ <b>lipo -create target/aarch64-apple-ios/debug/libanvil.a target/x86_64-apple-ios/debug/libanvil.a -output target/libanvil.a</b></pre>

In Finder, locate your iOS project folder and go to the `Anvil` subdirectory. Delete the old `anvil.h` and `libanvil.a`. Copy in the latest versions of `anvil.h` and `libanvil.a` from the `target` directory of your Rust project.

Build and run your iOS app in the simulator. It should still be working, calling the same function from before.

Try adding the new functions inside `ViewController.swift`:

```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        hello_devworld()
        let answer = add_numbers(15, 25)
        print("The answer is: \(answer)")
        let string = "for an anvil, this library sure is lightweight"
        print("The length is: \(string_length(string))")
    }
```

## Exercises

### 1. Unit test your library

If you used the examples above, try adding the following the following test at the bottom of `src/lib.rs`:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn basic_addition() {
        assert_eq!(add_numbers(2, 5), 7);
    }
}
```

Then inside the `anvil` folder ask cargo to run the tests.

<pre>$ <b>cargo test</b>
   Compiling anvil v0.1.0 (/Users/tk/code/anvil)
    Finished dev [unoptimized + debuginfo] target(s) in 0.40s
     Running target/debug/deps/anvil-9f86c8c40b30b57a

running 1 test
test tests::basic_addition ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests anvil

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out</pre>

What happens when it fails? Invent some new tests!

### 2. Add more functions

Invent more Rust functions that do different things. How do they come out in the header file? Can you use them in Swift?
