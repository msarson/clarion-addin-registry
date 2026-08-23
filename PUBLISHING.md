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
  checked by Addin Finder, and an entry pointing anywhere else is dropped. The same applies to the
  `githubRepo` of an addin that ships as a setup installer

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
> subfolder of `accessory\addins` at start-up and refuses to start at all if two declare the same
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

> `id` is also the folder name the addin installs into, under `accessory\addins`.

### Choosing `downloadUrls` vs `downloadZipUrl`

| Scenario | Use |
|----------|-----|
| Single DLL + `.addin` only | `downloadUrls` + `addinFileUrl` |
| Multiple DLLs, no resources | `downloadUrls` (list all DLLs) + `addinFileUrl` |
| Any resource files / sub-folders | `downloadZipUrl` (zip must include `.addin`) |
| Ships as a Windows setup `.exe` | a `setupAddins` entry with `githubRepo` — see below |

### If your addin ships as a setup installer

Some addins are distributed as a Windows setup `.exe`: they register components, write outside
`accessory\addins`, or install into several Clarion versions at once. Addin Finder cannot place
those files itself, so it does the one thing it honestly can — fetches the installer and hands it
over.

List these under a **`setupAddins`** key, a sibling of `addins`:

```json
{
  "version": 1,
  "publisher": "your-github-account",
  "addins": [ "..." ],
  "setupAddins": [
    { "id": "YourAddinId", "name": "Your Addin",
      "description": "What it does.",
      "author": "you", "authorUrl": "https://github.com/you", "license": "MIT",
      "category": "Utilities",
      "githubRepo": "you/your-addin",
      "homepageUrl": "https://github.com/you/your-addin" }
  ]
}
```

**No `version`, and no download URLs.** Addin Finder resolves the latest release of `githubRepo`
through the GitHub API, because an installer's file name changes every release and a pinned URL
would 404 almost at once. The useful consequence is that a setup addin needs **no registry edit to
release**: tag, upload the installer, done. The release tag is the version, so tag it the way you
want it read — `v1.2.1` shows as `1.2.1`.

What Addin Finder does with one, and what it leaves to you:

| | |
|---|---|
| Button reads | **Download**, not Install |
| Where it lands | the user's `Downloads` folder |
| Who runs it | **the user** — it is never executed for them |
| Recorded as installed | no; the disk scan finds it once your setup has run |
| Remove offered | no; uninstalling is your installer's job, through Windows |

Not running it is deliberate. A setup elevates, and it picks its own Clarion targets — possibly not
the installation running the pad. Handing over the file leaves both the decision and the elevation
prompt with the user, where they belong.

Four rules that apply to these and not to ordinary addins:

1. **`id` must be exactly the folder name your installer creates** under `accessory\addins`, and
   that folder must contain your `.addin` manifest. For an ordinary addin `id` *decides* the folder
   name, because Addin Finder creates it. Here your installer decides, and if the two disagree the
   addin never shows as installed and never reports a version — it just keeps offering Download to
   someone who already has it.
2. **`<Identity version>` in your manifest must match the release tag.** There is no separate file
   to declare a version in, and none is wanted: your installer already writes a manifest, and that
   manifest is what Addin Finder reads to find out which version is on the machine. The version it
   compares against is the release tag with any leading `v` removed. So tag `v1.2.1`, ship
   `<Identity version="1.2.1"/>`, and the pad reads **Installed**. Ship a manifest that says
   something else and it reads **Update available** forever, because installing your setup again
   cannot change what your manifest says.

   `1.2` and `1.2.0` are treated as the same version, so trailing zeroes will not catch you out.
   Anything else will. The cheap guard is a build step that fails when your project version and your
   manifest disagree — see `CheckAddinVersion` in
   [FlattenCode.csproj](https://github.com/msarson/FlattenCode/blob/master/FlattenCode.csproj).
3. **`githubRepo` must be under your own account**, exactly as download URLs must be. It is checked
   when your list is read, and checked again against the resolved asset before anything is fetched,
   because a repository can be transferred and GitHub follows the move silently. A mismatch drops
   the entry, not your list.
4. **Ship a working uninstaller, and do not put it in the addin folder.** Addin Finder will not
   offer Remove, so yours is the only way out.

   Installers put their uninstaller in the application directory by default — `{app}` in Inno
   Setup, `INSTALLDIR` in WiX. Pointing that at `accessory\addins\<YourAddinId>\` so everything
   lives together is the tidy-looking choice and the one to avoid: an uninstaller has to delete
   itself last, and if it leaves that folder behind empty, that is the shape that stops Clarion
   starting. The neat thing quietly ships the failure.

   The shape that works, and the one Clarion Assistant's installer already uses:

   - the application directory goes somewhere of your own — `{autopf}\YourAddin` — and is never
     the Clarion root. The user is not asked about it;
   - the addin files go to `<resolved Clarion root>\accessory\addins\<YourAddinId>`, resolved at
     install time, once per Clarion version you support;
   - uninstall removes that folder **and its contents**, not just the files you laid down. In Inno
     that is `Type: filesandordirs`, which also sweeps anything your addin generated at runtime —
     deleting only what you installed can leave a log or a cache behind, and a folder that is not
     quite empty is no better than one that is.

   Installing into several Clarions at once follows naturally from this, which is worth knowing:
   Addin Finder can only ever see the one it is running in.

> **`setupAddins` needs Addin Finder 0.9.0 or later.** Older builds read only the `addins` key and
> cannot see these entries — which is exactly why they live under a separate key rather than as a
> flag on an ordinary one. Shown such an entry, an older client would find no URLs, download nothing,
> create an *empty* folder under `accessory\addins` regardless and record a phantom install — and an
> empty folder in the folder Clarion scans at start-up is the shape that stops an IDE opening. So do
> not put a `githubRepo` entry in your `addins` list to reach older clients: current ones refuse it,
> and older ones are precisely who it would hurt.

---

## Step 5 — Releasing updates

1. Bump the version in your project **and** in `<Identity version>` — they must agree
2. Update `CHANGELOG.md`
3. Create a GitHub Release with the new tag (e.g. `v1.0.1`)
4. Edit `addins.json` **in your own repository**: new `version`, new download URLs

That is the whole process. There is no pull request here, and nothing to wait for -- Addin Finder
picks it up on the next refresh.

> A **setup addin stops at step 3.** There is no version or URL in its entry to update, so the
> release itself is the publication -- and there is nothing left to fall out of step with the
> release you tagged.

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
