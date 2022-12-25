# Rust Language Knowledge
## Types
### Scalar types
* signed integers: `i8`, `i16`, `i32`, `i64`, `i128` and `isize` (pointer size)
  * from $-2^{n-1}$ to $2^{n-1} - 1$
* unsigned integers: `u8`, `u16`, `u32`, `u64`, `u128` and `usize` (pointer size)
  * from $0$ to $2^{n} - 1$
* floating point: `f32`, `f64`
* [`char`](https://doc.rust-lang.org/std/primitive.char.html) Unicode scalar values like `'a'`, `'α'` and `'∞'` (4 bytes each)
  * Basic letters and symbols usually need 1 out of 4 bytes: a b 1 2 + - = $ @
  * Other letters like German Umlauts or accents need 2 out of 4 bytes: ä ö ü ß è é à ñ
  * Korean, Japanese or Chinese characters need 3 or 4 bytes: 国 안 녕
* [bool](https://doc.rust-lang.org/std/primitive.bool.html) either `true` or `false` - uses one byte of memory
  * **&&** - logical AND
  * **||** - logical OR
  * **!** - logical NOT
* and the unit type `()`, whose only possible value is an empty tuple: `()`

Despite the value of a unit type being a tuple, it is not considered a compound type because it does not contain multiple values.

Variables can always be *type annotated*. Numbers may additionally be annotated via a *suffix* or by *default*. Integers default to `i32` and floats to `f64`. Note that Rust can also infer types from context.

```rust
fn main() {
    println!("U8 min is {} max is {}", u8::MIN, u8::MAX);
    println!("I8 min is {} max is {}", i8::MIN, i8::MAX);
    println!("U16 min is {} max is {}", u16::MIN, u16::MAX);
    println!("I16 min is {} max is {}", i16::MIN, i16::MAX);
    println!("U32 min is {} max is {}", u32::MIN, u32::MAX);
    println!("I32 min is {} max is {}", i32::MIN, i32::MAX);
    println!("U64 min is {} max is {}", u64::MIN, u64::MAX);
    println!("I64 min is {} max is {}", i64::MIN, i64::MAX);
    println!("U128 min is {} max is {}", u128::MIN, u128::MAX);
    println!("I128 min is {} max is {}", i128::MIN, i128::MAX);
    println!("USIZE min is {} max is {}", usize::MIN, usize::MAX);
    println!("ISIZE min is {} max is {}", isize::MIN, isize::MAX);
}
```

### Compound types
* arrays like `[1, 2, 3]`
* tuples like `(1, true)`

#### [Arrays](https://doc.rust-lang.org/std/primitive.array.html) and [Slices](https://doc.rust-lang.org/std/primitive.slice.html)

An array is a collection of objects of the same type `T`, stored in contiguous memory. Arrays are created using brackets `[]`, and their length, which is known at compile time, is part of their type signature `[T; length]`. They are stack allocated.

Slices are similar to arrays, but their length is not known at compile time. Instead, a slice is a two-word object, the first word is a pointer to the data, and the second word is the length of the slice. The word size is the same as usize, determined by the processor architecture e.g. 64 bits on an x86-64. Slices can be used to borrow a section of an array, and have the type signature `&[T]`.

```rust
use std::mem;

// This function borrows a slice
fn analyze_slice(slice: &[i32]) {
    println!("first element of the slice: {}", slice[0]);
    println!("the slice has {} elements", slice.len());
}

fn main() {
    // Fixed-size array (type signature is superfluous)
    let xs: [i32; 5] = [1, 2, 3, 4, 5];

    // All elements can be initialized to the same value
    let ys: [i32; 500] = [0; 500];

    // Indexing starts at 0
    println!("first element of the array: {}", xs[0]);
    println!("second element of the array: {}", xs[1]);

    // `len` returns the count of elements in the array
    println!("number of elements in array: {}", xs.len());

    // Arrays are stack allocated
    println!("array occupies {} bytes", mem::size_of_val(&xs));

    // Arrays can be automatically borrowed as slices
    println!("borrow the whole array as a slice");
    analyze_slice(&xs);

    // Slices can point to a section of an array
    // They are of the form [starting_index..ending_index]
    // starting_index is the first position in the slice
    // ending_index is one more than the last position in the slice
    println!("borrow a section of the array as a slice");
    analyze_slice(&ys[1 .. 4]);

    // Example of empty slice `&[]`
    let empty_array: [u32; 0] = [];
    assert_eq!(&empty_array, &[]);
    assert_eq!(&empty_array, &[][..]); // same but more verbose

    // Arrays can be safely accessed using `.get`, which returns an
    // `Option`. This can be matched as shown below, or used with
    // `.expect()` if you would like the program to exit with a nice
    // message instead of happily continue.
    for i in 0..xs.len() + 1 { // OOPS, one element too far
        match xs.get(i) {
            Some(xval) => println!("{}: {}", i, xval),
            None => println!("Slow down! {} is too far!", i),
        }
    }

    // Out of bound indexing causes runtime error
    //println!("{}", xs[5]);
}
```

#### [Tuples](https://doc.rust-lang.org/std/primitive.tuple.html)
A tuple is a collection of values of different types. Tuples are constructed using parentheses `()`, and each tuple itself is a value with type signature `(T1, T2, ...)`, where `T1`, `T2` are the types of its members. Functions can use tuples to return multiple values, as tuples can hold any number of values.

```rust
// Tuples can be used as function arguments and as return values
fn reverse(pair: (i32, bool)) -> (bool, i32) {
    // `let` can be used to bind the members of a tuple to variables
    let (int_param, bool_param) = pair;

    (bool_param, int_param)
}

// The following struct is for the activity.
#[derive(Debug)]
struct Matrix(f32, f32, f32, f32);

fn main() {
    // A tuple with a bunch of different types
    let long_tuple = (1u8, 2u16, 3u32, 4u64,
                      -1i8, -2i16, -3i32, -4i64,
                      0.1f32, 0.2f64,
                      'a', true);

    // Values can be extracted from the tuple using tuple indexing
    println!("long tuple first value: {}", long_tuple.0);
    println!("long tuple second value: {}", long_tuple.1);

    // Tuples can be tuple members
    let tuple_of_tuples = ((1u8, 2u16, 2u32), (4u64, -1i8), -2i16);

    // Tuples are printable
    println!("tuple of tuples: {:?}", tuple_of_tuples);
    
    // But long Tuples (more than 12 elements) cannot be printed
    // let too_long_tuple = (1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13);
    // println!("too long tuple: {:?}", too_long_tuple);
    // TODO ^ Uncomment the above 2 lines to see the compiler error

    let pair = (1, true);
    println!("pair is {:?}", pair);

    println!("the reversed pair is {:?}", reverse(pair));

    // To create one element tuples, the comma is required to tell them apart
    // from a literal surrounded by parentheses
    println!("one element tuple: {:?}", (5u32,));
    println!("just an integer: {:?}", (5u32));

    //tuples can be destructured to create bindings
    let tuple = (1, "hello", 4.5, true);

    let (x, y);
    // .. throws away rest of tuple
    (x, ..) = (3, 4, 20, 100);
    // _ throws away one entry
    [_, y] = [1, 2];

    let (a, b, c, d) = tuple;
    println!("{:?}, {:?}, {:?}, {:?}", a, b, c, d);

    let matrix = Matrix(1.1, 1.2, 2.1, 2.2);
    println!("{:?}", matrix);
}
```

### Std library types
#### [String](https://doc.rust-lang.org/std/string/struct.String.html) and [&str](https://doc.rust-lang.org/std/str/)

There are two types of strings in Rust: `String` and `&str`.

A `String` is stored as a vector of bytes (`Vec<u8>`), but guaranteed to always be a valid UTF-8 sequence. `String` is heap allocated, growable and not null terminated.

`&str` is a slice (`&[u8]`) that always points to a valid UTF-8 sequence, and can be used to view into a String, just like `&[T]` is a view into `Vec<T>`.

```rust
fn main() {
    // (all the type annotations are superfluous)
    // A reference to a string allocated in read only memory
    let pangram: &'static str = "the quick brown fox jumps over the lazy dog";
    println!("Pangram: {}", pangram);

    // Iterate over words in reverse, no new string is allocated
    println!("Words in reverse");
    for word in pangram.split_whitespace().rev() {
        println!("> {}", word);
    }

    // Copy chars into a vector, sort and remove duplicates
    let mut chars: Vec<char> = pangram.chars().collect();
    chars.sort();
    chars.dedup();

    // Create an empty and growable `String`
    let mut string = String::new();
    for c in chars {
        // Insert a char at the end of string
        string.push(c);
        // Insert a string at the end of string
        string.push_str(", ");
    }

    // The trimmed string is a slice to the original string, hence no new
    // allocation is performed
    let chars_to_trim: &[char] = &[' ', ','];
    let trimmed_str: &str = string.trim_matches(chars_to_trim);
    println!("Used characters: {}", trimmed_str);

    // Heap allocate a string
    let alice = String::from("I like dogs");
    // Allocate new memory and store the modified string there
    let bob: String = alice.replace("dog", "cat");

    println!("Alice says: {}", alice);
    println!("Bob says: {}", bob);
}
```

##### Useful methods

```rust
fn from(value: T) -> Self
```
Converts to this type from the input type.

```rust
let hello = String::from("Hello, world!");
```

```rust
// length of string
let s = String::from("hello");
assert_eq!(s.chars().count(), 5);
```

```rust
pub fn as_str(&self) -> &str
```

Extracts a string slice containing the entire `String`.

```rust
let s = String::from("foo");

assert_eq!("foo", s.as_str());
```

```rust
pub fn push_str(&mut self, string: &str)
```

Appends a given string slice onto the end of this `String`.

```rust
let mut s = String::from("foo");

s.push_str("bar");

assert_eq!("foobar", s);
```

```rust
pub fn capacity(&self) -> usize
```

Returns this `String`'s capacity, in bytes.

```rust
let s = String::with_capacity(10);

assert!(s.capacity() >= 10);
```

```rust
pub fn chars(&self) -> Chars<'_>
```

Returns an iterator over the `chars` of a string slice.

```rust
let word = "goodbye";

let count = word.chars().count();
assert_eq!(7, count);

let mut chars = word.chars();

assert_eq!(Some('g'), chars.next());
assert_eq!(Some('o'), chars.next());
assert_eq!(Some('o'), chars.next());
assert_eq!(Some('d'), chars.next());
assert_eq!(Some('b'), chars.next());
assert_eq!(Some('y'), chars.next());
assert_eq!(Some('e'), chars.next());

assert_eq!(None, chars.next());
```

```rust
pub fn contains<'a, P>(&'a self, pat: P) -> bool
where
    P: Pattern<'a>,
```

Returns `true` if the given pattern matches a sub-slice of this string slice.

Returns `false` if it does not.

The pattern can be a `&str`, `char`, a slice of `char`s, or a function or closure that determines if a character matches.

```rust
let bananas = "bananas";

assert!(bananas.contains("nana"));
assert!(!bananas.contains("apples"));
```

```rust
pub fn starts_with<'a, P>(&'a self, pat: P) -> bool
where
    P: Pattern<'a>,
```

Returns `true` if the given pattern matches a prefix of this string slice.

Returns `false` if it does not.

The pattern can be a &`str`, `char`, a slice of `char`s, or a function or closure that determines if a character matches.

```rust
let bananas = "bananas";

assert!(bananas.starts_with("bana"));
assert!(!bananas.starts_with("nana"));
```

```rust
pub fn ends_with<'a, P>(&'a self, pat: P) -> bool
where
    P: Pattern<'a>,
    <P as Pattern<'a>>::Searcher: ReverseSearcher<'a>,
```

Returns `true` if the given pattern matches a suffix of this string slice.

Returns `false` if it does not.

The pattern can be a `&str`, `char`, a slice of `char`s, or a function or closure that determines if a character matches.

```rust
let bananas = "bananas";

assert!(bananas.ends_with("anas"));
assert!(!bananas.ends_with("nana"));
```

```rust
pub fn find<'a, P>(&'a self, pat: P) -> Option<usize>
where
    P: Pattern<'a>,
```

Returns the byte index of the first character of this string slice that matches the pattern.

Returns `None` if the pattern doesn’t match.

The pattern can be a `&str`, `char`, a slice of `char`s, or a function or closure that determines if a character matches.

```rust
let s = "Löwe 老虎 Léopard Gepardi";

assert_eq!(s.find('L'), Some(0));
assert_eq!(s.find('é'), Some(14));
assert_eq!(s.find("pard"), Some(17));
```

```rust
pub fn split<'a, P>(&'a self, pat: P) -> Split<'a, P>
where
    P: Pattern<'a>,
```

An iterator over substrings of this string slice, separated by characters matched by a pattern.

The pattern can be a `&str`, `char`, a slice of `char`s, or a function or closure that determines if a character matches.

```rust
let v: Vec<&str> = "Mary had a little lamb".split(' ').collect();
assert_eq!(v, ["Mary", "had", "a", "little", "lamb"]);

let v: Vec<&str> = "".split('X').collect();
assert_eq!(v, [""]);

let v: Vec<&str> = "lionXXtigerXleopard".split('X').collect();
assert_eq!(v, ["lion", "", "tiger", "leopard"]);

let v: Vec<&str> = "lion::tiger::leopard".split("::").collect();
assert_eq!(v, ["lion", "tiger", "leopard"]);

let v: Vec<&str> = "abc1def2ghi".split(char::is_numeric).collect();
assert_eq!(v, ["abc", "def", "ghi"]);

let v: Vec<&str> = "lionXtigerXleopard".split(char::is_uppercase).collect();
assert_eq!(v, ["lion", "tiger", "leopard"]);
```

```rust
pub fn trim(&self) -> &str
```

Returns a string slice with leading and trailing whitespace removed.

```rust
let s = "\n Hello\tworld\t\n";

assert_eq!("Hello\tworld", s.trim());
```

```rust
pub fn make_ascii_uppercase(&mut self)
```

Converts this string to its ASCII upper case equivalent in-place.

To return a new uppercased value without modifying the existing one, use `to_ascii_uppercase()`.

```rust
let mut s = String::from("Grüße, Jürgen ❤");

s.make_ascii_uppercase();

assert_eq!("GRüßE, JüRGEN ❤", s);
```

```rust
pub fn make_ascii_lowercase(&mut self)
```

Converts this string to its ASCII lower case equivalent in-place.

To return a new lowercased value without modifying the existing one, use `to_ascii_lowercase()`.

```rust
let mut s = String::from("GRÜßE, JÜRGEN ❤");

s.make_ascii_lowercase();

assert_eq!("grÜße, jÜrgen ❤", s);
```

```rust
pub fn replace<'a, P>(&'a self, from: P, to: &str) -> String
where
    P: Pattern<'a>,
```

Replaces all matches of a pattern with another string.

```rust
let s = "this is old";

assert_eq!("this is new", s.replace("old", "new"));
assert_eq!("than an old", s.replace("is", "an"));
```

##### [Complete List of String Methods](https://doc.rust-lang.org/std/string/struct.String.html#implementations)

#### Box, stack and heap
All values in Rust are stack allocated by default. Values can be *boxed* (allocated on the heap) by creating a `Box<T>`. A box is a smart pointer to a heap allocated value of type `T`. When a box goes out of scope, its destructor is called, the inner object is destroyed, and the memory on the heap is freed.

Boxed values can be dereferenced using the `*` operator; this removes one layer of indirection.

##### Get memory address of pointer

```rust
fn main() {
   let x = 5;
   let p = Box::new(x);

   println!("the memory address of x is {:p}", p); // One possible output: 0x16fa3ac84
}
```

```rust
use std::mem;

#[allow(dead_code)]
#[derive(Debug, Clone, Copy)]
struct Point {
    x: f64,
    y: f64,
}

// A Rectangle can be specified by where its top left and bottom right 
// corners are in space
#[allow(dead_code)]
struct Rectangle {
    top_left: Point,
    bottom_right: Point,
}

fn origin() -> Point {
    Point { x: 0.0, y: 0.0 }
}

fn boxed_origin() -> Box<Point> {
    // Allocate this point on the heap, and return a pointer to it
    Box::new(Point { x: 0.0, y: 0.0 })
}

fn main() {
    // (all the type annotations are superfluous)
    // Stack allocated variables
    let point: Point = origin();
    let rectangle: Rectangle = Rectangle {
        top_left: origin(),
        bottom_right: Point { x: 3.0, y: -4.0 }
    };

    // Heap allocated rectangle
    let boxed_rectangle: Box<Rectangle> = Box::new(Rectangle {
        top_left: origin(),
        bottom_right: Point { x: 3.0, y: -4.0 },
    });

    // The output of functions can be boxed
    let boxed_point: Box<Point> = Box::new(origin());

    // Double indirection
    let box_in_a_box: Box<Box<Point>> = Box::new(boxed_origin());

    println!("Point occupies {} bytes on the stack",
             mem::size_of_val(&point));
    println!("Rectangle occupies {} bytes on the stack",
             mem::size_of_val(&rectangle));

    // box size == pointer size
    println!("Boxed point occupies {} bytes on the stack",
             mem::size_of_val(&boxed_point));
    println!("Boxed rectangle occupies {} bytes on the stack",
             mem::size_of_val(&boxed_rectangle));
    println!("Boxed box occupies {} bytes on the stack",
             mem::size_of_val(&box_in_a_box));

    // Copy the data contained in `boxed_point` into `unboxed_point`
    let unboxed_point: Point = *boxed_point;
    println!("Unboxed point occupies {} bytes on the stack",
             mem::size_of_val(&unboxed_point));
}
```

#### [Rc](https://doc.rust-lang.org/std/rc/index.html)
When multiple ownership is needed, `Rc`(Reference Counting) can be used. `Rc` keeps track of the number of the references which means the number of owners of the value wrapped inside an `Rc`.

Reference count of an `Rc` increases by 1 whenever an `Rc` is cloned, and decreases by 1 whenever one cloned `Rc` is dropped out of the scope. When an `Rc`'s reference count becomes zero (which means there are no remaining owners), both the `Rc` and the value are all dropped.

Cloning an `Rc` never performs a deep copy. Cloning creates just another pointer to the wrapped value, and increments the count.

```rust
use std::rc::Rc;

fn main() {
    let rc_examples = "Rc examples".to_string();
    {
        println!("--- rc_a is created ---");
        
        let rc_a: Rc<String> = Rc::new(rc_examples);
        println!("Reference Count of rc_a: {}", Rc::strong_count(&rc_a));
        
        {
            println!("--- rc_a is cloned to rc_b ---");
            
            let rc_b: Rc<String> = Rc::clone(&rc_a);
            println!("Reference Count of rc_b: {}", Rc::strong_count(&rc_b));
            println!("Reference Count of rc_a: {}", Rc::strong_count(&rc_a));
            
            // Two `Rc`s are equal if their inner values are equal
            println!("rc_a and rc_b are equal: {}", rc_a.eq(&rc_b));
            
            // We can use methods of a value directly
            println!("Length of the value inside rc_a: {}", rc_a.len());
            println!("Value of rc_b: {}", rc_b);
            
            println!("--- rc_b is dropped out of scope ---");
        }
        
        println!("Reference Count of rc_a: {}", Rc::strong_count(&rc_a));
        
        println!("--- rc_a is dropped out of scope ---");
    }
    
    // Error! `rc_examples` already moved into `rc_a`
    // And when `rc_a` is dropped, `rc_examples` is dropped together
    // println!("rc_examples: {}", rc_examples);
    // TODO ^ Try uncommenting this line
}
```

#### [Arc](https://doc.rust-lang.org/std/sync/struct.Arc.html)
When shared ownership between threads is needed, `Arc`(Atomically Reference Counted) can be used. This struct, via the `Clone` implementation can create a reference pointer for the location of a value in the memory heap while increasing the reference counter. As it shares ownership between threads, when the last reference pointer to a value is out of scope, the variable is dropped.

```rust
use std::time::Duration;
use std::sync::Arc;
use std::thread;

fn main() {
    // This variable declaration is where its value is specified.
    let apple = Arc::new("the same apple");

    for _ in 0..10 {
        // Here there is no value specification as it is a pointer to a
        // reference in the memory heap.
        let apple = Arc::clone(&apple);

        thread::spawn(move || {
            // As Arc was used, threads can be spawned using the value allocated
            // in the Arc variable pointer's location.
            println!("{:?}", apple);
        });
    }

    // Make sure all Arc instances are printed from spawned threads.
    thread::sleep(Duration::from_secs(1));
}
```

#### [Vectors](https://doc.rust-lang.org/std/vec/)
Vectors are re-sizable arrays. Like slices, their size is not known at compile time, but they can grow or shrink at any time. A vector is represented using 3 parameters:

* pointer to the data
* length
* capacity

The capacity indicates how much memory is reserved for the vector. The vector can grow as long as the length is smaller than the capacity. When this threshold needs to be surpassed, the vector is reallocated with a larger capacity.

```rust
fn main() {
    // Iterators can be collected into vectors
    let collected_iterator: Vec<i32> = (0..10).collect();
    println!("Collected (0..10) into: {:?}", collected_iterator);

    // The `vec!` macro can be used to initialize a vector
    let mut xs = vec![1i32, 2, 3];
    println!("Initial vector: {:?}", xs);

    // Insert new element at the end of the vector
    println!("Push 4 into the vector");
    xs.push(4);
    println!("Vector: {:?}", xs);

    // Error! Immutable vectors can't grow
    collected_iterator.push(0);
    // FIXME ^ Comment out this line

    // The `len` method yields the number of elements currently stored in a vector
    println!("Vector length: {}", xs.len());

    // Indexing is done using the square brackets (indexing starts at 0)
    println!("Second element: {}", xs[1]);

    // `pop` removes the last element from the vector and returns it
    println!("Pop last element: {:?}", xs.pop());

    // Out of bounds indexing yields a panic
    println!("Fourth element: {}", xs[3]);
    // FIXME ^ Comment out this line

    // `Vector`s can be easily iterated over
    println!("Contents of xs:");
    for x in xs.iter() {
        println!("> {}", x);
    }

    // A `Vector` can also be iterated over while the iteration
    // count is enumerated in a separate variable (`i`)
    for (i, x) in xs.iter().enumerate() {
        println!("In position {} we have value {}", i, x);
    }

    // Thanks to `iter_mut`, mutable `Vector`s can also be iterated
    // over in a way that allows modifying each value
    for x in xs.iter_mut() {
        *x *= 3;
    }
    println!("Updated vector: {:?}", xs);
}
```

##### Useful methods
```rust
pub fn capacity(&self) -> usize
```

Returns the total number of elements the vector can hold without reallocating.

```rust
let mut vec: Vec<i32> = Vec::with_capacity(10);
vec.push(42);
assert_eq!(vec.capacity(), 10);
```

```rust
pub fn clear(&mut self)
```

Clears the vector, removing all values.

```rust
let mut v = vec![1, 2, 3];

v.clear();

assert!(v.is_empty());
```

```rust
pub fn len(&self) -> usize
```

Returns the number of elements in the vector, also referred to as its 'length'.

```rust
let a = vec![1, 2, 3];
assert_eq!(a.len(), 3);
```

```rust
pub fn is_empty(&self) -> bool
```

Returns `true` if the vector contains no elements.

```rust
let mut v = Vec::new();
assert!(v.is_empty());

v.push(1);
assert!(!v.is_empty());
```

```rust
pub fn drain<R>(&mut self, range: R) -> Drain<'_, T, A>
where
    R: RangeBounds<usize>,
```

Removes the specified range from the vector in bulk, returning all removed elements as an iterator. If the iterator is dropped before being fully consumed, it drops the remaining removed elements.

```rust
let mut v = vec![1, 2, 3];
let u: Vec<_> = v.drain(1..).collect();
assert_eq!(v, &[1]);
assert_eq!(u, &[2, 3]);

// A full range clears the vector, like `clear()` does
v.drain(..);
assert_eq!(v, &[]);
```

```rust
pub fn reverse(&mut self)
```

Reverses the order of elements in the slice, in place.

```rust
let mut v = [1, 2, 3];
v.reverse();
assert!(v == [3, 2, 1]);
```

```rust
pub fn insert(&mut self, index: usize, element: T)
```

Inserts an element at position `index` within the vector, shifting all elements after it to the right.

```rust
let mut vec = vec![1, 2, 3];
vec.insert(1, 4);
assert_eq!(vec, [1, 4, 2, 3]);
vec.insert(4, 5);
assert_eq!(vec, [1, 4, 2, 3, 5]);
```

```rust
pub fn remove(&mut self, index: usize) -> T
```

Removes and returns the element at position `index` within the vector, shifting all elements after it to the left.

```rust
let mut v = vec![1, 2, 3];
assert_eq!(v.remove(1), 2);
assert_eq!(v, [1, 3]);
```

```rust
pub fn retain<F>(&mut self, f: F)
where
    F: FnMut(&T) -> bool,
```

Retains only the elements specified by the predicate.

```rust
let mut vec = vec![1, 2, 3, 4];
vec.retain(|&x| x % 2 == 0);
assert_eq!(vec, [2, 4]);
```

```rust
pub fn push(&mut self, value: T)
```

Appends an element to the back of a collection.

```rust
let mut vec = vec![1, 2];
vec.push(3);
assert_eq!(vec, [1, 2, 3]);
```

```rust
pub fn pop(&mut self) -> Option<T>
```

Removes the last element from a vector and returns it, or `None` if it is empty.

```rust
let mut vec = vec![1, 2, 3];
assert_eq!(vec.pop(), Some(3));
assert_eq!(vec, [1, 2]);
```

```rust
pub fn append(&mut self, other: &mut Vec<T, A>)
```

Moves all the elements of other into `self`, leaving `other` empty.

```rust
let mut vec = vec![1, 2, 3];
let mut vec2 = vec![4, 5, 6];
vec.append(&mut vec2);
assert_eq!(vec, [1, 2, 3, 4, 5, 6]);
assert_eq!(vec2, []);
```

```rust
pub fn contains(&self, x: &T) -> bool
where
    T: PartialEq<T>,
```

Returns `true` if the slice contains an element with the given value.

This operation is O(n).

```rust
let v = [10, 40, 30];
assert!(v.contains(&30));
assert!(!v.contains(&50));
```

```rust
pub fn to_vec(&self) -> Vec<T, Global>
where
    T: Clone,
```

Copies `self` into a new `Vec`.

```rust
let s = [10, 40, 30];
let x = s.to_vec();
// Here, `s` and `x` can be modified independently.
```

```rust
pub fn sort(&mut self)
where
    T: Ord,
```

This sort is stable (i.e., does not reorder equal elements) and O(n * log(n)) worst-case.

```rust
let mut v = [-5, 4, 1, -3, 2];

v.sort();
assert!(v == [-5, -3, 1, 2, 4]);
```
 
```rust
pub fn sort_unstable(&mut self)
where
    T: Ord,
```

Sorts the slice, but might not preserve the order of equal elements.

This sort is unstable (i.e., may reorder equal elements), in-place (i.e., does not allocate), and O(n * log(n)) worst-case.

```rust
let mut v = [-5, 4, 1, -3, 2];

v.sort_unstable();
assert!(v == [-5, -3, 1, 2, 4]);
```

```rust
pub fn sort_by<F>(&mut self, compare: F)
where
    F: FnMut(&T, &T) -> Ordering,
```

Sorts the slice with a comparator function.

```rust
let mut floats = [5f64, 4.0, 1.0, 3.0, 2.0];
floats.sort_by(|a, b| a.partial_cmp(b).unwrap());
assert_eq!(floats, [1.0, 2.0, 3.0, 4.0, 5.0]);
```

##### [Complete List of Vec Methods](https://doc.rust-lang.org/std/vec/struct.Vec.html#implementations)

#### [HashMaps](https://doc.rust-lang.org/std/collections/hash_map/struct.HashMap.html)
Where vectors store values by an integer index, `HashMaps` store values by key. `HashMap` keys can be booleans, integers, strings, or any other type that implements the `Eq` and `Hash` traits. More on this in the next section.

Like vectors, `HashMaps` are growable, but HashMaps can also shrink themselves when they have excess space. You can create a HashMap with a certain starting capacity using `HashMap::with_capacity(uint)`, or use `HashMap::new()` to get a HashMap with a default initial capacity (recommended).

```rust
use std::collections::HashMap;

fn call(number: &str) -> &str {
    match number {
        "798-1364" => "We're sorry, the call cannot be completed as dialed. 
            Please hang up and try again.",
        "645-7689" => "Hello, this is Mr. Awesome's Pizza. My name is Fred.
            What can I get for you today?",
        _ => "Hi! Who is this again?"
    }
}

fn main() { 
    let mut contacts = HashMap::new();

    contacts.insert("Daniel", "798-1364");
    contacts.insert("Ashley", "645-7689");
    contacts.insert("Katie", "435-8291");
    contacts.insert("Robert", "956-1745");

    // Takes a reference and returns Option<&V>
    match contacts.get(&"Daniel") {
        Some(&number) => println!("Calling Daniel: {}", call(number)),
        _ => println!("Don't have Daniel's number."),
    }

    // `HashMap::insert()` returns `None`
    // if the inserted value is new, `Some(value)` otherwise
    contacts.insert("Daniel", "164-6743");

    match contacts.get(&"Ashley") {
        Some(&number) => println!("Calling Ashley: {}", call(number)),
        _ => println!("Don't have Ashley's number."),
    }

    contacts.remove(&"Ashley"); 

    // `HashMap::iter()` returns an iterator that yields 
    // (&'a key, &'a value) pairs in arbitrary order.
    for (contact, &number) in contacts.iter() {
        println!("Calling {}: {}", contact, call(number)); 
    }
}
```

##### Useful methods

```rust
pub fn capacity(&self) -> usize
```

Returns the number of elements the map can hold without reallocating.

```rust
use std::collections::HashMap;
let map: HashMap<i32, i32> = HashMap::with_capacity(100);
assert!(map.capacity() >= 100);
```

```rust
pub fn keys(&self) -> Keys<'_, K, V>
```

An iterator visiting all keys in arbitrary order. The iterator element type is `&'a K`.

```rust
use std::collections::HashMap;

let map = HashMap::from([
    ("a", 1),
    ("b", 2),
    ("c", 3),
]);

for key in map.keys() {
    println!("{key}");
}
```

```rust
pub fn values(&self) -> Values<'_, K, V>
```

An iterator visiting all values in arbitrary order. The iterator element type is `&'a V`.

```rust
use std::collections::HashMap;

let map = HashMap::from([
    ("a", 1),
    ("b", 2),
    ("c", 3),
]);

for val in map.values() {
    println!("{val}");
}
```

```rust
pub fn values_mut(&mut self) -> ValuesMut<'_, K, V>
```

An iterator visiting all values mutably in arbitrary order. The iterator element type is `&'a mut V`.

```rust
use std::collections::HashMap;

let mut map = HashMap::from([
    ("a", 1),
    ("b", 2),
    ("c", 3),
]);

for val in map.values_mut() {
    *val = *val + 10;
}

for val in map.values() {
    println!("{val}");
}
```

```rust
pub fn len(&self) -> usize
```

Returns the number of elements in the map.

```rust
use std::collections::HashMap;

let mut a = HashMap::new();
assert_eq!(a.len(), 0);
a.insert(1, "a");
assert_eq!(a.len(), 1);
```

```rust
pub fn is_empty(&self) -> bool
```

Returns `true` if the map contains no elements.

```rust
use std::collections::HashMap;

let mut a = HashMap::new();
assert!(a.is_empty());
a.insert(1, "a");
assert!(!a.is_empty());
```

```rust
pub fn retain<F>(&mut self, f: F)
where
    F: FnMut(&K, &mut V) -> bool,
```

Retains only the elements specified by the predicate.

```rust
use std::collections::HashMap;

let mut map: HashMap<i32, i32> = (0..8).map(|x| (x, x*10)).collect();
map.retain(|&k, _| k % 2 == 0);
assert_eq!(map.len(), 4);
```

```rust
pub fn clear(&mut self)
```

Clears the map, removing all key-value pairs. Keeps the allocated memory for reuse.

```rust
use std::collections::HashMap;

let mut a = HashMap::new();
a.insert(1, "a");
a.clear();
assert!(a.is_empty());
```

```rust
pub fn get<Q: ?Sized>(&self, k: &Q) -> Option<&V>
where
    K: Borrow<Q>,
    Q: Hash + Eq,
```

Returns a reference to the value corresponding to the key.

```rust
use std::collections::HashMap;

let mut map = HashMap::new();
map.insert(1, "a");
assert_eq!(map.get(&1), Some(&"a"));
assert_eq!(map.get(&2), None);
```

```rust
pub fn insert(&mut self, k: K, v: V) -> Option<V>
```

Inserts a key-value pair into the map.

If the map did not have this key present, `None` is returned.

```rust
use std::collections::HashMap;

let mut map = HashMap::new();
assert_eq!(map.insert(37, "a"), None);
assert_eq!(map.is_empty(), false);

map.insert(37, "b");
assert_eq!(map.insert(37, "c"), Some("b"));
assert_eq!(map[&37], "c");
```

```rust
pub fn remove<Q: ?Sized>(&mut self, k: &Q) -> Option<V>
where
    K: Borrow<Q>,
    Q: Hash + Eq,
```

Removes a key from the map, returning the value at the key if the key was previously in the map.

```rust
use std::collections::HashMap;

let mut map = HashMap::new();
map.insert(1, "a");
assert_eq!(map.remove(&1), Some("a"));
assert_eq!(map.remove(&1), None);
```

```rust
pub fn entry(&mut self, key: K) -> Entry<'_, K, V>
```

Gets the given key's corresponding entry in the map for in-place manipulation.

```rust
use std::collections::HashMap;

let mut letters = HashMap::new();

for ch in "a short treatise on fungi".chars() {
    letters.entry(ch).and_modify(|counter| *counter += 1).or_insert(1);
}

assert_eq!(letters[&'s'], 2);
assert_eq!(letters[&'t'], 3);
assert_eq!(letters[&'u'], 1);
assert_eq!(letters.get(&'y'), None);
```

```rust
pub fn and_modify<F>(self, f: F) -> Self
where
    F: FnOnce(&mut V),
```

Provides in-place mutable access to an occupied entry before any potential inserts into the map.

```rust
use std::collections::HashMap;

let mut map: HashMap<&str, u32> = HashMap::new();

map.entry("poneyland")
   .and_modify(|e| { *e += 1 })
   .or_insert(42);
assert_eq!(map["poneyland"], 42);

map.entry("poneyland")
   .and_modify(|e| { *e += 1 })
   .or_insert(42);
assert_eq!(map["poneyland"], 43);
```

```rust
pub fn or_default(self) -> &'a mut V
```

Ensures a value is in the entry by inserting the default value if empty, and returns a mutable reference to the value in the entry.

```rust
use std::collections::HashMap;

let mut map: HashMap<&str, Option<u32>> = HashMap::new();
map.entry("poneyland").or_default();

assert_eq!(map["poneyland"], None);
```

```rust
pub fn or_insert(self, default: V) -> &'a mut V
```

Ensures a value is in the entry by inserting the default if empty, and returns a mutable reference to the value in the entry.

```rust
use std::collections::HashMap;

let mut map: HashMap<&str, u32> = HashMap::new();

map.entry("poneyland").or_insert(3);
assert_eq!(map["poneyland"], 3);

*map.entry("poneyland").or_insert(10) *= 2;
assert_eq!(map["poneyland"], 6);
```

```rust
pub fn or_insert_with<F: FnOnce() -> V>(self, default: F) -> &'a mut V
```

Ensures a value is in the entry by inserting the result of the default function if empty, and returns a mutable reference to the value in the entry.

```rust
use std::collections::HashMap;

let mut map: HashMap<&str, String> = HashMap::new();
let s = "hoho".to_string();

map.entry("poneyland").or_insert_with(|| s);

assert_eq!(map["poneyland"], "hoho".to_string());
```

##### [Complete List of Hashmap Methods](https://doc.rust-lang.org/std/collections/struct.HashMap.html#implementations)

#### [HashSets](https://doc.rust-lang.org/std/collections/hash_set/struct.HashSet.html)
Consider a `HashSet` as a `HashMap` where we just care about the keys ( `HashSet<T>` is, in actuality, just a wrapper around `HashMap<T, ()>`).

A `HashSet`'s unique feature is that it is guaranteed to not have duplicate elements. That's the contract that any set collection fulfills. `HashSet` is just one implementation. (`BTreeSet`)

If you insert a value that is already present in the `HashSet`, (i.e. the new value is equal to the existing and they both have the same hash), then the new value will replace the old.

This is great for when you never want more than one of something, or when you want to know if you've already got something.

Sets have 4 primary operations (all of the following calls return an iterator):

`union`: get all the unique elements in both sets.

`difference`: get all the elements that are in the first set but not the second.

`intersection`: get all the elements that are only in both sets.

`symmetric_difference`: get all the elements that are in one set or the other, but not both.

```rust
use std::collections::HashSet;

fn main() {
    let mut a: HashSet<i32> = vec![1i32, 2, 3].into_iter().collect();
    let mut b: HashSet<i32> = vec![2i32, 3, 4].into_iter().collect();

    assert!(a.insert(4));
    assert!(a.contains(&4));

    // `HashSet::insert()` returns false if
    // there was a value already present.
    assert!(b.insert(4), "Value 4 is already in set B!");
    // FIXME ^ Comment out this line

    b.insert(5);

    // If a collection's element type implements `Debug`,
    // then the collection implements `Debug`.
    // It usually prints its elements in the format `[elem1, elem2, ...]`
    println!("A: {:?}", a);
    println!("B: {:?}", b);

    // Print [1, 2, 3, 4, 5] in arbitrary order
    println!("Union: {:?}", a.union(&b).collect::<Vec<&i32>>());

    // This should print [1]
    println!("Difference: {:?}", a.difference(&b).collect::<Vec<&i32>>());

    // Print [2, 3, 4] in arbitrary order.
    println!("Intersection: {:?}", a.intersection(&b).collect::<Vec<&i32>>());

    // Print [1, 5]
    println!("Symmetric Difference: {:?}",
             a.symmetric_difference(&b).collect::<Vec<&i32>>());
}
```

#### Option
The `Option<T>` enum has two variants:

* `None`, to indicate failure or lack of value, and
* `Some(value)`, a tuple struct that wraps a `value` with type `T`.

```rust
// An integer division that doesn't `panic!`
fn checked_division(dividend: i32, divisor: i32) -> Option<i32> {
    if divisor == 0 {
        // Failure is represented as the `None` variant
        None
    } else {
        // Result is wrapped in a `Some` variant
        Some(dividend / divisor)
    }
}

// This function handles a division that may not succeed
fn try_division(dividend: i32, divisor: i32) {
    // `Option` values can be pattern matched, just like other enums
    match checked_division(dividend, divisor) {
        None => println!("{} / {} failed!", dividend, divisor),
        Some(quotient) => {
            println!("{} / {} = {}", dividend, divisor, quotient)
        },
    }
}

fn main() {
    try_division(4, 2);
    try_division(1, 0);

    // Binding `None` to a variable needs to be type annotated
    let none: Option<i32> = None;
    let _equivalent_none = None::<i32>;

    let optional_float = Some(0f32);

    // Unwrapping a `Some` variant will extract the value wrapped.
    println!("{:?} unwraps to {:?}", optional_float, optional_float.unwrap());

    // Unwrapping a `None` variant will `panic!`
    println!("{:?} unwraps to {:?}", none, none.unwrap());
}
```

##### Useful Methods

```rust
pub const fn is_some(&self) -> bool
```

Returns `true` if the option is a `Some` value.

```rust
let x: Option<u32> = Some(2);
assert_eq!(x.is_some(), true);

let x: Option<u32> = None;
assert_eq!(x.is_some(), false);
```

```rust
pub const fn is_none(&self) -> bool
```

Returns `true` if the option is a `None` value.

```rust
let x: Option<u32> = Some(2);
assert_eq!(x.is_none(), false);

let x: Option<u32> = None;
assert_eq!(x.is_none(), true);
```

```rust
pub fn expect(self, msg: &str) -> T
```

Returns the contained `Some` value, consuming the `self` value.

Panics
Panics if the value is a `None` with a custom panic message provided by `msg`.

```rust
let item = slice.get(0)
    .expect("slice should not be empty");
```

```rust
pub fn unwrap(self) -> T
```

Returns the contained `Some` value, consuming the `self` value.

Because this function may panic, its use is generally discouraged. Instead, prefer to use pattern matching and handle the `None` case explicitly, or call `unwrap_or`, `unwrap_or_else`, or `unwrap_or_default`.

```rust
let x = Some("air");
assert_eq!(x.unwrap(), "air");
```

```rust
pub fn unwrap_or(self, default: T) -> T
```

Returns the contained `Some` value or a provided default.

Arguments passed to `unwrap_or` are eagerly evaluated; if you are passing the result of a function call, it is recommended to use `unwrap_or_else`, which is lazily evaluated.

```rust
assert_eq!(Some("car").unwrap_or("bike"), "car");
assert_eq!(None.unwrap_or("bike"), "bike");`
```

```rust
pub fn unwrap_or_else<F>(self, f: F) -> T
where
    F: FnOnce() -> T,
```

Returns the contained `Some` value or computes it from a closure.

```rust
let k = 10;
assert_eq!(Some(4).unwrap_or_else(|| 2 * k), 4);
assert_eq!(None.unwrap_or_else(|| 2 * k), 20);
```

```rust
pub fn unwrap_or_default(self) -> T
where
    T: Default,
```

Returns the contained `Some` value or a default.

```rust
let x: Option<u32> = None;
let y: Option<u32> = Some(12);

assert_eq!(x.unwrap_or_default(), 0);
assert_eq!(y.unwrap_or_default(), 12);
```

##### [Complete List of Option Methods](https://doc.rust-lang.org/std/option/enum.Option.html#implementations)

#### [Result](https://doc.rust-lang.org/std/result/index.html)
The `Result<T, E>` enum has two variants:

* `Ok(value)` which indicates that the operation succeeded, and wraps the `value` returned by the operation. (`value` has type `T`)
* `Err(why)`, which indicates that the operation failed, and wraps `why`, which (hopefully) explains the cause of the failure. (`why` has type `E`)

```rust
mod checked {
    // Mathematical "errors" we want to catch
    #[derive(Debug)]
    pub enum MathError {
        DivisionByZero,
        NonPositiveLogarithm,
        NegativeSquareRoot,
    }

    pub type MathResult = Result<f64, MathError>;

    pub fn div(x: f64, y: f64) -> MathResult {
        if y == 0.0 {
            // This operation would `fail`, instead let's return the reason of
            // the failure wrapped in `Err`
            Err(MathError::DivisionByZero)
        } else {
            // This operation is valid, return the result wrapped in `Ok`
            Ok(x / y)
        }
    }

    pub fn sqrt(x: f64) -> MathResult {
        if x < 0.0 {
            Err(MathError::NegativeSquareRoot)
        } else {
            Ok(x.sqrt())
        }
    }

    pub fn ln(x: f64) -> MathResult {
        if x <= 0.0 {
            Err(MathError::NonPositiveLogarithm)
        } else {
            Ok(x.ln())
        }
    }
}

// `op(x, y)` === `sqrt(ln(x / y))`
fn op(x: f64, y: f64) -> f64 {
    // This is a three level match pyramid!
    match checked::div(x, y) {
        Err(why) => panic!("{:?}", why),
        Ok(ratio) => match checked::ln(ratio) {
            Err(why) => panic!("{:?}", why),
            Ok(ln) => match checked::sqrt(ln) {
                Err(why) => panic!("{:?}", why),
                Ok(sqrt) => sqrt,
            },
        },
    }
}

fn main() {
    // Will this fail?
    println!("{}", op(1.0, 10.0));
}
```

##### ?
Chaining results using match can get pretty untidy; luckily, the `?` operator can be used to make things pretty again. `?` is used at the end of an expression returning a `Result`, and is equivalent to a match expression, where the `Err(err)` branch expands to an early return `Err(From::from(err))`, and the `Ok(ok)` branch expands to an `ok` expression.

```rust
mod checked {
    #[derive(Debug)]
    enum MathError {
        DivisionByZero,
        NonPositiveLogarithm,
        NegativeSquareRoot,
    }

    type MathResult = Result<f64, MathError>;

    fn div(x: f64, y: f64) -> MathResult {
        if y == 0.0 {
            Err(MathError::DivisionByZero)
        } else {
            Ok(x / y)
        }
    }

    fn sqrt(x: f64) -> MathResult {
        if x < 0.0 {
            Err(MathError::NegativeSquareRoot)
        } else {
            Ok(x.sqrt())
        }
    }

    fn ln(x: f64) -> MathResult {
        if x <= 0.0 {
            Err(MathError::NonPositiveLogarithm)
        } else {
            Ok(x.ln())
        }
    }

    // Intermediate function
    fn op_(x: f64, y: f64) -> MathResult {
        // if `div` "fails", then `DivisionByZero` will be `return`ed
        let ratio = div(x, y)?;

        // if `ln` "fails", then `NonPositiveLogarithm` will be `return`ed
        let ln = ln(ratio)?;

        sqrt(ln)
    }

    pub fn op(x: f64, y: f64) {
        match op_(x, y) {
            Err(why) => panic!("{}", match why {
                MathError::NonPositiveLogarithm
                    => "logarithm of non-positive number",
                MathError::DivisionByZero
                    => "division by zero",
                MathError::NegativeSquareRoot
                    => "square root of negative number",
            }),
            Ok(value) => println!("{}", value),
        }
    }
}

fn main() {
    checked::op(1.0, 10.0);
}
```

##### Useful Methods

```rust
pub const fn is_ok(&self) -> bool
```

Returns `true` if the result is `Ok`.

```rust
let x: Result<i32, &str> = Ok(-3);
assert_eq!(x.is_ok(), true);

let x: Result<i32, &str> = Err("Some error message");
assert_eq!(x.is_ok(), false);
```

```rust
pub const fn is_err(&self) -> bool`
```

Returns `true` if the result is `Err`.

```rust
let x: Result<i32, &str> = Ok(-3);
assert_eq!(x.is_err(), false);

let x: Result<i32, &str> = Err("Some error message");
assert_eq!(x.is_err(), true);
```

```rust
pub fn expect(self, msg: &str) -> T
where
    E: Debug,
```

Returns the contained `Ok` value, consuming the `self` value.

Because this function may panic, its use is generally discouraged. Instead, prefer to use pattern matching and handle the `Err` case explicitly, or call `unwrap_or`, `unwrap_or_else`, or `unwrap_or_default`.

```rust
let path = std::env::var("IMPORTANT_PATH")
    .expect("env variable `IMPORTANT_PATH` should be set by `wrapper_script.sh`");
```

```rust
pub fn unwrap(self) -> T
where
    E: Debug,
```

Returns the contained `Ok` value, consuming the `self` value.

Because this function may panic, its use is generally discouraged. Instead, prefer to use pattern matching and handle the `Err` case explicitly, or call `unwrap_or`, `unwrap_or_else`, or `unwrap_or_default`.

```rust
let x: Result<u32, &str> = Ok(2);
assert_eq!(x.unwrap(), 2);
```

```rust
pub fn unwrap_or(self, default: T) -> T
```

Returns the contained `Ok` value or a provided default.

Arguments passed to `unwrap_or` are eagerly evaluated; if you are passing the result of a function call, it is recommended to use `unwrap_or_else`, which is lazily evaluated.

```rust
let default = 2;
let x: Result<u32, &str> = Ok(9);
assert_eq!(x.unwrap_or(default), 9);

let x: Result<u32, &str> = Err("error");
assert_eq!(x.unwrap_or(default), default);
```

```rust
pub fn unwrap_or_else<F>(self, op: F) -> T
where
    F: FnOnce(E) -> T,
```

Returns the contained `Ok` value or computes it from a closure.

```rust
fn count(x: &str) -> usize { x.len() }

assert_eq!(Ok(2).unwrap_or_else(count), 2);
assert_eq!(Err("foo").unwrap_or_else(count), 3);
```

```rust
pub fn unwrap_or_default(self) -> T
where
    T: Default,
```

Returns the contained `Ok` value or a default

```rust
let good_year_from_input = "1909";
let bad_year_from_input = "190blarg";
let good_year = good_year_from_input.parse().unwrap_or_default();
let bad_year = bad_year_from_input.parse().unwrap_or_default();

assert_eq!(1909, good_year);
assert_eq!(0, bad_year);
```

##### [Complete List of Result Methods](https://doc.rust-lang.org/std/result/enum.Result.html#implementations)

### Get Type of Variable

```rust
use std::any::type_name;
fn print_type_of<T>(_: &T) {
    println!("{}", std::any::type_name::<T>())
}

fn main() {
    let s = "Hello";
    let i = 42;

    print_type_of(&s); // &str
    print_type_of(&i); // i32
}
```

### Type Conversions
#### String to Integer
To convert a string to an integer in Rust, use `parse()` function. The parse function needs to know what type, which can be specified on the left-side of assignment like so:

```rust
let str = "123";
let num: i32 = str.parse().unwrap();
```

```rust
let str = "123";
let num = str.parse::<i32>().unwrap();
```

#### String to Float
To convert a string to a float in Rust, uses the same `parse()` function. You just specify f64 for the type.

```rust
let pi = "3.14159";
let num: f64 = pi.parse().unwrap()
```

#### Char to a Integer
To convert a single char to an integer in Rust, use `.to_digit(RADIX)`. The radix value is used for conversion, 10 for decimal, 16 for hexadecimal.

```rust
let ch = '2';
let num = ch.to_digit(10).unwrap();
```

```rust
let ch = 'f';
let num = ch.to_digit(16).unwrap();
println!("{}", num);
// 15
```

#### Integer to String
To convert an integer to a string in Rust, use `.to_string()`

```rust
let num = 123;
let str = num.to_string();
```

#### Integer to Float
To convert an integer to a float in Rust, use `as f64`. This is a useful means to convert from various interchangeable types, you can use inline also.

```rust
let num = 13;
let flt = num as f64;
println!("{}", 1.2 * flt);
println!("{}", 1.2 * num as f64);
```

#### Float to Integer
Use the same `as` to cast a float to an integer type. By default, it will discard the decimal portion of the number giving you an equivalent of `floor()`. Use `.round()` or `.ceil()` prior to control how you want the float to be converted.

```rust
let num = 13.6 as f32;
let i = num as i32;
let j = num.round() as i32;
let k = num.ceil() as i32;
println!("{} {} {}", i, j, k);
```

### Custom Types
#### [Structures](https://doc.rust-lang.org/std/keyword.struct.html)
There are three types of structures ("structs") that can be created using the `struct` keyword:

* Unit structs, which are field-less, are useful for generics.
* Tuple structs, which are, basically, named tuples.
* The classic C structs

```rust
// An attribute to hide warnings for unused code.
#![allow(dead_code)]

#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

// A unit struct
struct Unit;

// A tuple struct
struct Pair(i32, f32);

// A struct with two fields
struct Point {
    x: f32,
    y: f32,
}

// Structs can be reused as fields of another struct
struct Rectangle {
    // A rectangle can be specified by where the top left and bottom right
    // corners are in space.
    top_left: Point,
    bottom_right: Point,
}

fn main() {
    // Create struct with field init shorthand
    let name = String::from("Peter");
    let age = 27;
    let peter = Person { name, age };

    // Print debug struct
    println!("{:?}", peter);

    // Instantiate a `Point`
    let point: Point = Point { x: 10.3, y: 0.4 };

    // Access the fields of the point
    println!("point coordinates: ({}, {})", point.x, point.y);

    // Make a new point by using struct update syntax to use the fields of our
    // other one
    let bottom_right = Point { x: 5.2, ..point };

    // `bottom_right.y` will be the same as `point.y` because we used that field
    // from `point`
    println!("second point: ({}, {})", bottom_right.x, bottom_right.y);

    // Destructure the point using a `let` binding
    let Point { x: left_edge, y: top_edge } = point;

    let _rectangle = Rectangle {
        // struct instantiation is an expression too
        top_left: Point { x: left_edge, y: top_edge },
        bottom_right: bottom_right,
    };

    // Instantiate a unit struct
    let _unit = Unit;

    // Instantiate a tuple struct
    let pair = Pair(1, 0.1);

    // Access the fields of a tuple struct
    println!("pair contains {:?} and {:?}", pair.0, pair.1);

    // Destructure a tuple struct
    let Pair(integer, decimal) = pair;

    println!("pair contains {:?} and {:?}", integer, decimal);
}
```

#### [Enums](https://doc.rust-lang.org/std/keyword.enum.html)
The `enum` keyword allows the creation of a type which may be one of a few different variants. Any variant which is valid as a `struct` is also valid as an `enum`.

```rust
// Create an `enum` to classify a web event. Note how both
// names and type information together specify the variant:
// `PageLoad != PageUnload` and `KeyPress(char) != Paste(String)`.
// Each is different and independent.
enum WebEvent {
    // An `enum` may either be `unit-like`,
    PageLoad,
    PageUnload,
    // like tuple structs,
    KeyPress(char),
    Paste(String),
    // or c-like structures.
    Click { x: i64, y: i64 },
}

// A function which takes a `WebEvent` enum as an argument and
// returns nothing.
fn inspect(event: WebEvent) {
    match event {
        WebEvent::PageLoad => println!("page loaded"),
        WebEvent::PageUnload => println!("page unloaded"),
        // Destructure `c` from inside the `enum`.
        WebEvent::KeyPress(c) => println!("pressed '{}'.", c),
        WebEvent::Paste(s) => println!("pasted \"{}\".", s),
        // Destructure `Click` into `x` and `y`.
        WebEvent::Click { x, y } => {
            println!("clicked at x={}, y={}.", x, y);
        },
    }
}

fn main() {
    let pressed = WebEvent::KeyPress('x');
    // `to_owned()` creates an owned `String` from a string slice.
    let pasted  = WebEvent::Paste("my text".to_owned());
    let click   = WebEvent::Click { x: 20, y: 80 };
    let load    = WebEvent::PageLoad;
    let unload  = WebEvent::PageUnload;

    inspect(pressed);
    inspect(pasted);
    inspect(click);
    inspect(load);
    inspect(unload);
}

// Enums can also be created with explicit discriminator
enum Number {
    Zero,
    One,
    Two,
}

enum Number1 {
    Zero = 0,
    One,
    Two,
}

// C-like enum
enum Number2 {
    Zero = 0,
    One = 1,
    Two = 2,
}

fn main() {
    // a enum variant can be converted to a integer by `as`
    assert_eq!(Number::One as u8, Number1::One as u8);
    assert_eq!(Number1::One as u8, Number2::One as u8);
} 
```

#### constants
Rust has two different types of constants which can be declared in any scope including global. Both require explicit type annotation:

* `const`: An unchangeable value (the common case).
* `static`: A possibly `mut`able variable with ['static](https://doc.rust-lang.org/rust-by-example/scope/lifetime/static_lifetime.html) lifetime. The static lifetime is inferred and does not have to be specified. Accessing or modifying a mutable static variable is [unsafe](https://doc.rust-lang.org/rust-by-example/unsafe.html).

```rust
// Globals are declared outside all other scopes.
static LANGUAGE: &str = "Rust";
const THRESHOLD: i32 = 10;

fn is_big(n: i32) -> bool {
    // Access constant in some function
    n > THRESHOLD
}

fn main() {
    let n = 16;

    // Access constant in the main thread
    println!("This is {}", LANGUAGE);
    println!("The threshold is {}", THRESHOLD);
    println!("{} is {}", n, if is_big(n) { "big" } else { "small" });

    // Error! Cannot modify a `const`.
    THRESHOLD = 5;
    // FIXME ^ Comment out this line
}
```

### Casting
Rust provides no implicit type conversion (coercion) between primitive types. But, explicit type conversion (casting) can be performed using the `as` keyword.

```rust
// Suppress all warnings from casts which overflow.
#![allow(overflowing_literals)]

fn main() {
    let decimal = 65.4321_f32;

    // Error! No implicit conversion
    let integer: u8 = decimal;
    // FIXME ^ Comment out this line

    // Explicit conversion
    let integer = decimal as u8;
    let character = integer as char;

    // Error! There are limitations in conversion rules. 
    // A float cannot be directly converted to a char.
    let character = decimal as char;
    // FIXME ^ Comment out this line

    println!("Casting: {} -> {} -> {}", decimal, integer, character);

    // when casting any value to an unsigned type, T,
    // T::MAX + 1 is added or subtracted until the value
    // fits into the new type

    // 1000 already fits in a u16
    println!("1000 as a u16 is: {}", 1000 as u16);

    // 1000 - 256 - 256 - 256 = 232
    // Under the hood, the first 8 least significant bits (LSB) are kept,
    // while the rest towards the most significant bit (MSB) get truncated.
    println!("1000 as a u8 is : {}", 1000 as u8);
    // -1 + 256 = 255
    println!("  -1 as a u8 is : {}", (-1i8) as u8);

    // For positive numbers, this is the same as the modulus
    println!("1000 mod 256 is : {}", 1000 % 256);

    // When casting to a signed type, the (bitwise) result is the same as
    // first casting to the corresponding unsigned type. If the most significant
    // bit of that value is 1, then the value is negative.

    // Unless it already fits, of course.
    println!(" 128 as a i16 is: {}", 128 as i16);
    
    // 128 as u8 -> 128, whose value in 8-bit two's complement representation is:
    println!(" 128 as a i8 is : {}", 128 as i8);

    // repeating the example above
    // 1000 as u8 -> 232
    println!("1000 as a u8 is : {}", 1000 as u8);
    // and the value of 232 in 8-bit two's complement representation is -24
    println!(" 232 as a i8 is : {}", 232 as i8);
    
    // Since Rust 1.45, the `as` keyword performs a *saturating cast* 
    // when casting from float to int. If the floating point value exceeds 
    // the upper bound or is less than the lower bound, the returned value 
    // will be equal to the bound crossed.
    
    // 300.0 as u8 is 255
    println!(" 300.0 as u8 is : {}", 300.0_f32 as u8);
    // -100.0 as u8 is 0
    println!("-100.0 as u8 is : {}", -100.0_f32 as u8);
    // nan as u8 is 0
    println!("   nan as u8 is : {}", f32::NAN as u8);
    
    // This behavior incurs a small runtime cost and can be avoided 
    // with unsafe methods, however the results might overflow and 
    // return **unsound values**. Use these methods wisely:
    unsafe {
        // 300.0 as u8 is 44
        println!(" 300.0 as u8 is : {}", 300.0_f32.to_int_unchecked::<u8>());
        // -100.0 as u8 is 156
        println!("-100.0 as u8 is : {}", (-100.0_f32).to_int_unchecked::<u8>());
        // nan as u8 is 0
        println!("   nan as u8 is : {}", f32::NAN.to_int_unchecked::<u8>());
    }
}
```

### Literals
Numeric literals can be type annotated by adding the type as a suffix. As an example, to specify that the literal `42` should have the type `i32`, write `42i32`.

The type of unsuffixed numeric literals will depend on how they are used. If no constraint exists, the compiler will use `i32` for integers, and `f64` for floating-point numbers.

```rust
fn main() {
    // Suffixed literals, their types are known at initialization
    let x = 1u8;
    let y = 2u32;
    let z = 3f32;

    // Unsuffixed literals, their types depend on how they are used
    let i = 1;
    let f = 1.0;

    // `size_of_val` returns the size of a variable in bytes
    println!("size of `x` in bytes: {}", std::mem::size_of_val(&x));
    println!("size of `y` in bytes: {}", std::mem::size_of_val(&y));
    println!("size of `z` in bytes: {}", std::mem::size_of_val(&z));
    println!("size of `i` in bytes: {}", std::mem::size_of_val(&i));
    println!("size of `f` in bytes: {}", std::mem::size_of_val(&f));
}
```

### Aliasing
The `type` statement can be used to give a new name to an existing type. Types must have `UpperCamelCase` names, or the compiler will raise a warning. The exception to this rule are the primitive types: `usize`, `f32`, etc.

```rust
// `NanoSecond`, `Inch`, and `U64` are new names for `u64`.
type NanoSecond = u64;
type Inch = u64;
type U64 = u64;

fn main() {
    // `NanoSecond` = `Inch` = `U64` = `u64`.
    let nanoseconds: NanoSecond = 5 as U64;
    let inches: Inch = 2 as U64;

    // Note that type aliases *don't* provide any extra type safety, because
    // aliases are *not* new types
    println!("{} nanoseconds + {} inches = {} unit?",
             nanoseconds,
             inches,
             nanoseconds + inches);
}
```

The main use of aliases is to reduce boilerplate; for example the `IoResult<T>` type is an alias for the `Result<T, IoError>` type.

## [cargo commands](https://doc.rust-lang.org/cargo/commands/index.html)

### run 
Run a binary or example of the local package

### run --release
Run optimized artifacts with the release profile

### build
Compile the current package

### check
Analyze the current package and report errors, but don't build object files

## Control flow
### if/else

```rust
if x == 4 {
    println!("x is four");
} else if x == 3 {
    println!("x is three");
} else {
    println!("x is something else");
}
```

### conditional expressions

```rust
let y = if 12 * 15 > 150 {
    "Bigger"
} else {
    "Smaller"
};
```

### function definitions

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();
    println!("The value of x is: {x}");
}
```

### infinite loops

```rust
loop { 
    println!("I live.");
    break;
}
```

### predicate loops 

```rust
let mut i = 0;

while i < 10 {
    println!("hello");
    i = i + 1;
}
```

### predicate pattern loops

```rust
let mut x = vec![1, 2, 3];

while let Some(y) = x.pop() {
    println!("y = {}", y);
}

while let _ = 5 {
    println!("Irrefutable patterns are always true");
    break;
}
```

### for loops

```rust
let v = &["apples", "cake", "coffee"];

for text in v {
    println!("I like {}.", text);
}
```

### range expressions
| Production           | Syntax      | Type                       | Range                     |
|----------------------|-------------|----------------------------|---------------------------|
| RangeExpr            | start..end  | [std::ops::Range](https://doc.rust-lang.org/std/ops/struct.Range.html)            | $start \leq x < end$      |
| RangeFromExpr        | start..     | [std::ops::RangeFrom](https://doc.rust-lang.org/std/ops/struct.RangeFrom.html)        | $x \geq start$            |
| RangeToExpr          | ..end       | [std::ops::RangeTo](https://doc.rust-lang.org/std/ops/struct.RangeTo.html)        | $x < end$                   |
| RangeFullExpr        | ..          | [std::ops::RangeFull](https://doc.rust-lang.org/std/ops/struct.RangeFull.html)        | $-$                         |
| RangeInclusiveExpr   | start..=end | [std::ops::RangeInclusive](https://doc.rust-lang.org/std/ops/struct.RangeInclusive.html)   | $start \leq x \leq end$ |
| RangeToInclusiveExpr | ..=end      | [std::ops::RangeToInclusive](https://doc.rust-lang.org/std/ops/struct.RangeToInclusive.html) | $x \leq end$              |

```rust
1..2;   // std::ops::Range
3..;    // std::ops::RangeFrom
..4;    // std::ops::RangeTo
..;     // std::ops::RangeFull
5..=6;  // std::ops::RangeInclusive
..=7;   // std::ops::RangeToInclusive
```

```rust
for i in 0..10 {
    println!("{}", i);
}
```

### [break](https://doc.rust-lang.org/std/keyword.break.html)

```rust
loop {
    println!("This prints once because of break.");
    break;
}
```

### [continue](https://doc.rust-lang.org/std/keyword.continue.html)

```rust
for i in 0..10 {
    if i == 5 {
        // skips to next iteration of loop
        continue;
    }
    // does not print i == 5
    println!("{}", i);
}
```

### nesting and labeling loops

```rust
#![allow(unreachable_code)]

fn main() {
    'outer: loop {
        println!("Entered the outer loop");

        'inner: loop {
            println!("Entered the inner loop");

            // This would break only the inner loop
            //break;

            // This breaks the outer loop
            break 'outer;
        }

        println!("This point will never be reached");
    }

    println!("Exited the outer loop");
}
```


### [match statements](https://doc.rust-lang.org/std/keyword.match.html)

```rust
fn main() {
    let number = 13;
    // TODO ^ Try different values for `number`

    println!("Tell me about {}", number);
    match number {
        // Match a single value
        1 => println!("One!"),
        // Match several values
        2 | 3 | 5 | 7 | 11 => println!("This is a prime"),
        // TODO ^ Try adding 13 to the list of prime values
        // Match an inclusive range
        13..=19 => println!("A teen"),
        // Handle the rest of cases
        _ => println!("Ain't special"),
        // TODO ^ Try commenting out this catch-all arm
    }

    let boolean = true;
    // Match is an expression too
    let binary = match boolean {
        // The arms of a match must cover all the possible values
        false => 0,
        true => 1,
        // TODO ^ Try commenting out one of these arms
    };

    println!("{} -> {}", boolean, binary);
}
```

### if let statements

```rust
fn main() {
    // All have type `Option<i32>`
    let number = Some(7);
    let letter: Option<i32> = None;
    let emoticon: Option<i32> = None;

    // The `if let` construct reads: "if `let` destructures `number` into
    // `Some(i)`, evaluate the block (`{}`).
    if let Some(i) = number {
        println!("Matched {:?}!", i);
    }

    // If you need to specify a failure, use an else:
    if let Some(i) = letter {
        println!("Matched {:?}!", i);
    } else {
        // Destructure failed. Change to the failure case.
        println!("Didn't match a number. Let's go with a letter!");
    }

    // Provide an altered failing condition.
    let i_like_letters = false;

    if let Some(i) = emoticon {
        println!("Matched {:?}!", i);
    // Destructure failed. Evaluate an `else if` condition to see if the
    // alternate failure branch should be taken:
    } else if i_like_letters {
        println!("Didn't match a number. Let's go with a letter!");
    } else {
        // The condition evaluated false. This branch is the default:
        println!("I don't like letters. Let's go with an emoticon :)!");
    }
}
```

## Rust memory semantics

### immutable variables
Immutable variables cannot be modified after created. They are the default.

```rust
let a: i32 = 100;
```

### mutable variables
Mutable variables can be modified after created.

```rust
let mut a: i32 = 100;
```

### mutable and immutable references
Mutable data can be mutably borrowed using `&mut T`. This is called a mutable reference and gives read/write access to the borrower. In contrast, `&T` borrows the data via an immutable reference, and the borrower can read the data but not modify it

```rust
fn main() {
    // original variable must be mutable in order to use a mutable reference
    let mut s = String::from("hello, ");
    borrow_object(&mut s);
    println!("Success!");
}

fn borrow_object(s: &mut String) {}
```

#### Rules
* At most one mutable reference at a time
* Multiple immutable references allowed
* No mutable and immutable references at the same time

Mutability can be changed when ownership is transferred:

```rust
fn main() {
    let s = String::from("hello, ");
    
    let mut s1 = s;

    s1.push_str("world");

    println!("Success!");
}
```

### aliasing
Data can be immutably borrowed any number of times, but while immutably borrowed, the original data can't be mutably borrowed. On the other hand, only *one* mutable borrow is allowed at a time. The original data can be borrowed again only *after* the mutable reference has been used for the last time.

### ref pattern
When doing pattern matching or destructuring via the `let` binding, the `ref` keyword can be used to take references to the fields of a struct/tuple. The example below shows a few instances where this can be useful:

```rust
#[derive(Clone, Copy)]
struct Point { x: i32, y: i32 }

fn main() {
    let c = 'Q';

    // A `ref` borrow on the left side of an assignment is equivalent to
    // an `&` borrow on the right side.
    let ref ref_c1 = c;
    let ref_c2 = &c;

    println!("ref_c1 equals ref_c2: {}", *ref_c1 == *ref_c2);

    let point = Point { x: 0, y: 0 };

    // `ref` is also valid when destructuring a struct.
    let _copy_of_x = {
        // `ref_to_x` is a reference to the `x` field of `point`.
        let Point { x: ref ref_to_x, y: _ } = point;

        // Return a copy of the `x` field of `point`.
        *ref_to_x
    };

    // A mutable copy of `point`
    let mut mutable_point = point;

    {
        // `ref` can be paired with `mut` to take mutable references.
        let Point { x: _, y: ref mut mut_ref_to_y } = mutable_point;

        // Mutate the `y` field of `mutable_point` via a mutable reference.
        *mut_ref_to_y = 1;
    }

    println!("point is ({}, {})", point.x, point.y);
    println!("mutable_point is ({}, {})", mutable_point.x, mutable_point.y);

    // A mutable tuple that includes a pointer
    let mut mutable_tuple = (Box::new(5u32), 3u32);
    
    {
        // Destructure `mutable_tuple` to change the value of `last`.
        let (_, ref mut last) = mutable_tuple;
        *last = 2u32;
    }
    
    println!("tuple is {:?}", mutable_tuple);
}
```

Note that:

```rust
let c = '中';
let ref r1 = c;
let ref r2 = c;
```

Is equivalent to:

```rust
let c = '中';
let r1 = &c;
let r2 = &c;
```

### partial move

Within the destructuring of a single variable, both by-move and by-reference pattern bindings can be used at the same time. Doing this will result in a partial move of the variable, which means that parts of the variable will be moved while other parts stay. In such a case, the parent variable cannot be used afterwards as a whole, however the parts that are only referenced (and not moved) can still be used.

```rust
fn main() {
    #[derive(Debug)]
    struct Person {
        name: String,
        age: Box<u8>,
    }

    let person = Person {
        name: String::from("Alice"),
        age: Box::new(20),
    };

    // `name` is moved out of person, but `age` is referenced
    let Person { name, ref age } = person;

    println!("The person's age is {}", age);

    println!("The person's name is {}", name);

    // Error! borrow of partially moved value: `person` partial move occurs
    //println!("The person struct is {:?}", person);

    // `person` cannot be used but `person.age` can be used as it is not moved
    println!("The person's age from person struct is {}", person.age);
}
```

## Traits
A `trait` is a collection of methods defined for an unknown type: `Self`. They can access other methods declared in the same trait.

Traits can be implemented for any data type. In the example below, we define `Animal`, a group of methods. The `Animal` `trait` is then implemented for the `Sheep` data type, allowing the use of methods from `Animal` with a `Sheep`.

```rust
struct Sheep { naked: bool, name: &'static str }

trait Animal {
    // Associated function signature; `Self` refers to the implementor type.
    fn new(name: &'static str) -> Self;

    // Method signatures; these will return a string.
    fn name(&self) -> &'static str;
    fn noise(&self) -> &'static str;

    // Traits can provide default method definitions.
    fn talk(&self) {
        println!("{} says {}", self.name(), self.noise());
    }
}

impl Sheep {
    fn is_naked(&self) -> bool {
        self.naked
    }

    fn shear(&mut self) {
        if self.is_naked() {
            // Implementor methods can use the implementor's trait methods.
            println!("{} is already naked...", self.name());
        } else {
            println!("{} gets a haircut!", self.name);

            self.naked = true;
        }
    }
}

// Implement the `Animal` trait for `Sheep`.
impl Animal for Sheep {
    // `Self` is the implementor type: `Sheep`.
    fn new(name: &'static str) -> Sheep {
        Sheep { name: name, naked: false }
    }

    fn name(&self) -> &'static str {
        self.name
    }

    fn noise(&self) -> &'static str {
        if self.is_naked() {
            "baaaaah?"
        } else {
            "baaaaah!"
        }
    }
    
    // Default trait methods can be overridden.
    fn talk(&self) {
        // For example, we can add some quiet contemplation.
        println!("{} pauses briefly... {}", self.name, self.noise());
    }
}

fn main() {
    // Type annotation is necessary in this case.
    let mut dolly: Sheep = Animal::new("Dolly");
    // TODO ^ Try removing the type annotations.

    dolly.talk();
    dolly.shear();
    dolly.talk();
}
```

### Derive
The compiler is capable of providing basic implementations for some traits via the `#[derive]` attribute. These traits can still be manually implemented if a more complex behavior is required.

The following is a list of derivable traits:

* Comparison traits: [Eq](https://doc.rust-lang.org/std/cmp/trait.Eq.html), [PartialEq](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html), [Ord](https://doc.rust-lang.org/std/cmp/trait.Ord.html), [PartialOrd](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html).
  
```rust
#[derive(PartialEq,Eq,Ord,Debug)]
struct Point2 {
    x: i32,
    y: i32,
}

// Let's assume we want to compare point by their distance to the origin
impl std::cmp::PartialOrd for Point2 {
    fn partial_cmp(&self, other: &Self) -> Option<std::cmp::Ordering> {
        let this = self.x * self.x + self.y * self.y;
        let that = other.x * other.x + other.y * other.y;
        return this.partial_cmp(&that);
    }
}

let p = Point2{x:3,y:1};
let q = Point2{x:2,y:100};
println!("{}", p < q);
println!("{}", p > q);
```
* [Clone](https://doc.rust-lang.org/std/clone/trait.Clone.html), to create `T` from `&T` via a copy.
* [Copy](https://doc.rust-lang.org/core/marker/trait.Copy.html), to give a type 'copy semantics' instead of 'move semantics'.
* [Hash](https://doc.rust-lang.org/std/hash/trait.Hash.html), to compute a hash from `&T`.
* [Default](https://doc.rust-lang.org/std/default/trait.Default.html), to create an empty instance of a data type.
* [Debug](https://doc.rust-lang.org/std/fmt/trait.Debug.html), to format a value using the `{:?}` formatter.
  * [Display Trait](https://doc.rust-lang.org/rust-by-example/hello/print/print_display.html):
  
```rust
// Import (via `use`) the `fmt` module to make it available.
use std::fmt;

// Define a structure for which `fmt::Display` will be implemented. This is
// a tuple struct named `Structure` that contains an `i32`.
struct Structure(i32);

// To use the `{}` marker, the trait `fmt::Display` must be implemented
// manually for the type.
impl fmt::Display for Structure {
    // This trait requires `fmt` with this exact signature.
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        // Write strictly the first element into the supplied output
        // stream: `f`. Returns `fmt::Result` which indicates whether the
        // operation succeeded or failed. Note that `write!` uses syntax which
        // is very similar to `println!`.
        write!(f, "{}", self.0)
    }
}
```

```rust
// `Centimeters`, a tuple struct that can be compared
#[derive(PartialEq, PartialOrd)]
struct Centimeters(f64);

// `Inches`, a tuple struct that can be printed
#[derive(Debug)]
struct Inches(i32);

impl Inches {
    fn to_centimeters(&self) -> Centimeters {
        let &Inches(inches) = self;

        Centimeters(inches as f64 * 2.54)
    }
}

// `Seconds`, a tuple struct with no additional attributes
struct Seconds(i32);

fn main() {
    let _one_second = Seconds(1);

    // Error: `Seconds` can't be printed; it doesn't implement the `Debug` trait
    //println!("One second looks like: {:?}", _one_second);
    // TODO ^ Try uncommenting this line

    // Error: `Seconds` can't be compared; it doesn't implement the `PartialEq` trait
    //let _this_is_true = (_one_second == _one_second);
    // TODO ^ Try uncommenting this line

    let foot = Inches(12);

    println!("One foot equals {:?}", foot);

    let meter = Centimeters(100.0);

    let cmp =
        if foot.to_centimeters() < meter {
            "smaller"
        } else {
            "bigger"
        };

    println!("One foot is {} than one meter.", cmp);
}
```

### Dyn
The Rust compiler needs to know how much space every function's return type requires. This means all your functions have to return a concrete type. Unlike other languages, if you have a trait like `Animal`, you can't write a function that returns `Animal`, because its different implementations will need different amounts of memory.

However, there's an easy workaround. Instead of returning a trait object directly, our functions return a `Box` which contains some `Animal`. A `box` is just a reference to some memory in the heap. Because a reference has a statically-known size, and the compiler can guarantee it points to a heap-allocated `Animal`, we can return a trait from our function!

Rust tries to be as explicit as possible whenever it allocates memory on the heap. So if your function returns a pointer-to-trait-on-heap in this way, you need to write the return type with the `dyn` keyword, e.g. `Box<dyn Animal>`.

```rust
struct Sheep {}
struct Cow {}

trait Animal {
    // Instance method signature
    fn noise(&self) -> &'static str;
}

// Implement the `Animal` trait for `Sheep`.
impl Animal for Sheep {
    fn noise(&self) -> &'static str {
        "baaaaah!"
    }
}

// Implement the `Animal` trait for `Cow`.
impl Animal for Cow {
    fn noise(&self) -> &'static str {
        "moooooo!"
    }
}

// Returns some struct that implements Animal, but we don't know which one at compile time.
fn random_animal(random_number: f64) -> Box<dyn Animal> {
    if random_number < 0.5 {
        Box::new(Sheep {})
    } else {
        Box::new(Cow {})
    }
}

fn main() {
    let random_number = 0.234;
    let animal = random_animal(random_number);
    println!("You've randomly chosen an animal, and it says {}", animal.noise());
}
```

### Drop 
The Drop trait only has one method: `drop`, which is called automatically when an object goes out of scope. The main use of the `Drop` trait is to free the resources that the implementor instance owns.

`Box`, `Vec`, `String`, `File`, and `Process` are some examples of types that implement the `Drop` trait to free resources. The `Drop` trait can also be manually implemented for any custom data type.

The following example adds a print to console to the drop function to announce when it is called.

```rust
struct Droppable {
    name: &'static str,
}

// This trivial implementation of `drop` adds a print to console.
impl Drop for Droppable {
    fn drop(&mut self) {
        println!("> Dropping {}", self.name);
    }
}

fn main() {
    let _a = Droppable { name: "a" };

    // block A
    {
        let _b = Droppable { name: "b" };

        // block B
        {
            let _c = Droppable { name: "c" };
            let _d = Droppable { name: "d" };

            println!("Exiting block B");
        }
        println!("Just exited block B");

        println!("Exiting block A");
    }
    println!("Just exited block A");

    // Variable can be manually dropped using the `drop` function
    drop(_a);
    // TODO ^ Try commenting this line

    println!("end of the main function");

    // `_a` *won't* be `drop`ed again here, because it already has been
    // (manually) `drop`ed
}
```

We can also use the `std::mem::drop` function.

```rust
fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    println!("CustomSmartPointer created.");
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}
```

### Iterators
The [Iterator](https://doc.rust-lang.org/core/iter/trait.Iterator.html) trait is used to implement iterators over collections such as arrays.

There are three common methods which can create iterators from a collection:

`iter()`, which iterates over `&T`.
`iter_mut()`, which iterates over `&mut T`.
`into_iter()`, which iterates over `T`.

```rust
struct Fibonacci {
    curr: u32,
    next: u32,
}

// Implement `Iterator` for `Fibonacci`.
// The `Iterator` trait only requires a method to be defined for the `next` element.
impl Iterator for Fibonacci {
    // We can refer to this type using Self::Item
    type Item = u32;

    // Here, we define the sequence using `.curr` and `.next`.
    // The return type is `Option<T>`:
    //     * When the `Iterator` is finished, `None` is returned.
    //     * Otherwise, the next value is wrapped in `Some` and returned.
    // We use Self::Item in the return type, so we can change
    // the type without having to update the function signatures.
    fn next(&mut self) -> Option<Self::Item> {
        let current = self.curr;

        self.curr = self.next;
        self.next = current + self.next;

        // Since there's no endpoint to a Fibonacci sequence, the `Iterator` 
        // will never return `None`, and `Some` is always returned.
        Some(current)
    }
}

// Returns a Fibonacci sequence generator
fn fibonacci() -> Fibonacci {
    Fibonacci { curr: 0, next: 1 }
}

fn main() {
    // `0..3` is an `Iterator` that generates: 0, 1, and 2.
    let mut sequence = 0..3;

    println!("Four consecutive `next` calls on 0..3");
    println!("> {:?}", sequence.next());
    println!("> {:?}", sequence.next());
    println!("> {:?}", sequence.next());
    println!("> {:?}", sequence.next());

    // `for` works through an `Iterator` until it returns `None`.
    // Each `Some` value is unwrapped and bound to a variable (here, `i`).
    println!("Iterate through 0..3 using `for`");
    for i in 0..3 {
        println!("> {}", i);
    }

    // The `take(n)` method reduces an `Iterator` to its first `n` terms.
    println!("The first four terms of the Fibonacci sequence are: ");
    for i in fibonacci().take(4) {
        println!("> {}", i);
    }

    // The `skip(n)` method shortens an `Iterator` by dropping its first `n` terms.
    println!("The next four terms of the Fibonacci sequence are: ");
    for i in fibonacci().skip(4).take(4) {
        println!("> {}", i);
    }

    let array = [1u32, 3, 3, 7];

    // The `iter` method produces an `Iterator` over an array/slice.
    println!("Iterate the following array {:?}", &array);
    for i in array.iter() {
        println!("> {}", i);
    }
}
```

### Impl Trait
`impl Trait` can be used in two locations:

1. as an argument type
2. as a return type

`&impl` -> static dispatch (relevant in the context of return values)
`&dyn` and `&T` (also `Box<T>`) -> dynamic dispatch (again relevant in the context of return values)

#### As an argument type

```rust
fn parse_csv_document<R: std::io::BufRead>(src: R) -> std::io::Result<Vec<Vec<String>>> {
    src.lines()
        .map(|line| {
            // For each line in the source
            line.map(|line| {
                // If the line was read successfully, process it, if not, return the error
                line.split(',') // Split the line separated by commas
                    .map(|entry| String::from(entry.trim())) // Remove leading and trailing whitespace
                    .collect() // Collect all strings in a row into a Vec<String>
            })
        })
        .collect() // Collect all lines into a Vec<Vec<String>>
}
```

```rust
fn parse_csv_document(src: impl std::io::BufRead) -> std::io::Result<Vec<Vec<String>>> {
    src.lines()
        .map(|line| {
            // For each line in the source
            line.map(|line| {
                // If the line was read successfully, process it, if not, return the error
                line.split(',') // Split the line separated by commas
                    .map(|entry| String::from(entry.trim())) // Remove leading and trailing whitespace
                    .collect() // Collect all strings in a row into a Vec<String>
            })
        })
        .collect() // Collect all lines into a Vec<Vec<String>>
}
```

#### As a return type

```rust
use std::iter;
use std::vec::IntoIter;

// This function combines two `Vec<i32>` and returns an iterator over it.
// Look how complicated its return type is!
fn combine_vecs_explicit_return_type(
    v: Vec<i32>,
    u: Vec<i32>,
) -> iter::Cycle<iter::Chain<IntoIter<i32>, IntoIter<i32>>> {
    v.into_iter().chain(u.into_iter()).cycle()
}

// This is the exact same function, but its return type uses `impl Trait`.
// Look how much simpler it is!
fn combine_vecs(
    v: Vec<i32>,
    u: Vec<i32>,
) -> impl Iterator<Item=i32> {
    v.into_iter().chain(u.into_iter()).cycle()
}

fn main() {
    let v1 = vec![1, 2, 3];
    let v2 = vec![4, 5];
    let mut v3 = combine_vecs(v1, v2);
    assert_eq!(Some(1), v3.next());
    assert_eq!(Some(2), v3.next());
    assert_eq!(Some(3), v3.next());
    assert_eq!(Some(4), v3.next());
    assert_eq!(Some(5), v3.next());
    println!("all done");
}
```

```rust
// Returns a function that adds `y` to its input
fn make_adder_function(y: i32) -> impl Fn(i32) -> i32 {
    let closure = move |x: i32| { x + y };
    closure
}

fn main() {
    let plus_one = make_adder_function(1);
    assert_eq!(plus_one(2), 3);
}
```

```rust
fn double_positives<'a>(numbers: &'a Vec<i32>) -> impl Iterator<Item = i32> + 'a {
    numbers
        .iter()
        .filter(|x| x > &&0)
        .map(|x| x * 2)
}

fn main() {
    let singles = vec![-3, -2, 2, 3];
    let doubles = double_positives(&singles);
    assert_eq!(doubles.collect::<Vec<i32>>(), vec![4, 6]);
}
```

### Clone and Copy
When dealing with resources, the default behavior is to transfer them during assignments or function calls. However, sometimes we need to make a copy of the resource as well.

The [Clone](https://doc.rust-lang.org/std/clone/trait.Clone.html) trait helps us do exactly this. Most commonly, we can use the `.clone()` method defined by the `Clone` trait.

Note: A type that implements `Copy` must also implement `Clone`, because a type that implements `Copy` has a trivial implementation of `Clone` that performs the same task as `Copy`.

Difference between `Clone` and `Copy` is that `Clone` requires the `.clone()` method while `Copy` implicitly does copies.

```rust
// A unit struct without resources
#[derive(Debug, Clone, Copy)]
struct Unit;

// A tuple struct with resources that implements the `Clone` trait
#[derive(Clone, Debug)]
struct Pair(Box<i32>, Box<i32>);

fn main() {
    // Instantiate `Unit`
    let unit = Unit;
    // Copy `Unit`, there are no resources to move
    let copied_unit = unit;

    // Both `Unit`s can be used independently
    println!("original: {:?}", unit);
    println!("copy: {:?}", copied_unit);

    // Instantiate `Pair`
    let pair = Pair(Box::new(1), Box::new(2));
    println!("original: {:?}", pair);

    // Move `pair` into `moved_pair`, moves resources
    let moved_pair = pair;
    println!("moved: {:?}", moved_pair);

    // Error! `pair` has lost its resources
    //println!("original: {:?}", pair);
    // TODO ^ Try uncommenting this line

    // Clone `moved_pair` into `cloned_pair` (resources are included)
    let cloned_pair = moved_pair.clone();
    // Drop the original pair using std::mem::drop
    drop(moved_pair);

    // Error! `moved_pair` has been dropped
    //println!("copy: {:?}", moved_pair);
    // TODO ^ Try uncommenting this line

    // The result from .clone() can still be used!
    println!("clone: {:?}", cloned_pair);
}
```

### Supertraits
Rust doesn't have "inheritance", but you can define a trait as being a superset of another trait.

```rust
trait Person {
    fn name(&self) -> String;
}

// Person is a supertrait of Student.
// Implementing Student requires you to also impl Person.
trait Student: Person {
    fn university(&self) -> String;
}

trait Programmer {
    fn fav_language(&self) -> String;
}

// CompSciStudent (computer science student) is a subtrait of both Programmer 
// and Student. Implementing CompSciStudent requires you to impl both supertraits.
trait CompSciStudent: Programmer + Student {
    fn git_username(&self) -> String;
}

fn comp_sci_student_greeting(student: &dyn CompSciStudent) -> String {
    format!(
        "My name is {} and I attend {}. My favorite language is {}. My Git username is {}",
        student.name(),
        student.university(),
        student.fav_language(),
        student.git_username()
    )
}

fn main() {}
```

### Disambiguating overlapping traits
A type can implement many different traits. What if two traits both require the same name? For example, many traits might have a method named `get()`. They might even have different return types!

Good news: because each trait implementation gets its own `impl` block, it's clear which trait's `get` method you're implementing.

What about when it comes time to call those methods? To disambiguate between them, we have to use Fully Qualified Syntax.

```rust
trait UsernameWidget {
    // Get the selected username out of this widget
    fn get(&self) -> String;
}

trait AgeWidget {
    // Get the selected age out of this widget
    fn get(&self) -> u8;
}

// A form with both a UsernameWidget and an AgeWidget
struct Form {
    username: String,
    age: u8,
}

impl UsernameWidget for Form {
    fn get(&self) -> String {
        self.username.clone()
    }
}

impl AgeWidget for Form {
    fn get(&self) -> u8 {
        self.age
    }
}

fn main() {
    let form = Form {
        username: "rustacean".to_owned(),
        age: 28,
    };

    // If you uncomment this line, you'll get an error saying
    // "multiple `get` found". Because, after all, there are multiple methods
    // named `get`.
    // println!("{}", form.get());

    let username = <Form as UsernameWidget>::get(&form);
    assert_eq!("rustacean".to_owned(), username);
    let age = <Form as AgeWidget>::get(&form);
    assert_eq!(28, age);
}
```

## Generics 
### Implementation

```rust
#![allow(unused)]
fn main() {
struct S; // Concrete type `S`
struct GenericVal<T>(T); // Generic type `GenericVal`

// impl of GenericVal where we explicitly specify type parameters:
impl GenericVal<f32> {} // Specify `f32`
impl GenericVal<S> {} // Specify `S` as defined above

// `<T>` Must precede the type to remain generic
impl<T> GenericVal<T> {}
}
```

```rust
struct Val {
    val: f64,
}

struct GenVal<T> {
    gen_val: T,
}

// impl of Val
impl Val {
    fn value(&self) -> &f64 {
        &self.val
    }
}

// impl of GenVal for a generic type `T`
impl<T> GenVal<T> {
    fn value(&self) -> &T {
        &self.gen_val
    }
}

fn main() {
    let x = Val { val: 3.0 };
    let y = GenVal { gen_val: 3i32 };

    println!("{}, {}", x.value(), y.value());
}
```
### Traits

```rust
// Non-copyable types.
struct Empty;
struct Null;

// A trait generic over `T`.
trait DoubleDrop<T> {
    // Define a method on the caller type which takes an
    // additional single parameter `T` and does nothing with it.
    fn double_drop(self, _: T);
}

// Implement `DoubleDrop<T>` for any generic parameter `T` and
// caller `U`.
impl<T, U> DoubleDrop<T> for U {
    // This method takes ownership of both passed arguments,
    // deallocating both.
    fn double_drop(self, _: T) {}
}

fn main() {
    let empty = Empty;
    let null  = Null;

    // Deallocate `empty` and `null`.
    empty.double_drop(null);

    //empty;
    //null;
    // ^ TODO: Try uncommenting these lines.
}
```
### Where Clauses
A bound can also be expressed using a `where` clause immediately before the opening `{`, rather than at the type's first mention. Additionally, `where` clauses can apply bounds to arbitrary types, rather than just to type parameters.

Some cases that a `where` clause is useful:

* When specifying generic types and bounds separately is clearer:
  
```rust
impl <A: TraitB + TraitC, D: TraitE + TraitF> MyTrait<A, D> for YourType {}

// Expressing bounds with a `where` clause
impl <A, D> MyTrait<A, D> for YourType where
    A: TraitB + TraitC,
    D: TraitE + TraitF {}
```
When using a `where` clause is more expressive than using normal syntax. The `impl` in this example cannot be directly expressed without a `where` clause:

```rust
use std::fmt::Debug;

trait PrintInOption {
    fn print_in_option(self);
}

// Because we would otherwise have to express this as `T: Debug` or 
// use another method of indirect approach, this requires a `where` clause:
impl<T> PrintInOption for T where
    Option<T>: Debug {
    // We want `Option<T>: Debug` as our bound because that is what's
    // being printed. Doing otherwise would be using the wrong bound.
    fn print_in_option(self) {
        println!("{:?}", Some(self));
    }
}

fn main() {
    let vec = vec![1, 2, 3];

    vec.print_in_option();
}
```

## [Modules](https://doc.rust-lang.org/rust-by-example/mod.html)
A module is a collection of items: functions, structs, traits, `impl` blocks, and even other modules.

### Visibility
By default, the items in a module have private visibility, but this can be overridden with the `pub` modifier. Only the public items of a module can be accessed from outside the module scope.

### Struct Visibility
Structs have an extra level of visibility with their fields. The visibility defaults to private, and can be overridden with the `pub` modifier. This visibility only matters when a struct is accessed from outside the module where it is defined, and has the goal of hiding information (encapsulation).

### Paths to modules
* Global paths: start from `crate`
* Local paths:
  * going one or many levels up via `super`

### The use declaration
The `use` declaration can be used to bind a full path to a new name, for easier access. It is often used like this:

```rust
use crate::deeply::nested::{
    my_first_function,
    my_second_function,
    AndATraitType
};

fn main() {
    my_first_function();
}
```

You can use the `as` keyword to bind imports to a different name:

```rust
// Bind the `deeply::nested::function` path to `other_function`.
use deeply::nested::function as other_function;

fn function() {
    println!("called `function()`");
}

mod deeply {
    pub mod nested {
        pub fn function() {
            println!("called `deeply::nested::function()`");
        }
    }
}

fn main() {
    // Easier access to `deeply::nested::function`
    other_function();

    println!("Entering block");
    {
        // This is equivalent to `use deeply::nested::function as function`.
        // This `function()` will shadow the outer one.
        use crate::deeply::nested::function;

        // `use` bindings have a local scope. In this case, the
        // shadowing of `function()` is only in this block.
        function();

        println!("Leaving block");
    }

    function();
}
```

## [Crates](https://doc.rust-lang.org/rust-by-example/crates.html)
A crate is a compilation unit in Rust. It can be compiled into a binary or into a library.

### Add crates
Tell Rust you want to use it:
* edit `Cargo.toml`
* add `[crate_name]="[version]"` below `dependencies`

### Useful Crates
#### [rand](https://crates.io/crates/rand)
* generate a random integer from 1 through 100
  
```rust
use rand::Rng;
rand::thread_rng().gen_range(1..=100);
```

#### [serde](https://crates.io/crates/serde)

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let point = Point { x: 1, y: 2 };

    // Convert the Point to a JSON string.
    let serialized = serde_json::to_string(&point).unwrap();

    // Prints serialized = {"x":1,"y":2}
    println!("serialized = {}", serialized);

    // Convert the JSON string back to a Point.
    let deserialized: Point = serde_json::from_str(&serialized).unwrap();

    // Prints deserialized = Point { x: 1, y: 2 }
    println!("deserialized = {:?}", deserialized);
}
```

#### [csv](https://crates.io/crates/csv)

```rust
use std::error::Error;
use std::io;
use std::process;

fn example() -> Result<(), Box<dyn Error>> {
    // Build the CSV reader and iterate over each record.
    let mut rdr = csv::Reader::from_reader(io::stdin());
    for result in rdr.records() {
        // The iterator yields Result<StringRecord, Error>, so we check the
        // error here.
        let record = result?;
        println!("{:?}", record);
    }
    Ok(())
}

fn main() {
    if let Err(err) = example() {
        println!("error running example: {}", err);
        process::exit(1);
    }
}
```

#### [utf8_slice](https://docs.rs/utf8_slice/1.0.0/utf8_slice/)

```rust
pub fn len(s: &str) -> usize
```

Fetches the length in characters of an utf8/unicode string

```rust
pub fn slice(s: &str, begin: usize, end: usize) -> &str
```

Fetches a slice of a string from a begin to an end index taking into account utf8/unicode character indices.

```rust
let s = "The 🚀 goes to the 🌑!";

let rocket = utf8_slice::slice(s, 4, 5);
// Will equal "🚀"
```

## File I/O
The `File` struct represents a file that has been opened (it wraps a file descriptor), and gives read and/or write access to the underlying file.

Since many things can go wrong when doing file I/O, all the `File` methods return the `io::Result<T>` type, which is an alias for `Result<T, io::Error>`.

### Open
The `open` function can be used to open a file in read-only mode.

A `File` owns a resource, the file descriptor and takes care of closing the file when it is `drop`ped.

```rust
use std::fs::File;
use std::io::prelude::*;
use std::path::Path;

fn main() {
    // Create a path to the desired file
    let path = Path::new("hello.txt");
    let display = path.display();

    // Open the path in read-only mode, returns `io::Result<File>`
    let mut file = match File::open(&path) {
        Err(why) => panic!("couldn't open {}: {}", display, why),
        Ok(file) => file,
    };

    // Read the file contents into a string, returns `io::Result<usize>`
    let mut s = String::new();
    match file.read_to_string(&mut s) {
        Err(why) => panic!("couldn't read {}: {}", display, why),
        Ok(_) => print!("{} contains:\n{}", display, s),
    }

    // `file` goes out of scope, and the "hello.txt" file gets closed
}
```

### create
The `create` function opens a file in write-only mode. If the file already existed, the old content is destroyed. Otherwise, a new file is created.

```rust
static LOREM_IPSUM: &str =
    "Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
";

use std::fs::File;
use std::io::prelude::*;
use std::path::Path;

fn main() {
    let path = Path::new("lorem_ipsum.txt");
    let display = path.display();

    // Open a file in write-only mode, returns `io::Result<File>`
    let mut file = match File::create(&path) {
        Err(why) => panic!("couldn't create {}: {}", display, why),
        Ok(file) => file,
    };

    // Write the `LOREM_IPSUM` string to `file`, returns `io::Result<()>`
    match file.write_all(LOREM_IPSUM.as_bytes()) {
        Err(why) => panic!("couldn't write to {}: {}", display, why),
        Ok(_) => println!("successfully wrote to {}", display),
    }
}
```

### read_lines
The method `lines()` returns an iterator over the lines of a file.

`File::open` expects a generic, `AsRef<Path>`. That's what `read_lines()` expects as input.

```rust
use std::fs::File;
use std::io::{self, BufRead};
use std::path::Path;

fn main() {
    // File hosts must exist in current path before this produces output
    if let Ok(lines) = read_lines("./hosts") {
        // Consumes the iterator, returns an (Optional) String
        for line in lines {
            if let Ok(ip) = line {
                println!("{}", ip);
            }
        }
    }
}

// The output is wrapped in a Result to allow matching on errors
// Returns an Iterator to the Reader of the lines of the file.
fn read_lines<P>(filename: P) -> io::Result<io::Lines<io::BufReader<File>>>
where P: AsRef<Path>, {
    let file = File::open(filename)?;
    Ok(io::BufReader::new(file).lines())
}
```

## [Lifetimes](https://doc.rust-lang.org/rust-by-example/scope/lifetime.html)
A *lifetime* is a construct the compiler (or more specifically, its *borrow checker*) uses to ensure all borrows are valid. Specifically, a variable's lifetime begins when it is created and ends when it is destroyed. While lifetimes and scopes are often referred to together, they are not the same.

```rust
// Lifetimes are annotated below with lines denoting the creation
// and destruction of each variable.
// `i` has the longest lifetime because its scope entirely encloses 
// both `borrow1` and `borrow2`. The duration of `borrow1` compared 
// to `borrow2` is irrelevant since they are disjoint.
fn main() {
    let i = 3; // Lifetime for `i` starts. ────────────────┐
    //                                                     │
    { //                                                   │
        let borrow1 = &i; // `borrow1` lifetime starts. ──┐│
        //                                                ││
        println!("borrow1: {}", borrow1); //              ││
    } // `borrow1 ends. ──────────────────────────────────┘│
    //                                                     │
    //                                                     │
    { //                                                   │
        let borrow2 = &i; // `borrow2` lifetime starts. ──┐│
        //                                                ││
        println!("borrow2: {}", borrow2); //              ││
    } // `borrow2` ends. ─────────────────────────────────┘│
    //                                                     │
}   // Lifetime ends. ─────────────────────────────────────┘
```

### Explicit annotation

```rust
// `print_refs` takes two references to `i32` which have different
// lifetimes `'a` and `'b`. These two lifetimes must both be at
// least as long as the function `print_refs`.
fn print_refs<'a, 'b>(x: &'a i32, y: &'b i32) {
    println!("x is {} and y is {}", x, y);
}

// A function which takes no arguments, but has a lifetime parameter `'a`.
fn failed_borrow<'a>() {
    let _x = 12;

    // ERROR: `_x` does not live long enough
    let y: &'a i32 = &_x;
    // Attempting to use the lifetime `'a` as an explicit type annotation 
    // inside the function will fail because the lifetime of `&_x` is shorter
    // than that of `y`. A short lifetime cannot be coerced into a longer one.
}

fn main() {
    // Create variables to be borrowed below.
    let (four, nine) = (4, 9);
    
    // Borrows (`&`) of both variables are passed into the function.
    print_refs(&four, &nine);
    // Any input which is borrowed must outlive the borrower. 
    // In other words, the lifetime of `four` and `nine` must 
    // be longer than that of `print_refs`.
    
    failed_borrow();
    // `failed_borrow` contains no references to force `'a` to be 
    // longer than the lifetime of the function, but `'a` is longer.
    // Because the lifetime is never constrained, it defaults to `'static`.
}
```

### Functions

```rust
// One input reference with lifetime `'a` which must live
// at least as long as the function.
fn print_one<'a>(x: &'a i32) {
    println!("`print_one`: x is {}", x);
}

// Mutable references are possible with lifetimes as well.
fn add_one<'a>(x: &'a mut i32) {
    *x += 1;
}

// Multiple elements with different lifetimes. In this case, it
// would be fine for both to have the same lifetime `'a`, but
// in more complex cases, different lifetimes may be required.
fn print_multi<'a, 'b>(x: &'a i32, y: &'b i32) {
    println!("`print_multi`: x is {}, y is {}", x, y);
}

// Returning references that have been passed in is acceptable.
// However, the correct lifetime must be returned.
fn pass_x<'a, 'b>(x: &'a i32, _: &'b i32) -> &'a i32 { x }

//fn invalid_output<'a>() -> &'a String { &String::from("foo") }
// The above is invalid: `'a` must live longer than the function.
// Here, `&String::from("foo")` would create a `String`, followed by a
// reference. Then the data is dropped upon exiting the scope, leaving
// a reference to invalid data to be returned.

fn main() {
    let x = 7;
    let y = 9;
    
    print_one(&x);
    print_multi(&x, &y);
    
    let z = pass_x(&x, &y);
    print_one(z);

    let mut t = 3;
    add_one(&mut t);
    print_one(&t);
}
```

### Methods

```rust
struct Owner(i32);

impl Owner {
    // Annotate lifetimes as in a standalone function.
    fn add_one<'a>(&'a mut self) { self.0 += 1; }
    fn print<'a>(&'a self) {
        println!("`print`: {}", self.0);
    }
}

fn main() {
    let mut owner = Owner(18);

    owner.add_one();
    owner.print();
}
```

### Structs

```rust
// A type `Borrowed` which houses a reference to an
// `i32`. The reference to `i32` must outlive `Borrowed`.
#[derive(Debug)]
struct Borrowed<'a>(&'a i32);

// Similarly, both references here must outlive this structure.
#[derive(Debug)]
struct NamedBorrowed<'a> {
    x: &'a i32,
    y: &'a i32,
}

// An enum which is either an `i32` or a reference to one.
#[derive(Debug)]
enum Either<'a> {
    Num(i32),
    Ref(&'a i32),
}

fn main() {
    let x = 18;
    let y = 15;

    let single = Borrowed(&x);
    let double = NamedBorrowed { x: &x, y: &y };
    let reference = Either::Ref(&x);
    let number    = Either::Num(y);

    println!("x is borrowed in {:?}", single);
    println!("x and y are borrowed in {:?}", double);
    println!("x is borrowed in {:?}", reference);
    println!("y is *not* borrowed in {:?}", number);
}
```

### Traits

```rust
// A struct with annotation of lifetimes.
#[derive(Debug)]
struct Borrowed<'a> {
    x: &'a i32,
}

// Annotate lifetimes to impl.
impl<'a> Default for Borrowed<'a> {
    fn default() -> Self {
        Self {
            x: &10,
        }
    }
}

fn main() {
    let b: Borrowed = Default::default();
    println!("b is {:?}", b);
}
```

### Bounds
Just like generic types can be bounded, lifetimes (themselves generic) use bounds as well. The `:` character has a slightly different meaning here, but `+` is the same. Note how the following read:

`T: 'a`: All references in `T` must outlive lifetime `'a`.
`T: Trait + 'a`: Type `T` must implement trait `Trait` and all references in `T` must outlive `'a`.

The example below shows the above syntax in action used after keyword `where`:

```rust
use std::fmt::Debug; // Trait to bound with.

#[derive(Debug)]
struct Ref<'a, T: 'a>(&'a T);
// `Ref` contains a reference to a generic type `T` that has
// an unknown lifetime `'a`. `T` is bounded such that any
// *references* in `T` must outlive `'a`. Additionally, the lifetime
// of `Ref` may not exceed `'a`.

// A generic function which prints using the `Debug` trait.
fn print<T>(t: T) where
    T: Debug {
    println!("`print`: t is {:?}", t);
}

// Here a reference to `T` is taken where `T` implements
// `Debug` and all *references* in `T` outlive `'a`. In
// addition, `'a` must outlive the function.
fn print_ref<'a, T>(t: &'a T) where
    T: Debug + 'a {
    println!("`print_ref`: t is {:?}", t);
}

fn main() {
    let x = 7;
    let ref_x = Ref(&x);

    print_ref(&ref_x);
    print(ref_x);
}
```

### Coercion
A longer lifetime can be coerced into a shorter one so that it works inside a scope it normally wouldn't work in. This comes in the form of inferred coercion by the Rust compiler, and also in the form of declaring a lifetime difference:

```rust
// Here, Rust infers a lifetime that is as short as possible.
// The two references are then coerced to that lifetime.
fn multiply<'a>(first: &'a i32, second: &'a i32) -> i32 {
    first * second
}

// `<'a: 'b, 'b>` reads as lifetime `'a` is at least as long as `'b`.
// Here, we take in an `&'a i32` and return a `&'b i32` as a result of coercion.
fn choose_first<'a: 'b, 'b>(first: &'a i32, _: &'b i32) -> &'b i32 {
    first
}

fn main() {
    let first = 2; // Longer lifetime
    
    {
        let second = 3; // Shorter lifetime
        
        println!("The product is {}", multiply(&first, &second));
        println!("{} is the first", choose_first(&first, &second));
    };
}
```

### Static
Rust has a few reserved lifetime names. One of those is `'static`. You might encounter it in two situations:

```rust
// A reference with 'static lifetime:
let s: &'static str = "hello world";

// 'static as part of a trait bound:
fn generic<T>(x: T) where T: 'static {}
```

## Closures
Closures are functions that can capture the enclosing environment.

Characteristics of closures:

* using `||` instead of `()` around input variables.
* optional body delimination (`{}`) for a single expression (mandatory otherwise).
* the ability to capture the outer environment variables.

```rust
fn main() {
    // Increment via closures and functions.
    fn function(i: i32) -> i32 { i + 1 }

    // Closures are anonymous, here we are binding them to references
    // Annotation is identical to function annotation but is optional
    // as are the `{}` wrapping the body. These nameless functions
    // are assigned to appropriately named variables.
    let closure_annotated = |i: i32| -> i32 { i + 1 };
    let closure_inferred  = |i     |          i + 1  ;

    let i = 1;
    // Call the function and closures.
    println!("function: {}", function(i));
    println!("closure_annotated: {}", closure_annotated(i));
    println!("closure_inferred: {}", closure_inferred(i));
    // Once closure's type has been inferred, it cannot be inferred again with another type.
    //println!("cannot reuse closure_inferred with another type: {}", closure_inferred(42i64));
    // TODO: uncomment the line above and see the compiler error.

    // A closure taking no arguments which returns an `i32`.
    // The return type is inferred.
    let one = || 1;
    println!("closure returning one: {}", one());

}
```

### Capturing
Closures are inherently flexible and will do what the functionality requires to make the closure work without annotation. This allows capturing to flexibly adapt to the use case, sometimes moving and sometimes borrowing. Closures can capture variables:

* by reference: `&T`
* by mutable reference: `&mut T`
* by value: `T`
  
They preferentially capture variables by reference and only go lower when required.

```rust
fn main() {
    use std::mem;
    
    let color = String::from("green");

    // A closure to print `color` which immediately borrows (`&`) `color` and
    // stores the borrow and closure in the `print` variable. It will remain
    // borrowed until `print` is used the last time. 
    //
    // `println!` only requires arguments by immutable reference so it doesn't
    // impose anything more restrictive.
    let print = || println!("`color`: {}", color);

    // Call the closure using the borrow.
    print();

    // `color` can be borrowed immutably again, because the closure only holds
    // an immutable reference to `color`. 
    let _reborrow = &color;
    print();

    // A move or reborrow is allowed after the final use of `print`
    let _color_moved = color;


    let mut count = 0;
    // A closure to increment `count` could take either `&mut count` or `count`
    // but `&mut count` is less restrictive so it takes that. Immediately
    // borrows `count`.
    //
    // A `mut` is required on `inc` because a `&mut` is stored inside. Thus,
    // calling the closure mutates the closure which requires a `mut`.
    let mut inc = || {
        count += 1;
        println!("`count`: {}", count);
    };

    // Call the closure using a mutable borrow.
    inc();

    // The closure still mutably borrows `count` because it is called later.
    // An attempt to reborrow will lead to an error.
    // let _reborrow = &count; 
    // ^ TODO: try uncommenting this line.
    inc();

    // The closure no longer needs to borrow `&mut count`. Therefore, it is
    // possible to reborrow without an error
    let _count_reborrowed = &mut count; 

    
    // A non-copy type.
    let movable = Box::new(3);

    // `mem::drop` requires `T` so this must take by value. A copy type
    // would copy into the closure leaving the original untouched.
    // A non-copy must move and so `movable` immediately moves into
    // the closure.
    let consume = || {
        println!("`movable`: {:?}", movable);
        mem::drop(movable);
    };

    // `consume` consumes the variable so this can only be called once.
    consume();
    // consume();
    // ^ TODO: Try uncommenting this line.
}
```

### As input parameters
Input parameters to functions must be annotated with one of these `traits`:

* Fn: the closure uses the captured value by reference (&T)
* FnMut: the closure uses the captured value by mutable reference (&mut T)
* FnOnce: the closure uses the captured value by value (T)

```rust
// A function which takes a closure as an argument and calls it.
// <F> denotes that F is a "Generic type parameter"
fn apply<F>(f: F) where
    // The closure takes no input and returns nothing.
    F: FnOnce() {
    // ^ TODO: Try changing this to `Fn` or `FnMut`.

    f();
}

// A function which takes a closure and returns an `i32`.
fn apply_to_3<F>(f: F) -> i32 where
    // The closure takes an `i32` and returns an `i32`.
    F: Fn(i32) -> i32 {

    f(3)
}

fn main() {
    use std::mem;

    let greeting = "hello";
    // A non-copy type.
    // `to_owned` creates owned data from borrowed one
    let mut farewell = "goodbye".to_owned();

    // Capture 2 variables: `greeting` by reference and
    // `farewell` by value.
    let diary = || {
        // `greeting` is by reference: requires `Fn`.
        println!("I said {}.", greeting);

        // Mutation forces `farewell` to be captured by
        // mutable reference. Now requires `FnMut`.
        farewell.push_str("!!!");
        println!("Then I screamed {}.", farewell);
        println!("Now I can sleep. zzzzz");

        // Manually calling drop forces `farewell` to
        // be captured by value. Now requires `FnOnce`.
        mem::drop(farewell);
    };

    // Call the function which applies the closure.
    apply(diary);

    // `double` satisfies `apply_to_3`'s trait bound
    let double = |x| 2 * x;

    println!("3 doubled: {}", apply_to_3(double));
}
```

### As output parameters
The valid traits for returning a closure are:

* `Fn`
* `FnMut`
* `FnOnce`

```rust
fn create_fn() -> impl Fn() {
    let text = "Fn".to_owned();

    move || println!("This is a: {}", text)
}

fn create_fnmut() -> impl FnMut() {
    let text = "FnMut".to_owned();

    move || println!("This is a: {}", text)
}

fn create_fnonce() -> impl FnOnce() {
    let text = "FnOnce".to_owned();

    move || println!("This is a: {}", text)
}

fn main() {
    let fn_plain = create_fn();
    let mut fn_mut = create_fnmut();
    let fn_once = create_fnonce();

    fn_plain();
    fn_mut();
    fn_once();
}
```

## Higher Order Functions

```rust
fn next(&mut self) -> Option<Self::Item>
```

Advances the iterator and returns the next value.

```rust
let a = [1, 2, 3];

let mut iter = a.iter();

// A call to next() returns the next value...
assert_eq!(Some(&1), iter.next());
assert_eq!(Some(&2), iter.next());
assert_eq!(Some(&3), iter.next());

// ... and then None once it's over.
assert_eq!(None, iter.next());

// More calls may or may not return `None`. Here, they always will.
assert_eq!(None, iter.next());
assert_eq!(None, iter.next());
```

```rust
fn collect<B>(self) -> B
where
    B: FromIterator<Self::Item>,
    Self: Sized,
```

Transforms an iterator into a collection.

```rust
let a = [1, 2, 3];

let doubled: Vec<i32> = a.iter()
                         .map(|&x| x * 2)
                         .collect();

assert_eq!(vec![2, 4, 6], doubled);
```

```rust
fn take(self, n: usize) -> Take<Self>
where
    Self: Sized,
```

Creates an iterator that yields the first n elements, or fewer if the underlying iterator ends sooner.

```rust
let mut iter = (0..).take(3);

assert_eq!(iter.next(), Some(0));
assert_eq!(iter.next(), Some(1));
assert_eq!(iter.next(), Some(2));
assert_eq!(iter.next(), None);
```

```rust
fn cycle(self) -> Cycle<Self>
where
    Self: Sized + Clone,
```

Repeats an iterator endlessly.

```rust
let a = [1, 2, 3];

let mut it = a.iter().cycle();

assert_eq!(it.next(), Some(&1));
assert_eq!(it.next(), Some(&2));
assert_eq!(it.next(), Some(&3));
assert_eq!(it.next(), Some(&1));
assert_eq!(it.next(), Some(&2));
assert_eq!(it.next(), Some(&3));
assert_eq!(it.next(), Some(&1));
```

```rust
fn for_each<F>(self, f: F)
where
    Self: Sized,
    F: FnMut(Self::Item),
```

Calls a closure on each element of an iterator.

```rust
(0..5).flat_map(|x| x * 100 .. x * 110)
      .enumerate()
      .filter(|&(i, x)| (i + x) % 3 == 0)
      .for_each(|(i, x)| println!("{i}:{x}"));
```

```rust
fn filter<P>(self, predicate: P) -> Filter<Self, P>
where
    Self: Sized,
    P: FnMut(&Self::Item) -> bool,
```

Creates an iterator which uses a closure to determine if an element should be yielded.

```rust
let a = [0i32, 1, 2];

let mut iter = a.iter().filter(|x| x.is_positive());

assert_eq!(iter.next(), Some(&1));
assert_eq!(iter.next(), Some(&2));
assert_eq!(iter.next(), None);
```

```rust
fn map<B, F>(self, f: F) -> Map<Self, F>
where
    Self: Sized,
    F: FnMut(Self::Item) -> B,
```

Takes a closure and creates an iterator which calls that closure on each element.

```rust
let a = [1, 2, 3];

let mut iter = a.iter().map(|x| 2 * x);

assert_eq!(iter.next(), Some(2));
assert_eq!(iter.next(), Some(4));
assert_eq!(iter.next(), Some(6));
assert_eq!(iter.next(), None);
```

```rust
// don't do this:
(0..5).map(|x| println!("{x}"));

// it won't even execute, as it is lazy. Rust will warn you about this.

// Instead, use for:
for x in 0..5 {
    println!("{x}");
}
```

```rust
fn any<F>(&mut self, f: F) -> bool
where
    Self: Sized,
    F: FnMut(Self::Item) -> bool,
```

Tests if any element of the iterator matches a predicate.

any() is short-circuiting; in other words, it will stop processing as soon as it finds a true, given that no matter what else happens, the result will also be true.

```rust
let a = [1, 2, 3];

let mut iter = a.iter();

assert!(iter.any(|&x| x != 2));

// we can still use `iter`, as there are more elements.
assert_eq!(iter.next(), Some(&2));
```

```rust
fn fold<B, F>(self, init: B, f: F) -> B
where
    Self: Sized,
    F: FnMut(B, Self::Item) -> B,
```

Folds every element into an accumulator by applying an operation, returning the final result.

fold() takes two arguments: an initial value, and a closure with two arguments: an ‘accumulator', and an element. The closure returns the value that the accumulator should have for the next iteration.

```rust
let a = [1, 2, 3];

// the sum of all of the elements of the array
let sum = a.iter().fold(0, |acc, x| acc + x);

assert_eq!(sum, 6);
```

```rust
fn reduce<F>(self, f: F) -> Option<Self::Item>
where
    Self: Sized,
    F: FnMut(Self::Item, Self::Item) -> Self::Item,
```

Reduces the elements to a single one, by repeatedly applying a reducing operation.

For iterators with at least one element, this is the same as `fold()` with the first element of the iterator as the initial accumulator value, folding every subsequent element into it.

```rust
let reduced: i32 = (1..10).reduce(|acc, e| acc + e).unwrap();
assert_eq!(reduced, 45);

// Which is equivalent to doing it with `fold`:
let folded: i32 = (1..10).fold(0, |acc, e| acc + e);
assert_eq!(reduced, folded);
```

```rust
fn zip<U>(self, other: U) -> Zip<Self, <U as IntoIterator>::IntoIter>
where
    Self: Sized,
    U: IntoIterator,
```

'Zips up' two iterators into a single iterator of pairs.

```rust
let a1 = [1, 2, 3];
let a2 = [4, 5, 6];

let mut iter = a1.iter().zip(a2.iter());

assert_eq!(iter.next(), Some((&1, &4)));
assert_eq!(iter.next(), Some((&2, &5)));
assert_eq!(iter.next(), Some((&3, &6)));
assert_eq!(iter.next(), None);
```

```rust
fn count(self) -> usize
where
    Self: Sized,
```

Consumes the iterator, counting the number of iterations and returning it.

```rust
let a = [1, 2, 3];
assert_eq!(a.iter().count(), 3);

let a = [1, 2, 3, 4, 5];
assert_eq!(a.iter().count(), 5);
```

```rust
fn last(self) -> Option<Self::Item>
where
    Self: Sized,
```

Consumes the iterator, returning the last element.

```rust
let a = [1, 2, 3];
assert_eq!(a.iter().last(), Some(&3));

let a = [1, 2, 3, 4, 5];
assert_eq!(a.iter().last(), Some(&5));
```

```rust
fn chain<U>(self, other: U) -> Chain<Self, <U as IntoIterator>::IntoIter>
where
    Self: Sized,
    U: IntoIterator<Item = Self::Item>,
```

Takes two iterators and creates a new iterator over both in sequence.

```rust
let a1 = [1, 2, 3];
let a2 = [4, 5, 6];

let mut iter = a1.iter().chain(a2.iter());

assert_eq!(iter.next(), Some(&1));
assert_eq!(iter.next(), Some(&2));
assert_eq!(iter.next(), Some(&3));
assert_eq!(iter.next(), Some(&4));
assert_eq!(iter.next(), Some(&5));
assert_eq!(iter.next(), Some(&6));
assert_eq!(iter.next(), None);
```

```rust
fn enumerate(self) -> Enumerate<Self>
where
    Self: Sized,
```

Creates an iterator which gives the current iteration count as well as the next value.

```rust
let a = ['a', 'b', 'c'];

let mut iter = a.iter().enumerate();

assert_eq!(iter.next(), Some((0, &'a')));
assert_eq!(iter.next(), Some((1, &'b')));
assert_eq!(iter.next(), Some((2, &'c')));
assert_eq!(iter.next(), None);
```

```rust
fn take_while<P>(self, predicate: P) -> TakeWhile<Self, P>
where
    Self: Sized,
    P: FnMut(&Self::Item) -> bool,
```

Creates an iterator that yields elements based on a predicate.

```rust
let a = [-1i32, 0, 1];

let mut iter = a.iter().take_while(|x| x.is_negative());

assert_eq!(iter.next(), Some(&-1));
assert_eq!(iter.next(), None);
```

```rust
fn flatten(self) -> Flatten<Self>
where
    Self: Sized,
    Self::Item: IntoIterator,
```

Creates an iterator that flattens nested structure.

```rust
let data = vec![vec![1, 2, 3, 4], vec![5, 6]];
let flattened = data.into_iter().flatten().collect::<Vec<u8>>();
assert_eq!(flattened, &[1, 2, 3, 4, 5, 6]);
```

```rust
let words = ["alpha", "beta", "gamma"];

// chars() returns an iterator
let merged: String = words.iter()
                          .map(|s| s.chars())
                          .flatten()
                          .collect();
assert_eq!(merged, "alphabetagamma");
```

```rust
fn collect<B>(self) -> B
where
    B: FromIterator<Self::Item>,
    Self: Sized,
```

Transforms an iterator into a collection.

```rust
let a = [1, 2, 3];

let doubled: Vec<i32> = a.iter()
                         .map(|&x| x * 2)
                         .collect();

assert_eq!(vec![2, 4, 6], doubled);
```

```rust
fn partition<B, F>(self, f: F) -> (B, B)
where
    Self: Sized,
    B: Default + Extend<Self::Item>,
    F: FnMut(&Self::Item) -> bool,
```

Consumes an iterator, creating two collections from it.

```rust
let a = [1, 2, 3];

let (even, odd): (Vec<_>, Vec<_>) = a
    .into_iter()
    .partition(|n| n % 2 == 0);

assert_eq!(even, vec![2]);
assert_eq!(odd, vec![1, 3]);
```

```rust
fn rev(self) -> Rev<Self>
where
    Self: Sized + DoubleEndedIterator,
```

Reverses an iterator's direction.

```rust
let a = [1, 2, 3];

let mut iter = a.iter().rev();

assert_eq!(iter.next(), Some(&3));
assert_eq!(iter.next(), Some(&2));
assert_eq!(iter.next(), Some(&1));

assert_eq!(iter.next(), None);
```

```rust
fn min(self) -> Option<Self::Item>
where
    Self: Sized,
    Self::Item: Ord,
```

Returns the minimum element of an iterator.

```rust
let a = [1, 2, 3];
let b: Vec<u32> = Vec::new();

assert_eq!(a.iter().min(), Some(&1));
assert_eq!(b.iter().min(), None);
```

```rust
fn max(self) -> Option<Self::Item>
where
    Self: Sized,
    Self::Item: Ord,
```

Returns the maximum element of an iterator.

```rust
let a = [1, 2, 3];
let b: Vec<u32> = Vec::new();

assert_eq!(a.iter().max(), Some(&3));
assert_eq!(b.iter().max(), None);
```

```rust
fn sum<S>(self) -> S
where
    Self: Sized,
    S: Sum<Self::Item>,
```

Sums the elements of an iterator.

```rust
let a = [1, 2, 3];
let sum: i32 = a.iter().sum();

assert_eq!(sum, 6);
```

```rust
fn product<P>(self) -> P
where
    Self: Sized,
    P: Product<Self::Item>,
```

Iterates over the entire iterator, multiplying all the elements

```rust
fn factorial(n: u32) -> u32 {
    (1..=n).product()
}
assert_eq!(factorial(0), 1);
assert_eq!(factorial(1), 1);
assert_eq!(factorial(5), 120);
```

```rust
fn step_by(self, step: usize) -> StepBy<Self>
where
    Self: Sized,
```

Creates an iterator starting at the same point, but stepping by the given amount at each iteration.

```rust
let a = [0, 1, 2, 3, 4, 5];
let mut iter = a.iter().step_by(2);

assert_eq!(iter.next(), Some(&0));
assert_eq!(iter.next(), Some(&2));
assert_eq!(iter.next(), Some(&4));
assert_eq!(iter.next(), None);
```

### [Complete List of Higher Order Functions](https://doc.rust-lang.org/stable/std/iter/trait.Iterator.html#provided-methods)

## Diverging Functions
Diverging functions never return. They are marked using `!`, which is an empty type.

```rust
fn foo() -> ! {
    panic!("This call never returns.");
}
```

As opposed to all the other types, this one cannot be instantiated, because the set of all possible values this type can have is empty. Note that, it is different from the `()` type, which has exactly one possible value.

```rust
fn main() {
    fn sum_odd_numbers(up_to: u32) -> u32 {
        let mut acc = 0;
        for i in 0..up_to {
            // Notice that the return type of this match expression must be u32
            // because of the type of the "addition" variable.
            let addition: u32 = match i%2 == 1 {
                // The "i" variable is of type u32, which is perfectly fine.
                true => i,
                // On the other hand, the "continue" expression does not return
                // u32, but it is still fine, because it never returns and therefore
                // does not violate the type requirements of the match expression.
                false => continue,
            };
            acc += addition;
        }
        acc
    }
    println!("Sum of odd numbers up to 9 (excluding): {}", sum_odd_numbers(9));
}
```


## Paralellism
### Threads
Rust provides a mechanism for spawning native OS threads via the `spawn` function, the argument of this function is a moving closure.

```rust
use std::thread;

const NTHREADS: u32 = 10;

// This is the `main` thread
fn main() {
    // Make a vector to hold the children which are spawned.
    let mut children = vec![];

    for i in 0..NTHREADS {
        // Spin up another thread
        children.push(thread::spawn(move || {
            println!("this is thread number {}", i);
        }));
    }

    for child in children {
        // Wait for the thread to finish. Returns a result.
        let _ = child.join();
    }
}
```

### Channels
Rust provides asynchronous `channels` for communication between threads. Channels allow a unidirectional flow of information between two end-points: the `Sender` and the `Receiver`.

```rust
use std::sync::mpsc::{Sender, Receiver};
use std::sync::mpsc;
use std::thread;

static NTHREADS: i32 = 3;

fn main() {
    // Channels have two endpoints: the `Sender<T>` and the `Receiver<T>`,
    // where `T` is the type of the message to be transferred
    // (type annotation is superfluous)
    let (tx, rx): (Sender<i32>, Receiver<i32>) = mpsc::channel();
    let mut children = Vec::new();

    for id in 0..NTHREADS {
        // The sender endpoint can be copied
        let thread_tx = tx.clone();

        // Each thread will send its id via the channel
        let child = thread::spawn(move || {
            // The thread takes ownership over `thread_tx`
            // Each thread queues a message in the channel
            thread_tx.send(id).unwrap();

            // Sending is a non-blocking operation, the thread will continue
            // immediately after sending its message
            println!("thread {} finished", id);
        });

        children.push(child);
    }

    // Here, all the messages are collected
    let mut ids = Vec::with_capacity(NTHREADS as usize);
    for _ in 0..NTHREADS {
        // The `recv` method picks a message from the channel
        // `recv` will block the current thread if there are no messages available
        ids.push(rx.recv());
    }
    
    // Wait for the threads to complete any remaining work
    for child in children {
        child.join().expect("oops! the child thread panicked");
    }

    // Show the order in which the messages were sent
    println!("{:?}", ids);
}
```

### Parallel Iterators
[Higher Order Functions can be used](#higher-order-functions)

```rust
fn par_iter(&'data self) -> Self::Iter
```

Converts `self` into a parallel iterator.

```rust
use rayon::prelude::*;

let v: Vec<_> = (0..100).collect();
assert_eq!(v.par_iter().sum::<i32>(), 100 * 99 / 2);

// `v.par_iter()` is shorthand for `(&v).into_par_iter()`,
// producing the exact same references.
assert!(v.par_iter().zip(&v)
         .all(|(a, b)| std::ptr::eq(a, b)));
```

```rust
fn par_iter_mut(&'data mut self) -> Self::Iter
```

Creates the parallel iterator from `self`.

```rust
use rayon::prelude::*;

let mut v = vec![0usize; 5];
v.par_iter_mut().enumerate().for_each(|(i, x)| *x = i);
assert_eq!(v, [0, 1, 2, 3, 4]);
```

```rust
fn into_par_iter(self) -> Self::Iter
```

Converts `self` into a parallel iterator.

```rust
use rayon::prelude::*;

println!("counting in parallel:");
(0..100).into_par_iter()
    .for_each(|i| println!("{}", i));
```


### [Scope](https://docs.rs/rayon/latest/rayon/fn.scope.html)


### Lock
#### [RwLock](https://doc.rust-lang.org/std/sync/struct.RwLock.html)
A reader-writer lock

This type of lock allows a number of readers or at most one writer at any point in time

```rust
use std::sync::RwLock;

let lock = RwLock::new(5);

// many reader locks can be held at once
{
    let r1 = lock.read().unwrap();
    let r2 = lock.read().unwrap();
    assert_eq!(*r1, 5);
    assert_eq!(*r2, 5);
} // read locks are dropped at this point

// only one write lock may be held, however
{
    let mut w = lock.write().unwrap();
    *w += 1;
    assert_eq!(*w, 6);
} // write lock is dropped here
```

#### [Mutex](https://doc.rust-lang.org/std/sync/struct.Mutex.html)
A mutual exclusion primitive useful for protecting shared data

This mutex will block threads waiting for the lock to become available

```rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::sync::mpsc::channel;

const N: usize = 10;

// Spawn a few threads to increment a shared variable (non-atomically), and
// let the main thread know once all increments are done.
//
// Here we're using an Arc to share memory among threads, and the data inside
// the Arc is protected with a mutex.
let data = Arc::new(Mutex::new(0));

let (tx, rx) = channel();
for _ in 0..N {
    let (data, tx) = (Arc::clone(&data), tx.clone());
    thread::spawn(move || {
        // The shared state can only be accessed once the lock is held.
        // Our non-atomic increment is safe because we're the only thread
        // which can access the shared state when the lock is held.
        //
        // We unwrap() the return value to assert that we are not expecting
        // threads to ever fail while holding the lock.
        let mut data = data.lock().unwrap();
        *data += 1;
        if *data == N {
            tx.send(()).unwrap();
        }
        // the lock is unlocked here when `data` goes out of scope.
    });
}

rx.recv().unwrap();
```