# RoslynMeow.Math.Bit

[中文](README.md) | [English](README.en.md)

[![GitHub Packages](https://img.shields.io/badge/GitHub%20Packages-RoslynMeow.Math.Bit-2ea44f?logo=nuget)](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Bit)
[![Release](https://img.shields.io/github/v/release/RoslynMeow/Math?include_prereleases&sort=semver&label=release)](https://github.com/RoslynMeow/Math/releases)
[![License](https://img.shields.io/github/license/RoslynMeow/Math)](https://github.com/RoslynMeow/Math/blob/main/LICENSE)
[![.NET](https://img.shields.io/badge/.NET-netstandard2.0-512BD4)](https://dotnet.microsoft.com)

位操作工具与基于 `Marshal` 的二进制报文生成。

## 目录

- [快速开始](#快速开始)
- [`Bit` 结构](#bit-结构)
- [`BitUtil` 扩展](#bitutil-扩展)
- [`Msg<T>` 报文](#msgt-报文)
- [安装](#安装)

## 快速开始

```csharp
using RoslynMeow.Math.Bit;

// 从 Byte 构造
var bits = new Bit(0b1010_0101);

// 索引 0-7(0 为最低位)
bool first = bits[0];
bits[1] = true;

byte raw = (byte)bits; // 显式转换
```

## `Bit` 结构

`Bit` 是一个轻量的 8 位容器,位索引方式为 `0x [7][6][5][4][3][2][1][0]`。

- `new Bit(params bool[] flags)` — 从最多 8 个布尔值构造。
- `new Bit(byte data)` — 从 `byte` 构造。
- `bit[index]` — 读取/写入第 `index` 位(`0`-`7`)。
- `ToByte()` / `(byte)bit` — 取回 `byte`。
- `implicit operator Bit(byte)` — 支持从 `byte` 隐式赋值。

## `BitUtil` 扩展

`BitUtil` 为 `byte` 提供扩展方法(位索引 `1`-`8`)。

```csharp
byte data = 0;

data.SetBit(1, true);           // 第 1 位置 1
data.SetBit(true, false, true); // 一次设置多个位

bool bit1 = data.GetBit(1);     // 读取第 1 位
bool[] all = data.GetBit();     // 读取全部位
```

## `Msg<T>` 报文

`Msg<T>` 使用 `Marshal` 在「结构体」与「字节数组」之间转换,适合固定布局的二进制报文。

```csharp
using System.Runtime.InteropServices;
using RoslynMeow.Math.Bit;

[StructLayout(LayoutKind.Sequential)]
struct Header
{
    public int Id;
    public short Size;
}

byte[] packet = Msg<Header>.Build(new Header { Id = 1, Size = 16 });
Header header = Msg<Header>.Decon(packet);
```

## 安装

本包发布在 **GitHub Packages**,使用前需在 `NuGet.config` 中配置源与凭据:

```xml
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="github" value="https://nuget.pkg.github.com/RoslynMeow/index.json" />
  </packageSources>
  <packageSourceCredentials>
    <github>
      <add key="Username" value="YOUR_GITHUB_USERNAME" />
      <!-- 需要带 read:packages 权限的 PAT -->
      <add key="ClearTextPassword" value="YOUR_GITHUB_PAT" />
    </github>
  </packageSourceCredentials>
</configuration>
```

```bash
dotnet add package RoslynMeow.Math.Bit
```

## License

[MIT](../LICENSE) © 2023 RoslynMeow
