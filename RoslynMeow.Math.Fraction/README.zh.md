# RoslynMeow.Math.Fraction

[English](README.md) | [中文](README.zh.md)

[![GitHub Packages](https://img.shields.io/badge/GitHub%20Packages-RoslynMeow.Math.Fraction-2ea44f?logo=nuget)](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Fraction)
[![Release](https://img.shields.io/github/v/release/RoslynMeow/Math?include_prereleases&sort=semver&label=release)](https://github.com/RoslynMeow/Math/releases)
[![License](https://img.shields.io/github/license/RoslynMeow/Math)](https://github.com/RoslynMeow/Math/blob/main/LICENSE)
[![.NET](https://img.shields.io/badge/.NET-netstandard2.0-512BD4)](https://dotnet.microsoft.com)

## 目录

- [简介](#简介)
- [结构](#结构)
- [运算 / 计算](#运算--计算)
- [特性](#特性)
- [模式](#模式)
- [运算符](#运算符)
- [技巧](#技巧)

## 简介
>这是一个用两个 `BigInteger` 表示分数的仓库

>计算时在 `BigInteger`(即分子/分母)上操作,输出时才计算并打印出数值。

>它使用 `堆分配` 表示一个数,从而保证该数不会像 IEEE 标准那样被截断。

## 结构
```csharp
public readonly struct Fraction
{
	private readonly BigInteger num;
	private readonly BigInteger den;
}
```

## 运算 / 计算
```csharp
Fraction a = 1;
a /= 3;
```

## 特性
1. 它会自动(隐式)把任意 `整数类型` 转换为 `Fraction`

1. 它会自动(隐式)把任意 `浮点类型` 转换为 `Fraction`
> 但 `Double` 类型只保留前 `15 位`(有效数字),这可能带来问题。

## 模式
* 它始终让数保持在分数模式,`且是精确的`。
* 计算时数`仍保持精确模式`,以分子和分母的形式进行。
* 只有在`打印`时才转入非精确模式,计算到指定的小数位数。

## 运算符
1. `[运算符]` `{var} >> {int}`,获取该分数的精确小数位数。
1. `[运算符]` `! {var}`,化简该分数。
1. `[运算符]` `~ {var}`,尝试把该分数转换成带分数。

```csharp
Fraction a = 52163;
a /= 16604;
Console.WriteLine(a >> 20); // (3, 14159238737653577451)
Console.WriteLine(a >> 50); // (3, 14159238737653577451216574319441098530474584437484)
Console.WriteLine(~a); // (3, 2351 / 16604)
Console.WriteLine(!a); // 52163 / 16604
```

## 技巧
* 在编写分数运算时,VS 中运算符会变色。把鼠标悬停在运算符上即可确认它是否正确。
