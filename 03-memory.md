# Memory

## Overview

1. Why Do We Need Time?
2. Clock and Discrete Time
3. CPU and Memory
4. Registers
5. RAM
6. Program Counter (PC)
7. Combinational vs. Sequential Logic
8. Data Flip-Flop (DFF)
9. From DFF to Register
10. Sub-bussing
11. RAM8 Architecture
12. Memory Hierarchy
13. Memory Components at a Glance
14. Key Takeaways

---

# 1. Why Do We Need Time?

Hardware signals take time to propagate through circuits and stabilize.

Before signals stabilize, the output may be temporarily meaningless.

Therefore, digital computers use a **clock** to synchronize hardware operations.

> **Hardware needs time because signals do not propagate instantaneously.**

---

# 2. Clock and Discrete Time

## Discrete Time

Instead of dealing with continuous physical time, digital systems divide time into fixed **cycles (time-steps)**.

Within each cycle:

* We ignore the exact timing of signal propagation.
* We allow enough time for signals to propagate and stabilize.
* We observe the result at the end of the cycle.

The cycle length must be long enough for the computation to stabilize.

> **Cycle length > maximum signal propagation delay**

## Clock

A **clock** provides a common timing signal for digital hardware.

The clock divides time into synchronized steps:

```text
tick → tock → tick → tock → ...
```

Different components use the same clock so that state changes occur synchronously.

### Physical Clock

A physical oscillator generates a continuous sequence of clock signals.

In chip diagrams, a triangle indicates a **clock input**.

The `tick/tock` behavior used in the Nand2Tetris simulator is a simplified model of a physical clock.

> **Within a cycle:** ignore timing details.
> **At the cycle boundary:** observe the stable result.

---

# 3. Computer Architecture: CPU and Memory

A computer can be broadly divided into **processing** and **storage**.

## CPU

The **CPU (Central Processing Unit)** processes instructions and performs computations.

It contains:

* **Registers** — small, fast storage locations
* **ALU (Arithmetic Logic Unit)** — performs arithmetic and logical operations
* **Control logic** — coordinates the execution of instructions

> **CPU = computation + a small amount of fast storage**

## Memory

Memory provides larger storage for data and instructions.

A computer's CPU:

```text
Memory → CPU → Memory
```

The CPU:

1. Reads data or instructions from memory.
2. Processes them.
3. Writes results back to memory.

> **Memory = large storage**
>
> **CPU = computation + small fast storage**

---

# 4. Register

A **Register** is a small storage unit that holds a value over time.

A register has:

* `in` — input value
* `load` — determines whether to store the new value
* `out` — stored value

## Register Behavior

```text
load = 1 → store new value
load = 0 → keep old value
clock    → determines when the value is updated
```

Formally:

```text
if load(t):
    out(t + 1) = in(t)
else:
    out(t + 1) = out(t)
```

The important point is that the output changes based on the **previous time-step**.

> **load = 1 → store new value**
>
> **load = 0 → maintain previous value**

## 1-Bit Register

A 1-bit register stores one binary value.

### When `load = 0`

```text
    t                    t+1
──────────────────────────────────
in      21
load     0
out     17   ───────────→ 17
```

The old value is maintained.

### When `load = 1`

```text
    t                    t+1
──────────────────────────────────
in      21
load     1
out     17   ───────────→ 21
```

The new input value is stored.

## Multi-Bit Register

The same behavior applies to multiple bits.

```text
if load(t):
    out(t + 1) = in(t)
else:
    out(t + 1) = out(t)
```

For example:

```text
16-bit Register

in   = 1011010010110010
out  = 0000000000000000
load = 1

        ↓

    next cycle

        ↓

out = 1011010010110010
```

---

# 5. RAM

**RAM (Random Access Memory)** is a collection of addressable registers.

A RAM can be described as:

> **RAM = n addressable w-bit Registers**

Where:

* `n` = number of registers
* `w` = width of each register
* `address` = selects which register to access
* `in` = data to be written
* `load` = write control
* `out` = data read from the selected register

For the Hack computer:

```text
w = 16 bits
```

## Address Bits

If a RAM contains `n` registers, the number of address bits required is:

$$
k = \log_2(n)
$$

For example, `RAM8` contains 8 registers:

```text
n = 8
log₂(8) = 3
```

Therefore, `RAM8` uses 3 address bits:

```text
address[3]
```

The possible addresses are:

```text
000
001
010
011
100
101
110
111
```

---

# 6. RAM Read and Write

## Read

The `address` selects a register.

The selected register's stored value appears at `out`.

```text
set address = i
probe out
```

Conceptually:

```text
out = RAM[address]
```

## Write

To write a value:

```text
set address = i
set in = v
set load = 1
```

Conceptually:

```text
RAM[i] ← v
```

The selected register stores the new value when the clock advances.

All other registers remain unchanged.

## Why "Random Access"?

The term **random access** means that any memory location can be accessed directly using its address.

For example:

```text
address = 3 → access RAM[3]
address = 7 → access RAM[7]
address = 1 → access RAM[1]
```

The physical location of the register does not determine how it is accessed.

> **Random access = directly access any memory location by its address.**

---

# 7. Program Counter (PC)

The **Program Counter (PC)** is a special CPU register.

It stores the **address of the next instruction to execute**.

The PC can perform three important operations.

### Increment

```text
PC++
```

Move to the next instruction.

### Load

```text
PC = n
```

Jump to instruction address `n`.

### Reset

```text
PC = 0
```

Start execution from the first instruction.

## PC Behavior

The PC can be described as:

```text
if reset(t):
    out(t + 1) = 0
else if load(t):
    out(t + 1) = in(t)
else if inc(t):
    out(t + 1) = out(t) + 1
else:
    out(t + 1) = out(t)
```

### Priority

When multiple control signals are possible, the operations have the following priority:

```text
reset
  ↓
load
  ↓
inc
  ↓
maintain
```

---

# 8. Combinational vs. Sequential Logic

One of the most important distinctions in digital logic is between **combinational** and **sequential** logic.

## Combinational Logic

The output depends only on the **current inputs**.

```text
current input
      ↓
    logic
      ↓
current output
```

It does not remember previous inputs.

Examples:

* AND
* OR
* NOT
* Mux
* Adder
* ALU

> **Combinational logic = computation without memory**

## Sequential Logic

The output depends on:

```text
current inputs + previous state
```

Sequential circuits can remember information.

Examples:

* DFF
* Register
* RAM
* Program Counter

They use a clock to synchronize state changes.

> **Sequential logic = memory + clock**

## Comparison

| Feature               | Combinational Logic              | Sequential Logic                |
| --------------------- | -------------------------------- | ------------------------------- |
| Depends on            | Current inputs                   | Current inputs + previous state |
| Remembers past values | No                               | Yes                             |
| Clock                 | Not required for logic operation | Used to update state            |
| Main purpose          | Computation                      | Storage / state                 |
| Examples              | ALU, Adder, Mux                  | DFF, Register, RAM, PC          |

---

# 9. Data Flip-Flop (DFF)

The **Data Flip-Flop (DFF)** is the most basic sequential memory element.

It stores **one bit** of information.

Its behavior can be represented as:

```text
out(t + 1) = in(t)
```

This means:

> The output at the next time-step equals the input from the current time-step.

The DFF is the basic building block used to construct registers and larger memory components.

> **DFF = 1-bit memory**

---

# 10. From DFF to a 1-Bit Register

A 1-bit Register can be built using:

```text
1-bit Register = Mux + DFF
```

The Mux determines what value should be stored in the DFF.

## Mux Inputs

```text
a   = current / old value
b   = new input value
sel = load
```

The behavior is:

```text
load = 0
    ↓
select old value
    ↓
DFF stores old value
    ↓
maintain state
```

```text
load = 1
    ↓
select new input
    ↓
DFF stores new value
    ↓
update state
```

Therefore:

```text
if load(t):
    out(t + 1) = in(t)
else:
    out(t + 1) = out(t)
```

### Conceptual Structure

```text
                 ┌─────────┐
old value ──────→│         │
                 │   Mux   ├────→ DFF ─────→ out
new input ──────→│         │       │
                 └────┬────┘       │
                      ↑             │
                    load            │
                                    │
                                    └── feedback
```

The feedback path allows the register to maintain its previous value.

---

# 11. w-Bit Register

A multi-bit register can be constructed from multiple 1-bit registers.

```text
w-bit Register = w × 1-bit Registers
```

For example:

```text
16-bit Register
= 16 × 1-bit Registers
```

Each bit stores one bit of the input.

The same `load` signal controls the entire register.

```text
load = 1 → store new w-bit value
load = 0 → maintain current w-bit value
```

> **1-bit Register × w → w-bit Register**

---

# 12. Sub-bussing

A register's `load` signal applies to the **entire bus**.

Therefore:

> **Partial loading is not directly supported.**

However, sub-bussing allows us to access and manipulate only part of a stored value.

For example:

```text
Register:

10110011 11001100
         ↑
       out[0..7]
```

We can:

1. Read a subset of the bits.
2. Perform calculations on those bits.
3. Combine the modified bits with the remaining bits.
4. Create a new full-width value.
5. Load the full value back into the register.

Conceptually:

```text
Register
    ↓
select bits
    ↓
calculate
    ↓
combine
    ↓
New 16-bit value
    ↓
load
    ↓
Register
```

> **Sub-bussing allows partial access and manipulation, but not partial loading.**

---

# 13. RAM8 Architecture

`RAM8` contains:

```text
8 registers × w bits
```

It requires:

```text
3 address bits
```

because:

```text
log₂(8) = 3
```

The RAM8 architecture can be understood as two main paths:

```text
                    ┌────────────────┐
address ───────────→│      DMux      │
load ──────────────→│ Address Decode │
                    └───────┬────────┘
                            │
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
            Reg0          Reg1          ... Reg7
              │             │              │
              └─────────────┼──────────────┘
                            ↓
                           Mux
                            ↓
                           out
```

## RAM8 Write Operation

### Step 1 — Address Decoding

The `address` and `load` signals enter a **DMux**.

The DMux activates only the register corresponding to the selected address.

For example:

```text
address = 010
load = 1
```

activates:

```text
Register 2
```

### Step 2 — Storage

The selected register receives:

```text
load = 1
```

and stores the input value when the clock advances.

All other registers remain unchanged.

```text
Register 2 ← in
```

## RAM8 Read Operation

Each register continuously provides its stored value.

The `address` signal controls a **Mux** that selects one register's output.

For example:

```text
address = 010
```

causes:

```text
Register 2 → Mux → out
```

Therefore:

```text
out = RAM[address]
```

---

# 14. Memory Hierarchy

Memory components are constructed hierarchically from smaller components.

A simplified hierarchy is:

```text
DFF
 ↓
1-bit Register
 ↓
16-bit Register
 ↓
RAM8
 ↓
RAM64
 ↓
RAM512
 ↓
RAM4K
 ↓
RAM16K
```

The key idea is **hierarchical construction**:

> Larger memory components can be built by combining smaller memory components.

---

# 15. Memory Components at a Glance

| Component          |       Stores | Main Role                      | Key Feature                          |
| ------------------ | -----------: | ------------------------------ | ------------------------------------ |
| **DFF**            |        1 bit | Remember one bit               | Basic sequential memory element      |
| **1-bit Register** |        1 bit | Store / maintain one bit       | `load=1` updates, `load=0` maintains |
| **w-bit Register** |       w bits | Store a w-bit value            | Built from w 1-bit registers         |
| **RAM8**           |   8 × w bits | Store multiple w-bit values    | 3-bit address selects one register   |
| **RAM64**          |  64 × w bits | Larger memory                  | 6-bit address                        |
| **RAM512**         | 512 × w bits | Larger memory                  | 9-bit address                        |
| **PC**             |       w bits | Store next instruction address | Increment, load, and reset           |

---

# 16. Key Takeaways

The main ideas from this section are:

* Hardware signals take physical time to propagate and stabilize.
* A **clock** divides time into discrete cycles and synchronizes state changes.
* **Combinational logic** computes outputs from current inputs without remembering past values.
* **Sequential logic** stores state and depends on previous values.
* A **DFF** is the basic 1-bit memory element.
* A **Register** can be built from DFFs and a Mux.
* A **w-bit Register** consists of w 1-bit registers.
* **RAM** is a collection of addressable registers.
* `address` selects which register is accessed.
* `load` determines whether the selected register is updated.
* **Sub-bussing** allows partial access and manipulation but not partial loading.
* The **Program Counter (PC)** stores the address of the next instruction.
* Memory components can be built hierarchically from smaller components.

## Core Relationship

```text
Combinational Logic
        ↓
   Computation
        ↓
Sequential Logic
        ↓
      State
        ↓
     Memory
        ↓
DFF → Register → RAM → Computer Memory
```

> **The key idea of sequential logic is state: the circuit can remember what happened in the past.**
