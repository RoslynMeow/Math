# RoslynMeow.Math

[中文](README.md) | [English](README.en.md)

[![Release](https://img.shields.io/github/v/release/RoslynMeow/Math?include_prereleases&sort=semver&label=release)](https://github.com/RoslynMeow/Math/releases)
[![Release workflow](https://github.com/RoslynMeow/Math/actions/workflows/release.yml/badge.svg)](https://github.com/RoslynMeow/Math/actions/workflows/release.yml)
[![License](https://img.shields.io/github/license/RoslynMeow/Math)](https://github.com/RoslynMeow/Math/blob/main/LICENSE)
[![.NET](https://img.shields.io/badge/.NET-netstandard2.0%20%7C%202.1-512BD4)](https://dotnet.microsoft.com)
[![Stars](https://img.shields.io/github/stars/RoslynMeow/Math?style=social)](https://github.com/RoslynMeow/Math/stargazers)

一组专注于**精确与任意精度**数值计算的 .NET 数学库,包含精确分数、任意精度整数/有理数、图算法与位操作工具。

## 包

| 包 | 目标框架 | 说明 |
|---|---|---|
| [`RoslynMeow.Math.Fraction`](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Fraction) | `netstandard2.0` | 基于两个 `BigInteger` 的精确分数 |
| [`RoslynMeow.Math.Number`](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Number) | `netstandard2.1` | 任意精度整数 `GrandInt` 与有理数 `BigFraction` |
| [`RoslynMeow.Math.Graph`](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Graph) | `netstandard2.0` | 图数据结构与算法:最短路、最小生成树、最大流、拓扑排序 |
| [`RoslynMeow.Math.Bit`](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Bit) | `netstandard2.0` | 位操作工具与基于 `Marshal` 的二进制报文 |

## 安装

包发布在 **GitHub Packages**。该 NuGet 源即使对公开包也需要认证,先在本机配置源与凭据:

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
      <!-- 需要带 read:packages 权限的 PAT -->
      <add key="ClearTextPassword" value="YOUR_GITHUB_PAT" />
    </github>
  </packageSourceCredentials>
</configuration>
```

```bash
dotnet add package RoslynMeow.Math.Fraction
```

## 目录结构

```
RoslynMeow.Math.Fraction/     精确分数库
RoslynMeow.Math.Number/       GrandInt / BigFraction(含测试、示例、基准)
RoslynMeow.Math.Graph/        图库(含测试、示例)
RoslynMeow.Math.Bit/          位操作库
.github/workflows/release.yml 打包发布流水线
```

## 构建与测试

```bash
dotnet build RoslynMeow.Math.Fraction/Fraction.csproj -c Release
dotnet build RoslynMeow.Math.Number/BigNumber/BigNumber.csproj -c Release
dotnet build RoslynMeow.Math.Graph/GraphX/GraphX.csproj -c Release
dotnet build RoslynMeow.Math.Bit/Bit.csproj -c Release

dotnet test RoslynMeow.Math.Graph/GraphX.Tests/GraphX.Tests.csproj -c Release
dotnet test RoslynMeow.Math.Number/BigNumberTest/BigNumberTest.csproj -c Release
```

## 发布

在 `main` 上打并推送 `v*` 标签即可发布到 GitHub Packages,并自动创建 GitHub Release:

```bash
git tag v1.0.0-pre
git push origin v1.0.0-pre
```

版本号由标签统一决定(去掉前导 `v`);版本中含 `-` 的会标记为预发布。

## License

[MIT](LICENSE) © 2023 RoslynMeow
