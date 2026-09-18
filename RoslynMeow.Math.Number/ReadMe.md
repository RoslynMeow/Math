# RoslynMeow.Math.Number

[中文](ReadMe.md) | [English](ReadMe.en.md)

[![GitHub Packages](https://img.shields.io/badge/GitHub%20Packages-RoslynMeow.Math.Number-2ea44f?logo=nuget)](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Number)
[![Release](https://img.shields.io/github/v/release/RoslynMeow/Math?include_prereleases&sort=semver&label=release)](https://github.com/RoslynMeow/Math/releases)
[![License](https://img.shields.io/github/license/RoslynMeow/Math)](https://github.com/RoslynMeow/Math/blob/main/LICENSE)
[![.NET](https://img.shields.io/badge/.NET-netstandard2.1-512BD4)](https://dotnet.microsoft.com)

## 目录

- [简介](#简介)
- [主要特性（概览）](#主要特性概览)
- [API 快速参考 — GrandInt](#api-快速参考--grandint)
- [API 快速参考 — BigFraction](#api-快速参考--bigfraction)
- [vs 原生 int / long / double 的比较](#vs-原生-int--long--double-的比较)
- [使用建议与场景](#使用建议与场景)
- [数学证明：有理数的整数放大性质及其扩充](#数学证明有理数的整数放大性质及其扩充)
- [构建与运行（重复）](#构建与运行重复)

## 简介

本仓库实现了任意精度大整数类型 `GrandInt`（位于 `RoslynMeow.Math.Number`代码库），以及基于它的有理数类型 `BigFraction`。
它们适用于需要高精度整数与分数运算、定制解析规则与紧凑二进制序列化的场景。

## 主要特性（概览）

- 任意精度整数 `GrandInt`：以小端字节数组存储幅度（least-significant byte first），使用单字节 `Flags` 存放符号。支持加减乘除（含取商与余数）、比较、序列化/反序列化、字符串解析与格式化。
- 有理数 `BigFraction`：分子/分母均由 `GrandInt` 支持，构造时自动约简，支持四则运算、与 IEEE754 的双精度互转（精确还原双精度为分数）以及若干便捷显示操作符。
- 性能：关键幅度运算按字节实现，部分循环使用 `unsafe` + `fixed` 优化。后备数组按倍数增长以降低重分配。

## API 快速参考 — `GrandInt`

主要构造与转换：

- `new GrandInt(long)` / `new GrandInt(ulong)`：从64 位整数构造。
- `GrandInt.Parse(string)` / `GrandInt.TryParse(string, out _)`：解析十进制、`0x` 十六进制、`0b` 二进制，允许 `_` 分隔符，例如 `"0xFF_FF"`。
- `GrandInt.ToSerialized()` / `GrandInt.FromSerialized(byte[])`：紧凑二进制序列化格式为 `[8 字节 little-endian length][Flags][data bytes...]`。
- `GrandInt.ToString()`：返回十进制字符串；`ToString("X")` 返回十六进制。
- 显式 / 隐式转换：支持与基本整数类型的隐式（向 GrandInt）与部分显式（从 double/decimal 到 GrandInt）转换；`ToDouble()` / `ToDecimal()` 将丢失超出目标类型精度的信息。

算术与常用方法：

- 运算符：`+`, `-`, `*`（已实现，带符号处理）。
- 除法：`GrandInt.DivRem(GrandInt dividend, GrandInt divisor)` 返回 `(quotient, remainder)`；余数为非负。
- 辅助：`Gcd(GrandInt, GrandInt)`、`Abs()`、`IsZero()`、`IsNegative`、`IsEven()`、`Length`（幅度字节数）、`GetMagnitudeBytes()`、`GetHeapAllocatedSize()`。
- 与 `System.Numerics.BigInteger`互操作：`ToBigInteger()` / `FromBigInteger(BigInteger)`。

示例：

```csharp
//解析与基本运算
var a = GrandInt.Parse("123456789012345678901234567890");
var b = GrandInt.Parse("0xDEADBEEF_F00D");
var c = a * b;
Console.WriteLine(c.ToString());

// 序列化
var buf = c.ToSerialized();
var d = GrandInt.FromSerialized(buf);
```

注意事项（转换/精度/边界）：

- `ToDouble()` 使用逐字节乘256 累积的方式构造 double，可能在极大幅度时造成溢出或失去低位信息；`(double)grandInt` 并不会保留全部精度。
- 从 `double`/`decimal` 到 `GrandInt` 的显式转换会截断小数部分并在超出 `ulong.MaxValue` 时抛出 `OverflowException`（实现采用最简单的截断路径）。

## API 快速参考 — `BigFraction`

主要构造与性质：

- 构造：`new BigFraction(GrandInt numerator, GrandInt denominator)`（构造时会自动约简，并保证分母为正）。
- `new BigFraction(GrandInt integer)`：创建整数分数（denominator =1）。
- `BigFraction.FromDouble(double d)`：基于 IEEE754 精确还原 double 为分数（精确表示 double 的有限二进制小数）。
- `ToDouble()`：将分数近似为 double（可能丢失精度）。

算术：

- 已实现 `+`, `-`, `*`, `/`，内部借助 `BigInteger` 做中间运算并在必要时做交叉约简以减小中间值大小。

便捷显示/操作符：

- `a >> digits`：返回 `(integerPart, decimalDigitsString)`，用于按位输出小数扩展（保留指定小数位数）。
- `~a`：返回 `(integerPart, remainderString)`，其中 remainderString 格式为 `numerator / denominator`。
- `!a`：返回 `"numerator / denominator"` 字符串。

示例：

```csharp
var n = GrandInt.Parse("355");
var d = GrandInt.Parse("113");
var piApprox = new BigFraction(n, d); //355/113
var (intPart, decimals) = piApprox >>20; // 获取整数部分和20位小数
Console.WriteLine(intPart); //3
Console.WriteLine(decimals.Substring(0,10)); // 前10位小数

// 从 double 精确还原
var f = BigFraction.FromDouble(0.1); //0.1 在 double 中并非精确1/10
```

注意事项（约简与性能）：

- `BigFraction` 在构造和运算时会使用 `BigInteger`进行约简或交叉约简以保证正确与尽可能小的中间表示，因而在非常大的分子/分母上会有额外 CPU/内存开销。

## vs 原生 `int` / `long` / `double` 的比较

- 精度与范围：
 - `int`/`long`：固定宽度（32/64 位），会发生溢出（除非使用 checked），无法表示超出范围的整数。
 - `double`：64 位 IEEE754 浮点，能表示非常大或非常小的量级，但只有有限的显式有效数字（约15-17 十进制位），并存在舍入误差，无法精确表示许多有理数（如0.1）。
 - `GrandInt`：任意精度整数，只有受内存与实现限制（如单次长度受 int.MaxValue 限制），不会发生溢出；非常适合大整数、密码学、精确计数场景。
 - `BigFraction`：表示精确有理数（分子/分母为任意精度整数），可准确表达像1/3、355/113 等分数而无舍入误差（直到你转换为有限精度类型）。

- 精确性与数学语义：
 - 使用 `double` 做累加/比较在数值敏感场景容易产生误差（例如金融、代数推导、精确解析）；`BigFraction` 提供数学上准确的有理运算。

- 性能与内存：
 - 原生 `int`/`double` 在常见 CPU 上速度更快、内存占用更小。
 - `GrandInt`/`BigFraction` 在小数或中等大小数值下有较高开销（分配、逐字节运算），但在需要超64 位或精确分数时是必要且正确的选择。为性能优化，应避免频繁的临时分配或不必要的从/到浮点转换；可考虑引入池化（如 `ArrayPool<byte>`）来降低 GC 压力。

- 可互操作性：
 -代码提供 `ToBigInteger()` / `FromBigInteger()`，便于与 `System.Numerics.BigInteger`互操作，亦可与现有库集成。

## 使用建议与场景

- 使用 `GrandInt`：
 -需要任意精度整数（大于64 位）的场景，例如加密、精确计数、大整数分布式算法、解析并操作很长的整数文本表示。
- 使用 `BigFraction`：
 -需要精确有理运算的场景，例如数值代数、符号计算、精确比例/比率计算、从浮点精确还原并分析其分数表示。

性能提示：
- 尽量复用 `GrandInt` 实例的内部缓冲区（通过 API读取后再一次性操作），避免频繁在循环中创建大量临时 `GrandInt`/`BigFraction`。
- 对于大量数值运算的热路径，考虑基准（Benchmarks 项目）并针对热点使用池化或本地优化。

## 数学证明：有理数的整数放大性质及其扩充

> 本节是 `BigFraction` / `GrandInt` 精确表示有理数的理论基础。原始 LaTeX 见 [`_Proofs/Proof.tex`](_Proofs/Proof.tex),编译版见 [`_Proofs/Proof.pdf`](_Proofs/Proof.pdf)。

### 记号与补充约定

- $\mathbb{Q}$ 表示有理数集,$\mathbb{Z}$ 表示整数集,$\mathbb{Z}_{+}$ 表示正整数集,$\mathbb{N}_0=\{0,1,2,\dots\}$。
- 若 $n$ 为正整数,写其素因子分解 $n=\prod_{\pi}\pi^{e_\pi(n)}$,其中 $\pi$ 遍历素数。
- 对素数 $\pi$,对整数 $n>0$ 定义 $v_\pi(n):=e_\pi(n)$;对有理数 $r=\frac{a}{b}$(互素表示)推广为 $v_\pi(r):=v_\pi(a)-v_\pi(b)$。
- 对固定基数 $p\ge2$(不必为素数),关心是否存在 $k\in\mathbb{N}_0$ 使 $p^k q\in\mathbb{Z}$。

### 定理 1(有理数的整数放大)

**定理.** 对任意 $q\in\mathbb{Q}$,存在正整数 $K\in\mathbb{Z}_{+}$ 使得 $Kq\in\mathbb{Z}$。

**证明.** 将 $q$ 表示为最简分数 $q=\frac{a}{b}$($a\in\mathbb{Z},\ b\in\mathbb{Z}_{+},\ \gcd(a,b)=1$)。取 $K=b$,则 $Kq=b\cdot\frac{a}{b}=a\in\mathbb{Z}$。∎

### 定理 2(基 $p$ 的等价条件)

**定理.** 设固定基数 $p\ge2$,令 $q=\frac{a}{b}$ 为最简分数。下列命题等价:

1. 存在 $k\in\mathbb{N}_0$ 使得 $p^k q\in\mathbb{Z}$;
2. 存在 $k\in\mathbb{N}_0$ 使得 $b\mid p^k$;
3. $b$ 的所有素因子都出现在 $p$ 的素因子分解中:对任意素数 $\pi$,若 $v_\pi(b)>0$ 则 $v_\pi(p)>0$。

特别地,当 $p$ 为素数时,上述等价于 $b=p^m$ 对某个 $m\in\mathbb{N}_0$。

**证明.**

- (1)$\Rightarrow$(2):若 $p^k q\in\mathbb{Z}$,则 $b\mid p^k a$;由 $\gcd(a,b)=1$ 得 $b\mid p^k$。
- (2)$\Rightarrow$(1):若 $b\mid p^k$,则 $p^k q=p^k\frac{a}{b}\in\mathbb{Z}$。
- (2)$\Leftrightarrow$(3):写 $p=\prod_\pi \pi^{v_\pi(p)}$、$b=\prod_\pi \pi^{v_\pi(b)}$。$b\mid p^k$ 等价于对每个素数 $\pi$ 有 $v_\pi(b)\le k\,v_\pi(p)$。若某 $\pi$ 满足 $v_\pi(b)>0$ 且 $v_\pi(p)=0$,右端恒为 $0$,矛盾;故必须有 $v_\pi(b)>0\Rightarrow v_\pi(p)>0$。反之,若对 $b$ 的每个素因子都有 $v_\pi(p)\ge1$,取

$$k\ge \max_{\pi:\,v_\pi(b)>0}\left\lceil\frac{v_\pi(b)}{v_\pi(p)}\right\rceil,$$

即可保证 $b\mid p^k$。∎

### 最小的 $k$ 的显式公式

设 $p$ 的素因子集合为 $\{\pi_1,\dots,\pi_r\}$,$v_{\pi_i}(p)=f_i\ge1$,并记 $e_i:=v_{\pi_i}(b)$(若某 $\pi_i$ 未出现在 $b$ 中则为 $0$)。若存在 $\pi\notin\{\pi_1,\dots,\pi_r\}$ 使 $v_\pi(b)>0$,则不存在使 $b\mid p^k$ 的 $k$;否则最小的 $k$ 为

$$k_{\min}=\max_{1\le i\le r}\left\lceil\frac{e_i}{f_i}\right\rceil.$$

当 $p$ 为素数时只有一项且 $f_1=1$,故 $k_{\min}=v_p(b)$,此时 $b$ 必为 $p$ 的幂。

### 基下有限小数表示的等价性

在基 $p$ 的位置表示中,右移小数点 $k$ 位相当于乘以 $p^k$,因此

$$q\ \text{在基 }p\text{ 下有有限小数表示}\iff \exists k\in\mathbb{N}_0,\ p^k q\in\mathbb{Z}.$$

结合定理 2:$q=a/b$ 在基 $p$ 下有有限小数表示,当且仅当 $b$ 的所有素因子都出现在 $p$ 的素因子分解中。对十进制($p=10$),即分母只含素因子 $2$ 与 $5$。

### 关于推广到实数集的不可能性

固定 $p\ge2$,定义 $S_p:=\{x\in\mathbb{R}:\exists k\in\mathbb{N}_0,\ p^k x\in\mathbb{Z}\}$。有 $S_p=\bigcup_{k\ge0}p^{-k}\mathbb{Z}$,每个 $p^{-k}\mathbb{Z}$ 可数,故 $S_p$ 可数;而 $\mathbb{R}$ 不可数(Cantor 对角线论证),所以 $S_p\neq\mathbb{R}$。具体反例:$\sqrt{2}$——若存在非零整数 $K$ 使 $K\sqrt2\in\mathbb{Z}$,则 $\sqrt2$ 为有理数,矛盾。

**Cantor 对角线论证.** 若 $\mathbb{R}$ 可数,则可枚举所有实数;构造一个第 $n$ 位小数与枚举中第 $n$ 个数不同的新实数,产生矛盾,故 $\mathbb{R}$ 不可数,不可能包含于任何可数集。

**实数的稠密性.** $\mathbb{Q}$ 虽可数,但在 $\mathbb{R}$ 中稠密:对任意 $x\in\mathbb{R},\varepsilon>0$,取 $n\in\mathbb{N}$ 使 $1/n<\varepsilon$,令 $k=\lfloor nx\rfloor$,则 $|x-k/n|<\varepsilon$。这说明任意小邻域内都有有理数,但不意味着所有实数都能写成分母为某固定形式的有理数。

### 结论

$$\forall q\in\mathbb{Q}\ \exists K\in\mathbb{Z}_{+}:\ Kq\in\mathbb{Z}.$$

$$\text{设 }q=a/b\ (\text{最简}),\ p\ge2\text{ 固定,则}\quad
\bigl(\exists k:\ p^k q\in\mathbb{Z}\bigr)\iff\bigl(\exists k:\ b\mid p^k\bigr)\iff\bigl(\forall\pi,\ v_\pi(b)>0\Rightarrow v_\pi(p)>0\bigr).$$

### 参考资料

- D. S. Dummit, R. M. Foote, *Abstract Algebra*(整环与分式域章节)。
- T. W. Hungerford, *Algebra*(分式域与局部化)。
- W. Rudin, *Principles of Mathematical Analysis*(实数与有理数、稠密性)。

## 构建与运行（重复）

在仓库根目录执行：

- `dotnet build`
- `dotnet test`
- `dotnet run --project BigNumberExample`



