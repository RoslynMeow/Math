# RoslynMeow.Math.Bit

[中文](README.md) | [English](README.en.md)

[![GitHub Packages](https://img.shields.io/badge/GitHub%20Packages-RoslynMeow.Math.Bit-2ea44f?logo=nuget)](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Bit)
[![Release](https://img.shields.io/github/v/release/RoslynMeow/Math?include_prereleases&sort=semver&label=release)](https://github.com/RoslynMeow/Math/releases)
[![License](https://img.shields.io/github/license/RoslynMeow/Math)](https://github.com/RoslynMeow/Math/blob/main/LICENSE)
[![.NET](https://img.shields.io/badge/.NET-netstandard2.0-512BD4)](https://dotnet.microsoft.com)

Bit helpers and a `Marshal`-based binary message builder.

## Contents

- [Quick start](#quick-start)
- [The `Bit` struct](#the-bit-struct)
- [`BitUtil` extensions](#bitutil-extensions)
- [`Msg<T>` messages](#msgt-messages)
- [Installation](#installation)

## Quick start

```csharp
using RoslynMeow.Math.Bit;

// Construct from a byte
var bits = new Bit(0b1010_0101);

// Index 0-7 (0 is the least-significant bit)
bool first = bits[0];
bits[1] = true;

byte raw = (byte)bits; // explicit conversion
```

## The `Bit` struct

`Bit` is a lightweight 8-bit container; bit indexing follows `0x [7][6][5][4][3][2][1][0]`.

- `new Bit(params bool[] flags)` — build from up to 8 booleans.
- `new Bit(byte data)` — build from a `byte`.
- `bit[index]` — read or write bit `index` (`0`-`7`).
- `ToByte()` / `(byte)bit` — get the underlying `byte`.
- `implicit operator Bit(byte)` — implicit assignment from `byte`.

## `BitUtil` extensions

`BitUtil` provides extension methods on `byte` (bit index `1`-`8`).

```csharp
byte data = 0;

data.SetBit(1, true);           // set bit 1
data.SetBit(true, false, true); // set several bits at once

bool bit1 = data.GetBit(1);     // read bit 1
bool[] all = data.GetBit();     // read all bits
```

## `Msg<T>` messages

`Msg<T>` converts between a blittable struct and a `byte[]` via `Marshal`, useful for fixed-layout packets.

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

## Installation

This package is published to **GitHub Packages**; configure the source and credentials in `NuGet.config` first:

```xml
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="github" value="https://nuget.pkg.github.com/RoslynMeow/index.json" />
  </packageSources>
  <packageSourceCredentials>
    <github>
      <add key="Username" value="YOUR_GITHUB_USERNAME" />
      <!-- a PAT with read:packages -->
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
