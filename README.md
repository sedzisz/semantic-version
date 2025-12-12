# 🚀 Semantic Version GitHub Action

Automatically calculate the next semantic version (`MAJOR.MINOR.PATCH`) based on commit messages, branch names, or pull-request labels.

---

## ✨ Features

- Detects change type from commit messages, branch names, or PR labels
- Configurable token → version bump mapping
- Outputs: `version`, `release_needed`, `release_id`
- Docker-based for consistency
- **Version detection based on Git tags** (format: `v*.*.*` or `*.*.*`)

---

## 📦 Quick Start

### Basic Usage (Recommended)

```yaml
- name: Determine version
  id: version
  uses: sedzisz/semantic-version-action@v1
  with:
    type: label  # or 'commit', 'branch'
    map: |
      {
        "major": ["breaking"],
        "minor": ["feature"],
        "patch": ["fix", "bug", "docs"]
      }

- name: Use version
  run: echo "New version: ${{ steps.version.outputs.version }}"
```

### Docker Method

```yaml
- name: Determine version
  id: version
  uses: docker://ghcr.io/sedzisz/semantic-version-action:latest
  env:
    INPUT_TYPE: label
    INPUT_MAP: '{"major":["breaking"],"minor":["feature"],"patch":["fix","bug","docs"]}'
```

---

## 📋 Inputs & Outputs

**Inputs:**
- `type` – Detection mode: `label`, `commit`, or `branch` (default: `label`)
- `map` – JSON mapping of tokens to bump types (required)

**Outputs:**
- `version` – e.g. `v1.4.2`
- `release_needed` – `true` or `false`
- `release_id` – e.g. `1.4.2`

---

## 🔄 Versioning Logic

The action automatically determines the current version by scanning Git tags in the repository:
- Searches for tags matching `v*.*.*` (e.g., `v1.2.3`) or `*.*.*` (e.g., `1.2.3`)
- Uses the latest semantic version tag as the base
- If no valid tags exist, starts from `v0.0.0`
- Increments the appropriate version component (MAJOR, MINOR, or PATCH) based on detected changes

---

## 🎯 Detection Modes

**`label`** – Reads first PR label (e.g., `feature`, `bug`)  
**`commit`** – Extracts from commit message: `[feature]` or `fix:`  
**`branch`** – Extracts from branch name: `feature/login`

---

## 🔧 Complete Example

```yaml
name: Auto Version

on:
  pull_request:
    types: [closed]
    branches: [main]

jobs:
  release:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Required for Git tag detection

      - name: Calculate version
        id: version
        uses: sedzisz/semantic-version-action@v1
        with:
          type: label

      - name: Create tag
        if: steps.version.outputs.release_needed == 'true'
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git tag ${{ steps.version.outputs.version }}
          git push origin ${{ steps.version.outputs.version }}
```

---

## 🐛 Troubleshooting

**No version generated?**  
→ Check if token matches your `map` keys. View logs for detection messages.

**Invalid JSON error?**  
→ Use double quotes and no trailing commas. For `docker://`, use single-line JSON.

**Empty outputs?**  
→ Add `id: version` to your step. For `docker://`, use `env:` not `with:`.

**Version starts at v0.0.0?**  
→ Ensure `fetch-depth: 0` is set in checkout step to access all Git tags.

---

## 📊 Method Comparison

| Feature | `uses: repo@v1` | `uses: docker://` |
|---------|-----------------|-------------------|
| Syntax | `with:` ✅ | `env:` only |
| Multi-line JSON | ✅ | ❌ |
| Recommended | ✅ | Testing only |
