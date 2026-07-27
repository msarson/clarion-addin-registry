# Clarion Addin Registry

Community registry of Clarion IDE addins. Used by the **AddinFinder** addin to discover and install addins directly from within the IDE.

## Resources

- **[DEVELOPING.md](./DEVELOPING.md)** — tips and tricks for building a Clarion IDE addin from scratch
- **[PUBLISHING.md](./PUBLISHING.md)** — how to prepare your release and submit a PR to list it here

## Adding Your Addin

Submit a PR adding your entry to [`registry.json`](./registry.json).  
For full entry format and requirements, see [PUBLISHING.md](./PUBLISHING.md).

### Entry format — simple addin (single DLL)

```json
{
  "id": "YourAddinId",
  "name": "Display Name",
  "description": "What your addin does.",
  "author": "YourGitHubUsername",
  "authorUrl": "https://github.com/YourGitHubUsername",
  "license": "MIT",
  "category": "One of: Source Control | Editor | Utilities | Templates | Other",
  "version": "1.0.0",
  "targetFramework": "net40",
  "downloadUrls": [
    "https://github.com/you/your-addin/releases/download/v1.0.0/YourAddin.dll"
  ],
  "addinFileUrl": "https://github.com/you/your-addin/releases/download/v1.0.0/YourAddin.addin",
  "homepageUrl": "https://github.com/you/your-addin",
  "changelogUrl": "https://github.com/you/your-addin/blob/master/CHANGELOG.md"
}
```

### Entry format — complex addin (multiple files / resource folders)

Use `downloadZipUrl` instead of `downloadUrls` when your addin ships resource files, sub-folders, or multiple dependent DLLs.

```json
{
  "id": "YourAddinId",
  "name": "Display Name",
  "description": "What your addin does.",
  "author": "YourGitHubUsername",
  "authorUrl": "https://github.com/YourGitHubUsername",
  "license": "MIT",
  "category": "Editor",
  "version": "1.0.0",
  "targetFramework": "net48",
  "downloadZipUrl": "https://github.com/you/your-addin/releases/download/v1.0.0/YourAddin-v1.0.0.zip",
  "homepageUrl": "https://github.com/you/your-addin",
  "changelogUrl": "https://github.com/you/your-addin/blob/master/CHANGELOG.md"
}
```

The zip is extracted directly into the addin's install folder, preserving any sub-folder structure (e.g. `Resources\`). The `.addin` manifest file must be included inside the zip.

### Entry format — fork

Add `fork: true` and `upstreamUrl` so users can see where the addin originates:

```json
{
  ...
  "fork": true,
  "upstreamUrl": "https://github.com/original-author/original-repo"
}
```

### Optional fields

| Field | Purpose |
|-------|---------|
| `addinFileUrl` | Direct URL to the `.addin` manifest (single-DLL addins only — not needed when using `downloadZipUrl`, which bundles it). |
| `fork` / `upstreamUrl` | Mark an addin as a fork and point at the original repo (see above). |

### Requirements

- Addin must be **open source** (MIT or compatible license)
- **`authorUrl` is required** — a link to the developer's page (e.g. your GitHub profile `https://github.com/you`). Addin Finder turns the author name in the detail panel into a clickable link so users can find more of your work and know who to contact.
- All download URLs must point to **GitHub Release** assets (direct download, no redirects)
- `targetFramework` must be `net40` through `net48` — net5+ cannot load in Clarion's CLR v4
- If using `downloadZipUrl`, the zip must include the `.addin` manifest file
- Do **not** include debug symbols (`.pdb`) or test assemblies in the release assets

## Current Addins

| Name | Author | Category | Version | Notes |
|------|--------|----------|---------|-------|
| [GitPane](https://github.com/msarson/Clarion-GitPane) | [msarson](https://github.com/msarson) | Source Control | 1.0.9 | |
| [Flatten Code](https://github.com/msarson/FlattenCode) | [msarson](https://github.com/msarson) | Editor | 1.0.3 | |
| [List Format Parser](https://github.com/msarson/ListFormatParser) | [msarson](https://github.com/msarson) | Editor | 1.3.0 | |
| [Clarion LSP](https://github.com/msarson/clarion-lsp) | [msarson](https://github.com/msarson) | Utilities | 1.4.1 | |
| [Clarion Markdown Editor](https://github.com/msarson/ClarionMarkdownEditor) | [msarson](https://github.com/msarson) | Editor | 1.2.0 | Fork of [peterparker57](https://github.com/peterparker57/ClarionMarkdownEditor) |
| [Clarion CSV Editor](https://github.com/msarson/clarion-csv-editor) | [msarson](https://github.com/msarson) | Editor | 0.2.0 | Requires WebView2 |
| [Clarion Transformer](https://github.com/asantarelli/ClarionTransformer) | [asantarelli](https://github.com/asantarelli) | Editor | 1.0.0 | |
| [Clarion Formatter](https://github.com/asantarelli/ClarionFormatter) | [asantarelli](https://github.com/asantarelli) | Editor | 2.0.1 | |
