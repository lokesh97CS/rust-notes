
# Rust Lecture one: short notes
1.Rust uses the u8 type for byte values. For example, reading data from a binary file or socket yields a stream of u8 values.

2.The usize and isize types are analogous to size_t and ptrdiff_t in C and C++.Their precision matches the size of the address space on the target machine: they are 32 bits long on 32-bit architectures, and 64 bits long on 64-bit architectures.

3.Rust requires array indices to be usize values. Values representing the sizes of arrays or vectors or counts of the number of elements in some data structure also generally have the usize type

4.Integer literals in Rust can take a suffix indicating their type: 42u8 is a u8 value

Unlike C and C++, Rust treats characters as distinct from the numeric types: a char is not a u8, nor is it a u32 (though it is 32 bits long). We describe Rust’s char type in “Characters”.

Although numeric types and the char type are distinct, Rust does provide byte literals, character-like literals for u8 values: b'X' represents the ASCII code for the character X, as a u8 value. For example, since the ASCII code for A is 65, the literals b'A' and 65u8 are exactly equivalent. Only ASCII characters may appear in byte literals.
```
b’\’’ → ascii value of ‘ is 39u8

few characters are represented with backslash

assert_eq!( 10_i8 as u16, 10_u16); // in range , type cast

assert_eq!( -1_i16 as i32, -1_i32); // sign-extended 

assert_eq!(65535_u16 as i32, 65535_i32); // zero-extended

assert_eq!( -1_i8 as u8, 255_u8); assert_eq!( 255_u8 as i8, -1_i8);
```

// Conversions that are out of range for the destination// produce values that are equivalent to the original modulo 2^N, // where N is the width of the destination in bits. This// is sometimes called "truncation."assert_eq!( 1000_**i16 as u8**, 232_**u8**); //1000%256

println!("{}", (-4_**i32**).abs()); println!("{}", **i32**::abs(-4));

Note that method calls have a higher precedence than unary prefix operators, so be careful when applying methods to negated values.
```
let mut i: i32 = 1;
loop {


      // panic: multiplication overflowed (in any build)
// i =i*10; // will be infinte loop .when to stop the loop.\, how to stop the loop
      i = i.checked_mul(10).expect("multiplication overflowed");
  }
```

Checked operations return an Option of the result: Some(v) if the mathematically correct result can be represented as a value of that type, or None if it cannot. 
The sum of 10 and 20 can be represented as a u8.
```
assert_eq!(10_u8.checked_add(20), Some(30));
assert_eq!(100_u8.checked_add(200), None);
```

## Checked, Wrapping, Saturating, and Overflowing Arithmetic When an integer arithmetic operation overflows


// Operations on signed types may wrap to negative values.
```
assert_eq!(500_i16.wrapping_mul(500), -12144);
```

// In bitwise shift operations, the shift distance  is wrapped to fall within the size of the value.
// So a shift of 17 bits in a 16-bit type is a shift 1
```
assert_eq!(5_i16.wrapping_shl(17), 10);
```
//saturating
```
assert_eq!(32760_i16.saturating_add(10), 32767);
assert_eq!((-32760_i16).saturating_sub(10), -32768);
```
There are no saturating division, remainder, or bitwise shift methods
// overflow occured or not checking 
```
assert_eq!(255_u8.overflowing_sub(2), (253, false));
```

overflowing_shl and overflowing_shr deviate from the pattern a bit: they return true foroverflowed only if the shift distance was as large or larger than the bit width of the type itself. The actual shift applied is the requested shift modulo the bit width of the type:

// A shift of 17 bits is too large for `u16`, and 17 modulo 16 is 1. 
```
assert_eq!(5_u16.overflowing_shl(17), (10, true));
```
Multiplication mul Division div Remainder rem Negation neg Absolute value abs
Exponentiation pow Bitwise left shift shl
```
128_u8.saturating_mul(3) == 255

64_u16.wrapping_div(8) == 8

(-32768_i16).wrapping_rem(-1) == 0

(-128_i8).checked_neg() == None

(-32768_i16).wrapping_abs() == -3276


3_u8.checked_pow(4) == Some(81)

10_u32.wrapping_shl(34) == 40

Bitwise right shr
 40_u64.wrapping_shr(66) == 10 shift

assert_eq!(false as i32, 0);
//However, as won’t convert in the other direction, from numeric types to bool. Instead, you must write out an explicit comparison like x != 0.

```
# Tuples
tuples allow only constants as indices, like t.4. You can’t write t.i or t[i] to get the ith element.
if you like, you may include a comma after a tuple’s last element: the types (&str, i32,) and (&str, i32) are equivalent, as are the expressions ("Brazil", 1985,) and ("Brazil", 1985).
## three pointer types here: references, boxes, and unsafe pointers.
# References
A value of type &String (pronounced “ref String”) is a reference to a String value, a &i32 is a reference to an i32, and so on.<br>

It’s easiest to get started by thinking of references as Rust’s basic pointer type. At run time, a reference to an i32 is a single machine word holding the address of the i32, which may be on the stack or in the heap.<br>
The expression &x produces a reference to x; in Rust terminology, we say that it borrows a reference to x. <br>
Given a reference r, the expression *r refers to the value r points to. These are very much like the & and * operators in C and C++. And like a C pointer, a reference does not automatically free any resources when it goes out of scope.<br>
Rust references are never null: there is simply no way to produce a null reference in safe Rust.<br>
Rust tracks the ownership and lifetimes of values, so mistakes like dangling pointers, double frees, and pointer invalidation are ruled out at compile time.<br>
# &T
An immutable, shared reference. You can have many shared references to a given value at a time, but they are read-only: modifying the value they point to is forbidden, as with const T* in C.
# &mut T 
A mutable, exclusive reference. You can read and modify the value it points to, as with a T* in C. But for as long as the reference exists, you may not have any other references of any kind to that value. In fact, the only way you may access the value at all is through the mutable reference.
# Boxes
The simplest way to allocate a value in the heap is to use Box::new:
```
let t = (12, "eggs");
let b = Box::new(t); // allocate a tuple in the heap
The type of t is (i32, &str), so the type of b is Box<(i32, &str)>.
```
# raw pointers
Rust also has the raw pointer types *mut T and *const T. Raw pointers really are just like pointers in C++. Using a raw pointer is unsafe, because Rust makes no effort to track what it points to. 
# Arrays, Vectors, and Slices
Rust has three types for representing a sequence of values in memory:<br>
The type [T; N] represents an array of N values, each of type T. An array’s size is a constant determined at compile time and is part of the type; you can’t append new elements or shrink an array. <br>
The type Vec<T>, called a vector of Ts, is a dynamically allocated, growable sequence of values of type T. A vector’s elements live on the heap, so <br>
you can resize vectors at will: push new elements onto them, append other vectors to them, delete elements, and so on.<br>

The types &[T] and &mut [T], called a shared slice of Ts and mutable slice of Ts, are references to a series of elements that are a part of some other value, like an array or vector.<br>
 A mutable slice &mut [T] lets you read and modify elements, but can’t be shared; a shared slice &[T] lets you share access among several readers, but doesn’t let you modify elements<br>
 



