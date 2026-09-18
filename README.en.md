# RoslynMeow.Math

[中文](README.md) | [English](README.en.md)

[![Release](https://img.shields.io/github/v/release/RoslynMeow/Math?include_prereleases&sort=semver&label=release)](https://github.com/RoslynMeow/Math/releases)
[![Release workflow](https://github.com/RoslynMeow/Math/actions/workflows/release.yml/badge.svg)](https://github.com/RoslynMeow/Math/actions/workflows/release.yml)
[![License](https://img.shields.io/github/license/RoslynMeow/Math)](https://github.com/RoslynMeow/Math/blob/main/LICENSE)
[![.NET](https://img.shields.io/badge/.NET-netstandard2.0%20%7C%202.1-512BD4)](https://dotnet.microsoft.com)
[![Stars](https://img.shields.io/github/stars/RoslynMeow/Math?style=social)](https://github.com/RoslynMeow/Math/stargazers)

A collection of .NET math libraries focused on **exact and arbitrary-precision** numerics: exact fractions, arbitrary-precision integers & rationals, graph algorithms, and bit utilities.

## Packages

| Package | Target | Description |
|---|---|---|
| [`RoslynMeow.Math.Fraction`](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Fraction) | `netstandard2.0` | Exact fractions backed by two `BigInteger` values |
| [`RoslynMeow.Math.Number`](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Number) | `netstandard2.1` | Arbitrary-precision `GrandInt` and `BigFraction` |
| [`RoslynMeow.Math.Graph`](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Graph) | `netstandard2.0` | Graph data structures and algorithms: shortest path, MST, max flow, topological sort |
| [`RoslynMeow.Math.Bit`](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Bit) | `netstandard2.0` | Bit helpers and a `Marshal`-based binary message builder |

## Installation

Packages are published to **GitHub Packages**. This NuGet feed requires authentication even for public packages, so configure the source and credentials first:

```xml
<!-- NuGet.config -->
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
dotnet add package RoslynMeow.Math.Fraction
```

## Repository layout

```
RoslynMeow.Math.Fraction/     exact fractions
RoslynMeow.Math.Number/       GrandInt / BigFraction (with tests, examples, benchmarks)
RoslynMeow.Math.Graph/        graphs (with tests, examples)
RoslynMeow.Math.Bit/          bit utilities
.github/workflows/release.yml release pipeline
```

## Build & test

```bash
dotnet build RoslynMeow.Math.Fraction/Fraction.csproj -c Release
dotnet build RoslynMeow.Math.Number/BigNumber/BigNumber.csproj -c Release
dotnet build RoslynMeow.Math.Graph/GraphX/GraphX.csproj -c Release
dotnet build RoslynMeow.Math.Bit/Bit.csproj -c Release

dotnet test RoslynMeow.Math.Graph/GraphX.Tests/GraphX.Tests.csproj -c Release
dotnet test RoslynMeow.Math.Number/BigNumberTest/BigNumberTest.csproj -c Release
```

## Release

Tag `main` with `v*` and push to publish to GitHub Packages and create a GitHub Release automatically:

```bash
git tag v1.0.0-pre
git push origin v1.0.0-pre
```

The version is taken from the tag (leading `v` stripped); versions containing `-` are marked as pre-release.

## License

[MIT](LICENSE) © 2023 RoslynMeow
