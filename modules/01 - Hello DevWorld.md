# 01 - Hello DevWorld

## Overview

* Create a static library in Rust
* Define a function that prints to screen
* Test it from Rust

## Instructions

Create a new library project and give it a name. In these notes we will use the meaningless name `anvil`, so the resulting static library will called `libanvil.a`.

<pre>
$ <b>cd ~/code</b>
$ <b>cargo new --lib anvil</b>
     Created library `anvil` package
$
</pre>

Open `Cargo.toml` and add a `[lib]` section to select the output formats:

<pre>
[package]
name = "anvil"
version = "0.1.0"
authors = ["Thomas Karpiniec &lt;tom.karpiniec@outlook.com&gt;"]
edition = "2018"

<b>[lib]
crate-type = ["lib", "staticlib"]</b>

[dependencies]
</pre>

Open `src/lib.rs` and delete the generated test. Add a function like this one:

```rust
#[no_mangle]
pub extern "C" fn hello_devworld() {
    println!("Hello, /dev/world 2019!");
}
```

Inside the `anvil` project directory, make sure it compiles:

<pre>$ <b>cargo build</b>
   Compiling anvil v0.1.0 (/Users/tk/code/anvil)
    Finished dev [unoptimized + debuginfo] target(s) in 0.36s
</pre>

Check that you can see both the rust library and the static library in the `target` subdirectory:

<pre>$ <b>ls -l target/debug/lib*</b>
-rw-r--r--  2 tk  staff  4416376 24 Aug 11:05 target/debug/libanvil.a
-rw-r--r--  1 tk  staff       78 24 Aug 11:05 target/debug/libanvil.d
-rw-r--r--  2 tk  staff    18520 24 Aug 11:05 target/debug/libanvil.rlib
</pre>

Check that your function is being exported from the static library. This means it can be used from C (and Swift)!

<pre>$ <b>nm target/debug/libanvil.a | grep hello</b>
0000000000000000 T _hello_devworld
</pre>

Next, let's provide a way to test the library function by calling it from a normal Rust application. Inside the `anvil` directory, manually create a new directory and a file inside it: `examples/testanvil.rs`.

<pre>$ <b>mkdir examples</b>
$ <b>cd examples</b>
$ <b>touch testanvil.rs</b>
</pre>

Your project folder should now be laid out like this:

<pre>
├── Cargo.lock
├── Cargo.toml
├── examples
│   └── testanvil.rs
├── src
│   └── lib.rs
...
</pre>

Open `testanvil.rs` and enter the following Rust program:

```rust
use anvil::*;

fn main() {
    hello_devworld();
}
```

You can now use this test program to call into your anvil library.

<pre>$ <b>cargo run --example testanvil</b>
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/examples/testanvil`
Hello, /dev/world 2019!
</pre>

## Exercises

### 1. An experiment with crates

Searching on https://crates.io for something interesting we might spot the crate `ferris-says`.

Let's make our function more exciting. Open `Cargo.toml` and add a reference to this crate underneath `[dependencies]`.

```toml
[dependencies]
ferris-says = "0.1"
```

Change `src/lib.rs` to use this new crate:

```rust
use std::io::{stdout, BufWriter};

#[no_mangle]
pub extern "C" fn hello_devworld() {
    let phrase = b"Hello, /dev/world/2019!";
    let stdout = stdout();
    let mut writer = BufWriter::new(stdout.lock());
    ferris_says::say(phrase, 30, &mut writer).unwrap();
}
```

Now we get a more interesting printout.

<pre>$ <b>cargo run --example testanvil</b>
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/examples/testanvil`
    ----------------------------------
    | Hello, /dev/world/2019!        |
    ----------------------------------
                \
                 \
                     _~^~^~_
                 \) /  o o  \ (/
                   '_   -   _'
                   / '-----' \
</pre>

### 2. Test the library from C

If you don't feel like going all the way to Swift yet, you can try out the library from an ordinary C program.

We need to make sure we link in all the required system libraries to support our library and its use of the Rust standard library. Run this command to see what flags you will need - note that this may vary from system to system.

<pre>$ <b>cargo rustc -- --print native-static-libs</b>
   Compiling anvil v0.1.0 (/Users/tk/code/anvil)
note: Link against the following native artifacts when linking against this static library. The order and any duplication can be significant on some platforms.

note: native-static-libs: <b>-lSystem -lresolv -lc -lm</b>

    Finished dev [unoptimized + debuginfo] target(s) in 0.71s
</pre>

Create a new folder out of the way, and copy over your compiled static library.

<pre>
$ <b>mkdir ~/code/hello_c</b>
$ <b>cd ~/code/hello_c</b>
$ <b>cp ~/code/anvil/target/debug/libanvil.a .</b>
$ <b>touch hello.c</b>
</pre>

Edit `hello.c` and fill in a program that calls the library function. We don't have an `anvil.h` header yet (that comes later!) so we'll need to declare the function prototype ourselves.

```c
extern void hello_devworld();

int main(int argc, char *argv[]) {
	hello_devworld();
    return 0;
}
```

Compile and run it:

<pre>
$ <b>cc hello.c libanvil.a -lSystem -lresolv -lc -lm -o hello</b>
$ <b>./hello</b>
    ----------------------------------
    | Hello, /dev/world/2019!        |
    ----------------------------------
                \
                 \
                     _~^~^~_
                 \) /  o o  \ (/
                   '_   -   _'
                   / '-----' \
</pre>
