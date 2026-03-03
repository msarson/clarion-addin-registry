# Developing a Clarion IDE Addin

A practical guide to building, debugging, and shipping Clarion IDE addins — distilled from real-world experience.

---

## How Clarion loads addins

Clarion is built on **SharpDevelop 3.x** (ICSharpCode). Addins are `.NET` assemblies described by an `.addin` XML manifest. At startup, Clarion scans `accessory\addins\**\*.addin`, loads the manifests, then loads the DLLs on demand.

The runtime is **.NET CLR v4** — your addin must target `net40` through `net48`. `net5+` assemblies cannot be loaded.

---

## Project setup

Use an SDK-style `.csproj` for modern tooling with legacy targets:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net40</TargetFramework>
    <UseWindowsForms>true</UseWindowsForms>
    <GenerateAssemblyInfo>false</GenerateAssemblyInfo>  <!-- use Properties\AssemblyInfo.cs -->
    <Platforms>x86</Platforms>
    <PlatformTarget>x86</PlatformTarget>               <!-- Clarion is 32-bit -->
  </PropertyGroup>

  <ItemGroup>
    <Reference Include="ICSharpCode.Core">
      <HintPath>$(ClarionBin)\ICSharpCode.Core.dll</HintPath>
      <Private>false</Private>
    </Reference>
    <Reference Include="ICSharpCode.SharpDevelop">
      <HintPath>$(ClarionBin)\ICSharpCode.SharpDevelop.dll</HintPath>
      <Private>false</Private>
    </Reference>
  </ItemGroup>
</Project>
```

Set `ClarionBin` in `Directory.Build.props` or via environment variable:

```xml
<!-- Directory.Build.props -->
<Project>
  <PropertyGroup>
    <ClarionBin Condition="'$(ClarionBin)' == ''">C:\Clarion\Clarion11.1\bin</ClarionBin>
  </PropertyGroup>
</Project>
```

Build with:
```
dotnet build -c Release
```

> ⚠️ **net40 stale bin bug:** `dotnet build` on net40 projects correctly compiles to `obj\x86\Release\net40\` but sometimes skips the copy to `bin\`. Always verify the bin DLL timestamp matches before releasing. If stale, copy manually from `obj\`.

---

## The .addin manifest

The manifest wires your classes into Clarion's extension points:

```xml
<?xml version="1.0" encoding="utf-8"?>
<AddIn name="MyAddin" author="you" description="What it does">

  <Manifest>
    <Identity name="MyAddin"/>
  </Manifest>

  <Runtime>
    <Import assembly="MyAddin.dll"/>
  </Runtime>

  <!-- Run a class at IDE startup (before pads open) -->
  <Path name="/Workspace/Autostart">
    <Class id="MyAddinInit" class="MyAddin.InitCommand"/>
  </Path>

  <!-- Register a dockable pad -->
  <Path name="/SharpDevelop/Workbench/Pads">
    <Pad id="MyAddinPad"
         category="Main"
         title="My Addin"
         icon="MyAddin.MyIcon"
         shortcut="Control|Alt|M"
         class="MyAddin.MyPad"/>
  </Path>

  <!-- Add a top-level menu item -->
  <Path name="/SharpDevelop/Workbench/MainMenu/Edit">
    <MenuItem id="MyCommand"
              label="Do Something"
              icon="MyAddin.MyIcon"
              class="MyAddin.MyCommand"
              insertbefore="Find"/>
  </Path>

  <!-- Context menu in the source editor -->
  <Path name="/SharpDevelop/ViewContent/DefaultTextEditor/ContextMenu">
    <MenuItem id="MyContextItem"
              label="Do Something"
              class="MyAddin.MyCommand"
              conditions="MyAddin.MyCondition"/>
  </Path>

</AddIn>
```

---

## Key base classes

### `AbstractCommand` — a menu/toolbar action

```csharp
using ICSharpCode.Core;

public class MyCommand : AbstractCommand
{
    public override void Run()
    {
        // your logic here
        var editor = SD.GetActiveTextEditor();
    }
}
```

### `AbstractPadContent` — a dockable pad

```csharp
using ICSharpCode.SharpDevelop.Gui;
using System.Windows.Forms;

public class MyPad : AbstractPadContent
{
    private readonly Panel _panel = new Panel();

    public override Control Control => _panel;

    public MyPad()
    {
        // build your WinForms UI in _panel
    }
}
```

To update the pad title at runtime, walk up the parent chain:

```csharp
private void SetTitle(string title)
{
    Control parent = _panel.Parent;
    while (parent != null)
    {
        if (parent is Form f) { f.Text = title; return; }
        parent = parent.Parent;
    }
}
```

### `IConditionEvaluator` — show/hide menu items contextually

```csharp
using ICSharpCode.Core;

public class MyCondition : IConditionEvaluator
{
    public bool IsValid(object caller, Condition condition)
    {
        // return true to show the menu item
        var editor = SD.GetActiveTextEditor();
        return editor != null && /* your check */;
    }
}
```

In the manifest: `conditions="MyAddin.MyCondition"` (or `conditionFailedAction="Disable"` to grey out instead of hide).

---

## Registering icons

Clarion uses a `ResourceManager`-based icon system. To register a PNG embedded in your assembly:

```csharp
// In your Autostart class Run() method:
using (var stream = Assembly.GetExecutingAssembly()
           .GetManifestResourceStream("MyAddin.Resources.MyIcon.png"))
{
    if (stream != null)
        ResourceService.RegisterNeutralImages(
            new EmbeddedIconManager("MyAddin.MyIcon", new Bitmap(stream)));
}

// Helper class:
internal sealed class EmbeddedIconManager : System.Resources.ResourceManager
{
    private readonly string _key;
    private readonly Bitmap _bmp;
    public EmbeddedIconManager(string key, Bitmap bmp)
        : base(key, Assembly.GetExecutingAssembly()) { _key = key; _bmp = bmp; }
    public override object GetObject(string name) => name == _key ? _bmp : null;
    public override object GetObject(string name, CultureInfo c) => GetObject(name);
}
```

Add the PNG to your csproj:
```xml
<ItemGroup>
  <EmbeddedResource Include="Resources\MyIcon.png"/>
</ItemGroup>
```

Reference the icon key in the manifest: `icon="MyAddin.MyIcon"`.

---

## Threading

Clarion's UI runs on the main thread. Long operations (network, file I/O, git) must run on a background thread:

```csharp
ThreadPool.QueueUserWorkItem(_ =>
{
    var result = DoSlowWork();

    // Marshal back to UI thread
    _panel.BeginInvoke(new Action(() =>
    {
        _myLabel.Text = result;
    }));
});
```

> Never call `Control.Invoke` or `Control.BeginInvoke` before the control's handle is created. Use `VisibleChanged` or `HandleCreated` events to trigger initial background work.

---

## Deployment

Copy the DLL and `.addin` file into a named subfolder under Clarion's addins directory:

```
C:\Clarion\Clarion11.1\accessory\addins\MyAddin\
    MyAddin.dll
    MyAddin.addin
```

**A Clarion restart is required** whenever a DLL is added, replaced, or removed — Clarion loads addins at startup and keeps DLLs locked for the session.

### Replacing a loaded DLL

Clarion holds DLLs with `FILE_SHARE_DELETE`. You **cannot** overwrite a loaded DLL with `File.Copy(overwrite:true)` — it throws `UnauthorizedAccessException`. You **can** rename it (even while loaded):

```csharp
File.Move("MyAddin.dll", "MyAddin.dll.old");  // allowed — FILE_SHARE_DELETE
File.Copy(newDll, "MyAddin.dll");              // now succeeds
```

This is the technique used by AddinFinder's staged update mechanism.

---

## Common pitfalls

| Problem | Cause | Fix |
|---------|-------|-----|
| "Cannot create object: MyAddin.MyClass" | Wrong class name in manifest, or DLL not loaded | Check manifest `class=` matches fully-qualified name exactly |
| Addin loads but commands don't appear | Missing or wrong `/Path name=` in manifest | Check path matches Clarion's extension point exactly |
| TLS errors fetching URLs | .NET 4.x defaults to TLS 1.0; GitHub requires 1.2+ | Set `ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12` at startup |
| Stale bin DLL after build | net40 SDK projects sometimes skip the bin copy | Compare `obj\x86\Release\net40\` vs `bin\` timestamp — copy manually if needed |
| `IOException` copying DLL | File locked by Clarion | Use rename-then-copy pattern (see above) |
| Autostart class not found | `<Class>` used but needs `<Command>` or vice versa | Use `<Class>` for Autostart initialisation, `<Command>` for menu/toolbar actions |

---

## Publishing to the registry

Once your addin is working, see [PUBLISHING.md](./PUBLISHING.md) for how to prepare a release and submit a PR to list it here.
