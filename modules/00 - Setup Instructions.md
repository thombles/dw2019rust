# 00 - Setup Instructions

## Assumptions

I'm assuming you have a favourite text editor that you will use to edit Rust and TOML files. If you only have Xcode I would recommend installing something like [Visual Studio Code](https://code.visualstudio.com/download).

I'm also assuming that you're familiar with terminal basics - opening `Terminal`, running commands, and moving between directories. Don't panic if you're not! Just ask for help.

In these notes, if you see <code>$ <b>cargo build</b></code> it means this is a command you need to type, but don't include the `$` symbol! Just type <code><b>cargo build</b></code> into your terminal and press enter.

## Software Installation

1. **Xcode**: If necessary, install it from the App Store.
2. **Xcode command line tools**: Run `xcode-select --install` in Terminal and make sure it shows they're installed:
<pre>
$ <b>xcode-select --install</b>
xcode-select: error: command line tools are already installed, use "Software Update" to install updates
</pre>
3. **Xcode path**: Run `xcode-select --print-path` in Terminal and make it sure it points inside where your copy of Xcode is actually located. If it does not, use the `--switch` option to point it to the `Contents/Developer` subdirectory of your Xcode.
<pre>
$ <b>xcode-select --print-path</b>
/Applications/Xcode.app/Contents/Developer
</pre>
4. **Rust**: Install Rust via Rustup. If you have installed it via Homebrew or other means, I would recommend uninstalling it and using Rustup instead. It will also interfere with some instructions in this workshop. Rustup is both an installation script and a command line tool that you can use to manage your Rust versions and features. Visit https://rustup.rs and follow the instructions. Use the default `stable` toolchain. When it finishes, remember to either run the provided `source` command or open a new shell!
5. **Rust extra features**: We need to add Rust targets to build for 64-bit iOS platforms - physical and simulator. 32-bit versions are also available but hopefully you don't need them.
<pre>
$ <b>rustup target add aarch64-apple-ios x86_64-apple-ios</b>
</pre>

## Is my Rust working?

Check that `rustup` thinks everything is installed. Open a terminal and run this command. It should look similar to this:

<pre>
$ <b>rustup show</b>
Default host: x86_64-apple-darwin

installed targets for active toolchain
--------------------------------------

aarch64-apple-ios
x86_64-apple-darwin
x86_64-apple-ios

active toolchain
----------------

stable-x86_64-apple-darwin (default)
rustc 1.37.0 (eae3437df 2019-08-13)
</pre>


Now let's do a quick test of the compiler. Go to a directory for your programming projects, e.g.: <code>$ <b>cd ~/code</b></code>

Rust's project management tool is called `cargo`. Use it to create, compile and run the default "Hello, world!" project:

<pre>
$ <b>cargo new --bin hello</b>
     Created binary (application) `hello` package
$ <b>cd hello</b>
$ <b>cargo run</b>
   Compiling hello v0.1.0 (/Users/tk/code/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.12s
     Running `target/debug/hello`
Hello, world!
$
</pre>

Take a minute to explore the `hello` directory. For this workshop you will need to know where you can find the main source code file, the `Cargo.toml` configuration file, and the compiled targets!
