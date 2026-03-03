# Clarion Addin Registry

Community registry of Clarion IDE addins. Used by the **AddinFinder** addin to discover and install addins directly from within the IDE.

## Adding Your Addin

Submit a PR adding your entry to [`registry.json`](./registry.json).

### Entry format

```json
{
  "id": "YourAddinId",
  "name": "Display Name",
  "description": "What your addin does.",
  "author": "YourGitHubUsername",
  "license": "MIT",
  "category": "One of: Source Control | Editor | Utilities | Templates | Other",
  "version": "1.0.0",
  "targetFramework": "net40",
  "downloadUrls": [
    "https://github.com/you/your-addin/releases/download/v1.0.0/YourAddin.dll",
    "https://github.com/you/your-addin/releases/download/v1.0.0/SomeDependency.dll"
  ],
  "addinFileUrl": "https://github.com/you/your-addin/releases/download/v1.0.0/YourAddin.addin",
  "homepageUrl": "https://github.com/you/your-addin",
  "changelogUrl": "https://github.com/you/your-addin/blob/master/CHANGELOG.md"
}
```

### Requirements

- Addin must be **open source** (MIT or compatible license)
- `downloadUrl` and `addinFileUrl` must point to a **GitHub Release** asset (direct download)
- `targetFramework` must be `net40` through `net48` — net5+ cannot load in Clarion's CLR v4

## Current Addins

| Name | Author | Category | Version |
|------|--------|----------|---------|
| [GitPane](https://github.com/msarson/Clarion-GitPane) | msarson | Source Control | 1.0.7 |
