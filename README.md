<!-- Copyright (c) Herman Schoenfeld 2018 - Present. Author: Herman Schoenfeld. Distributed under the MIT software license; see LICENSE. -->

# Sphere10.VisualRenderer

Build a visual document in C# and render it to HTML or plain text. An independent Sphere10 component (separate solution and NuGet package from [Framework](https://github.com/HermanSchoenfeld/Sphere10.Framework)), distributed under the [MIT License](LICENSE). Targets .NET 10. Independent of Framework, Notion, and LocalNotion.

![Version](https://img.shields.io/badge/version-3.1.3-blue)
![.NET](https://img.shields.io/badge/.NET-10.0-purple)
![License](https://img.shields.io/badge/license-MIT-green)

## Install

```shell
dotnet add package Sphere10.VisualRenderer
```

## Build, test, pack

```powershell
dotnet test "tests/Sphere10.VisualRenderer.Tests/Sphere10.VisualRenderer.Tests.csproj" -c Release
.\pack.ps1
.\publish.ps1 -IncludeSymbols -WhatIf
```

The package version is `VersionPrefix` in [Directory.Build.props](Directory.Build.props) (currently `3.1.3`). Packed output goes to `nuget-packages`.

## Documentation

- [Package README](src/Sphere10.VisualRenderer/README.md) — usage, CDN HTML examples, snippets, packing
- [Independent rendering](docs/visual-renderer/renderer.md)
- [Themes](docs/visual-renderer/renderer-themes.md)
- [Coverage](docs/visual-renderer/renderer-coverage.md)
