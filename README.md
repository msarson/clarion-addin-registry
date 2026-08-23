# Clarion Addin Registry

Community registry of Clarion IDE addins. Used by the **AddinFinder** addin to discover and install addins directly from within the IDE.

## Resources

- **[DEVELOPING.md](./DEVELOPING.md)** — tips and tricks for building a Clarion IDE addin from scratch
- **[PUBLISHING.md](./PUBLISHING.md)** — how to prepare a release and keep your own addin list

> **Publishing here?** Read [PUBLISHERS.md](PUBLISHERS.md) first — what being listed
> does and does not mean, what you are responsible for, and how to ask to be added.

## Adding Your Addin

Addins are published by their authors. This registry records **publishers**; each keeps their own
`addins.json` in their own repository, so releasing a new version means editing that file and
nothing else — no pull request here, and nobody to wait for.

1. Ask to be listed as a publisher — [PUBLISHERS.md](./PUBLISHERS.md) covers what that means and
   how to ask.
2. Keep your addins in your own `addins.json`. The entry format below is unchanged, so anything
   already listed here carries over verbatim. Working example:
   [msarson/clarion-addins](https://github.com/msarson/clarion-addins/blob/main/addins.json).

For the full walkthrough, see [PUBLISHING.md](./PUBLISHING.md).

> The `addins` list in [`registry.json`](./registry.json) is the older flat format. It is still read,
> so addins that have not moved keep working, and it will be retired once every publisher has a list
> of their own — not merely once federation works.

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

### Entry format — addin distributed as a setup installer

Some addins ship as a Windows setup `.exe` rather than as files Addin Finder can place. These go in
a **separate `setupAddins` list**, alongside `addins` rather than inside it:

```json
{
  "version": 1,
  "publisher": "your-github-account",
  "addins": [ "..." ],
  "setupAddins": [
    {
      "id": "YourAddinId",
      "name": "Display Name",
      "description": "What your addin does.",
      "author": "YourGitHubUsername",
      "authorUrl": "https://github.com/YourGitHubUsername",
      "license": "MIT",
      "category": "Utilities",
      "githubRepo": "YourGitHubUsername/your-addin",
      "homepageUrl": "https://github.com/YourGitHubUsername/your-addin",
      "changelogUrl": "https://github.com/YourGitHubUsername/your-addin/blob/main/CHANGELOG.md"
    }
  ]
}
```

**No `version`, and no download URLs.** `githubRepo` records the repository, and Addin Finder resolves
the current release from it — publishers rename the installer every release
(`YourAddin-1.2-Setup.exe`, then `-1.2.1-`), so a pinned URL would 404 almost immediately. It also
means there is no version left for you to keep in step with anything: the release tag is the version.

**Addin Finder downloads the installer and stops.** The button reads **Download**, the file lands in
the user's Downloads folder, and they run it themselves. It is not executed for them: a setup
elevates, and it picks its own Clarion installations — possibly not the one running the pad. Nothing
is recorded as installed and **Remove is not offered**, because those files belong to your
uninstaller and deleting them behind its back would leave Windows believing the addin is still
there. It still shows as installed once the setup has run, because Addin Finder reconciles against
what is actually on disk.

> **`id` must be exactly the folder name your installer creates** under `accessory\addins`, and that
> folder must contain your `.addin` manifest. For an ordinary addin, Addin Finder creates the folder
> and `id` decides its name; for a setup addin your installer decides, and the two only match if you
> make them. Get it wrong and the addin never shows as installed and never reports a version.

> **`<Identity version>` must match the release tag.** No separate version file is needed or wanted:
> your installer already writes a manifest, and that manifest is what Addin Finder reads to find out
> what is on the machine. It is compared against the release tag with any leading `v` removed — so
> tag `v1.2.1`, ship `<Identity version="1.2.1"/>`. A manifest that says anything else leaves the
> addin reading *Update available* permanently, since running your setup again cannot change what
> your manifest says. `1.2` and `1.2.0` count as equal; nothing else does.

> Requires Addin Finder **0.8.1 or later**. Earlier builds read only the `addins` key, so they cannot
> see a `setupAddins` entry — which is the whole reason for the separate key. Shown one, an older
> client would have created an *empty* folder under `accessory\addins` and recorded a phantom
> install, and an empty folder in the folder Clarion scans at start-up can stop the IDE opening.

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
| `githubRepo` | `owner/repo` for an addin distributed as a setup installer. Only valid inside `setupAddins`, where it replaces the version and the download URLs (see above). |

### Requirements

- Addin must be **open source** (MIT or compatible license)
- **`authorUrl` is required** — a link to the developer's page (e.g. your GitHub profile `https://github.com/you`). Addin Finder turns the author name in the detail panel into a clickable link so users can find more of your work and know who to contact.
- All download URLs must point to **GitHub Release** assets (direct download, no redirects)
- **Everything must come from your own account.** Download URLs must be under
  `github.com/<your-publisher-id>/`, and a `githubRepo` must be owned by that same id. Both are
  checked by the client, and an entry that fails is dropped — one bad entry, not your whole list.
  For a setup addin the check runs again at the moment of download, since a repository that was
  yours when you listed it can be transferred later and GitHub follows the move silently.
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
