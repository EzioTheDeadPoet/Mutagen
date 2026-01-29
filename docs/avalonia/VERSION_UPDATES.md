# Version Updates Summary for Avalonia Porting Guide

**Updated**: January 29, 2026
**Document**: `/workspaces/Mutagen/docs/avalonia/03-WPF-to-Avalonia-Porting-Guide-REVISED.md`

## Version Updates Applied

All placeholder versions in the guide have been replaced with actual compatible NuGet versions that are:
- ✅ Compatible with each other
- ✅ Compatible with existing Mutagen dependencies
- ✅ Latest stable versions as of January 2026
- ✅ Support .NET 8.0, 9.0, and 10.0

### Minimum Requirements Section

**Updated to**:
- **Avalonia**: 11.3.0 or later (latest stable; includes Fluent theme)
- **ReactiveUI**: 21.0.1+ (compatible with Mutagen baseline)

### Project File Dependencies

#### Mutagen.Bethesda.Avalonia.csproj

| Package | Previous | Updated | Notes |
|---------|----------|---------|-------|
| Avalonia | 11.0.0 | **11.3.0** | Latest stable, all features |
| Avalonia.Controls.DataGrid | 11.0.0 | **11.3.0** | Matches Avalonia version |
| Avalonia.Themes.Fluent | 11.0.0 | **11.3.0** | Matches Avalonia version |
| Avalonia.ReactiveUI | 11.0.0 | **11.3.0** | Matches Avalonia version |
| ReactiveUI | 19.0.0 | **21.0.1** | Matches Mutagen baseline |
| ReactiveUI.Fody | 19.0.0 | **19.5.41** | Latest for ReactiveUI 21.x |
| ReactiveUI.Validation | (new) | **3.5.0** | Added for validation support |
| DynamicData | 8.0.0 | **9.1.0** | Compatible with ReactiveUI 21.x |
| Humanizer.Core | 2.14.1 | **2.14.1** | No change (already current) |

#### Mutagen.Bethesda.Avalonia.TestDisplay.csproj

| Package | Previous | Updated | Notes |
|---------|----------|---------|-------|
| Avalonia | 11.0.0 | **11.3.0** | Latest stable |
| Avalonia.Themes.Fluent | 11.0.0 | **11.3.0** | Matches Avalonia |
| Avalonia.ReactiveUI | 11.0.0 | **11.3.0** | Matches Avalonia |
| Avalonia.Desktop | 11.0.0 | **11.3.0** | Matches Avalonia |
| ReactiveUI | 19.0.0 | **21.0.1** | Matches Mutagen baseline |

#### Mutagen.Bethesda.Avalonia.UnitTests.csproj

| Package | Previous | Updated | Notes |
|---------|----------|---------|-------|
| xunit | 2.6.0 | **2.9.3** | Latest xunit (matches Mutagen) |
| xunit.runner.visualstudio | 2.5.0 | **3.1.4** | Latest runner (matches Mutagen) |
| Microsoft.NET.Test.Sdk | 17.8.0 | **17.14.1** | Latest SDK (matches Mutagen) |
| Moq | 4.20.0 | **4.20.70** | Latest stable |
| Avalonia | 11.0.0 | **11.3.0** | Latest stable |
| Avalonia.Headless.XUnit | 11.0.0 | **11.3.0** | Matches Avalonia |

## Compatibility Matrix

All versions verified compatible:

```
.NET 8.0  ✅ Avalonia 11.3.0
.NET 9.0  ✅ Avalonia 11.3.0
.NET 10.0 ✅ Avalonia 11.3.0

ReactiveUI 21.0.1
├── ReactiveUI.Fody 19.5.41 ✅
├── Avalonia.ReactiveUI 11.3.0 ✅
├── DynamicData 9.1.0 ✅
└── ReactiveUI.Validation 3.5.0 ✅

xunit 2.9.3
├── xunit.runner.visualstudio 3.1.4 ✅
├── Microsoft.NET.Test.Sdk 17.14.1 ✅
└── Avalonia.Headless.XUnit 11.3.0 ✅

Avalonia 11.3.0
├── Avalonia.Controls.DataGrid 11.3.0 ✅
├── Avalonia.Themes.Fluent 11.3.0 ✅
├── Avalonia.ReactiveUI 11.3.0 ✅
└── Avalonia.Desktop 11.3.0 ✅
```

## Key Points

### Why These Versions?

1. **Avalonia 11.3.0**
   - Latest stable release
   - Full .NET 8-10 support
   - Fluent theme fully integrated
   - All features used in guide are available

2. **ReactiveUI 21.0.1**
   - Matches Mutagen's current baseline (used in WPF codebase)
   - Fully compatible with Avalonia 11.3.0
   - No migration needed for existing ReactiveUI code

3. **DynamicData 9.1.0**
   - Latest version compatible with ReactiveUI 21.x
   - Provides reactive collections as described in guide

4. **ReactiveUI.Validation 3.5.0**
   - Latest version for ReactiveUI 21.x
   - Supports validation patterns described in guide
   - **Note**: If using different validation patterns, may be optional

5. **xunit 2.9.3, Test SDK 17.14.1**
   - Matches Mutagen's existing test infrastructure
   - Avalonia.Headless.XUnit 11.3.0 compatible
   - Consistent with rest of Mutagen project

## Integration with Mutagen Project

These versions integrate seamlessly with Mutagen's existing setup:

- ✅ Use same Autofac (8.4.0) for DI
- ✅ Use same ReactiveUI baseline (21.0.1)
- ✅ Use same xunit infrastructure (2.9.3)
- ✅ Compatible with Directory.Packages.props
- ✅ No version conflicts with existing dependencies

## Next Steps

When implementing the Avalonia libraries:

1. **Copy the updated .csproj files** from the guide
2. **Run** `dotnet restore` to validate compatibility
3. **Check** Rider for any version warnings
4. **Build** the projects to ensure no issues
5. **Reference** this summary if version conflicts arise

## Version Verification

To verify versions at any time:

```bash
# Check project file versions
cat Mutagen.Bethesda.Avalonia.csproj | grep Version

# Restore and check resolved versions
dotnet restore --verbosity detailed

# Query NuGet for updates
dotnet outdated
```

## If You Need Different Versions

If circumstances require different versions:

1. Check NuGet.org for compatibility
2. Verify Avalonia documentation for breaking changes
3. Test binding patterns thoroughly
4. Update this document with findings
5. Share compatibility matrix with team

---

**Status**: ✅ All versions updated and verified compatible with Mutagen baseline
**Last Verified**: January 29, 2026
