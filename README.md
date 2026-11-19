# Mirage Editor — Extensions Catalog

[![Extension Gatekeeper](https://github.com/VenturaNotes/mirage-editor-extensions/actions/workflows/validate-registry.yml/badge.svg)](https://github.com/VenturaNotes/mirage-editor-extensions/actions/workflows/validate-registry.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Mirage%20Editor-705df2.svg)](https://github.com/VenturaNotes/mirage-editor)

This repository serves as the official registry index and automated gatekeeper for **[Mirage Editor](https://github.com/VenturaNotes/mirage-editor)** community extensions.

Unlike traditional package registries that store heavy binaries in Git history or rely on central web servers, Mirage uses a **decentralized, serverless architecture**: extension code lives in the author’s own GitHub Releases, while this repository compiles a lightweight, indexed **SQLite database with FTS5 (Full-Text Search)** and an incremental delta stream distributed globally via GitHub Releases.

## Key Highlights

* **Decentralized Storage**: Compiled JavaScript (`main.js`), assets, and stylesheets (`styles.css`) are hosted directly on the author's own GitHub Releases. This repository stores only lightweight verified metadata pointers.
* **Autonomous CI Gatekeeper**: Pull requests are automatically evaluated in parallel by static security analyzers, cryptographic integrity verifiers, and anti-cloning checks, then merged into `main` without manual intervention.
* **Anti-Cloning & Originality Protection**: The gatekeeper verifies both raw SHA-256 and whitespace/comment-normalized hashes against the catalog to prevent re-uploading uncredited or cosmetic copies of other authors' work.
* **Immutable Capability Badges**: Security tags (`Vault-Only`, `Network`, `Terminal`, `Audio`, `Status Bar`) are derived directly from static code inspection and cannot be falsified by authors.
* **Sub-Millisecond Search at Scale**: On merge, GitHub Actions compiles extensions into `catalog.sqlite.gz` with an FTS5 virtual table and B-tree indexes, alongside an incremental delta stream (`catalog-delta.jsonl.gz`), allowing desktop clients to sync updates in milliseconds with zero UI lag.
* **Pre-Install Client Audits**: Users can run on-demand client-side security audits using their own Google Gemini API key before installing community extensions.

## How to Publish an Extension

To ensure cryptographic integrity (`hash`, `normalizedHash`, `fileHashes`), compatibility with the latest version of Mirage Editor, and safe release asset packaging, **all extensions must be published directly through Mirage Editor**:

### 1. Connect GitHub
* In Mirage Editor, open **Settings** (`Cmd+,`) -> **Extensions**.
* Click **"Connect GitHub"** in the top-right corner to authenticate via OAuth Device Flow. Your GitHub handle (e.g., `@alice`) becomes your verified author namespace.

### 2. Review & Pre-Flight
* Under the **Installed** tab, locate your personal extension in the `local/` section and click **`Publish`** (or **`Publish Update`**).
* Mirage automatically syncs the latest catalog, verifies code originality, and uses Gemini to:
  * Analyze your implementation and recommend a Semantic Version (`MAJOR`, `MINOR`, or `PATCH`).
  * Verify that forked extensions introduce meaningful functional or UX changes relative to upstream.
  * Generate a catalog card summary and formatted `README.md` documentation with proper upstream attribution.
* Review or tweak the version, card description, search keywords, and documentation.

### 3. Automated Release & Auto-Merge
* Choose your repository destination (your `mirage-personal-extensions` collection repo or a dedicated standalone repo).
* Click **"Confirm & Continue to GitHub ➔"**.
* Mirage automatically:
  1. Configures your target GitHub repository.
  2. Commits your extension files (`main.js`, `manifest.json`, `styles.css`, `README.md`).
  3. Creates a tagged GitHub Release and uploads release assets.
  4. Forks this catalog repository and opens an automated Pull Request adding `authors/<your-username>/<extension-id>.json`.
* The automated CI Gatekeeper verifies the PR, approves it, and merges it into the community catalog within seconds.

> **Note**: Each author is permitted **one active pull request at a time** in the submission queue to prevent spam and preserve CI throughput.

## Automated Security & Gatekeeper Policies

Every submission undergoes an automated verification pipeline in GitHub Actions:

```text
PR Opened ──► 1. 1-PR Queue Limit Check (1 active PR per author)
          ──► 2. Identity & Namespace Check (PR author == authors/<author>/)
          ──► 3. App Version Compatibility (Mirage Editor version >= latest release)
          ──► 4. Release Asset & Multi-File SHA-256 Integrity Verification
          ──► 5. Anti-Clone & Originality Check (Raw & normalized hash checks)
          ──► 6. AST Static Security Scan (No eval, no dynamic remote imports)
          ──► 7. Capability Badge Derivation & Verification
          ──► 8. Auto-Approve & Auto-Merge into main
          ──► 9. Incremental SQLite Catalog & Delta Compilation
```

### Security Invariants Checked in CI
1. **Dynamic Code Execution**: Calls to `eval()`, `new Function()`, `window.eval`, or `globalThis.eval` are strictly prohibited and cause immediate PR rejection.
2. **Dynamic Remote Imports**: `import("https://...")` or `require("https://...")` are blocked. All runtime dependencies must be bundled or self-contained.
3. **Remote Fetch-to-Eval**: Fetching remote strings and passing them into execution runtimes is forbidden.
4. **Obfuscation**: Code must be published in human-readable form. Heavy hexadecimal-encoded arrays or encrypted string decoders will trigger an obfuscation violation.
5. **Namespace Protection**: The author namespace strictly matches the GitHub username of the pull request creator. Nobody can create or modify files in another developer's directory. Reserved author names (`local`, `mirage`, `official`) are permanently blocked.
6. **Code Originality**: `main.js` cannot match the raw SHA-256 or normalized (comment- and whitespace-stripped) hash of an existing extension by another author.

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

1. Maintainers add the extension identifier (`<author>/<id>`) to `blocklist.txt` in the root of this repository.
2. The CI workflow compiles the ID into the `blocklist` table of `catalog.sqlite` and updates `catalog-meta.json`.
3. When Mirage Editor clients launch or perform their 6-hour background sync, any running extension matching the blocklist is **instantly unloaded and disabled locally**, protecting users who already installed it.

## Repository Layout

```text
mirage-editor-extensions/
├── .github/
│   └── workflows/
│       └── validate_registry.yml  # Autonomous gatekeeper & SQLite compiler
├── authors/                       # Isolated author namespace directories
│   ├── VenturaNotes/
│   │   └── lean-player.json
│   └── <author>/
│       └── <extension-id>.json
├── blocklist.txt                  # List of revoked extension IDs
└── README.md
```

*(Compiled catalog artifacts—`catalog.sqlite.gz`, `catalog-delta.jsonl.gz`, and `catalog-meta.json`—are automatically published to the [`catalog-latest`](https://github.com/VenturaNotes/mirage-editor-extensions/releases/tag/catalog-latest) release.)*

## License & Attribution

* The **Mirage Editor Extensions Catalog** and CI automation are released under the [MIT License](LICENSE).
* **Mirage Editor** is designed and created by **[Julian Ventura](https://github.com/VenturaNotes)**.
* Individual extensions and scripts listed in this catalog remain the intellectual property and copyright of their respective authors as declared in their individual repositories and licenses.
