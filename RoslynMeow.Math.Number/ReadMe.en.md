# RoslynMeow.Math.Number

[中文](ReadMe.md) | [English](ReadMe.en.md)

[![GitHub Packages](https://img.shields.io/badge/GitHub%20Packages-RoslynMeow.Math.Number-2ea44f?logo=nuget)](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Number)
[![Release](https://img.shields.io/github/v/release/RoslynMeow/Math?include_prereleases&sort=semver&label=release)](https://github.com/RoslynMeow/Math/releases)
[![License](https://img.shields.io/github/license/RoslynMeow/Math)](https://github.com/RoslynMeow/Math/blob/main/LICENSE)
[![.NET](https://img.shields.io/badge/.NET-netstandard2.1-512BD4)](https://dotnet.microsoft.com)

## Contents

- [Introduction](#introduction)
- [Main features](#main-features)
- [API quick reference — GrandInt](#api-quick-reference--grandint)
- [API quick reference — BigFraction](#api-quick-reference--bigfraction)
- [Compared with native int / long / double](#compared-with-native-int--long--double)
- [Usage recommendations](#usage-recommendations)
- [Mathematical proof](#mathematical-proof)

## Introduction

This library implements an arbitrary-precision integer type `GrandInt` (in the `RoslynMeow.Math.Number` codebase) and a rational type `BigFraction` built on top of it.
They suit scenarios that need high-precision integer and fraction arithmetic, custom parsing rules, and compact binary serialization.

## Main features

- Arbitrary-precision integer `GrandInt`: stores magnitude as a little-endian byte array (least-significant byte first) and uses a single-byte `Flags` for the sign. Supports add/subtract/multiply/divide (including quotient and remainder), comparison, serialization/deserialization, and string parsing/formatting.
- Rational `BigFraction`: numerator and denominator are both backed by `GrandInt`, automatically reduced on construction. Supports the four arithmetic operations, exact conversion to/from IEEE754 double (exactly reconstructing a double as a fraction), and several convenience display operators.
- Performance: core magnitude operations are implemented byte-wise, and some loops are optimized with `unsafe` + `fixed`. The backing array grows by a factor to reduce reallocations.

## API quick reference — `GrandInt`

Construction and conversion:

- `new GrandInt(long)` / `new GrandInt(ulong)`: construct from a 64-bit integer.
- `GrandInt.Parse(string)` / `GrandInt.TryParse(string, out _)`: parse decimal, `0x` hexadecimal, and `0b` binary, allowing `_` separators, e.g. `"0xFF_FF"`.
- `GrandInt.ToSerialized()` / `GrandInt.FromSerialized(byte[])`: compact binary format `[8-byte little-endian length][Flags][data bytes...]`.
- `GrandInt.ToString()`: decimal string; `ToString("X")`: hexadecimal.
- Explicit/implicit conversions: implicit conversion from primitive integer types (to GrandInt), and partial explicit conversions (from double/decimal to GrandInt); `ToDouble()` / `ToDecimal()` lose information beyond the target type's precision.

Arithmetic and common methods:

- Operators: `+`, `-`, `*` (implemented, sign-aware).
- Division: `GrandInt.DivRem(GrandInt dividend, GrandInt divisor)` returns `(quotient, remainder)`; the remainder is non-negative.
- Helpers: `Gcd(GrandInt, GrandInt)`, `Abs()`, `IsZero()`, `IsNegative`, `IsEven()`, `Length` (magnitude byte count), `GetMagnitudeBytes()`, `GetHeapAllocatedSize()`.
- Interop with `System.Numerics.BigInteger`: `ToBigInteger()` / `FromBigInteger(BigInteger)`.

Example:

```csharp
// Parsing and basic arithmetic
var a = GrandInt.Parse("123456789012345678901234567890");
var b = GrandInt.Parse("0xDEADBEEF_F00D");
var c = a * b;
Console.WriteLine(c.ToString());

// Serialization
var buf = c.ToSerialized();
var d = GrandInt.FromSerialized(buf);
```

Notes (conversion / precision / boundaries):

- `ToDouble()` accumulates byte-by-byte multiplied by 256, which may overflow or lose low bits at very large magnitudes; `(double)grandInt` does not preserve full precision.
- Explicit conversion from `double`/`decimal` to `GrandInt` truncates the fractional part and throws `OverflowException` when it exceeds `ulong.MaxValue` (the implementation takes the simplest truncation path).

## API quick reference — `BigFraction`

Construction and properties:

- `new BigFraction(GrandInt numerator, GrandInt denominator)` (automatically reduced on construction, denominator kept positive).
- `new BigFraction(GrandInt integer)`: creates an integer fraction (denominator = 1).
- `BigFraction.FromDouble(double d)`: exactly reconstructs a double as a fraction based on IEEE754 (representing the finite binary fraction of the double exactly).
- `ToDouble()`: approximates the fraction as a double (may lose precision).

Arithmetic:

- Implements `+`, `-`, `*`, `/`, using `BigInteger` internally for intermediate work and cross-reduction when needed to keep intermediate values small.

Convenience display/operators:

- `a >> digits`: returns `(integerPart, decimalDigitsString)` for emitting the decimal expansion (keeping the specified number of fractional digits).
- `~a`: returns `(integerPart, remainderString)`, where remainderString is `numerator / denominator`.
- `!a`: returns the string `"numerator / denominator"`.

Example:

```csharp
var n = GrandInt.Parse("355");
var d = GrandInt.Parse("113");
var piApprox = new BigFraction(n, d); // 355/113
var (intPart, decimals) = piApprox >> 20; // integer part and 20 fractional digits
Console.WriteLine(intPart); // 3
Console.WriteLine(decimals.Substring(0, 10)); // first 10 fractional digits

// Exact reconstruction from a double
var f = BigFraction.FromDouble(0.1); // 0.1 is not exactly 1/10 in a double
```

Notes (reduction and performance):

- `BigFraction` uses `BigInteger` for reduction or cross-reduction on construction and operations to stay correct and keep intermediates as small as possible, so very large numerators/denominators incur extra CPU/memory cost.

## Compared with native `int` / `long` / `double`

- Precision and range:
  - `int`/`long`: fixed width (32/64-bit), can overflow (unless checked), cannot represent integers beyond their range.
  - `double`: 64-bit IEEE754 floating point; represents very large or very small magnitudes but has only a limited number of significant digits (about 15-17 decimal digits) and rounding error, and cannot exactly represent many rationals (e.g. 0.1).
  - `GrandInt`: arbitrary-precision integer, limited only by memory and implementation (e.g. a single length limited by int.MaxValue), never overflows; ideal for big integers, cryptography, exact counting.
  - `BigFraction`: exact rationals (numerator/denominator are arbitrary-precision integers); represents values like 1/3 or 355/113 with no rounding error (until you convert to a finite-precision type).

- Exactness and mathematical semantics:
  - Using `double` for accumulation/comparison easily produces error in numerically sensitive work (e.g. finance, algebraic derivation, exact parsing); `BigFraction` provides mathematically exact rational arithmetic.

- Performance and memory:
  - Native `int`/`double` are faster and use less memory on common CPUs.
  - `GrandInt`/`BigFraction` have higher overhead for small or medium values (allocation, byte-wise operations), but are necessary and correct when more than 64 bits or exact fractions are needed. To optimize, avoid frequent temporary allocations or unnecessary float conversions; consider pooling (e.g. `ArrayPool<byte>`) to reduce GC pressure.

- Interoperability:
  - The code provides `ToBigInteger()` / `FromBigInteger()` to interoperate with `System.Numerics.BigInteger` and integrate with existing libraries.

## Usage recommendations

- Use `GrandInt`:
  - when you need arbitrary-precision integers (larger than 64 bits), e.g. cryptography, exact counting, large-integer distributed algorithms, parsing and manipulating very long integer text.
- Use `BigFraction`:
  - when you need exact rational arithmetic, e.g. numerical algebra, symbolic computation, exact ratio/proportion computation, exactly reconstructing and analyzing the fraction representation of floats.

Performance tips:

- Reuse the internal buffer of `GrandInt` instances where possible (read via the API and then operate in one go), avoiding creating many temporary `GrandInt`/`BigFraction` objects in loops.
- For hot paths with heavy numeric work, consider benchmarking (the Benchmarks project) and applying pooling or local optimizations to hotspots.

## Mathematical proof

> This section is the theoretical basis for `BigFraction` / `GrandInt` representing rationals exactly. The original LaTeX is in [`_Proofs/Proof.tex`](_Proofs/Proof.tex); a compiled version is in [`_Proofs/Proof.pdf`](_Proofs/Proof.pdf).

### Notation and conventions

- $\mathbb{Q}$ denotes the rationals, $\mathbb{Z}$ the integers, $\mathbb{Z}_{+}$ the positive integers, and $\mathbb{N}_0=\{0,1,2,\dots\}$.
- For a positive integer $n$, write its prime factorization as $n=\prod_{\pi}\pi^{e_\pi(n)}$, where $\pi$ runs over primes.
- For a prime $\pi$ and integer $n>0$, define $v_\pi(n):=e_\pi(n)$; for a rational $r=\frac{a}{b}$ (in lowest terms) extend this as $v_\pi(r):=v_\pi(a)-v_\pi(b)$.
- For a fixed base $p\ge2$ (not necessarily prime), we ask whether there is $k\in\mathbb{N}_0$ with $p^k q\in\mathbb{Z}$.

### Theorem 1 (integer scaling of rationals)

**Theorem.** For every $q\in\mathbb{Q}$ there is a positive integer $K\in\mathbb{Z}_{+}$ such that $Kq\in\mathbb{Z}$.

**Proof.** Write $q$ in lowest terms as $q=\frac{a}{b}$ with $a\in\mathbb{Z},\ b\in\mathbb{Z}_{+},\ \gcd(a,b)=1$. Take $K=b$; then $Kq=b\cdot\frac{a}{b}=a\in\mathbb{Z}$. ∎

### Theorem 2 (equivalence for base $p$)

**Theorem.** Fix a base $p\ge2$ and let $q=\frac{a}{b}$ be in lowest terms. The following are equivalent:

1. there is $k\in\mathbb{N}_0$ with $p^k q\in\mathbb{Z}$;
2. there is $k\in\mathbb{N}_0$ with $b\mid p^k$;
3. every prime factor of $b$ also occurs in the prime factorization of $p$: for every prime $\pi$, $v_\pi(b)>0$ implies $v_\pi(p)>0$.

In particular, when $p$ is prime this is equivalent to $b=p^m$ for some $m\in\mathbb{N}_0$.

**Proof.**

- (1)$\Rightarrow$(2): if $p^k q\in\mathbb{Z}$, then $b\mid p^k a$; since $\gcd(a,b)=1$, we get $b\mid p^k$.
- (2)$\Rightarrow$(1): if $b\mid p^k$, then $p^k q=p^k\frac{a}{b}\in\mathbb{Z}$.
- (2)$\Leftrightarrow$(3): write $p=\prod_\pi \pi^{v_\pi(p)}$ and $b=\prod_\pi \pi^{v_\pi(b)}$. The condition $b\mid p^k$ is equivalent to $v_\pi(b)\le k\,v_\pi(p)$ for every prime $\pi$. If some $\pi$ has $v_\pi(b)>0$ and $v_\pi(p)=0$, the right side is always $0$, a contradiction; hence we must have $v_\pi(b)>0\Rightarrow v_\pi(p)>0$. Conversely, if $v_\pi(p)\ge1$ for every prime factor of $b$, taking

$$k\ge \max_{\pi:\,v_\pi(b)>0}\left\lceil\frac{v_\pi(b)}{v_\pi(p)}\right\rceil$$

guarantees $b\mid p^k$. ∎

### Explicit formula for the minimal $k$

Let the prime factors of $p$ be $\{\pi_1,\dots,\pi_r\}$ with $v_{\pi_i}(p)=f_i\ge1$, and set $e_i:=v_{\pi_i}(b)$ (zero if $\pi_i$ does not occur in $b$). If some $\pi\notin\{\pi_1,\dots,\pi_r\}$ has $v_\pi(b)>0$, then no $k$ satisfies $b\mid p^k$; otherwise the minimal such $k$ is

$$k_{\min}=\max_{1\le i\le r}\left\lceil\frac{e_i}{f_i}\right\rceil.$$

When $p$ is prime there is a single term with $f_1=1$, so $k_{\min}=v_p(b)$, and $b$ must be a power of $p$.

### Equivalence with finite base-$p$ expansions

In a positional base-$p$ representation, shifting the radix point $k$ places is multiplication by $p^k$, so

$$q\ \text{has a finite base-}p\text{ expansion}\iff \exists k\in\mathbb{N}_0,\ p^k q\in\mathbb{Z}.$$

Combined with Theorem 2: $q=a/b$ has a finite base-$p$ expansion iff every prime factor of $b$ occurs in the prime factorization of $p$. For decimal ($p=10$) this means the denominator only contains the prime factors $2$ and $5$.

### Impossible to extend to the reals

Fix $p\ge2$ and define $S_p:=\{x\in\mathbb{R}:\exists k\in\mathbb{N}_0,\ p^k x\in\mathbb{Z}\}$. We have $S_p=\bigcup_{k\ge0}p^{-k}\mathbb{Z}$; each $p^{-k}\mathbb{Z}$ is countable, so $S_p$ is countable, whereas $\mathbb{R}$ is uncountable (Cantor's diagonal argument). Hence $S_p\neq\mathbb{R}$. A concrete counterexample: $\sqrt{2}$ — if some non-zero integer $K$ had $K\sqrt2\in\mathbb{Z}$, then $\sqrt2$ would be rational, a contradiction.

**Cantor's diagonal argument.** If $\mathbb{R}$ were countable, one could enumerate all reals; constructing a new real whose $n$-th digit differs from the $n$-th number in the enumeration yields a contradiction. Hence $\mathbb{R}$ is uncountable and cannot be contained in any countable set.

**Density of the reals.** Although $\mathbb{Q}$ is countable, it is dense in $\mathbb{R}$: for any $x\in\mathbb{R},\ \varepsilon>0$, pick $n\in\mathbb{N}$ with $1/n<\varepsilon$ and let $k=\lfloor nx\rfloor$; then $|x-k/n|<\varepsilon$. This shows every neighborhood, however small, contains a rational, but it does not mean every real can be written as a rational with a denominator of some fixed form.

### Conclusion

$$\forall q\in\mathbb{Q}\ \exists K\in\mathbb{Z}_{+}:\ Kq\in\mathbb{Z}.$$

$$\text{For }q=a/b\ (\text{lowest terms}),\ p\ge2\text{ fixed:}\quad
\bigl(\exists k:\ p^k q\in\mathbb{Z}\bigr)\iff\bigl(\exists k:\ b\mid p^k\bigr)\iff\bigl(\forall\pi,\ v_\pi(b)>0\Rightarrow v_\pi(p)>0\bigr).$$

### References

- D. S. Dummit, R. M. Foote, *Abstract Algebra* (integral domains and fields of fractions).
- T. W. Hungerford, *Algebra* (fields of fractions and localization).
- W. Rudin, *Principles of Mathematical Analysis* (reals, rationals, density).

## Build and run

From the repository root:

- `dotnet build`
- `dotnet test`
- `dotnet run --project BigNumberExample`
