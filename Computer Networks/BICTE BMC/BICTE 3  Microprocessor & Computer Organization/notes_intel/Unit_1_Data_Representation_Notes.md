# UNIT 1: DATA REPRESENTATION
**Course:** Computer Architecture and Organization  
**Course Code:** ICT. Ed. 436  
**Lecture Hours:** 4 LH

## Learning Objectives

By the end of this unit, students should be able to:

- Explain how different types of data are represented in a computer.
- Explain why complement numbers are used in computer arithmetic.

---

## 1.1 Data Types

A computer stores all data as **bits** (`0` and `1`). Different data types use different bit patterns and interpretations.

| Data type | Meaning | Example |
|---|---|---|
| Integer | Whole number | `25`, `-8` |
| Floating-point | Number with a fractional part | `3.14`, `-0.5` |
| Character | Letter, digit or symbol | `'A'`, `'7'` |
| Boolean | Logical value | `True`, `False` |

### Bits, bytes and number capacity

- **Bit:** One binary digit, either `0` or `1`.
- **Nibble:** 4 bits.
- **Byte:** 8 bits.

With \(n\) bits, there are \(2^n\) possible bit patterns.

Example: 8 bits can represent \(2^8 = 256\) different patterns.

### Unsigned and signed integers

An **unsigned** 8-bit number uses all bits for magnitude:

\[
0 \text{ to } 2^8-1 = 255
\]

A **signed** 8-bit integer commonly uses two's complement:

\[
-2^{7} \text{ to } 2^{7}-1 = -128 \text{ to } 127
\]

Example:

```text
Decimal 13 = Binary 00001101  (8-bit unsigned or positive signed)
```

### Character representation

Characters are stored using character-encoding standards.

- **ASCII:** A standard encoding for basic English letters, digits and symbols.
- **Unicode:** Supports characters from many languages and writing systems.

Example:

```text
'A' = ASCII decimal 65 = Binary 01000001
```

---

## 1.2 Complements

A complement is a way of representing negative numbers and simplifying subtraction in digital computers.

### 1's complement

To find the 1's complement, change every `0` to `1` and every `1` to `0`.

Example (8-bit):

```text
 +5       = 00000101
 1's comp = 11111010
```

### 2's complement

To find the 2's complement:

1. Find the 1's complement.
2. Add `1` to the result.

Example: Represent `-5` using 8 bits.

```text
 +5             = 00000101
 1's complement = 11111010
 Add 1          = 00000001
                  --------
 -5 (2's comp)  = 11111011
```

### Why is 2's complement used?

- It represents both positive and negative integers.
- The same binary adder circuit can perform addition and subtraction.
- It has only one representation for zero.

### Subtraction using 2's complement

Subtraction can be performed as:

\[
A-B = A + (\text{2's complement of } B)
\]

Example: \(7-3\), using 8 bits.

```text
  7             = 00000111
  3             = 00000011
  2's complement of 3 = 11111101

  00000111
+ 11111101
-----------
1 00000100

Discard the carry out:
  00000100 = 4
```

### Overflow

Overflow occurs when the result of an operation is outside the range that can be represented with the available number of bits.

For signed 8-bit integers, the range is \(-128\) to \(+127\).

Example: \(100+50=150\), but 150 is outside this range.

```text
  100 = 01100100
   50 = 00110010
        --------
        10010110
```

The result has a negative sign bit even though two positive numbers were added. This indicates **signed overflow**.

---

## 1.3 Fixed-Point Representation

In fixed-point representation, the binary point is assumed to be at a fixed position. The point is not usually stored as a separate bit.

Example:

```text
Binary value:  0011.0100
               |||| ||||
               Integer Fraction
```

The value is:

\[
0011.0100_2 = 3 + 0.25 = 3.25_{10}
\]

### Example using Q4.4 format

For this note, **Q4.4** means 4 bits before the binary point and 4 bits after it, using an 8-bit signed two's-complement value.

```text
3.25 = 0011.0100
       ---- ----
       4 bits 4 bits
```

- Resolution (smallest step): \(2^{-4}=0.0625\)
- Signed range: \(-8\) to \(7.9375\)

**Advantages:** Simple and fast arithmetic; useful in embedded and digital signal-processing systems.

**Limitation:** The range and precision are fixed by the chosen format.

---

## 1.4 Floating-Point Representation

Floating-point representation stores numbers using a sign, exponent and significand (also called mantissa in many textbooks). It is useful for representing very large, very small and fractional values.

A normalized binary floating-point number has the form:

\[
(-1)^S \times 1.F \times 2^{(E-\text{Bias})}
\]

Where:

- \(S\) = sign bit
- \(F\) = fraction field
- \(E\) = stored exponent
- Bias = exponent bias

### IEEE 754 single precision (32-bit)

```text
| Sign | Exponent | Fraction                  |
| 1 bit| 8 bits   | 23 bits                   |
```

- Sign: 0 for positive, 1 for negative.
- Exponent: Stored using a bias of 127.
- Fraction: Stores the fractional part of the significand for normalized values.

### Example: Represent 13.25

Convert the number to binary:

\[
13.25_{10}=1101.01_2
\]

Normalize:

\[
1101.01_2=1.10101_2 \times 2^3
\]

Therefore:

```text
Sign     = 0
Exponent = 3 + 127 = 130 = 10000010
Fraction = 10101000000000000000000
```

The 32-bit IEEE 754 representation is:

```text
0 10000010 10101000000000000000000
```

Hexadecimal form:

```text
41540000
```

**Advantages:** Supports a wide range of values.

**Limitation:** Some decimal fractions cannot be represented exactly in binary, so small rounding errors may occur.

---

## Quick Comparison

| Feature | Fixed-point | Floating-point |
|---|---|---|
| Binary point | Fixed position | Position changes using exponent |
| Range | Limited | Much wider |
| Precision | Fixed scale | Varies with magnitude |
| Typical use | Embedded/control systems | Scientific and general numerical computing |

---

## Practical Work 1.1

**Task:** Write a program to visualize complement numbers, integers, floating-point numbers and character data, and detect overflow while adding integers.

### Short Python demonstration

```python
# 8-bit integer and complement demonstration

n = 5
bits = 8

binary = format(n, "08b")
ones_complement = format((~n) & 0xFF, "08b")
twos_complement = format(((~n) + 1) & 0xFF, "08b")

print("Integer:", n)
print("Binary:", binary)
print("1's complement:", ones_complement)
print("2's complement of 5:", twos_complement)

# Character representation
ch = 'A'
print("Character:", ch)
print("ASCII value:", ord(ch))
print("8-bit binary:", format(ord(ch), "08b"))

# Floating-point example
x = 3.25
print("Floating-point value:", x)

# Signed 8-bit overflow detection
a, b = 100, 50
result = a + b
minimum, maximum = -128, 127

print("Addition:", a, "+", b, "=", result)

if result < minimum or result > maximum:
    print("Signed 8-bit overflow detected")
else:
    print("No signed 8-bit overflow")
```

### Expected observation

- The program displays the binary form of an integer.
- It displays the 1's and 2's complements.
- It displays the character code and its binary form.
- It displays a floating-point value.
- It detects that \(100+50=150\) exceeds the signed 8-bit range.

---

## Unit 1: Key Points to Remember

1. All computer data is stored as binary bits.
2. One byte contains 8 bits.
3. 1's complement is obtained by inverting every bit.
4. 2's complement is obtained by adding 1 to the 1's complement.
5. Fixed-point uses a fixed binary-point position.
6. Floating-point uses sign, exponent and fraction fields.
7. Overflow occurs when a result is outside the representable range.
