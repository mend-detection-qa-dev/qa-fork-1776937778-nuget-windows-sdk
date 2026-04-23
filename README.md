# nuget-windows-sdk

## Feature exercised

Exercises Mend SCA detection of Windows App SDK NuGet packages that use
non-standard, platform-specific version schemes (`1.5.240627000` date-encoded
and `10.0.22621.x` Windows SDK four-part builds) under a Windows-specific
target framework moniker (`net8.0-windows10.0.19041.0`).

## Expected dependency tree

### Direct dependencies

| Package | Version | Notes |
|---------|---------|-------|
| `Microsoft.WindowsAppSDK` | `1.5.240627000` | Date-encoded minor version segment |
| `Microsoft.Windows.SDK.BuildTools` | `10.0.22621.3233` | Four-part 10.x SDK version |
| `Microsoft.Windows.CsWinRT` | `2.0.7` | C#/WinRT interop |
| `Newtonsoft.Json` | `13.0.3` | Baseline non-Windows package |

### Transitive dependencies

| Package | Version | Brought in by |
|---------|---------|--------------|
| `Microsoft.WindowsAppSDK.Foundation` | `1.5.240627000` | `Microsoft.WindowsAppSDK` |
| `System.Memory` | `4.5.5` | `Microsoft.Windows.CsWinRT` |

### Resolution expectations

- Target framework: `net8.0-windows10.0.19041.0` (Windows-only TFM).
- All four direct packages must appear as top-level nodes.
- `Microsoft.WindowsAppSDK` must carry `Microsoft.Windows.SDK.BuildTools` and
  `Microsoft.Windows.CsWinRT` as children (they are both direct AND transitive;
  Mend should report them at the correct level — direct wins).
- `Microsoft.Windows.CsWinRT` must appear with child `System.Memory`.
- Version strings must be preserved exactly:
  - `1.5.240627000` — Mend must not truncate or normalise the date segment.
  - `10.0.22621.3233` — Mend must handle the four-part 10.x scheme.
- Source: `https://api.nuget.org/v3/index.json` (standard nuget.org feed).

## Probe metadata

```
pattern:       nuget-windows-sdk
target:        local
pm:            nuget
tfm:           net8.0-windows10.0.19041.0
created:       2026-04-22
purpose:       Validates Mend detects Windows App SDK packages with unusual
               version schemes (10.0.22621.x, 1.5.240627000) via standard
               NuGet PackageReference + packages.lock.json.
detection:     lockfile-driven (packages.lock.json present)
known-risk:    Mend may normalise 1.5.240627000 to 1.5.0 or drop the date
               segment; 10.0.22621.3233 may be matched as 10.0.22621.
```
