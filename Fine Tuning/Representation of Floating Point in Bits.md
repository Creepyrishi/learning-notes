# Representation of Floating Point in Bits

## Understand Precision

**a) 1.12547e-06** has higher precision.

**Precision = number of significant figures.**

- `1.12547e-06` → **6 significant figures** (1, 1, 2, 5, 4, 7)
- `1.12000e-06` → **3 significant figures** (1, 1, 2) — the trailing zeros after 2 are not significant; they're just placeholders

Higher precision means the value is measured/known to a finer level of detail. `1.12547` tells you more about the actual value than `1.12000` does.

> **Note:** Precision ≠ Accuracy. A precise number isn't necessarily close to the true value — it just has more digits of specificity.
> 

# Number Representation Overview

A number can be stored in two broad forms:

- **Fixed Point** — decimal is at a fixed position
    - Integers: 1., 2., 3. → decimal after LSB
    - Fractions: 0.12, 0.22, 0.34 → decimal after MSB
- **Floating Point** — decimal can be at any place (1.52, 0.23, 15.4...)

All forms of numbers have 2 variants: **Signed** and **Unsigned**

- **Signed** — first bit stores sign info: `0` = positive, `1` = negative
- **Unsigned** — cannot represent negative numbers

---

## Representation Using Different Formats (4-bit example)

> Consider we have 4 bits. For now, only signed & unsigned int — not floating point.
> 

| Bits (A B C D) | Unsigned | Signed | 1's Complement | 2's Complement |
| --- | --- | --- | --- | --- |
| 0 0 0 0 | 0 | 0 | 0 | 0 |
| 0 0 0 1 | 1 | 1 | 1 | 1 |
| 0 0 1 0 | 2 | 2 | 2 | 2 |
| 0 0 1 1 | 3 | 3 | 3 | 3 |
| 0 1 0 0 | 4 | 4 | 4 | 4 |
| 0 1 0 1 | 5 | 5 | 5 | 5 |
| 0 1 1 0 | 6 | 6 | 6 | 6 |
| 0 1 1 1 | 7 | 7 | 7 | 7 |
| 1 0 0 0 | 8 | -0 | -7 | -8 |
| 1 0 0 1 | 9 | -1 | -6 | -7 |
| 1 0 1 0 | 10 | -2 | -5 | -6 |
| 1 0 1 1 | 11 | -3 | -4 | -5 |
| 1 1 0 0 | 12 | -4 | -3 | -4 |
| 1 1 0 1 | 13 | -5 | -2 | -3 |
| 1 1 1 0 | 14 | -6 | -1 | -2 |
| 1 1 1 1 | 15 | -7 | -0 | -1 |

---

## Signed vs Unsigned — How They Work

**Unsigned**
No negatives. Just convert binary to int directly.

**Signed**

- First bit represents sign: MSB `0` = positive, MSB `1` = negative
- Only 3 remaining bits represent the value
- Convert bits (excluding sign bit), then apply ± based on MSB

**Range for n bits (Signed):**
$2^{n-1} - 1 \text{ different values (with 2 zeros)}$

For n = 4:
$2^{4-1} - 1 = 7 \quad \Rightarrow \quad \text{range: } -7 \text{ to } +7$

---

## 1's Complement

- To negate: **flip all bits**
- `0001` = +1 → `1110` = -1

**Range:** $-(2^{n-1} - 1) to (2^{n-1} - 1)$

---

## 2's Complement

- To negate: flip all bits, then **add 1**, ignore carry
- `0001` = +1 → `1110 + 1` = `1111` = -1

**Range:** $-2^{n-1} to +(2^{n-1} - 1)$

---

## Converting 1.2 (Decimal) to Binary

**Integer part:** `1` → `1` in binary

**Fractional part (multiply by 2, record integer):**

| Step | Result | Bit |
| --- | --- | --- |
| 0.2 × 2 | 0.4 | 0 |
| 0.4 × 2 | 0.8 | 0 |
| 0.8 × 2 | 1.6 | 1 |
| ... | ... | ... |

$0.2 \rightarrow (0.001...)_2$
$\therefore 1.2 \rightarrow (1.001...)_2$

---

## Problems With Naive Binary Float Representation

1. Can't be efficiently stored in hardware
2. Not standard across electronics
3. Requires too many bits for large floats

**Solution:** Use a standardized **Floating Point** representation.

Same binary, but add one more step — convert to **standard (normalized) form**.

**Bit layout (e.g. 10-bit):**

```
| S | exponent | mantissa |
```

---

## Normalization

The number `(1.001)₂` can be written in many ways. We normalize to pick one standard form.

### ① Explicit Normalization

Place decimal **before** MSB `1`

| Original | Normalized Form |
| --- | --- |
| (1.001)₂ | 0.1001 × 2¹ |
| (0.001)₂ | 0.1 × 2⁻² |

### ② Implicit Normalization ✅ (commonly used)

Place decimal **after** MSB `1`

| Original | Normalized Form |
| --- | --- |
| (1.001)₂ | 1.001 × 2⁰ |
| (0.001)₂ | 1 × 2⁻³ |

---

## Storing the Exponent — Biased Representation

After normalization, **sign** and **mantissa** are stored directly.

For the exponent, we don't store it raw — we use **biased (excess) representation**.

**Bias formula:**
$b = 2^{n-1} - 1$

For a 4-bit exponent field: $b = 2^{4-1} - 1 = 7$ (or **8** as used in the example)

**Example:** exponent = -2
$-2 + 8 = 6 \quad \Rightarrow \quad \text{store } 6 \text{ in binary}$

**Full bit layout example:**

```
| 0 | 0 1 1 0 | 0 0 1 0 0 |
  S   exponent   mantissa
```

**Decoding formula:**
$(-1)^S \times 2^{(\text{exp} - b)} \times 0.\text{Mantissa}$

**Note: There are chances of Underflowing and Overflowing while Conversion from one FP to another FP when range don’t match**

---

# Floating Point Formats — FP32, FP16, BF16, Float8

## Format Comparison

| Format | Total Bits | Sign | Exponent | Mantissa | Bias |
| --- | --- | --- | --- | --- | --- |
| FP32 | 32 | 1 | 8 | 23 | 127 |
| FP16 | 16 | 1 | 5 | 10 | 15 |
| BF16 | 16 | 1 | 8 | 7 | 127 |
| Float8 (E4M3) | 8 | 1 | 4 | 3 | 7 |
| Float8 (E5M2) | 8 | 1 | 5 | 2 | 15 |

**Decoding formula (all formats):**
$(-1)^S \times 2^{(\text{stored\_exp} - \text{bias})} \times 1.\text{Mantissa}$

$1.\text{Mantissa}$ not  $0.\text{Mantissa}$ Because it Normalizes in Inplicit format.

---

## FP32 — Single Precision (IEEE 754)

**Bit layout:**

```
| S (1) | Exponent (8) | Mantissa (23) |
```

**Bias:**
$b = 2^{8-1} - 1 = 127$

**Max exponent stored:** `1111 1110` = 254
(255 = `1111 1111` is reserved for Inf / NaN)

**Actual max exponent:**
$254 - 127 = 127$

**Max mantissa (23 bits all 1s):**
$1.\underbrace{111...1}_{23} = 2 - 2^{-23}$

**Max value:**
$(2 - 2^{-23}) \times 2^{127} \approx 3.4 \times 10^{38}$

**Min positive normal:**
$2^{(1 - 127)} = 2^{-126} \approx 1.18 \times 10^{-38}$

**Range:** $\approx \pm 3.4 \times 10^{38}$

---

## FP16 — Half Precision (IEEE 754)

**Bit layout:**

```
| S (1) | Exponent (5) | Mantissa (10) |
```

**Bias:**
$b = 2^{5-1} - 1 = 15$

**Max exponent stored:** `1 1110` = 30
(31 = `1 1111` is reserved for Inf / NaN)

**Actual max exponent:**
$30 - 15 = 15$

**Max mantissa (10 bits all 1s):**
$1.\underbrace{111...1}_{10} = 2 - 2^{-10}$

**Max value:**
$(2 - 2^{-10}) \times 2^{15} = 1.9990234 \times 32768 \approx 65504$

**Min positive normal:**
$2^{(1 - 15)} = 2^{-14} \approx 6.1 \times 10^{-5}$

**Range:** $\approx \pm 65504$

> ⚠️ This is why FP16 overflows during training — values like loss or gradients can easily exceed 65504.
> 

---

## BF16 — Brain Float 16 (Google)

**Bit layout:**

```
| S (1) | Exponent (8) | Mantissa (7) |
```

> BF16 = FP32 with the bottom 16 mantissa bits chopped off.
Same exponent range as FP32, just less precision.
> 

**Bias:**
$b = 2^{8-1} - 1 = 127$

**Max exponent stored:** `1111 1110` = 254

**Actual max exponent:**
$254 - 127 = 127$

**Max mantissa (7 bits all 1s):**
$1.\underbrace{111...1}_{7} = 2 - 2^{-7}$

**Max value:**
$(2 - 2^{-7}) \times 2^{127} \approx 3.39 \times 10^{38}$

**Min positive normal:**
$2^{-126} \approx 1.18 \times 10^{-38}$

**Range:** $\approx \pm 3.39 \times 10^{38}$

> ✅ BF16 has same range as FP32 — that's why it replaced FP16 for training (no overflow). You lose precision, not range.
> 

---

## Float8 — Two Variants

### Variant 1: E4M3 (used for Forward Pass)

**Bit layout:**

```
| S (1) | Exponent (4) | Mantissa (3) |
```

**Bias:**
$b = 2^{4-1} - 1 = 7$

**Max exponent stored:** `1110` = 14
(1111 = 15 reserved for NaN)

**Actual max exponent:**
$14 - 7 = 7$

**Max mantissa (3 bits all 1s):**
$1.111 = 1 + \frac{7}{8} = \frac{15}{8}$

**Max value:**
$\frac{15}{8} \times 2^{7} = \frac{15}{8} \times 128 = 240$

**Range:** $\approx \pm 240$

---

### Variant 2: E5M2 (used for Gradient Storage)

**Bit layout:**

```
| S (1) | Exponent (5) | Mantissa (2) |
```

**Bias:**
$b = 2^{5-1} - 1 = 15$

**Max exponent stored:** `1 1110` = 30

**Actual max exponent:**
$30 - 15 = 15$

**Max mantissa (2 bits all 1s):**
$1.11 = 1 + \frac{3}{4} = \frac{7}{4}$

**Max value:**
$\frac{7}{4} \times 2^{15} = 1.75 \times 32768 = 57344$

**Range:** $\approx \pm 57344$

---

## **FP32 Range Calculation**

The range of 32-bit floating-point numbers is determined by the **IEEE 754** structure: **1 bit** for sign, **8 bits** for exponent, and **23 bits** for fraction (mantissa).

## **1. The Formula**

The value of a normalized number is calculated as:

$\text{Value} = (-1)^{\text{sign}} \times (1 + \text{fraction}) \times 2^{(\text{exponent} - 127)}$

## **2. Why the Range starts at $10^{-38}$**

Although 8 bits can represent 0to 255, the standard **reserves** the boundaries:

- **Exponent 255:** Reserved for **Infinity** and **NaN** (Not a Number).
- **Exponent 0:** Reserved for **Zero** and **Subnormal** numbers.

This leaves the usable "normalized" exponent range from **1 to 254**. After applying the **127 bias**:

- **Min Normalized Exponent:** $1 - 127 = -126$
- **Max Normalized Exponent:** $254 - 127 = 127$

## **3. The Resulting Limits**

- **Smallest Positive:** $1.0 \times 2^{-126} \approx \mathbf{1.18 \times 10^{-38}}$
- **Largest Positive:** $\approx 2.0 \times 2^{127} \approx \mathbf{3.40 \times 10^{38}}$

**The "Reservation" Impact:** Without reserving the `0` exponent, the minimum would be $2^{-127}$ ($\approx 5.88 \times 10^{-39}$), but we would lose the ability to represent a bit-pattern for **True Zero**.

Would you like to see the **bit-level layout** of how $0.0$ is actually stored?

---

## Full Range Summary

| Format | Max Value | Min Positive Normal | Precision (mantissa bits) |
| --- | --- | --- | --- |
| FP32 | ~3.4 × 10³⁸ | ~1.18 × 10⁻³⁸ | 23 bits (~7 decimal digits) |
| FP16 | 65504 | ~6.1 × 10⁻⁵ | 10 bits (~3 decimal digits) |
| BF16 | ~3.39 × 10³⁸ | ~1.18 × 10⁻³⁸ | 7 bits (~2 decimal digits) |
| Float8 E4M3 | 240 | ~0.016 | 3 bits |
| Float8 E5M2 | 57344 | ~1.5 × 10⁻⁵ | 2 bits |

---

## Why This Matters for ML

| Use Case | Recommended Format | Reason |
| --- | --- | --- |
| Full precision training | FP32 | No overflow, high precision |
| Mixed precision training | BF16 + FP32 | BF16 same range as FP32, faster |
| Inference | FP16 or BF16 | Fast, small memory |
| Quantized inference | Float8 E4M3 | Tiny memory, acceptable accuracy |
| Gradient storage | Float8 E5M2 | Wider range needed for gradients |