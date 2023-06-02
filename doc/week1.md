## What's Rust?

* First stable release: 2015
* Originated at Mozilla, but development has been largely open-sourced
* Multi-purpose language, but generally more useful for application where performance is key, like system programming, embedded...

### Basics of "classic" Memory management

In Rust, like in C and other languages, memory is typically divided into two main areas: the stack and the heap. The stack is used for storing local variables and function call information, and it is managed automatically by the compiler. The heap is a region of memory used for dynamic memory allocation, and it requires manual management.

Here's how the Rust Book describe it:
```
Both the stack and the heap are parts of memory available to your code to use at runtime, but they are structured in different ways. The stack stores values in the order it gets them and removes the values in the opposite order. This is referred to as last in, first out. Think of a stack of plates: when you add more plates, you put them on top of the pile, and when you need a plate, you take one off the top. Adding or removing plates from the middle or bottom wouldn’t work as well! Adding data is called pushing onto the stack, and removing data is called popping off the stack. All data stored on the stack must have a known, fixed size. Data with an unknown size at compile time or a size that might change must be stored on the heap instead.

The heap is less organized: when you put data on the heap, you request a certain amount of space. The memory allocator finds an empty spot in the heap that is big enough, marks it as being in use, and returns a pointer, which is the address of that location. This process is called allocating on the heap and is sometimes abbreviated as just allocating (pushing values onto the stack is not considered allocating). Because the pointer to the heap is a known, fixed size, you can store the pointer on the stack, but when you want the actual data, you must follow the pointer. Think of being seated at a restaurant. When you enter, you state the number of people in your group, and the host finds an empty table that fits everyone and leads you there. If someone in your group comes late, they can ask where you’ve been seated to find you.

Pushing to the stack is faster than allocating on the heap because the allocator never has to search for a place to store new data; that location is always at the top of the stack. Comparatively, allocating space on the heap requires more work because the allocator must first find a big enough space to hold the data and then perform bookkeeping to prepare for the next allocation.

Accessing data in the heap is slower than accessing data on the stack because you have to follow a pointer to get there. Contemporary processors are faster if they jump around less in memory. Continuing the analogy, consider a server at a restaurant taking orders from many tables. It’s most efficient to get all the orders at one table before moving on to the next table. Taking an order from table A, then an order from table B, then one from A again, and then one from B again would be a much slower process. By the same token, a processor can do its job better if it works on data that’s close to other data (as it is on the stack) rather than farther away (as it can be on the heap).

When your code calls a function, the values passed into the function (including, potentially, pointers to data on the heap) and the function’s local variables get pushed onto the stack. When the function is over, those values get popped off the stack.

Keeping track of what parts of code are using what data on the heap, minimizing the amount of duplicate data on the heap, and cleaning up unused data on the heap so you don’t run out of space are all problems that ownership addresses. Once you understand ownership, you won’t need to think about the stack and the heap very often, but knowing that the main purpose of ownership is to manage heap data can help explain why it works the way it does.
```

Here are some key concepts related to "classic" memory management in C:

1. **Dynamic Memory Allocation**: Dynamic memory allocation allows you to allocate memory at runtime on the heap using functions like `malloc`, `calloc`, and `realloc`. It is typically used when you need to allocate memory for variables or data structures whose size is determined at runtime or when you want to manage memory explicitly. Dynamic memory allocation returns a pointer to the allocated memory, which you must handle carefully to avoid memory leaks or accessing deallocated memory.

2. **Memory Deallocation**: When you are done with dynamically allocated memory, it is important to deallocate it to free up the resources and avoid memory leaks. 

When writing C, you'll mess things up in 3 ways (most of the time): 

1. **Memory Leaks**: Memory leaks occur when you fail to deallocate dynamically allocated memory after you are done using it. To say it another way, you called `malloc` and then the pointer goes out of scope before you `free` it. The result is that the memory is still allocated on the heap, but your program can't do anything with it anymore. If memory leaks accumulate over time, it can lead to excessive memory consumption and potentially cause your program to run out of memory. 

Here's an example:

```c
#include <stdio.h>
#include <stdlib.h>

void someFunction() {
    int* dynamicArray = malloc(5 * sizeof(int));
    dynamicArray[0] = 1;
    dynamicArray[1] = 2;
    dynamicArray[2] = 3;
    dynamicArray[3] = 4;
    dynamicArray[4] = 5;

    // The dynamically allocated memory is not freed here,
    // leading to a memory leak.
}

int main() {
    someFunction();

    // We can't access the allocated memory here
    // But it's still there in our RAM
    
    return 0;
}
```

2. **Dangling Pointers**: Dangling pointers are pointers that point to memory that has been deallocated. Accessing such memory can lead to undefined behavior and crashes. 

```c
#include <stdio.h>
#include <stdlib.h>

int* createIntArray() {
    int* array = malloc(3 * sizeof(int));
    if (array != NULL) {
        array[0] = 1;
        array[1] = 2;
        array[2] = 3;
    }
    return array;
}

void printIntArray(int* arr) {
    printf("Array elements: %d, %d, %d\n", arr[0], arr[1], arr[2]);
}

int main() {
    int* myArray = createIntArray();
    
    free(myArray); // Deallocate the memory

    printIntArray(myArray); // Accessing a dangling pointer

    return 0;
}
```

3. **Double free**: Double freeing a pointer occurs when you attempt to deallocate memory that has already been freed. It is a serious issue that can lead to undefined behavior and can cause your program to crash or behave unpredictably.

The consequences of double freeing a pointer can vary depending on the specific implementation and platform. It can cause heap corruption, where the internal data structures used by the memory allocator become inconsistent, leading to crashes or memory leaks. It may also overwrite other valid data structures, leading to further memory access violations or unpredictable behavior.

### Classic solution to memory management issue: Garbage Collection

A garbage collector (GC) is a memory management technique used in programming languages to automatically reclaim memory that is no longer in use by the program. It relieves the programmer from manually deallocating memory and helps prevent memory leaks and other memory-related issues.

The primary task of a garbage collector is to identify and reclaim memory that is no longer reachable or referenced by the program. It works by tracing the references from the root objects (such as global variables, stack frames, and registers) and determining which objects are still reachable. Objects that are not reachable are considered garbage and can be safely deallocated.

Garbage collectors can be implemented in different ways, depending on the programming language and runtime environment. They typically run in the background as part of the language runtime, periodically reclaiming memory to keep the program's memory usage efficient.

### Memory management in Rust: Ownership

The ownership system in Rust provides memory safety without relying on manual memory management or garbage collection. It eliminates certain classes of bugs and enforces strict rules for memory usage, making it easier to write safe and concurrent code. However, it also requires a different mindset and understanding of ownership semantics compared to the manual memory management approach of C.

It basically enforces a set of rules at compile time. Here are the 3 rules that you need to keep in mind when coding in Rust:

* Each value in Rust has an owner.
* There can only be one owner at a time.
* When the owner goes out of scope, the value will be dropped.

Let's have a look at some example to understand those rules:

1. Each value in Rust has an owner: the owner is a variable. It means that you can't have a value written in memory without a valid variable that can access it.
```rust
let s = "hello"; // `s` is the owner of the value "hello"a
```

2. There can only be one owner at a time: you can't have 2 variables that own the same memory. Here `s1` owns a reference to memory allocated on the heap, and `s2 = s1` means that we pass the ownership of this reference to s2, effectively making `s1` void.
```rust
let s1 = String::from("hello");
let s2 = s1;

// println!("{}, world!", s1); // This would cause an error, because `s1` is no longer the owner
```

3. When the owner goes out of scope, the value will be dropped: the scope is the bounds into which variables live. Some can be `global` (meaning that they are accessible for the whole execution of your program), but most only live inside a function or a smaller scope defined inside a function.

![global_variable](../assets/global_variable.jpeg)

```rust
fn another_function() {
    let x = 10;
    println!("x: {}", x); 
    // println!("y: {}", y); -> This line would cause a compile error because y does not exist in this scope
    // Here x falls out of scope and is dropped from memory
}

fn main() {
    let y = 5;

    another_function();
    // println!("x: {}", x); -> This line would cause a compile error because x does not exist in this scope
    {
        let z = 'a';
        println!("z: {}", z);
    }
    // println!("z: {}", z); -> This line would cause a compile error because z does not exist in this scope
}
```

For a deeper understanding of ownership, we can simply read [the Rust book](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html) which is already perfect to explain that.

## Setup

### Install Rust

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

If you're on windows, download and run [rustup-init](https://static.rust-lang.org/rustup/dist/i686-pc-windows-gnu/rustup-init.exe)

### Hello, world!

```bash
$ cargo new hello_cargo
$ cd hello_cargo
$ cargo run
```

### Rust toolchain

```bash
// show active/installed toolchains
$ rustup show

// install a new toolchain
$ rustup install nightly

// set a new default
$ rustup default nightly

// add a target 
$ rustup target add x86_64-unknown-freebsd

// set a different default only for a specific package
$ rustup override stable
```

## Package, Crate, Module

### Package

In Rust, a package is a collection of one or more crates that provide the necessary functionality of a library or application. 

A package contains a `Cargo.toml` file, which is essentially the package manifest. The `Cargo.toml` file describes the package, its dependencies, and other metadata. 

A package can contain at most one library crate and any number of binary crates. This means you can have a package that contains a library, multiple binaries, or both. 

### Crate 

A crate is a collection of modules and other elements that together define a library or an executable application.

A binary crate is an executable application, whereas a library crate is a collection of code that other programs can use.

### Module

In Rust, a module (declared with the `mod` keyword) is a way to organize code into namespaces. This can help manage complexity in larger programs. Each module can contain functions, structs, traits, implementations, and even other modules.

Here is an example of a simple module:

```rust
mod game {
    pub fn start() {
        println!("Game started!");
    }
}
```

In this example, `game` is a module that contains a public function `start`.

### Hierarchy

The relationship between packages and crates can be summarized as follows:

- A package contains one or more crates (binary or/and library).
- A crate (library or binary) contains modules and other elements (structs, enums, etc.).
- A module, defined within a crate, is a namespace that contains other code elements (functions, structs, traits, etc.).

In other words, packages and crates are two levels of organization in Rust. A package is the highest level of organization: it is a collection of crates that are bundled together. 

![Structure example](../assets/1684762625_mai22_15%3A37%3A05.png)

### How to import functions or items from a module?

Rust provides the `use` keyword to bring certain items into scope. For example, to use the `start` function defined in the `game` module, we would do:

```rust
use game::start;

fn main() {
    start();  // We can call start() directly now.
}
```

In this example, the `use` keyword allows us to bring the `start` function from the `game` module into scope. 

### Module declaration

1. **Inline Module Declaration**

You can define a module directly in a file using the `mod` keyword. This is often done in `main.rs` or `lib.rs` for small projects. Here's an example:

```rust
mod module_name {
    pub fn function_name() {
        println!("Hello from module_name!");
    }
}
```

This will create a module called `module_name` with a function `function_name` inside it.

2. **Module Declaration in Separate Files**

For larger projects, it is common to put each module in a separate file. The file's name is the module's name. 

To declare a module from a separate file, you would write `mod module_name;` (no block follows the statement). For example, if you have a file named `module_name.rs`, you would declare it in `main.rs` or `lib.rs` (or another module file) like this:

```rust
mod module_name;
```

Then, in `module_name.rs`, you would define the module's contents:

```rust
pub fn function_name() {
    println!("Hello from module_name!");
}
```

3. **Nested Module Files**

If your module is complex and contains submodules, you can further break it down into multiple files using a directory. The directory name should match the parent module's name, and the submodules should each have their own `.rs` file within the directory.

For example, if you have a module named `game` and it has submodules `input`, `physics`, and `render`, you would structure your project like this:

```
src/
├── main.rs
└── game/
    ├── mod.rs
    ├── input.rs
    ├── physics.rs
    └── render.rs
```

In `main.rs`:

```rust
mod game;
```

In `game/mod.rs`:

```rust
mod input;
mod physics;
mod render;
```

In `game/input.rs` (similarly for `physics.rs` and `render.rs`):

```rust
pub fn handle_input() {
    // handle input logic here
}
```

In this way, you can structure a large project with multiple levels of modules.

4. **Public and Private Modules**

By default, all modules are private - they can only be accessed from their parent module. If you want a module to be accessible from outside its parent module, you need to declare it as `pub mod`.

Keep in mind that even if a module is public, its contents (functions, structs, etc.) are still private by default. You also need to use `pub` to make them public.

## Running Tetris

To make things more interesting we're going to work from an existing piece of code that implements a simple Tetris game inside the terminal. 

It will give us a very simple and readable example to learn basic concepts and we can upgrade it in different ways.


### Rust versioning

```
// clone the present repository
git clone 

// checkout tetris_console branch
git checkout tetris_console

// build
cargo build
```

The build should fail with some errors. Why?

Let's take a look at the `Cargo.toml` file:
```Cargo.toml
[package]
name = "rust_course"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
libc = "0.1.8"
rand = "0.3"

```

Let's comment out the `edition` key for now.

```Cargo.toml
[package]
name = "rust_course"
version = "0.1.0"
#edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
libc = "0.1.8"
rand = "0.3"

```

And build it again. Does it work now? 

This introduced you to Rust `edition`. Rust is a relatively young language and its syntax evolves.
Code that was valid in 2015 edition might become a `hard error` later, and that's what happened here.
This code is a copy/paste of [someone else version of the game](https://github.com/mikejquinn/rust-tetris.git), and it was written many years back, so this kind of issues is expected.  

Our first step is to update the code and make it compliant with 2021 standard. It's also a good occasion to learn a couple things:

1. `src/main.rs:222` is telling us `trait objects must include the `dyn` keyword`
    * a trait is a way to define shared behaviours between different types. It's similar to interfaces in other languages.
    * `fn each_point(&self, callback: &mut FnMut(i32, i32))`: `FnMut` is a trait, not a type. Here it means that `each_point` function expect as argument an object that implements this trait with 2 arguments of type `i32`. Or to put it another way, `callback` must be a closure (meaning a function) that takes 2 `i32` as arguments.
    * The problem is that we want to write a generic function that could be used to makes different things that we don't know yet. The content of callback will be defined at runtime instead of compile time.
    * The compiler tells us to add a `dyn` keyword before this trait. The `dyn` keyword stands for "dynamic" and is used to highlight that dynamic dispatch is happening at this point in the code. The trait object is "dynamic" because it dispatches method calls at runtime, as opposed to static dispatch, which happens at compile time.
    * While it was possible to pass a trait object implicitly as a type in the 2015 edition, it was later considered confusing and the `dyn` keyword was added to explicitly signal "This is a trait, not a type". 

2. `src/main.rs:491`, `src/main.rs:523` and `src/main.rs:530` are all the same error `format argument must be a string literal`
    * `panic!` and `format!` are not function, but macros. Macros are expanded at compile time and generate code. Contrary to function it can take variadic (ie. any number) arguments. You can think of it as code that writes more code, or meta-programming. 
    * `panic!` is crashing the program immediately, and can take a formatted string as argument.
    * `format!` is the equivalent of `sprintf` in other languages.
    * In Rust 2015, it was possible to directly pass `format!` as argument to `panic!`, but it was removed later. Now `panic!` actually as the same syntax than `format!`, with a formatted string and some arguments to fill the gap signaled with "{}".
    * While the compiler suggest to add an empty formatted string "{}" and keep the `format!` as argument, it is much more efficient and readable to get rid of the `format!`.

3. `src/display.rs:1` `unresolved import `util``
    * We simply need to prefix this import statement with `crate::`
    * Since Rust 2018, `crate::` unambiguously refers to this crate (it used to be able to refer to external crate in 2015).

4. Another modification we can make is removing the `extern crate` statements at the top of the main file, even if it's not a compile error, they're now unnecessary. 

Now we can build and run our Tetris.


## Write a prelude for our game

A prelude is a module that re-exports the most commonly used items of a library or a larger module, so that users of the library/module can conveniently import those items from the prelude, instead of remembering which sub-modules they belong to.

For example, imagine you have a library called `game_lib` that has several modules (`physics`, `rendering`, `input`, etc.) and each module has several items (functions, structs, constants, etc.). If a user wants to use a function from `physics` and a struct from `rendering`, they would have to do:

```rust
use game_lib::physics::apply_gravity;
use game_lib::rendering::Sprite;
```

If `game_lib` provides a prelude that re-exports `apply_gravity` and `Sprite`, the user could do this instead:

```rust
use game_lib::prelude::*;
```

This brings all items exported by the prelude into scope. It's especially convenient when you need to import many items from different modules.

A prelude module usually looks like this:

```rust
pub mod prelude {
    pub use super::physics::apply_gravity;
    pub use super::rendering::Sprite;
    // ...and so on for other commonly used items.
}
```

The `pub use` statement makes an item available for others to use when they import the prelude.

We can also use it in our own crate by defining it our main with some import statement that we will use in many places and some constants. As an example, let's move this import statement in `terminal.rs` into a crate prelude:

```rust
use libc::{c_ulong, c_int, c_uchar};
```

To make things a little more interesting, let's also move the `BOARD_{WIDTH,HEIGHT}` constant into the prelude and see how we can use it directly into `display.rs` without passing it in function arguments. 

## Basic types

1. **Booleans**: The `bool` type represents a boolean value and can have two possible values: `true` or `false`.

See the logic of [`advance_game()`](https://github.com/Sosthene00/tetrust/blob/77eda68991e9225769c8c378c706706830521ab6/src/main.rs#LL405C5-L405C5) in `main.rs`. It first calls `move_piece()` to try to drop 1 row, and if it returns false it means that the piece hit the ground and that we enter the lock phase. It then calls `place_new_piece()`, that also returns `true` or `false`. We can already keep that in mind for when we will try to implement game over conditions.

2. **Characters**: The `char` type represents a single Unicode character and is enclosed in single quotes, such as `'a'`, `'1'`, or `'😀'`. Beware it means a Rust `char` is 4 bytes, not 1!

Since we display our game in the terminal, we basically only need `char`s, and you can see how it is done in the [`set_text()`]() function.

3. **Numeric Types**:
   - **Integers**: Rust provides signed and unsigned integers with different sizes: `i8`, `i16`, `i32`, `i64`, `i128` for signed integers, and `u8`, `u16`, `u32`, `u64`, `u128` for unsigned integers. The number indicates the number of bits the type can hold. `isize` and `usize` are integers of variable size, dependent on the underlying machine architecture.
   - **Floating-Point**: Rust has two floating-point types: `f32` for single-precision and `f64` for double-precision.

What's interesting here is that we often need to `cast` a numeric types into another to satisfy some functions or API. See for example when we instantiate a new `Display()` how we need to cast the constant `BOARD_WIDTH` and `BOARD_HEIGHT`, that are `u32`, as `usize` to create `Vector`. 

**Question**: Why does it makes sense to impose `usize` type to express the number of elements in a `Vector`?

4. **Tuples**: A tuple is a collection of different values that may not be of the same type, for example `(1, 'a', 1.1)`. It is of fixed size, meaning that you can't add or remove elements from it once declared. You can destructure a tuple:

```rust
let tup = ('a', 1, 1.1);
let (x, y, z) = tup; // x == 'a', y == 1, z == 1.1
```

The `()` type, pronounced "unit," represents an empty tuple. It is used to indicate the absence of a meaningful value and is commonly used as the return type of functions that do not return a value.

5. **Arrays**: Array is a collection of values of the same type. It is of fixed-size.

```rust
let a: [i32;5] = [1,2,3,4,5];
let first = a[0] // 1
let second = a[1] //2
```

We can't add to or pop elements from an array. If we need a collection that can vary in size we need a `Vector` instead. 

It's worth noting that strings are not considered primitive types in Rust. Instead, they are represented by the `String` type, which is a dynamically sized, growable string. 

## Variable declaration

We declare a new variable with the keyword `let`:

```rust
let x = 5;
```

By default, variables in Rust are immutable. This means that once a value is bound to a name, you can’t change that value. To make a variable mutable, you can use `mut` before the variable name:

```rust
impl Display {
    pub fn new() -> Display {
        let width = BOARD_WIDTH * 2 + 100;
        let height = BOARD_HEIGHT + 2;
        let mut rows = Vec::with_capacity(height as usize);
        ...
    }
}
```

Here `width` and `height` are immutable, which makes sense because they're not expected to change during a game. `rows` on contrary will be constantly modified during the game, so it must be mutable.

Rust is a statically typed language, which means that it must know the types of all variables at compile time. The compiler can usually infer what type we want to use based on the value and how we use it.

Sometimes inference is not possible, and you need to tell the compiler about the exact type you want:

```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);
```

You can declare a new variable with the same name as a previous variable, and the new variable shadows the previous variable. It's called shadowing.

```rust
let x = 5;
let x = x + 1;
let x = x * 2;
```

Each `let` statement is valid because they each create a new variable named `x`, even though they're creating a new layer of "shadow" over the previous value.

Shadowing is different from marking a variable as `mut`, because we'll get a compile-time error if we accidentally try to reassign to this variable without using the `let` keyword. By using `let`, we're effectively creating a new variable, so this is mostly used in situations where you want to transform a value but want to reuse the variable name.