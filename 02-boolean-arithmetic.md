# Boolean Arithmetic

## Overview

1. Computer System and ALU
2. Representing Numbers
3. Binary Arithmetic and Overflow
4. Signed Integers and Two's Complement
5. Hack ALU
6. Half Adder and Full Adder
7. 16-bit Adder
8. ALU Implementation

---

# 1. Computer System and ALU

A computer system consists of several important components:

* **CPU (Central Processing Unit):** Executes instructions and processes data.
* **Register:** Small, fast storage inside the CPU for temporarily holding data.
* **Memory (RAM):** Stores data and instructions needed while programs are running.
* **ALU (Arithmetic Logic Unit):** Performs arithmetic and logical operations.

The ALU computes a function on two **n-bit inputs** (`x`, `y`) and produces an **n-bit output**.

### ALU Functions

#### Arithmetic

Examples:

```text
x + y
x - y
x + 1
x - 1
```

#### Logical

Examples:

```text
x & y
x | y
x
!x
```

---

## Hardware vs. Software Trade-off

|                 | Hardware Implementation  | Software Implementation |
| --------------- | ------------------------ | ----------------------- |
| **Speed**       | Faster                   | Slower                  |
| **Cost**        | More expensive           | Less expensive          |
| **Complexity**  | More hardware complexity | Simpler hardware        |
| **Flexibility** | Less flexible            | More flexible           |

A function implemented directly in hardware can be faster, while implementing the same function in software generally provides greater flexibility.

---

# 2. Representing Numbers

## Positional Representation

A numeral's value depends on the position of each digit.

* **Digit:** A symbol used to represent a number.
* **Base:** The number of available digits.
* **Numeral:** An ordered sequence of digits.
* **Position:** Counted from right to left, starting at `0`.
* **Value:** `digit × base^position`.

For example, in decimal:

```text
6507

= 6 × 10³
+ 5 × 10²
+ 0 × 10¹
+ 7 × 10⁰
```

---

# 3. Binary ↔ Decimal Conversion

## Binary → Decimal

Multiply each binary digit by its corresponding power of 2 and add the results.

### Example

```text
1011₂

= 1 × 2³
+ 0 × 2²
+ 1 × 2¹
+ 1 × 2⁰

= 8 + 0 + 2 + 1
= 11₁₀
```

Therefore:

```text
1011₂ = 11₁₀
```

---

## Decimal → Binary

Repeatedly divide the decimal number by 2 and record the remainders.

Read the remainders from **bottom to top**.

### Example: 13₁₀ → ?₂

```text
13 ÷ 2 = 6  remainder 1
 6 ÷ 2 = 3  remainder 0
 3 ÷ 2 = 1  remainder 1
 1 ÷ 2 = 0  remainder 1
```

Read from bottom to top:

```text
1101₂
```

Therefore:

```text
13₁₀ = 1101₂
```

---

# 4. Binary Addition and Overflow

Computers use a **fixed number of bits** to represent values.

For example, a 4-bit unsigned integer can represent:

```text
0000 → 0
...
1111 → 15
```

Therefore, 4 bits can represent **16 different values**.

When the result requires more bits than are available, an **overflow** occurs.

### Example

Using 4 bits:

```text
1111
+0001
-----
10000
```

The result requires 5 bits, but only 4 bits are available.

Therefore, the stored result is:

```text
0000
```

The extra carry bit is discarded.

---

# 5. Signed Integers

Signed integers represent both positive and negative values.

Common integer sizes include:

* `short` → 16 bits
* `int` → 32 bits
* `long` → 64 bits

Efficient signed-integer arithmetic is essential for computer design.

---

## Unsigned Integers

With `n` bits, there are:

```text
2ⁿ
```

possible bit patterns.

The range is:

```text
0 ~ 2ⁿ - 1
```

For example, with 4 bits:

```text
2⁴ = 16 values

0 ~ 15
```

---

# 6. Two's Complement

**Two's complement** is a representation used to store signed integers using a fixed number of bits.

With `n` bits:

```text
2ⁿ possible values
```

The range is:

```text
-2ⁿ⁻¹ ~ 2ⁿ⁻¹ - 1
```

For 4 bits:

```text
-8 ~ +7
```

### Sign Bit

The most significant bit (MSB) indicates the sign:

* `MSB = 0` → non-negative
* `MSB = 1` → negative

---

## Representing Negative Numbers

For an `n`-bit number, `-x` is represented as:

```text
2ⁿ - x
```

### Example: 4-bit representation

```text
-1 → 2⁴ - 1 = 15 → 1111
-8 → 2⁴ - 8 = 8  → 1000
```

Therefore:

```text
1111 → -1
1000 → -8
```

---

## Decoding Two's Complement

If the MSB is `0`, interpret the value normally.

If the MSB is `1`, subtract `2ⁿ` from the unsigned value.

### Example

```text
1111₂ = 15

15 - 16 = -1
```

Therefore:

```text
1111₂ = -1
```

Similarly:

```text
1000₂ = 8

8 - 16 = -8
```

Therefore:

```text
1000₂ = -8
```

---

# 7. Signed Integer Addition

Two's complement allows signed integers to be added using ordinary binary addition.

The ALU does not need a separate addition mechanism for positive and negative numbers.

With `n` bits, the result is effectively computed **modulo `2ⁿ`** because only `n` bits can be stored.

Any carry beyond the `n`th bit is discarded.

### Example: 4-bit Addition

```text
  1110   (-2)
+ 0011   (+3)
------
 10001
```

Keep only the lowest 4 bits:

```text
0001
```

Therefore:

```text
-2 + 3 = 1
```

---

# 8. Hack ALU

The Hack ALU takes two 16-bit inputs and produces a 16-bit output.

### Inputs

```text
x[16]
y[16]
```

### Control Bits

Six control bits determine the operation:

```text
zx nx zy ny f no
```

### Outputs

```text
out[16]
zr
ng
```

* **`out`** → 16-bit result
* **`zr`** → `1` if `out == 0`, otherwise `0`
* **`ng`** → `1` if `out < 0`, otherwise `0`

The ALU can compute **18 predefined arithmetic and logical functions**.

---

## Hack ALU Control Logic

The six control bits determine the operation in stages:

1. `zx` → zero the `x` input
2. `nx` → negate `x`
3. `zy` → zero the `y` input
4. `ny` → negate `y`
5. `f` → choose between AND and ADD
6. `no` → negate the final output

Conceptually:

```text
x → zero? → negate?
y → zero? → negate?
             ↓
        AND / ADD
             ↓
       negate output?
             ↓
           out
```

---

## Important ALU Identities

The ALU can construct subtraction and increment operations using two's complement.

### Negation

To compute `-x`:

```text
-x = !x + 1
```

For example, using 4 bits:

```text
x  = 0100   (+4)
!x = 1011
!x + 1
   = 1100   (-4)
```

Therefore:

```text
-4 = 1100
```

---

### Increment

To compute `x + 1`:

```text
x + 1 = !(!x + 1) + 1
```

More directly, the ALU can construct `x + 1` through its zeroing and negation controls.

For example:

```text
x = 0101   (+5)

x + 1 = 0110   (+6)
```

---

### Subtraction

Subtraction is performed using two's complement:

```text
x - y = x + (-y)
      = x + !y + 1
```

### Example

```text
x = 0010   (+2)
y = 0101   (+5)

!y = 1010
!y + 1 = 1011   (-5)

  0010
+ 1011
------
  1101
```

`1101` represents `-3` in 4-bit two's complement.

Therefore:

```text
2 - 5 = -3
```

---

## Bitwise AND with All Ones

A bitwise AND with all ones preserves the original value:

```text
x & 1111...1111 = x
```

### Example

```text
  0101
& 1111
------
  0101
```

Therefore:

```text
x & 1111 = x
```

This is useful when constructing ALU operations.

---

# 9. Half Adder

A **Half Adder** adds two 1-bit inputs:

```text
a
b
```

It produces:

* `sum`
* `carry`

The equations are:

```text
sum   = a XOR b
carry = a AND b
```

A Half Adder does **not** have a carry-in from a previous bit.

### HDL Implementation

```text
CHIP HalfAdder {
    IN a, b;
    OUT sum, carry;

    PARTS:
    Xor(a=a, b=b, out=sum);
    And(a=a, b=b, out=carry);
}
```

---

# 10. Full Adder

A **Full Adder** adds three 1-bit inputs:

```text
a
b
c
```

where `c` is the carry-in from the previous bit.

It produces:

```text
sum
carry
```

The first (rightmost) bit starts with:

```text
carry-in = 0
```

### HDL Implementation

```text
CHIP FullAdder {
    IN a, b, c;
    OUT sum, carry;

    PARTS:
    HalfAdder(a=a, b=b, sum=sum1, carry=carry1);
    HalfAdder(a=sum1, b=c, sum=sum, carry=carry2);
    Or(a=carry1, b=carry2, out=carry);
}
```

### Key Idea

A Full Adder can be constructed from:

```text
2 Half Adders + 1 OR gate
```

---

# 11. 16-bit Adder

A 16-bit adder adds two 16-bit values.

It can be constructed by connecting **16 Full Adders**.

The carry from each bit is passed to the next bit.

```text
bit 0 → bit 1 → bit 2 → ... → bit 15
```

The carry generated by the most significant bit is discarded.

### HDL Implementation

```text
CHIP Add16 {
    IN a[16], b[16];
    OUT out[16];

    PARTS:
    FullAdder(a=a[0], b=b[0], c=false,
              sum=out[0], carry=c1);

    FullAdder(a=a[1], b=b[1], c=c1,
              sum=out[1], carry=c2);

    FullAdder(a=a[2], b=b[2], c=c2,
              sum=out[2], carry=c3);

    FullAdder(a=a[3], b=b[3], c=c3,
              sum=out[3], carry=c4);

    // ...

    FullAdder(a=a[15], b=b[15], c=c15,
              sum=out[15], carry=ignored);
}
```

> **16-bit Adder = 16 Full Adders connected in series.**

---

# 12. HDL Implementation Tips

When implementing larger chips in HDL, reuse existing chips whenever possible.

For example:

```text
Add16
```

can be used as a component inside another chip.

### Setting Multiple Bits

A range of bits can be assigned the same value.

```text
x[0..3] = false;
```

produces:

```text
0000
```

Similarly:

```text
x[4..7] = true;
```

produces:

```text
1111
```

This is useful when constructing 16-bit constants.

---

# 13. ALU Implementation Strategy

The Hack ALU can be constructed from smaller components.

### Required Operations

The implementation needs to support:

* Zeroing `x`
* Negating `x`
* Zeroing `y`
* Negating `y`
* Selecting between `AND` and `ADD`
* Negating the final output
* Detecting whether the output is zero
* Detecting whether the output is negative

### Recommended Implementation Order

Build the ALU in stages:

1. Implement the `x` preprocessing.
2. Implement the `y` preprocessing.
3. Select between `AND` and `ADD`.
4. Apply output negation.
5. Implement `zr`.
6. Implement `ng`.

> **First make `out` work, then add `zr` and `ng`.**

This makes debugging much easier because the main ALU computation can be verified before implementing the status flags.

---

# Key Takeaways

* The **ALU** performs arithmetic and logical operations on binary values.
* Binary numbers use **positional representation** based on powers of 2.
* Fixed-width arithmetic can produce **overflow**.
* **Two's complement** allows computers to represent negative integers efficiently.
* Two's complement allows positive and negative numbers to use the same binary addition hardware.
* A **Half Adder** adds two bits without a carry-in.
* A **Full Adder** adds two bits plus a carry-in.
* A **16-bit Adder** can be built by connecting 16 Full Adders.
* The **Hack ALU** uses six control bits to select among 18 predefined functions.
* Larger hardware components can be constructed hierarchically from smaller components.
