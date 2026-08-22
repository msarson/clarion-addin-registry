# Publishing a Clarion Addin

This guide covers everything you need to do to publish a Clarion IDE addin.

Addins are published by their authors. Once you are listed as a publisher you keep your own
`addins.json`, and releasing a new version means editing that file in your own repository --
nothing is submitted here and nobody has to approve it.

Read [PUBLISHERS.md](./PUBLISHERS.md) first: it covers what being listed does and does not mean,
what you take responsibility for, and how to ask to be added.

---

## Prerequisites

- Your addin source is hosted in a **public GitHub repository**
- Your addin is licensed under **MIT** or another OSI-approved open source license
- Your addin targets **.NET Framework 4.0–4.8** (`net40`–`net48`)  
  > ⚠️ .NET 5+ (CoreCLR) is **not compatible** with Clarion's runtime. See [Clarion .NET Compatibility](#clarion-net-compatibility).
- Your repo has a `LICENSE` file and a `README.md`
- Downloads are served from **your own GitHub account** (`github.com/<your-id>/...`) -- this is
  checked by Addin Finder, and an entry pointing anywhere else is dropped

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

> **Bump `<Identity version>` on every release, and keep it matching your assembly version.**
>
> Addin Finder reads it to work out what is installed. FlattenCode shipped 1.0.1, 1.0.2 and 1.0.3
> all declaring `version="1.0"`, and the result was an update the pad offered forever and installing
> could never clear -- the manifest kept reasserting the old number. Nothing warns you: the build
> succeeds, the addin works, and only users see the symptom.
>
> The cheap guard is a build step that fails when the manifest and your project version disagree.
> See `CheckAddinVersion` in [FlattenCode.csproj](https://github.com/msarson/FlattenCode/blob/master/FlattenCode.csproj).

> **`<Identity name>` must be unique across every addin a user might install.** Clarion loads every
> subfolder of `accessoryddins` at start-up and refuses to start at all if two declare the same
> name -- the user gets *"Identity name used by multiple addins"* and no IDE. It does not have to
> match your addin id, but it does have to be yours alone, and changing it later looks like a
> different addin to any IDE that already has yours.

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

## Step 3 — Become a publisher, once

Ask to be listed by opening an issue on this repository **from the GitHub account you want listed**.
See [PUBLISHERS.md](./PUBLISHERS.md) for exactly what to include and what you are agreeing to.

You will be added to the `publishers` list in `registry.json`:

```json
{ "id": "your-github-account", "name": "Your Name",
  "repo": "your-addins-repo", "branch": "main" }
```

That happens once. After it, you own your listings.

## Step 4 — Keep your own `addins.json`

In the repository you named, create `addins.json`:

```json
{
  "version": 1,
  "publisher": "your-github-account",
  "updated": "2026-08-22",
  "addins": [ { "id": "MyAddin", "name": "My Addin", "...": "..." } ]
}
```

Entry format is unchanged from the old central list, so anything you already had carries over
verbatim. A complete working example:
[msarson/clarion-addins](https://github.com/msarson/clarion-addins/blob/main/addins.json).

> **Required:** every entry must include an **`authorUrl`** — a link to the developer's page
> (e.g. your GitHub profile, `https://github.com/you`). Addin Finder renders the author name in
> the detail panel as a clickable link so users can find more of your work and know who to contact.
> See the full field list and examples in the [registry README](./README.md#adding-your-addin).

> `id` is also the folder name the addin installs into, under `accessoryddins`.

### Choosing `downloadUrls` vs `downloadZipUrl`

| Scenario | Use |
|----------|-----|
| Single DLL + `.addin` only | `downloadUrls` + `addinFileUrl` |
| Multiple DLLs, no resources | `downloadUrls` (list all DLLs) + `addinFileUrl` |
| Any resource files / sub-folders | `downloadZipUrl` (zip must include `.addin`) |

---

## Step 5 — Releasing updates

1. Bump the version in your project **and** in `<Identity version>` — they must agree
2. Update `CHANGELOG.md`
3. Create a GitHub Release with the new tag (e.g. `v1.0.1`)
4. Edit `addins.json` **in your own repository**: new `version`, new download URLs

That is the whole process. There is no pull request here, and nothing to wait for -- Addin Finder
picks it up on the next refresh.

### Retiring an addin

Prefer marking it rather than deleting it. Users who already have it installed are told, and keep
working:

```json
{ "id": "MyAddin", "status": "deprecated",
  "statusNote": "No longer maintained; works on Clarion 11 and earlier.",
  "replacedBy": "MyOtherAddin" }
```

Deleting the entry, or the whole repository, is the blunt option: it carries the least information
to the people who have your addin installed, and a vanished list cannot be told apart from an outage
with any confidence. If you are winding down, mark your addins `deprecated` and ask to be recorded
as `abandoned`.

---

## Tips

- Use **SDK-style** `.csproj` (`<Project Sdk="Microsoft.NET.Sdk">`) — works with `dotnet build`
- Set `<PlatformTarget>x86</PlatformTarget>` — Clarion is a 32-bit process
- Use `<GenerateAssemblyInfo>false</GenerateAssemblyInfo>` to avoid conflicts with `AssemblyInfo.cs`
- Test your addin in both Clarion IDE **and** ClarionCL (headless) before publishing
- Set `<Private>false</Private>` on Clarion SDK references so they're not copied to your output
