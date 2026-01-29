# WPF to Avalonia: Critical Issues Assessment

## Document Purpose
This assessment identifies critical technical differences between WPF and Avalonia UI frameworks that affect the porting of Mutagen.Bethesda.WPF libraries to Avalonia equivalents. It focuses on concrete challenges based on how these frameworks differ architecturally, not on theoretical differences alone.

---

## Executive Summary

Avalonia and WPF share a common philosophy but differ significantly in:
- **Platform support** (Avalonia is cross-platform; WPF is Windows-only)
- **Control templating mechanism** (explicit vs implicit systems)
- **Event system** (Routed vs Direct events)
- **Dependency property implementation** (simplified in Avalonia)
- **Resource dictionaries** (different merging behavior)
- **Styling and theming** (CSS-like vs XAML-based)
- **Third-party ecosystem** (fewer controls available natively)

These differences do NOT prevent feature parity, but they require deliberate porting strategies.

---

## Critical Issue Categories

### 1. XAML and Control Templating

#### The Problem
**WPF**: Uses `TemplateBinding` and `TemplatePart` attributes with routed events and explicit template binding. Controls inherit from `Control` and define templates in code-behind using `OnApplyTemplate()`.

**Avalonia**: Uses `{TemplateBinding}` syntax (similar but not identical), supports `TemplatePartAttribute`, but the binding system is more lightweight. The `TemplateBinding` behaves differently for attached properties.

#### Impact on Mutagen.Bethesda.WPF
The Mutagen WPF library uses custom controls extensively:
- `AModKeyPicker`, `AFormKeyPicker`, `ModKeyBox`, `FormKeyBox` all inherit from `NoggogControl` (a WPF custom control)
- They use `TemplatePart` attributes and `OnApplyTemplate()` to wire up internal components
- Heavy reliance on Noggog.WPF's base classes

#### Specific Challenges
1. **TemplateBinding limitations**: Avalonia's `TemplateBinding` doesn't support all binding modes equally. Reverse binding (target→template) has different semantics.
2. **Attached properties in templates**: Avalonia treats attached properties differently in template contexts.
3. **Template lookup**: WPF uses `DefaultStyleKey` pattern; Avalonia uses a simpler but different system.

#### Mitigation Strategy
- Replace `TemplatePart`-based designs with direct binding where possible
- Use Avalonia's `IDataTemplate` for template flexibility instead of code-behind templates
- Test bidirectional binding scenarios early
- Prefer composition over template-heavy custom controls where feasible

---

### 2. Event System (Routed vs Direct Events)

#### The Problem
**WPF**: Uses a routed event system where events tunnel and bubble through the visual tree. Controls register handlers for specific routes (Tunneling, Direct, Bubbling).

**Avalonia**: Uses a simplified direct event system. There is no built-in tunneling/bubbling. Event propagation is handled through standard C# event inheritance.

#### Impact on Mutagen.Bethesda.WPF
The WPF library uses:
- `ReactiveUI` with `ReactiveMarbles.ObservableEvents.SourceGenerator` for observable event conversion
- Dependency on `Noggog.WPF` which wraps routed events
- Custom events on picker controls that may propagate through the visual tree

#### Specific Challenges
1. **Event not finding handlers**: Code relying on bubbling events won't work if events are handled at a higher level
2. **Re-raising events**: The routed event system's ability to re-raise events at different levels has no direct Avalonia equivalent
3. **Event.Original vs CurrentTarget semantics**: Different in Avalonia

#### Mitigation Strategy
- Migrate from routed events to observable streams (this aligns well with ReactiveUI)
- Use dependency properties with `PropertyChanged` notifications instead of events for state changes
- Leverage ReactiveUI's `IObserveNotifyPropertyChanged` for cross-cutting event concerns
- Test event propagation scenarios with unit tests, not by relying on framework behavior

---

### 3. Dependency Properties and Property System

#### The Problem
**WPF**: Full-featured dependency property system with complex metadata, property coercion, validation, property inheritance, and attached properties deeply integrated.

**Avalonia**: Simplified property system (`AvaloniaProperty`). Coercion is less flexible. Validation is looser. Property inheritance is limited.

#### Impact on Mutagen.Bethesda.WPF
The WPF library uses:
- Custom dependency properties for picker state (e.g., `IsOpen` in dropdown pickers)
- Property metadata with default values and change handlers
- Likely some coercion or validation on numeric/key inputs

#### Specific Challenges
1. **No PropertyMetadata coercion**: Custom coercion logic in WPF's `OnPropertyChanged` handlers won't translate directly
2. **Attached property inheritance**: WPF's attached properties can have different inheritance rules than Avalonia
3. **Validation**: WPF's validation system is deeper; Avalonia validation is more data-annotation focused

#### Mitigation Strategy
- Convert coercion logic to observable-based validation in ViewModels (reactive approach)
- Use `AvaloniaProperty.RegisterAttached()` for attached properties, test scope carefully
- Prefer data validation in Models/ViewModels over control-level validation
- Document property inheritance assumptions; test explicitly

---

### 4. Noggog.WPF Dependency

#### The Problem
Many Mutagen controls inherit from `NoggogControl` and use utilities from `Noggog.WPF`:
- Drag-and-drop helpers
- Custom focus behaviors
- Resource key constants
- Brush and color utilities

**No direct Avalonia equivalent exists** for Noggog.WPF.

#### Impact on Mutagen.Bethesda.WPF
Direct blockers:
- `NoggogControl` base class (need custom base or composition)
- `Noggog.WPF.Drag.ListBoxDragDrop<T>` utilities (must reimplement or use Avalonia's drag-drop)
- `Noggog.WPF.Brushes.Constants` (need custom constants or resource dictionary)

#### Specific Challenges
1. **No drop-in replacement**: Noggog.WPF is WPF-specific; no Avalonia version exists
2. **Utility patterns differ**: WPF's behavior-based approach differs from Avalonia's attached properties
3. **Community library gap**: Avalonia has fewer polished utility libraries compared to WPF

#### Mitigation Strategy
- Create `Mutagen.Bethesda.Avalonia.Common` to provide base classes/utilities (compose features from Avalonia directly)
- Use Avalonia's built-in attached properties and behaviors for what Noggog provided
- Leverage open-source Avalonia libraries: `Avalonia.Controls.DataGrid`, `Avalonia.FuncUI` (if functional approach desired)
- Accept some features may need reimplementation or simplified versions

---

### 5. Data Binding Differences

#### The Problem
**WPF**: Binding system is deeply integrated with dependency properties. Binding modes, update triggers, and validation integration are tightly coupled.

**Avalonia**: Binding system is designed to work with any C# properties, not just dependency properties. Less overhead but different semantics.

#### Impact on Mutagen.Bethesda.WPF
The library uses:
- XAML bindings to picker properties
- Two-way bindings for user selection state
- Converter usage for FormKey/ModKey formatting

#### Specific Challenges
1. **Binding.UpdateTrigger differences**: WPF's explicit trigger system differs from Avalonia's simpler approach
2. **StringFormat not available**: Avalonia doesn't have `StringFormat` binding property; must use converters
3. **ElementName binding scope**: Avalonia's ElementName resolution has different scoping rules

#### Mitigation Strategy
- Use value converters liberally (Avalonia's preferred pattern)
- Test binding direction and timing carefully
- Avoid relying on implicit binding trigger behavior; be explicit
- Create a small set of common converters (FormKeyConverter, ModKeyConverter, etc.)

---

### 6. Resource Dictionaries and Styling

#### The Problem
**WPF**: Resource dictionaries merge at parse time. Theme switching requires application restart or manual resource override.

**Avalonia**: Resources are more dynamic. Theme switching is easier, but resource merging order and precedence differ.

#### Impact on Mutagen.Bethesda.WPF
Current setup:
- Uses "Everything.xaml" to export all resources
- Depends on MahApps or Extended.Wpf.Toolkit for themes
- No mechanism for runtime theme switching (users set at app startup)

#### Specific Challenges
1. **Resource merge behavior**: Order of `<ResourceDictionary.MergedDictionaries>` has different impact
2. **No MahApps equivalent**: MahApps is WPF-only; Avalonia's theming is CSS-like and different
3. **Color/brush constants**: Noggog.WPF provides constants; Avalonia requires custom implementation

#### Mitigation Strategy
- Define brushes and colors in a dedicated ResourceDictionary
- Use Avalonia's theme system (light/dark) instead of MahApps
- Create a theme provider interface for runtime color customization
- Document theme assumptions and color slots clearly

---

### 7. Reflection and Type-Driven UI

#### The Problem
**Scope**: Mutagen uses reflection-driven UI generation in `Mutagen.Bethesda.WPF.Reflection` for auto-generating settings UI from property metadata.

**Challenge**: Avalonia has different reflection capabilities and no direct equivalent to WPF's `FrameworkElement` property introspection.

#### Impact on Mutagen.Bethesda.WPF
The reflection module:
- Scans object types and generates XAML dynamically
- Creates controls based on property types (combobox for enums, etc.)
- Uses WPF control registry

#### Specific Challenges
1. **No automatic control registration**: Avalonia doesn't have WPF's implicit type→control mapping
2. **XAML loading**: Dynamic XAML creation and loading differs between frameworks
3. **Control selection by type**: Need custom mapping logic

#### Mitigation Strategy
- Create explicit mapping from .NET type → Avalonia control type
- Use Avalonia's `DataTemplateSelector` for dynamic template selection
- Keep reflection logic in ViewModels (MVVM pattern), not framework-specific
- Write comprehensive unit tests for reflection-based generation

---

### 8. Cross-Platform Implications

#### The Problem
**WPF is Windows-only**. Avalonia is cross-platform (Windows, macOS, Linux).

**Target scope**: This guide assumes Windows primary, but Avalonia enables future cross-platform extension.

#### Platform-Specific Concerns
1. **File dialogs**: Different APIs on different platforms
2. **Registry access**: Windows-specific; no Avalonia equivalent
3. **DPI/scaling**: Different handling on macOS/Linux
4. **Threading**: Avalonia's dispatcher behaves slightly differently on non-Windows platforms

#### Mitigation Strategy
- Isolate platform-specific code in separate service interfaces
- Test on Windows primary, but don't assume platform-specific behavior
- Use Avalonia's `RuntimeInformation` for platform detection
- Consider abstracting system interactions behind interfaces (key for testability)

---

## Library-Specific Challenges

### Mutagen.Bethesda.Avalonia

**Primary goal**: Provide Avalonia-equivalent controls for FormKey, ModKey, and custom selection pickers.

**Key challenges**:
- Re-implementing `AModKeyPicker` and `AFormKeyPicker` without Noggog.WPF
- Providing drag-and-drop for list pickers
- Ensuring the same level of responsiveness and validation

**Priority mitigations**:
- Use composition instead of inheritance for base control behavior
- Implement drag-and-drop through Avalonia's native drag-drop system
- Test picker responsiveness and state management thoroughly

### Mutagen.Bethesda.Avalonia.TestDisplay

**Primary goal**: Desktop test harness that mirrors WPF test display app.

**Key challenges**:
- Dependency injection setup (likely using same Autofac, which is framework-agnostic)
- View/ViewModel binding patterns (ReactiveUI works with Avalonia)
- Test registration and discovery UI

**Priority mitigations**:
- Leverage ReactiveUI's Avalonia integration
- Keep test discovery logic in non-UI libraries (already done)
- Mirror WPF MVVM structure closely for maintainability

### Mutagen.Bethesda.Avalonia.UnitTests

**Primary goal**: Unit tests for custom Avalonia controls and integration points.

**Key challenges**:
- Testing custom controls without WPF's TestGrid
- Mocking visual tree interactions
- Testing template binding behavior

**Priority mitigations**:
- Use snapshot testing for control rendering
- Create helper test fixtures that replicate common scenarios
- Rely on integration tests, not low-level unit tests, for visual behavior
- Test ViewModels separately from Views

---

## Summary of High-Risk Areas

| Risk | Severity | Mitigation |
|------|----------|-----------|
| Noggog.WPF base classes | High | Create Mutagen base classes; use composition |
| Routed event system | Medium | Migrate to observable streams (ReactiveUI) |
| TemplateBinding semantics | Medium | Test bidirectional bindings; prefer composition |
| Reflection-driven UI | Medium | Create explicit type→control mappings |
| Resource theming | Low | Use Avalonia's theme system; document color slots |
| Dependency property coercion | Medium | Move coercion to ViewModels (reactive pattern) |

---

## Critical Success Factors

1. **Early reactive architecture adoption**: Don't try to replicate WPF's event system; embrace observables.
2. **Thorough binding testing**: Test data binding scenarios early and often.
3. **Custom base class strategy**: Decide early on composition vs inheritance for control base behavior.
4. **Test environment quality**: Invest in a good test harness to catch differences early.
5. **Documentation of mappings**: Document WPF→Avalonia patterns as you discover them.

---

## Next Steps

This assessment informs the porting guide (Section 2) and provides context for understanding why certain design decisions are made during the porting process. Each high-risk area should have a corresponding section in the porting guide with concrete examples and code patterns.
