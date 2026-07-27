# Publishing a Clarion Addin

This guide covers everything you need to do to publish your Clarion IDE addin to the community registry.

---

## Prerequisites

- Your addin source is hosted in a **public GitHub repository**
- Your addin is licensed under **MIT** or another OSI-approved open source license
- Your addin targets **.NET Framework 4.0–4.8** (`net40`–`net48`)  
  > ⚠️ .NET 5+ (CoreCLR) is **not compatible** with Clarion's runtime. See [Clarion .NET Compatibility](#clarion-net-compatibility).
- Your repo has a `LICENSE` file and a `README.md`

---

## Step 1 — Prepare Your Repository

### Recommended repo structure

```
YourAddin/
├── YourAddin.csproj       # SDK-style project, targets net40 or net48
├── YourAddin.addin        # SharpDevelop addin manifest
├── YourAddin.sln
├── src/                   # source files
├── CHANGELOG.md           # version history
├── README.md
└── LICENSE
```

### The `.addin` manifest

Every addin must have a SharpDevelop `.addin` XML file that registers pads, menu items, display bindings, etc.  
Minimal example:

```xml
<AddIn name="My Addin" author="YourName" description="What it does">
  <Manifest>
    <Identity name="MyAddin" version="1.0.0"/>
  </Manifest>
  <Runtime>
    <Import assembly="MyAddin.dll"/>
  </Runtime>
  <Path name="/SharpDevelop/Workbench/Pads">
    <Pad id="MyAddinPad" category="Tools" title="My Addin"
         shortcut="Control|Alt|M"
         class="MyAddin.MyAddinPad"/>
  </Path>
</AddIn>
```

### Clarion .NET Compatibility

| Framework | Compatible? | Notes |
|-----------|-------------|-------|
| net40     | ✅ Yes      | Minimum — matches Clarion's own runtime |
| net45–net48 | ✅ Yes    | Same CLR v4, more APIs available |
| net5+     | ❌ No       | CoreCLR — will not load in Clarion |

Clarion is compiled against `net40` (CLR v4). Any `net40`–`net48` addin runs on the same CLR. If you need modern APIs (e.g. `System.IO.Compression.ZipFile`, `JavaScriptSerializer`), target `net48`.

### ClarionCL safety (important!)

Clarion has a **headless command-line generator** (ClarionCL) used by CI/build servers.  
Any addin event handler that touches the IDE UI **must** guard against null:

```csharp
var form = WorkbenchSingleton.MainForm;
if (form == null) return; // Running in ClarionCL — no UI
```

Without this, `NullReferenceException` will crash every CI build with `error CLCE001`.

Also guard `solution.Name` before calling `.StartsWith()` — it can be null in CL mode.

---

## Step 2 — Build a Release

### Simple addin (single DLL + .addin file)

Build in Release mode and upload two files as GitHub Release assets:
- `YourAddin.dll`
- `YourAddin.addin`

```powershell
dotnet build -c Release
gh release create v1.0.0 bin\Release\net40\YourAddin.dll YourAddin.addin --title "YourAddin v1.0.0"
```

### Complex addin (resource files or multiple DLLs)

If your addin ships HTML/JS/CSS resources, sub-folders, or NuGet dependency DLLs, package everything into a **zip file** and upload that as the single release asset.

```powershell
# Example: create zip with DLL, addin, and Resources folder
Compress-Archive -Path bin\Release\net48\MyAddin.dll,
                       bin\Release\net48\MyAddin.addin,
                       bin\Release\net48\SomeDependency.dll,
                       bin\Release\net48\Resources `
                 -DestinationPath MyAddin-v1.0.0.zip

gh release create v1.0.0 MyAddin-v1.0.0.zip --title "MyAddin v1.0.0"
```

> **Important:** The zip is extracted **flat** into `<ClarionRoot>\accessory\addins\<YourAddinId>\`.  
> Sub-folders are preserved. The `.addin` file **must** be in the zip root (not in a sub-folder).

#### What to include in the zip

| File | Include? |
|------|----------|
| `YourAddin.dll` | ✅ Yes |
| `YourAddin.addin` | ✅ Yes |
| NuGet dependency DLLs | ✅ Yes (any not shipped with Clarion) |
| Native loader DLLs (e.g. WebView2Loader.dll) | ✅ Yes |
| Resource files (HTML, JS, CSS, images) | ✅ Yes |
| `.pdb` debug symbols | ❌ No |
| Test assemblies | ❌ No |
| Clarion SDK DLLs (ICSharpCode.Core, etc.) | ❌ No (already in Clarion\bin) |

#### Third-party runtime dependencies

If your addin requires a runtime that may not be installed on user machines (e.g. **WebView2**), document this clearly in your README and registry `description` field.

---

## Step 3 — Add to the Registry

1. Fork [msarson/clarion-addin-registry](https://github.com/msarson/clarion-addin-registry)
2. Add your entry to `registry.json`
3. Submit a Pull Request

> **Required:** every entry must include an **`authorUrl`** — a link to the developer's page
> (e.g. your GitHub profile, `https://github.com/you`). Addin Finder renders the author name in
> the detail panel as a clickable link so users can find more of your work and know who to contact.
> See the full field list and examples in the [registry README](./README.md#adding-your-addin).

### Choosing `downloadUrls` vs `downloadZipUrl`

| Scenario | Use |
|----------|-----|
| Single DLL + `.addin` only | `downloadUrls` + `addinFileUrl` |
| Multiple DLLs, no resources | `downloadUrls` (list all DLLs) + `addinFileUrl` |
| Any resource files / sub-folders | `downloadZipUrl` (zip must include `.addin`) |

---

## Step 4 — Releasing Updates

1. Build the new version
2. Update `CHANGELOG.md`
3. Create a new GitHub Release with the new tag (e.g. `v1.0.1`)
4. Submit a PR to this registry updating the `version` and download URL(s)

---

## Tips

- Use **SDK-style** `.csproj` (`<Project Sdk="Microsoft.NET.Sdk">`) — works with `dotnet build`
- Set `<PlatformTarget>x86</PlatformTarget>` — Clarion is a 32-bit process
- Use `<GenerateAssemblyInfo>false</GenerateAssemblyInfo>` to avoid conflicts with `AssemblyInfo.cs`
- Test your addin in both Clarion IDE **and** ClarionCL (headless) before publishing
- Set `<Private>false</Private>` on Clarion SDK references so they're not copied to your output
