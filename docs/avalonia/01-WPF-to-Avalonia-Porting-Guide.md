# WPF to Avalonia: Comprehensive Porting Guide for Mutagen UI Libraries

## Document Purpose

This guide provides concrete, actionable steps for converting WPF-based Mutagen UI libraries (`Mutagen.Bethesda.WPF*`) to Avalonia equivalents (`Mutagen.Bethesda.Avalonia*`). It assumes:

- **IDE**: JetBrains Rider
- **.NET SDK**: 10.0 (with compatibility down to .NET 8.0)
- **Primary platform**: Windows (with future cross-platform capability)
- **FOSS libraries only**: No commercial dependencies

This guide emphasizes feature parity while leveraging Avalonia-specific improvements where applicable.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Project Structure Setup](#project-structure-setup)
3. [Dependency Migration](#dependency-migration)
4. [Control Porting Strategy](#control-porting-strategy)
5. [MVVM and Data Binding](#mvvm-and-data-binding)
6. [Custom Control Implementation](#custom-control-implementation)
7. [Styling and Theming](#styling-and-theming)
8. [TestDisplay Application](#testdisplay-application)
9. [Unit Testing Strategy](#unit-testing-strategy)
10. [Optimization Opportunities](#optimization-opportunities)

---

## Architecture Overview

### Key Differences from WPF

| Aspect | WPF | Avalonia | Implication |
|--------|-----|---------|------------|
| **Platform** | Windows-only | Cross-platform | Use platform-agnostic interfaces |
| **Event system** | Routed events | Direct events + observables | Prefer ReactiveUI streams |
| **Controls** | Extensive native set | Curated set | Use composition; community libraries |
| **Styling** | XAML-based | XAML + CSS-like | Use both; CSS for dynamic themes |
| **Base classes** | FrameworkElement | Visual | Similar but lighter API |
| **Binding** | DP-integrated | Property-agnostic | More flexible; fewer assumptions |

### High-Level Structure

```
Mutagen.Bethesda.Avalonia/
├── Controls/
│   ├── Pickers/          (FormKeyPicker, ModKeyPicker, etc.)
│   ├── Common/           (Shared base classes, behaviors)
│   └── Utils/            (Converters, helpers)
├── Reflection/           (Auto-generated UI from metadata)
├── Styles/               (Themes, resource dictionaries)
└── Mutagen.Bethesda.Avalonia.csproj

Mutagen.Bethesda.Avalonia.TestDisplay/
├── ViewModels/
├── Views/
├── App.xaml / App.xaml.cs
└── Mutagen.Bethesda.Avalonia.TestDisplay.csproj

Mutagen.Bethesda.Avalonia.UnitTests/
├── Controls/             (Control-specific tests)
├── Reflection/           (Reflection generation tests)
└── Mutagen.Bethesda.Avalonia.UnitTests.csproj
```

---

## Project Structure Setup

### Step 1: Create Project Files

#### Mutagen.Bethesda.Avalonia.csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net8.0;net9.0;net10.0</TargetFrameworks>
    <IsPackable>true</IsPackable>
    <DebugType>portable</DebugType>
    <DebugSymbols>true</DebugSymbols>
    <PublishRepositoryUrl>true</PublishRepositoryUrl>
    <EmbedUntrackedSources>true</EmbedUntrackedSources>
    <IncludeSymbols>true</IncludeSymbols>
    <SymbolPackageFormat>snupkg</SymbolPackageFormat>
    <Description>A C# library for Mutagen/Bethesda related Avalonia controls and styling</Description>
    <WarningsAsErrors>nullable</WarningsAsErrors>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <LangVersion>latest</LangVersion>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Avalonia" />
    <PackageReference Include="Avalonia.Controls.DataGrid" />
    <PackageReference Include="Avalonia.Themes.Fluent" />
    <PackageReference Include="Avalonia.ReactiveUI" />
    <PackageReference Include="ReactiveUI" />
    <PackageReference Include="ReactiveUI.Fody" />
    <PackageReference Include="DynamicData" />
    <PackageReference Include="Humanizer.Core" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\Mutagen.Bethesda.Core\Mutagen.Bethesda.Core.csproj" />
    <ProjectReference Include="..\Mutagen.Bethesda.Kernel\Mutagen.Bethesda.Kernel.csproj" />
  </ItemGroup>
</Project>
```

Key differences from WPF:
- No `UseWPF` property
- Add `Avalonia.ReactiveUI` for ReactiveUI integration
- Use `Avalonia.Themes.Fluent` (open-source, modern alternative to MahApps)
- Target same .NET versions as WPF

#### Mutagen.Bethesda.Avalonia.TestDisplay.csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFrameworks>net8.0;net9.0;net10.0</TargetFrameworks>
    <DebugType>portable</DebugType>
    <LangVersion>latest</LangVersion>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Avalonia" />
    <PackageReference Include="Avalonia.Themes.Fluent" />
    <PackageReference Include="Avalonia.ReactiveUI" />
    <PackageReference Include="Avalonia.Desktop" />
    <PackageReference Include="ReactiveUI" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\Mutagen.Bethesda.Autofac\Mutagen.Bethesda.Autofac.csproj" />
    <ProjectReference Include="..\Mutagen.Bethesda.Avalonia\Mutagen.Bethesda.Avalonia.csproj" />
    <ProjectReference Include="..\Mutagen.Bethesda\Mutagen.Bethesda.csproj" />
  </ItemGroup>
</Project>
```

Note: Avalonia requires `Avalonia.Desktop` for desktop app functionality.

#### Mutagen.Bethesda.Avalonia.UnitTests.csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net8.0;net9.0;net10.0</TargetFrameworks>
    <IsPackable>false</IsPackable>
    <LangVersion>latest</LangVersion>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="xunit" />
    <PackageReference Include="xunit.runner.visualstudio" />
    <PackageReference Include="Microsoft.NET.Test.Sdk" />
    <PackageReference Include="Moq" />
    <PackageReference Include="Avalonia" />
    <PackageReference Include="Avalonia.Headless.XUnit" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\Mutagen.Bethesda.Avalonia\Mutagen.Bethesda.Avalonia.csproj" />
    <ProjectReference Include="..\Mutagen.Bethesda.UnitTests\Mutagen.Bethesda.UnitTests.csproj" />
  </ItemGroup>
</Project>
```

Note: `Avalonia.Headless.XUnit` provides testing infrastructure for Avalonia controls without display.

---

## Dependency Migration

### Direct Replacements

| WPF Dependency | Avalonia Equivalent | Purpose |
|---|---|---|
| `Noggog.WPF` | Custom base classes | Control base functionality |
| `Extended.Wpf.Toolkit` | `Avalonia.Controls.DataGrid` | Data grid control |
| `MahApps.Metro` | `Avalonia.Themes.Fluent` | Theming framework |
| `ReactiveUI.Wpf` | `Avalonia.ReactiveUI` | ReactiveUI integration |

### Noggog.WPF Replacement Strategy

**Challenge**: Noggog.WPF provides `NoggogControl` base class and utility methods. No direct Avalonia equivalent.

**Solution**: Create `Mutagen.Bethesda.Avalonia.Common` namespace with local implementations:

```csharp
// Mutagen.Bethesda.Avalonia/Common/AvaloniaControlBase.cs
using Avalonia.Controls;

namespace Mutagen.Bethesda.Avalonia.Common
{
    /// <summary>
    /// Base class for Mutagen custom controls.
    /// Replaces Noggog.WPF's NoggogControl.
    /// </summary>
    public class AvaloniaControlBase : Control
    {
        // Framework methods can be overridden here
        // Use composition instead of behavior-based approach
    }
}
```

**Key principle**: Replace Noggog utilities with Avalonia's native features:
- **Drag-drop**: Use Avalonia's `DragDrop` class directly
- **Focus management**: Use Avalonia's focus API
- **Resource constants**: Create local resource dictionaries

### Dependency Decision Tree

```
For each WPF NuGet dependency:
├── Is it framework-specific (WPF)?
│   └── YES → Need Avalonia replacement or local implementation
├── Is it framework-agnostic (e.g., Humanizer)?
│   └── YES → Use as-is
└── Is it domain-specific (e.g., Mutagen.Bethesda.Core)?
    └── YES → Use as-is (already done)
```

---

## Control Porting Strategy

### Phase 1: Identify Reusable Components

**Categories of WPF controls to port**:

1. **Custom pickers** (highest priority):
   - `FormKeyPicker`, `AFormKeyPicker`
   - `ModKeyPicker`, `AModKeyPicker`
   - `FormKeyBox`, `ModKeyBox`
   - `FormKeyMultiPicker`, `ModKeyMultiPicker`

2. **Integration points**:
   - Plugin order controls
   - Reflection-driven settings UI
   - Data validation overlays

3. **Support utilities**:
   - Value converters
   - Validation behaviors
   - Drag-drop helpers

### Phase 2: Template to Binding Conversion

**WPF Pattern** (TemplatePart-based):
```csharp
[TemplatePart(Name = "PART_FileNameBox", Type = typeof(TextBox))]
public class ModKeyBox : NoggogControl
{
    private TextBox? _fileNameBox;

    public override void OnApplyTemplate()
    {
        base.OnApplyTemplate();
        _fileNameBox = GetTemplateChild("PART_FileNameBox") as TextBox;
        if (_fileNameBox != null)
        {
            // Wire up events
            _fileNameBox.TextChanged += OnFileNameTextChanged;
        }
    }
}
```

**Avalonia Pattern** (Binding-first approach):
```csharp
public class ModKeyBox : UserControl
{
    public static readonly StyledProperty<string?> FileNameProperty =
        AvaloniaProperty.Register<ModKeyBox, string?>(nameof(FileName));

    public string? FileName
    {
        get => GetValue(FileNameProperty);
        set => SetValue(FileNameProperty, value);
    }

    public ModKeyBox()
    {
        InitializeComponent();
    }
}
```

**XAML**:
```xml
<UserControl x:Class="Mutagen.Bethesda.Avalonia.ModKeyBox">
    <TextBox Text="{Binding FileName, RelativeSource={RelativeSource AncestorType=local:ModKeyBox}}"/>
</UserControl>
```

**Why this matters**:
- Avalonia encourages binding over template lookup
- Simpler, more testable, less template-fragile
- Easier to override behavior in derived classes

### Phase 3: Implement Pickers with Composition

**Example: FormKeyPicker Porting**

WPF approach relies on:
- `NoggogControl` base class
- `ReactiveUI` for ViewModel binding
- Routed events for selection changed

Avalonia approach:
```csharp
namespace Mutagen.Bethesda.Avalonia.Plugins
{
    /// <summary>
    /// Picker control for selecting FormKeys with search capability.
    /// </summary>
    public partial class FormKeyPicker : UserControl
    {
        public static readonly StyledProperty<IObservable<FormKey>?> SelectedFormKeyProperty =
            AvaloniaProperty.Register<FormKeyPicker, IObservable<FormKey>?>(
                nameof(SelectedFormKey));

        public IObservable<FormKey>? SelectedFormKey
        {
            get => GetValue(SelectedFormKeyProperty);
            set => SetValue(SelectedFormKeyProperty, value);
        }

        public FormKeyPicker()
        {
            InitializeComponent();
        }
    }
}
```

**Key pattern**: 
- Use properties for configuration
- Use observables for state changes (aligns with ReactiveUI)
- Bind directly in XAML; avoid code-behind wiring

### Phase 4: ListBox Drag-Drop Implementation

**WPF** uses `Noggog.WPF.Drag.ListBoxDragDrop<T>()` helper.

**Avalonia** built-in drag-drop:
```csharp
public partial class ModKeyMultiPicker : UserControl
{
    public ModKeyMultiPicker()
    {
        InitializeComponent();
        InitializeDragDrop();
    }

    private void InitializeDragDrop()
    {
        var listBox = this.FindControl<ListBox>("ModKeyListBox");
        if (listBox != null)
        {
            DragDrop.SetAllowDrop(listBox, true);
            listBox.AddHandler(DragDrop.DropEvent, OnDrop);
            listBox.AddHandler(DragDrop.DragOverEvent, OnDragOver);
        }
    }

    private void OnDrop(object? sender, DragEventArgs e)
    {
        // Reorder items based on drop position
        if (e.Data.Contains(DataFormats.Text))
        {
            var text = e.Data.GetText();
            // Handle reordering logic
        }
    }

    private void OnDragOver(object? sender, DragEventArgs e)
    {
        e.DragEffects = DragDropEffects.Move;
    }
}
```

**Advantage**: No external dependency; simpler code.

---

## MVVM and Data Binding

### ReactiveUI Integration

**Good news**: ReactiveUI works seamlessly with Avalonia. The WPF `ReactiveUI.Wpf` NuGet simply becomes `ReactiveUI.Avalonia`.

**Pattern for ViewModels** (unchanged from WPF):

```csharp
using ReactiveUI;
using ReactiveUI.Fody.Helpers;

namespace Mutagen.Bethesda.Avalonia.Plugins
{
    public class FormKeyPickerVM : ReactiveObject
    {
        [Reactive]
        public FormKey? SelectedFormKey { get; set; }

        [Reactive]
        public string SearchText { get; set; } = string.Empty;

        private readonly ObservableAsPropertyHelper<IEnumerable<FormKey>> _filteredKeys;
        public IEnumerable<FormKey> FilteredKeys => _filteredKeys.Value;

        public FormKeyPickerVM()
        {
            _filteredKeys = this
                .WhenAnyValue(x => x.SearchText, x => x.SelectedFormKey)
                .Select(FilterKeys)
                .ToProperty(this, x => x.FilteredKeys);
        }

        private IEnumerable<FormKey> FilterKeys(string search, FormKey? selected)
        {
            // Return filtered results
            return [];
        }
    }
}
```

**Pattern for Views** (nearly identical):

```csharp
using Avalonia.Controls;
using Avalonia.ReactiveUI;
using ReactiveUI;

namespace Mutagen.Bethesda.Avalonia.Plugins
{
    public partial class FormKeyPickerView : ReactiveUserControl<FormKeyPickerVM>
    {
        public FormKeyPickerView()
        {
            InitializeComponent();
            this.WhenActivated(disposable =>
            {
                // Bind view to viewmodel
                this.Bind(ViewModel, x => x.SelectedFormKey, v => v.SelectedKeyTextBox.Text)
                    .DisposeWith(disposable);
            });
        }
    }
}
```

**Differences from WPF**:
- No `NoggogUserControl<T>` base; use `ReactiveUserControl<T>` directly
- `DisposeWith()` works identically
- `WhenActivated()` ensures bindings only active when view is displayed

### Binding Syntax Cheat Sheet

| Scenario | XAML |
|----------|------|
| Simple property binding | `{Binding PropertyName}` |
| Relative source | `{Binding PropertyName, RelativeSource={RelativeSource AncestorType=local:ControlType}}` |
| Converter | `{Binding PropertyName, Converter={StaticResource MyConverter}}` |
| String formatting | `{Binding PropertyName, StringFormat='Key: {0}'}`† |
| Two-way binding | `{Binding PropertyName, Mode=TwoWay}` |

†**Note**: Avalonia doesn't have `StringFormat`. Use a converter instead:
```csharp
public class FormKeyConverter : IValueConverter
{
    public object? Convert(object? value, Type targetType, object? parameter, CultureInfo? culture)
    {
        return value is FormKey fk ? $"Key: {fk}" : null;
    }

    public object? ConvertBack(object? value, Type targetType, object? parameter, CultureInfo? culture)
        => value is string s ? FormKey.TryFactory(s, out var fk) ? fk : null : null;
}
```

---

## Custom Control Implementation

### Template System

**WPF** relies on XAML templates applied at runtime via `Template` property.

**Avalonia** also uses templates but with different lookup semantics. Create templates inline:

```xml
<UserControl x:Class="Mutagen.Bethesda.Avalonia.Plugins.ModKeyBox"
             xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Design.DataContext>
        <local:ModKeyBoxVM/>
    </Design.DataContext>

    <!-- Template directly in user control -->
    <Grid ColumnDefinitions="*,Auto">
        <TextBox Grid.Column="0" 
                 Text="{Binding FileName}"
                 Watermark="Enter mod key..."/>
        <Button Grid.Column="1" Content="Browse"/>
    </Grid>
</UserControl>
```

**Advantages**:
- No template lookup; control structure is obvious
- Designer preview works better
- Easier to refactor

### Attached Behaviors

For complex interactive behavior, use attached properties instead of code-behind:

```csharp
namespace Mutagen.Bethesda.Avalonia.Common
{
    public static class SelectionBehavior
    {
        public static bool GetSelectOnGotFocus(TextBox textBox)
            => textBox.GetValue(SelectOnGotFocusProperty);

        public static void SetSelectOnGotFocus(TextBox textBox, bool value)
            => textBox.SetValue(SelectOnGotFocusProperty, value);

        public static readonly AttachedProperty<bool> SelectOnGotFocusProperty =
            AvaloniaProperty.RegisterAttached<TextBox, bool>(
                "SelectOnGotFocus",
                false,
                false,
                (textBox, value) =>
                {
                    if (value)
                    {
                        textBox.GotFocus += (s, e) => textBox.SelectAll();
                    }
                });
    }
}
```

**XAML usage**:
```xml
<TextBox local:SelectionBehavior.SelectOnGotFocus="True"/>
```

### Validation Integration

Unlike WPF's deep validation system, Avalonia uses data annotations:

```csharp
using System.ComponentModel.DataAnnotations;

public class FormKeyInputVM : ReactiveValidationObject
{
    [Required(ErrorMessage = "FormKey is required")]
    [CustomValidation(typeof(FormKeyValidator), nameof(FormKeyValidator.ValidateFormKey))]
    [Reactive]
    public string FormKeyText { get; set; } = string.Empty;

    public FormKeyInputVM()
    {
        this.ValidationRule(
            x => x.FormKeyText,
            formKey => FormKey.TryFactory(formKey, out _),
            "Invalid FormKey format");
    }
}
```

Use `ReactiveValidationObject` from `ReactiveUI.Validation` NuGet.

---

## Styling and Theming

### Moving from MahApps to Fluent

**WPF** uses MahApps.Metro for theming (no longer actively maintained).

**Avalonia** provides `Avalonia.Themes.Fluent` (open-source, Microsoft-inspired).

#### Step 1: Create Fluent Theme Override

```xml
<!-- Mutagen.Bethesda.Avalonia/Styles/Overrides.xaml -->
<Styles xmlns="https://github.com/avaloniaui">
    <Styles.Resources>
        <!-- Define custom colors -->
        <Color x:Key="MutagenPrimary">#FF0078D4</Color>
        <Color x:Key="MutagenAccent">#FFFF8C00</Color>
    </Styles.Resources>

    <Style Selector="TextBox">
        <Setter Property="Padding" Value="8"/>
    </Style>

    <Style Selector="Button.Primary">
        <Setter Property="Background" Value="{StaticResource MutagenAccent}"/>
        <Setter Property="Foreground" Value="White"/>
    </Style>
</Styles>
```

#### Step 2: Apply in App.xaml

```xml
<Application x:Class="Mutagen.Bethesda.Avalonia.TestDisplay.App"
             xmlns="https://github.com/avaloniaui">
    <Application.Styles>
        <FluentTheme Mode="Dark"/>
        <StyleInclude Source="avares://Mutagen.Bethesda.Avalonia/Styles/Overrides.xaml"/>
    </Application.Styles>
</Application>
```

#### Step 3: Runtime Theme Switching

**Implement theme provider** (Avalonia doesn't support theme switching out-of-the-box):

```csharp
public interface IThemeProvider
{
    void SetTheme(ThemeMode mode);
}

public enum ThemeMode
{
    Light,
    Dark,
    Auto
}

public class AvaloniaThemeProvider : IThemeProvider
{
    public void SetTheme(ThemeMode mode)
    {
        var app = Application.Current!;
        app.RequestedThemeVariant = mode switch
        {
            ThemeMode.Light => ThemeVariant.Light,
            ThemeMode.Dark => ThemeVariant.Dark,
            ThemeMode.Auto => ThemeVariant.Default,
            _ => ThemeVariant.Default
        };
    }
}
```

### Color Palette

Replace WPF's Noggog resource constants:

```xml
<!-- Mutagen.Bethesda.Avalonia/Styles/Colors.xaml -->
<Styles xmlns="https://github.com/avaloniaui">
    <Styles.Resources>
        <!-- Primary palette -->
        <Color x:Key="PrimaryColor">#FF0078D4</Color>
        <Color x:Key="AccentColor">#FFFF8C00</Color>
        <Color x:Key="WarningColor">#FFFFC107</Color>
        <Color x:Key="ErrorColor">#FFF44336</Color>

        <!-- Semantic colors -->
        <SolidColorBrush x:Key="PrimaryBrush" Color="{StaticResource PrimaryColor}"/>
        <SolidColorBrush x:Key="AccentBrush" Color="{StaticResource AccentColor}"/>
    </Styles.Resources>
</Styles>
```

---

## TestDisplay Application

### Project Structure

```
Mutagen.Bethesda.Avalonia.TestDisplay/
├── App.xaml
├── App.xaml.cs
├── MainWindow.xaml
├── MainWindow.xaml.cs
├── MainVM.cs (ViewModel)
├── Views/
│   ├── FormKeyPickerTestView.xaml
│   ├── ModKeyPickerTestView.xaml
│   └── ...
├── ViewModels/
│   ├── FormKeyPickerTestVM.cs
│   ├── ModKeyPickerTestVM.cs
│   └── ...
└── Mutagen.Bethesda.Avalonia.TestDisplay.csproj
```

### Bootstrap Application

**App.xaml.cs**:
```csharp
using Avalonia;
using Avalonia.Markup.Xaml;
using Autofac;
using Mutagen.Bethesda.Avalonia.TestDisplay.ViewModels;

namespace Mutagen.Bethesda.Avalonia.TestDisplay;

public partial class App : Application
{
    public override void Initialize()
    {
        AvaloniaXamlLoader.Load(this);
    }

    public override void OnFrameworkInitializationCompleted()
    {
        var builder = new ContainerBuilder();
        RegisterServices(builder);
        var container = builder.Build();

        MainWindow = new MainWindow
        {
            DataContext = container.Resolve<MainVM>()
        };

        base.OnFrameworkInitializationCompleted();
    }

    private void RegisterServices(ContainerBuilder builder)
    {
        builder.RegisterType<MainVM>();
        // Register other ViewModels and services
    }
}
```

**Program.cs** (entry point):
```csharp
using Avalonia;
using Avalonia.ReactiveUI;

namespace Mutagen.Bethesda.Avalonia.TestDisplay;

class Program
{
    [STAThread]
    public static void Main(string[] args) => BuildAvaloniaApp()
        .StartWithClassicDesktopLifetime(args);

    public static AppBuilder BuildAvaloniaApp()
        => AppBuilder.Configure<App>()
            .UsePlatformDetect()
            .LogToTrace()
            .UseReactiveUI();
}
```

### Test View Pattern

```csharp
// ViewModels/FormKeyPickerTestVM.cs
using ReactiveUI;
using ReactiveUI.Fody.Helpers;

public class FormKeyPickerTestVM : ReactiveObject
{
    [Reactive]
    public FormKey? SelectedFormKey { get; set; }

    [Reactive]
    public IEnumerable<FormKey> AvailableFormKeys { get; set; } = [];
}
```

```xml
<!-- Views/FormKeyPickerTestView.xaml -->
<UserControl x:Class="Mutagen.Bethesda.Avalonia.TestDisplay.Views.FormKeyPickerTestView"
             xmlns="https://github.com/avaloniaui"
             xmlns:local="clr-namespace:Mutagen.Bethesda.Avalonia.Plugins">
    <Grid RowDefinitions="Auto,*">
        <TextBlock Grid.Row="0" Text="FormKey Picker Test"/>
        <local:FormKeyPicker Grid.Row="1" 
                            SelectedFormKey="{Binding SelectedFormKey}"
                            AvailableFormKeys="{Binding AvailableFormKeys}"/>
    </Grid>
</UserControl>
```

---

## Unit Testing Strategy

### Test Structure

```
Mutagen.Bethesda.Avalonia.UnitTests/
├── Controls/
│   ├── FormKeyPickerTests.cs
│   ├── ModKeyPickerTests.cs
│   └── ModKeyBoxTests.cs
├── Reflection/
│   ├── SettingsGenerationTests.cs
│   └── ControlMappingTests.cs
├── Integration/
│   └── TestDisplayIntegrationTests.cs
└── Fixtures/
    ├── AvaloniaTestFixture.cs
    └── DataFixture.cs
```

### Headless Testing

Use `Avalonia.Headless.XUnit` for UI testing without a display:

```csharp
using Avalonia.Headless.XUnit;
using Xunit;
using Mutagen.Bethesda.Avalonia.Plugins;

namespace Mutagen.Bethesda.Avalonia.UnitTests.Controls;

public class ModKeyBoxTests
{
    [AvaloniaFact]
    public void SelectedModKey_Changed_RaisesNotification()
    {
        // Arrange
        var control = new ModKeyBox();
        var changed = false;
        control.PropertyChanged += (s, e) =>
        {
            if (e.PropertyName == nameof(ModKeyBox.SelectedModKey))
                changed = true;
        };

        // Act
        control.SelectedModKey = new ModKey("MyMod", ModType.Master);

        // Assert
        Assert.True(changed);
        Assert.Equal("MyMod", control.SelectedModKey?.Name);
    }

    [AvaloniaFact]
    public void Binding_TwoWay_UpdatesViewModel()
    {
        // Arrange
        var vm = new ModKeyBoxVM();
        var control = new ModKeyBox { DataContext = vm };
        var binding = new Binding(nameof(ModKeyBoxVM.SelectedModKey))
        {
            Mode = BindingMode.TwoWay,
            Source = vm
        };
        control.Bind(ModKeyBox.SelectedModKeyProperty, binding);

        // Act
        control.SelectedModKey = new ModKey("TestMod", ModType.Master);

        // Assert
        Assert.NotNull(vm.SelectedModKey);
        Assert.Equal("TestMod", vm.SelectedModKey.Name);
    }
}
```

### Integration Testing

Test complete workflows using the TestDisplay app:

```csharp
public class FormKeyPickerIntegrationTests
{
    [AvaloniaFact]
    public async Task Picker_SearchAndSelect_FiltersAndReturnsResult()
    {
        // Arrange
        var vm = new FormKeyPickerTestVM
        {
            AvailableFormKeys = new[]
            {
                new FormKey(ModKey.Master, 0x001),
                new FormKey(ModKey.Master, 0x002)
            }
        };
        var view = new FormKeyPickerTestView { DataContext = vm };

        // Act
        vm.SearchText = "0x001";
        await Task.Delay(100); // Allow filter to complete

        // Assert
        Assert.Single(vm.FilteredKeys);
        Assert.Equal(0x001U, vm.FilteredKeys.First().ID);
    }
}
```

### Test Fixtures

Create reusable test helpers:

```csharp
public class AvaloniaTestFixture : IAsyncLifetime
{
    public async Task InitializeAsync()
    {
        if (!AppBuilder.Instance.IsRunning)
        {
            AppBuilder.Configure<TestApp>()
                .UsePlatformDetect()
                .UseReactiveUI()
                .StartWithClassicDesktopLifetime([]);
        }
    }

    public Task DisposeAsync() => Task.CompletedTask;
}

[Collection("Avalonia")]
public class AvaloniaTestCollectionFixture : ICollectionFixture<AvaloniaTestFixture>
{
    // All tests using this will share Avalonia initialization
}
```

---

## Optimization Opportunities

### 1. Reactive Streams Over Events

**Benefit**: Avalonia + ReactiveUI excel with observable streams.

**Example**: Instead of event handlers, use observables for validation:

```csharp
public class FormKeyInputVM : ReactiveValidationObject
{
    [Reactive]
    public string FormKeyText { get; set; } = string.Empty;

    public IObservable<bool> IsValid { get; }

    public FormKeyInputVM()
    {
        IsValid = this
            .WhenAnyValue(x => x.FormKeyText)
            .Select(text => FormKey.TryFactory(text, out _))
            .ShareLatest();
    }
}
```

This replaces WPF's validation event pattern with reactive flow, which is:
- More composable
- Easier to test
- More performant (avoids event listener chaining)

### 2. DynamicData for List Operations

**Benefit**: `DynamicData` NuGet provides reactive collections with built-in filtering, sorting, and paging.

```csharp
public class ModListVM : ReactiveObject
{
    private readonly SourceList<ModKey> _modKeys = new();

    public IObservableCollection<ModKey> FilteredMods { get; }

    public ModListVM()
    {
        FilteredMods = _modKeys
            .Connect()
            .Filter(this.WhenAnyValue(x => x.SearchText)
                .Throttle(TimeSpan.FromMilliseconds(300))
                .Select<string, Func<ModKey, bool>>(search =>
                    mk => mk.Name.Contains(search, StringComparison.OrdinalIgnoreCase)))
            .AsObservableCollection();
    }

    public void AddMod(ModKey mod) => _modKeys.Add(mod);
}
```

**Why use DynamicData over ObservableCollection?**
- Native support for reactive filtering/sorting
- Better performance for large lists
- Easier to combine with other streams

### 3. Virtualized Lists for Performance

For large lists (100+ items), use virtualization:

```xml
<ListBox x:Name="ModKeyList" 
         Items="{Binding FilteredMods}"
         VirtualizationMode="Recycling">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Name}"/>
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```

Avalonia supports virtualization out-of-the-box; no special configuration needed.

### 4. Lazy Initialization for Heavy Views

Defer creation of complex controls until needed:

```xml
<ContentControl Content="{Binding CurrentView, Mode=OneWay}"/>
```

The ViewModel can swap views on-demand:

```csharp
public IObservable<Control> CurrentView { get; }

public SettingsVM()
{
    CurrentView = this
        .WhenAnyValue(x => x.SelectedTab)
        .Select(tab => tab switch
        {
            "Advanced" => new LazyAsync(() => CreateAdvancedView()),
            _ => new SimplifiedView()
        });
}
```

### 5. String Localization Integration

Avalonia has no built-in localization. Use a pattern like:

```csharp
public interface ILocalizer
{
    string GetString(string key, CultureInfo? culture = null);
}

public class AvaloniaLocalizer : ILocalizer
{
    public string GetString(string key, CultureInfo? culture = null)
    {
        // Load from resource file
        return ResourceManager.GetString(key, culture ?? CultureInfo.CurrentCulture) ?? key;
    }
}
```

Register in DI and use in ViewModels for culture-agnostic UI strings.

---

## Best Practices Summary

| Practice | Rationale |
|----------|-----------|
| **Prefer composition over inheritance** | Avalonia controls simpler than WPF; composition more flexible |
| **Use observables instead of events** | ReactiveUI + Avalonia designed for this pattern |
| **Bind in XAML, not code-behind** | Cleaner, more testable, better designer support |
| **Test ViewModels, not Views** | ViewModels are framework-agnostic and easier to test |
| **Isolate platform-specific code** | Avalonia enables future cross-platform; prepare for it |
| **Document Mutagen-specific controls** | Users expect same API as WPF versions |
| **Leverage DynamicData for lists** | Massively more powerful than ObservableCollection |
| **Use headless testing for controls** | Faster, more reliable than desktop testing |

---

## Migration Checklist

- [ ] Projects created with correct .csproj files
- [ ] All FOSS dependencies added (no commercial libraries)
- [ ] Custom base classes created to replace Noggog.WPF
- [ ] FormKey/ModKey pickers ported with full feature parity
- [ ] Multi-picker drag-drop implemented natively
- [ ] Reflection-driven UI controls ported
- [ ] Fluent theme applied with custom color overrides
- [ ] TestDisplay application runs without errors
- [ ] Unit tests for all custom controls pass
- [ ] Integration tests verify control workflows
- [ ] Documentation for Mutagen-specific patterns written
- [ ] Performance profiling shows acceptable responsiveness
- [ ] Cross-platform code paths tested (Windows primary)

---

## Next Steps

1. **Create the projects** following the structure in Section 2
2. **Implement core pickers** as described in Section 4-6
3. **Port styling** using patterns in Section 7
4. **Build TestDisplay** following Section 8 template
5. **Write unit tests** using Section 9 examples
6. **Validate with integration tests** across all components
7. **Document Avalonia-specific patterns** in Mutagen docs
8. **Prepare for community contribution** with clear guidelines

