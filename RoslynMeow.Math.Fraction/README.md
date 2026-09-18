# RoslynMeow.Math.Fraction

[English](README.md) | [中文](README.zh.md)

[![GitHub Packages](https://img.shields.io/badge/GitHub%20Packages-RoslynMeow.Math.Fraction-2ea44f?logo=nuget)](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Fraction)
[![Release](https://img.shields.io/github/v/release/RoslynMeow/Math?include_prereleases&sort=semver&label=release)](https://github.com/RoslynMeow/Math/releases)
[![License](https://img.shields.io/github/license/RoslynMeow/Math)](https://github.com/RoslynMeow/Math/blob/main/LICENSE)
[![.NET](https://img.shields.io/badge/.NET-netstandard2.0-512BD4)](https://dotnet.microsoft.com)

## Contents

- [Intro](#intro)
- [Struct](#struct)
- [Manipulation / Calculation](#manipulation--calculation)
- [Features](#features)
- [Mode](#mode)
- [Operators](#operators)
- [Tricks](#tricks)

## Intro
>
>this is a repo that use 2 BigInteger that represent a fraction

>when calculate it manipulate on BigInteger (thus numerator/denominator) when transcribe out, it calculate and output the number.

>it use a `heap allocation` represent a number, as if ensure a number is not truncate by IEEE standard.

## Struct

```csharp
public readonly struct Fraction
{
 private readonly BigInteger num;
 private readonly BigInteger den;
}
```

## Manipulation / Calculation

```csharp
Fraction a = 1;
a /= 3;
```

## Features

1. it auto (implicit) convert the type of any `Integer Type` into `Fraction`

1. it auto (implicit) convert the type of any `Double Type` into `Fraction`

> but `Double` Type only record the first `15 digit` (valid) which `may` cause issues.

## Mode

* it always keep the number in fraction mode `which is percise`.
- when calculate the number `still in percise mode`, which do as numerator and denominator.
- when `print` it into `none percise mode`, it will calculate into specific digit.

## Operators

1. `[operator]` `{var} >> {int}`, this operator let you get percise digit number of a fraction.
1. `[operator]` `! {var}` , this operator simplify the fraction.
1. `[operator]` `~ {var}` , this operator try turning the fraction into a complex fraction.

```csharp
Fraction a = 52163;
a /= 16604;
Console.WriteLine(a >> 20); // (3, 14159238737653577451)
Console.WriteLine(a >> 50); // (3, 14159238737653577451216574319441098530474584437484)
Console.WriteLine(~a); // (3, 2351 / 16604)
Console.WriteLine(!a); // 52163 / 16604
```

## Tricks

* when add into Fraction mode. in the VS, the operator will change color. hover the operator to see if it's correct.
