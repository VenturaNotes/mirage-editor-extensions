# Mirage Editor — Extensions Catalog

[![Extension Gatekeeper](https://github.com/VenturaNotes/mirage-editor-extensions/actions/workflows/validate-registry.yml/badge.svg)](https://github.com/VenturaNotes/mirage-editor-extensions/actions/workflows/validate-registry.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Mirage%20Editor-705df2.svg)](https://github.com/VenturaNotes/mirage-editor)

This repository serves as the official registry index and automated gatekeeper for **[Mirage Editor](https://github.com/VenturaNotes/mirage-editor)** community extensions.

Unlike traditional package registries that store heavy binaries in Git history or rely on central servers, Mirage uses a **decentralized, serverless architecture**: extension code lives in the author’s own GitHub Releases, while this repository compiles a lightweight, indexed **SQLite database with FTS5 (Full-Text Search)** distributed globally over GitHub Pages.

## Key Highlights

* **Decentralized Storage**: Compiled JavaScript (`main.js`) and stylesheets (`styles.css`) are hosted as release assets on the author's own GitHub repositories. This repository only stores lightweight metadata pointers.
* **Autonomous CI Gatekeeper**: Pull requests are automatically evaluated by an Abstract Syntax Tree (AST) static security scanner, verified against release assets, and merged into `main`.
* **Immutable Capability Badges**: Security tags (`Vault-Only`, `Network`, `Terminal`, `Audio`, `Status Bar`) are derived directly from static code inspection and cannot be falsified by authors.
* **Sub-Millisecond Search at Scale**: On merge, GitHub Actions compiles all registered extensions into `catalog.sqlite` with an FTS5 virtual table and B-tree indexes, allowing desktop clients to query hundreds of thousands of extensions with 0ms UI lag.
* **Pre-Install Client Audits**: Users can run on-demand client-side security audits using their own Google Gemini API key before installing untrusted extensions.

## How to Publish an Extension

### Method 1: In-Editor One-Click Publish (Recommended)

You can package and publish extensions directly from within Mirage Editor without touching a terminal:

1. **Connect GitHub**:
   * Open **Settings** (`Cmd+,`) -> **Extensions**.
   * Click **"Connect GitHub"** to authorize via the built-in Device Flow. Your authenticated handle (e.g. `@alice`) becomes your verified author namespace.
2. **Review & Pre-Flight**:
   * Under the **Installed** tab, find your local extension and click **`Publish`**.
   * Mirage uses your local Gemini API key to analyze your code, recommend a Semantic Version bump (`MAJOR`, `MINOR`, or `PATCH`), generate a crisp card summary, and draft a formatted `README.md`.
   * Review or edit the version, description, and keywords in the modal.
3. **Automated Release & PR**:
   * Choose your repository destination (your `mirage-extensions` collection repo or a dedicated standalone repo).
   * Click **"Confirm & Continue to GitHub ➔"**.
   * Mirage automatically provisions your repository, tags the release, uploads the bundle assets, and opens an automated Pull Request to register your extension.

### Method 2: Manual Pull Request (For Git / CLI Authors)

If you prefer to manage your repositories manually:

#### 1. Host your Release Assets
Create a GitHub Release on your own repository (e.g. `https://github.com/<username>/mirage-extensions/releases/tag/my-tool@v1.0.0`) and attach:
- `main.js` (Compiled entrypoint)
- `manifest.json` (Extension metadata)
- `styles.css` (Optional stylesheet)

#### 2. Fork this Repository
Fork `VenturaNotes/mirage-editor-extensions` to your GitHub account.

#### 3. Add your Metadata Entry
Create an isolated JSON file at `authors/<your-github-username>/<extension-id>.json`:

```json
{
  "id": "lean-player",
  "name": "Lean Player",
  "author": "VenturaNotes",
  "version": "1.0.0",
  "description": "Lightweight, zero-build local music player with queue management, global search, and status bar controls.",
  "repo": "VenturaNotes/mirage-extensions",
  "releaseTag": "lean-player@v1.0.0",
  "capabilities": ["action", "status-bar", "vault-only"],
  "searchKeywords": ["music", "player", "audio", "mp3", "queue"],
  "hash": "a1b2c3d4e5f6...",
  "updatedAt": 1731367680000
}
```

#### 4. Open a Pull Request
Open a Pull Request from your fork to `main`. 

> **Important**: To prevent Git merge conflicts, PRs must **only** add or update files inside `authors/<your-github-username>/`. PRs modifying any other directory will be rejected by the automated gatekeeper.

## Automated Security & Gatekeeper Policies

Every Pull Request undergoes an automated multi-step verification pipeline in GitHub Actions:

```text
PR Opened ──► 1. Identity Check (PR author == authors/<author>/)
          ──► 2. Release Asset Validation (main.js exists on author's repo)
          ──► 3. AST Static Security Scan (No eval, no dynamic remote imports)
          ──► 4. Capability Badge Derivation
          ──► 5. Auto-Approve & Auto-Merge into main
          ──► 6. Compile catalog.sqlite (FTS5) & Deploy
```

### Security Invariants Checked in CI
1. **Dynamic Code Execution**: Calls to `eval()`, `new Function()`, `window.eval`, or `globalThis.eval` are strictly prohibited and cause immediate PR rejection.
2. **Dynamic Remote Imports**: `import("https://...")` or `require("https://...")` are blocked. All runtime dependencies must be bundled or vendored.
3. **Remote Fetch-to-Eval**: Fetching remote strings and passing them into execution runtimes is forbidden.
4. **Obfuscation**: Code must be published in human-readable form. Heavy hexadecimal-encoded arrays or encrypted string decoders will trigger an obfuscation violation.
5. **Namespace Protection**: The author namespace strictly matches the GitHub username of the pull request creator. Nobody can create or modify files in another developer's directory. Reserved author names (`local`, `mirage`, `official`) are permanently blocked.

## Immutable Capability Badges

To protect users against unauthorized data exfiltration, capability badges are derived directly by the CI static code analyzer and **cannot be manually falsified by authors**:

| Badge | Meaning | Enforcement Rule |
| :--- | :--- | :--- |
| `Vault-Only` | **100% Offline & Private** | Provably contains **zero** network requests (`fetch`, `WebSocket`, `XMLHttpRequest`) and zero shell/terminal calls. |
| `Network` | **External API Access** | Code makes external network calls (e.g. calls to Spotify, YouTube, or AI endpoints). Disclosed prominently to users. |
| `Terminal` | **Local Process Execution** | Extension interacts with system shells (`child_process`, `node-pty`). |
| `Audio` | **Media Playback** | Utilizes HTML5 Audio or Web Audio API. |
| `Status Bar` | **UI Indicator** | Registers widgets or chips in the bottom status bar. |

## Emergency Killswitch (`blocklist.txt`)

If a malicious payload, zero-day exploit, or severe privacy violation is discovered in an already-published extension:

1. Maintainers add the extension ID or author handle to `blocklist.txt` in the root of this repository.
2. The CI workflow compiles the ID into the `blocklist` table of `catalog.sqlite`.
3. When Mirage Editor clients launch or sync the catalog, any installed extension matching the blocklist is **instantly unloaded and disabled locally**, protecting users who already downloaded the script.

## Repository Layout

```text
mirage-editor-extensions/
├── .github/
│   └── workflows/
│       └── validate-registry.yml  # Autonomous gatekeeper & SQLite compiler
├── authors/                       # Isolated author namespace directories
│   ├── VenturaNotes/
│   │   └── lean-player.json
│   └── <author>/
│       └── <extension-id>.json
├── catalog.sqlite                 # Pre-compiled SQLite FTS5 database (built by CI)
├── blocklist.txt                  # List of revoked extension IDs or hashes
└── README.md
```

## License & Attribution

* The **Mirage Editor Extensions Catalog** and CI automation are released under the [MIT License](LICENSE).
* **Mirage Editor** is designed and created by **[Julian Ventura](https://github.com/VenturaNotes)**.
* Individual extensions and scripts listed in this catalog remain the intellectual property and copyright of their respective authors as declared in their individual repositories and licenses.
