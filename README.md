# Unity Essentials

This module is part of the Unity Essentials ecosystem and follows the same lightweight, editor-first approach.
Unity Essentials is a lightweight, modular set of editor utilities and helpers that streamline Unity development. It focuses on clean, dependency-free tools that work well together.

All utilities are under the `UnityEssentials` namespace.

```csharp
using UnityEssentials;
```

## Installation

Install the Unity Essentials entry package via Unity's Package Manager, then install modules from the Tools menu.

- Add the entry package (via Git URL)
    - Window → Package Manager
    - "+" → "Add package from git URL…"
    - Paste: `https://github.com/CanTalat-Yakan/UnityEssentials.git`

- Install or update Unity Essentials packages
    - Tools → Install & Update UnityEssentials
    - Install all or select individual modules; run again anytime to update

---

# On Value Changed Attribute

> Quick overview: Invoke a method when specific serialized fields change in the Inspector. Annotate a public instance method with `[OnValueChanged("fieldA", "fieldB")]` to react after edits are applied. Supports no-arg methods or a single `string` parameter that receives the changed field’s name.

A lightweight way to add reactive editor behavior without custom inspectors. Register a method with `[OnValueChanged]` and it’ll be called when the referenced field’s value actually changes in the Inspector.

![screenshot](Documentation/Screenshot.png)

## Features
- Observe one or more serialized fields by name
- Fires only when the value changes (not on every repaint)
- Method signatures supported:
  - `void Method()`
  - `void Method(string fieldName)`
- Multiple attributes per method supported (monitor different sets)
- Works across many property types (int, float, string, bool, enums, vectors, UnityEngine.Object, etc.)
- Per-object tracking; state resets on selection change
- Editor-only; zero runtime overhead

## Requirements
- Unity Editor 6000.0+ (Editor-only; attribute lives in Runtime for convenience)
- Depends on the Unity Essentials Inspector Hooks module (provides the inspector pipeline and reflection helpers)

Tip: Only serialized fields visible in the Inspector are monitored. Make them public or annotate with `[SerializeField]`.

## Usage
Basic: react with no parameters

```csharp
using UnityEngine;
using UnityEssentials;

public class Example : MonoBehaviour
{
    [SerializeField] private float speed;

    [OnValueChanged(nameof(speed))]
    public void OnSpeedChanged()
    {
        Debug.Log($"Speed changed to {speed}");
    }
}
```

Receiving the field name

```csharp
public class MultiWatcher : MonoBehaviour
{
    public int width;
    public int height;

    [OnValueChanged(nameof(width), nameof(height))]
    public void OnSizeChanged(string field)
    {
        Debug.Log($"{field} changed → {width}x{height}");
    }
}
```

Multiple methods and attributes

```csharp
public class Settings : MonoBehaviour
{
    [SerializeField] private bool enableFeature;
    [SerializeField] private int quality;

    [OnValueChanged(nameof(enableFeature))]
    public void OnFeatureToggle() { /* … */ }

    [OnValueChanged(nameof(quality))]
    [OnValueChanged(nameof(enableFeature))]
    public void OnEitherChanged(string field) { /* … */ }
}
```

## How It Works
- On selection, the inspector pipeline builds a list of public instance methods declared on the component that have `[OnValueChanged]`
- For each referenced field name, it records a snapshot of the current serialized value
- After properties are applied (post-process), it compares snapshot vs current; if different, it updates the snapshot and invokes the method
- Invocation rules:
  - If the method has no parameters → invoke directly
  - If it has exactly one `string` parameter → pass the changed field’s name
  - Other signatures are ignored

## Notes and Limitations
- Supported methods: public instance methods declared on the component class (not inherited); static methods are not considered
- Field matching is by serialized property name; use top-level field names. Nested fields and array elements are not supported and may match ambiguously by leaf name
- Triggers when the serialized value changes; rapid edits may invoke multiple times
- Multi-object editing: methods are invoked per-inspected target
- Editor-only feature; has no effect at runtime

## Files in This Package
- `Runtime/OnValueChangedAttribute.cs` – `[OnValueChanged("fieldA", "fieldB")]` attribute marker
- `Editor/OnValueChangedEditor.cs` – Inspector integration (snapshot, change detection, invocation)
- `Runtime/UnityEssentials.OnValueChangedAttribute.asmdef` – Runtime assembly definition
- `Editor/UnityEssentials.OnValueChangedAttribute.Editor.asmdef` – Editor assembly definition

## Tags
unity, unity-editor, attribute, onvaluechanged, reactive, inspector, reflection, hooks, tools, workflow
