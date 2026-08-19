# Boolean Logic

## Overview

1. Elementary Logic Gates
2. Building a Chip
3. HDL Stub Files
4. Gate Interface and Implementation
5. HDL and Hardware Simulation
6. Multi-bit Buses
7. Multiplexors and Demultiplexors
8. 16-bit Variants
9. Multi-way Variants

---

# 1. Elementary Logic Gates

A **binary variable** has two possible states:

```text
0 / 1
```

With two binary variables, there are four possible combinations:

```text
00 / 01 / 10 / 11
```

Therefore, with **N binary variables**, the number of possible input combinations is:

$$
2^N
$$

## Number of Boolean Functions

A Boolean function maps each possible input combination to either `0` or `1`.

Since there are $2^N$ possible input combinations, and each input combination can independently produce two possible outputs, the total number of Boolean functions is:

$$
2^{2^N}
$$

### Why?

#### 1. Find the number of input combinations

Each binary variable has two possible values:

$$
\text{Input combinations} = 2^N
$$

#### 2. Assign an output to each input combination

For each input combination, there are two possible outputs:

```text
Input combination 1 → 0 or 1
Input combination 2 → 0 or 1
Input combination 3 → 0 or 1
...
Input combination 2^N → 0 or 1
```

Therefore, the number of possible Boolean functions is:

$$
2^{2^N}
$$

---

# 2. Building a Chip

The general process of building a hardware component can be summarized as:

1. Design the chip architecture
2. Specify the architecture in HDL
3. Test the chip using a hardware simulator
4. Optimize the design
5. Realize the optimized design in silicon

This process separates **hardware design**, **hardware description**, **verification**, and **physical implementation**.

---

# 3. HDL Stub Files

An **HDL stub file** is a pre-made skeleton that defines the basic interface of a chip while leaving its internal implementation empty.

For example:

```text
CHIP Xor {
    IN a, b;
    OUT out;

    PARTS:
}
```

### Components

* `CHIP Xor` → defines a chip named `Xor`
* `IN a, b;` → defines the input pins
* `OUT out;` → defines the output pin
* `PARTS:` → contains the internal chip implementation

The stub provides the **interface** that the implementation must satisfy.

---

# 4. Gate Interface and Implementation

## Gate Interface

The **interface** describes what a gate provides to the outside world.

It defines:

* Inputs
* Outputs
* Expected functionality

For example:

```text
AND

IN:  a, b
OUT: out
```

A user of the gate only needs to know how to interact with it, not how it is implemented internally.

## Gate Implementation

The **implementation** describes how the functionality is achieved internally.

For example, an XOR gate can be constructed using:

```text
NOT + AND + OR
```

One possible implementation is:

```text
CHIP Xor {
    IN a, b;
    OUT out;

    PARTS:
    Not(in=a, out=notA);
    Not(in=b, out=notB);
    And(a=a, b=notB, out=aAndNotB);
    And(a=notA, b=b, out=notAAndB);
    Or(a=aAndNotB, b=notAAndB, out=out);
}
```

The key idea is:

> **One interface, many possible implementations.**

This separation allows the internal implementation to change without changing how other chips interact with the gate.

---

# 5. HDL and Hardware Simulation

## HDL Observations

### 1. HDL is Declarative

HDL describes **what the hardware should be** rather than specifying a sequence of instructions to execute.

### 2. HDL Represents Hardware Structure

An HDL program can be viewed as a textual representation of a chip diagram.

For example, the following HDL describes how several gates are connected:

```text
Not(in=a, out=notA);
And(a=a, b=notB, out=aAndNotB);
```

### 3. Statement Order Is Not Execution Order

HDL statements do not represent sequential execution like statements in a typical programming language.

Instead, they describe the **structure and connections of hardware components**.

---

## Hardware Simulation

There are two main ways to test a chip:

* **Interactive simulation:** manually set input values and observe outputs.
* **Script-based simulation:** automatically test the chip using a predefined test script.

### Example: `Xor.hdl`

```text
CHIP Xor {
    IN a, b;
    OUT out;

    PARTS:
    Not(in=a, out=notA);
    Not(in=b, out=notB);
    And(a=a, b=notB, out=aAndNotB);
    And(a=notA, b=b, out=notAAndB);
    Or(a=aAndNotB, b=notAAndB, out=out);
}
```

### Example: `Xor.tst`

```text
load Xor.hdl,
output-file Xor.out,
compare-to Xor.cmp,
output-list a b out;

set a 0, set b 0, eval, output;
set a 0, set b 1, eval, output;
set a 1, set b 0, eval, output;
set a 1, set b 1, eval, output;
```

### Expected Output

| a | b | out |
| - | - | --: |
| 0 | 0 |   0 |
| 0 | 1 |   1 |
| 1 | 0 |   1 |
| 1 | 1 |   0 |

The truth table confirms that the implementation behaves as an XOR gate.

---

# 6. Multi-bit Buses

A **bus** is a group of bits treated as a single entity.

For example:

```text
10110110
```

is an **8-bit bus**.

* **MSB (Most Significant Bit):** the leftmost bit
* **LSB (Least Significant Bit):** the rightmost bit

Using buses allows hardware components to operate on multiple bits at once.

## Example: 16-bit Adder

A 16-bit adder can be constructed by connecting full adders in sequence.

```text
CHIP Adder {
    IN a[16], b[16];
    OUT out[16];

    PARTS:
    FullAdder(a=a[0],  b=b[0],  c=false,   sum=out[0],  carry=carry0);
    FullAdder(a=a[1],  b=b[1],  c=carry0,  sum=out[1],  carry=carry1);
    FullAdder(a=a[2],  b=b[2],  c=carry1,  sum=out[2],  carry=carry2);
    ...
    FullAdder(a=a[15], b=b[15], c=carry14, sum=out[15], carry=carry15);
}
```

The carry output from each full adder becomes the carry input of the next stage.

## Multi-bit Operations

A chip can also operate on multiple bits as a group.

For example, three 16-bit values can be added by composing two 16-bit adders:

```text
CHIP Adder3Way {
    IN a[16], b[16], c[16];
    OUT out[16];

    PARTS:
    Adder(a=a, b=b, out=ab);
    Adder(a=ab, b=c, out=out);
}
```

## Bit-wise Operations

A **bit-wise operation** applies the same operation independently to each bit.

For example:

```text
CHIP And4 {
    IN a[4], b[4];
    OUT out[4];

    PARTS:
    And(a=a[0], b=b[0], out=out[0]);
    And(a=a[1], b=b[1], out=out[1]);
    And(a=a[2], b=b[2], out=out[2]);
    And(a=a[3], b=b[3], out=out[3]);
}
```

Conceptually:

```text
out[i] = a[i] AND b[i]
```

for each bit `i`.

---

# 7. Multiplexors and Demultiplexors

## Multiplexor (Mux)

A **multiplexor**, or **Mux**, selects one input from multiple inputs and sends it to the output.

```text
Multiple inputs → One output
```

The `sel` signal determines which input is selected.

### Mux

```text
CHIP Mux {
    IN a, b, sel;
    OUT out;

    PARTS:
    Not(in=sel, out=notSel);
    And(a=a, b=notSel, out=aAndNotSel);
    And(a=b, b=sel, out=bAndSel);
    Or(a=aAndNotSel, b=bAndSel, out=out);
}
```

Conceptually:

```text
sel = 0 → out = a
sel = 1 → out = b
```

A Mux is therefore a basic **data selection** component.

---

## Demultiplexor (DMux)

A **demultiplexor**, or **DMux**, takes one input and routes it to one of multiple outputs.

```text
One input → One of multiple outputs
```

### DMux

```text
CHIP DMux {
    IN in, sel;
    OUT a, b;

    PARTS:
    Not(in=sel, out=notSel);
    And(a=in, b=notSel, out=a);
    And(a=in, b=sel, out=b);
}
```

Conceptually:

```text
sel = 0 → a = in, b = 0
sel = 1 → a = 0,  b = in
```

A DMux is therefore a basic **data routing** component.

---

# 8. 16-bit Variants

The same logic gates can be extended to operate on 16-bit buses.

## Not16

`Not16` applies NOT independently to each of the 16 bits.

```text
CHIP Not16 {
    IN in[16];
    OUT out[16];

    PARTS:
    Not(in=in[0], out=out[0]);
    Not(in=in[1], out=out[1]);
    ...
    Not(in=in[15], out=out[15]);
}
```

Conceptually:

```text
out[i] = NOT in[i]
```

---

## And16

`And16` performs a bit-wise AND operation on two 16-bit inputs.

```text
CHIP And16 {
    IN a[16], b[16];
    OUT out[16];

    PARTS:
    And(a=a[0], b=b[0], out=out[0]);
    And(a=a[1], b=b[1], out=out[1]);
    ...
    And(a=a[15], b=b[15], out=out[15]);
}
```

Conceptually:

```text
out[i] = a[i] AND b[i]
```

---

## Or16

`Or16` performs a bit-wise OR operation on two 16-bit inputs.

```text
CHIP Or16 {
    IN a[16], b[16];
    OUT out[16];

    PARTS:
    Or(a=a[0], b=b[0], out=out[0]);
    Or(a=a[1], b=b[1], out=out[1]);
    ...
    Or(a=a[15], b=b[15], out=out[15]);
}
```

Conceptually:

```text
out[i] = a[i] OR b[i]
```

---

## Mux16

`Mux16` selects one of two 16-bit inputs based on a single select bit.

```text
CHIP Mux16 {
    IN a[16], b[16], sel;
    OUT out[16];

    PARTS:
    Mux(a=a[0], b=b[0], sel=sel, out=out[0]);
    Mux(a=a[1], b=b[1], sel=sel, out=out[1]);
    ...
    Mux(a=a[15], b=b[15], sel=sel, out=out[15]);
}
```

Conceptually:

```text
sel = 0 → out = a
sel = 1 → out = b
```

The same select signal controls all 16 individual Muxes.

---

# 9. Multi-way Variants

## Or8Way

`Or8Way` performs an OR operation across eight input bits.

```text
CHIP Or8Way {
    IN in[8];
    OUT out;

    PARTS:
    Or(a=in[0], b=in[1], out=or01);
    Or(a=or01, b=in[2], out=or012);
    Or(a=or012, b=in[3], out=or0123);
    Or(a=or0123, b=in[4], out=or01234);
    Or(a=or01234, b=in[5], out=or012345);
    Or(a=or012345, b=in[6], out=or0123456);
    Or(a=or0123456, b=in[7], out=out);
}
```

The output is:

```text
out = 1
```

if **at least one input bit is `1`**.

The output is `0` only when all eight input bits are `0`.

---

## Mux4Way16

`Mux4Way16` selects one of four 16-bit inputs using a 2-bit select signal.

```text
sel = 00 → out = a
sel = 01 → out = b
sel = 10 → out = c
sel = 11 → out = d
```

Implementation:

```text
CHIP Mux4Way16 {
    IN a[16], b[16], c[16], d[16], sel[2];
    OUT out[16];

    PARTS:
    Mux16(a=a, b=b, sel=sel[0], out=ab);
    Mux16(a=c, b=d, sel=sel[0], out=cd);
    Mux16(a=ab, b=cd, sel=sel[1], out=out);
}
```

The design is hierarchical: a larger Mux is constructed from smaller Mux components.

---

## Mux8Way16

`Mux8Way16` selects one of eight 16-bit inputs using a 3-bit select signal.

```text
000 → a
001 → b
010 → c
011 → d
100 → e
101 → f
110 → g
111 → h
```

Implementation:

```text
CHIP Mux8Way16 {
    IN a[16], b[16], c[16], d[16],
       e[16], f[16], g[16], h[16],
       sel[3];
    OUT out[16];

    PARTS:
    Mux4Way16(
        a=a, b=b, c=c, d=d,
        sel=sel[0..1],
        out=abcd
    );

    Mux4Way16(
        a=e, b=f, c=g, d=h,
        sel=sel[0..1],
        out=efgh
    );

    Mux16(
        a=abcd,
        b=efgh,
        sel=sel[2],
        out=out
    );
}
```

This demonstrates how complex hardware can be built hierarchically from simpler components.

---

## DMux4Way

`DMux4Way` routes one input to one of four outputs using a 2-bit select signal.

```text
00 → a
01 → b
10 → c
11 → d
```

Implementation:

```text
CHIP DMux4Way {
    IN in, sel[2];
    OUT a, b, c, d;

    PARTS:
    DMux(in=in, sel=sel[1], a=ab, b=cd);
    DMux(in=ab, sel=sel[0], a=a, b=b);
    DMux(in=cd, sel=sel[0], a=c, b=d);
}
```

---

## DMux8Way

`DMux8Way` routes one input to one of eight outputs using a 3-bit select signal.

```text
000 → a
001 → b
010 → c
011 → d
100 → e
101 → f
110 → g
111 → h
```

Implementation:

```text
CHIP DMux8Way {
    IN in, sel[3];
    OUT a, b, c, d, e, f, g, h;

    PARTS:
    DMux(in=in, sel=sel[2], a=abcd, b=efgh);

    DMux4Way(
        in=abcd,
        sel=sel[0..1],
        a=a, b=b, c=c, d=d
    );

    DMux4Way(
        in=efgh,
        sel=sel[0..1],
        a=e, b=f, c=g, d=h
    );
}
```

---

# Key Takeaways

The main concepts from this section are:

* **Boolean logic** provides the foundation for digital hardware.
* A Boolean function maps binary inputs to binary outputs.
* **HDL** describes hardware structure rather than sequential program execution.
* A chip can be constructed by **composing smaller chips**.
* **Interfaces** describe what a component does, while **implementations** describe how it does it.
* **Buses** allow multiple bits to be processed as a group.
* **Bit-wise operations** apply the same logic independently to each bit.
* **Muxes** select data from multiple inputs.
* **DMuxes** route data to one of multiple outputs.
* Larger components such as `Mux4Way16` and `Mux8Way16` can be constructed hierarchically from smaller components.
* The same basic logic gates can be extended to **16-bit and multi-way components**.

## Project Context

This section covers the Boolean Logic chapter of **Nand2Tetris** and focuses on understanding how basic logic gates can be composed into increasingly complex digital hardware using HDL.
