# Mirage Editor Extensions

Catalog, registry, and monorepo for community plugins and user scripts designed for **[Mirage Editor](https://github.com/VenturaNotes/mirage-editor)**.

Unlike traditional plugin ecosystems that require complex build steps, bundlers, and package managers, Mirage Editor extensions are **zero-build, transparent, and remixable**. You can inspect the code, tweak it live inside your vault, or remix it using AI.


## Repository Structure

```text
mirage-editor-extensions/
├── plugins/                      # Full multi-file plugins
│   └── lean-player/              # Example: Local audio player
│       ├── manifest.json         # Extension metadata & capabilities
│       ├── main.js               # CommonJS entry point
│       └── styles.css            # (Optional) Scoped styling
├── scripts/                      # Single-file user scripts
│   └── formatTables.js           # Example: Standalone utility snippet
├── .github/
│   └── workflows/
│       └── build-registry.yml    # CI action that compiles registry.json
└── registry.json                 # Auto-generated catalog consumed by Mirage Editor
```

---

## Plugins vs. User Scripts

Mirage distinguishes between two types of extensions so you never write more boilerplate than necessary:

| Type | Best For | Structure | Capabilities |
| :--- | :--- | :--- | :--- |
| **Plugin** | Audio players, custom UI panels, status bar widgets, full tools | Directory with `manifest.json`, `main.js`, and optional `styles.css` | Lifecycle (`onload`/`onunload`), settings tabs, persistent `data.json`, event hooks |
| **Script** | Text formatters, word counters, scrapers, cleanups | Single `.js` file | Stateless, executes on demand or at startup, no persistent UI required |

---

## Quickstart: Building an Extension

### 1. Creating a Plugin

Create a folder inside `plugins/<your-plugin-id>/` with two required files:

#### `manifest.json`
```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "description": "A clean description of what this extension adds.",
  "author": "Your Name",
  "prefix": "MyPlugin",
  "main": "main.js"
}
```

#### `main.js`
```javascript
module.exports = class MyPlugin {
  constructor(app, manifest) {
    this.app = app;
    this.manifest = manifest;
  }

  async onload() {
    // 1. Load persistent data from data.json
    this.settings = (await this.loadData()) || { enabled: true };

    // 2. Add commands to the Command Palette (Cmd+P)
    this.addCommand({
      id: `${this.manifest.id}:run`,
      name: 'Run Action',
      callback: () => {
        this.app.notify({
          title: this.manifest.name,
          body: 'Plugin action executed!',
          type: 'success'
        });
      }
    });

    // 3. (Optional) Add a custom settings tab (Cmd+,)
    this.addSettingTab({
      name: this.manifest.name,
      settings: [
        {
          key: 'enabled',
          name: 'Enable Feature',
          type: 'toggle',
          default: true
        }
      ]
    });

    console.log(`[${this.manifest.name}] Loaded.`);
  }

  onSettingChange(key, value) {
    this.settings[key] = value;
    this.saveData(this.settings);
  }

  onunload() {
    console.log(`[${this.manifest.name}] Unloaded.`);
  }
};
```

---

### 2. Creating a User Script

For simple actions, create a single file inside `scripts/<scriptName>.js`:

```javascript
module.exports = async function(app) {
  const activeLeaf = app.workspace.activeLeaf;
  const activeTab = activeLeaf?.activeTab;

  if (activeTab?.type === 'markdown' && activeTab.editor) {
    const content = activeTab.editor.getContent();
    const words = content.trim().split(/\s+/).filter(Boolean).length;

    app.notify({
      title: 'Word Count',
      body: `This note contains ${words.toLocaleString()} words.`,
      type: 'info'
    });
  } else {
    app.notify({
      title: 'Word Count',
      body: 'No active markdown note open.',
      type: 'warning'
    });
  }
};
```

---

## Security & Capabilities

To maintain user trust without requiring a centralized app store approval process, declare your extension's capabilities in `manifest.json`:

* **`vault-only`**: The extension only reads/writes within the active vault. No network activity and no shell execution.
* **`audio`**: Uses web audio APIs or HTML5 audio playback.
* **`network`**: Makes outbound HTTP/WebSocket requests (e.g. sync services, dictionary lookups, APIs).
* **`shell`**: Executes system processes via terminal or shell runners.

---

## How to Contribute

We welcome community extensions! To add your plugin or script to the Mirage Editor in-app catalog:

1. **Fork** this repository.
2. Add your extension:
   * For **Plugins**: Add a folder under `plugins/<plugin-id>/`.
   * For **User Scripts**: Add your single `.js` file under `scripts/<scriptName>.js`.
3. Open a **Pull Request** against `main`.
4. Our automated GitHub Action will:
   * Scan your code for disallowed patterns (e.g. `eval`, obfuscated code).
   * Automatically compile and update `registry.json`.
5. Once merged, your extension will immediately appear in the **Explore Catalog** tab for all Mirage Editor users worldwide.

---

## The Remix Philosophy

Mirage Editor extensions are designed to solve the "abandoned plugin" crisis. If an extension is missing a feature you want:

1. Open **Settings → Community Plugins**.
2. Click **`✨ Remix`** on any installed or catalog extension.
3. Choose to **Copy Prompt Context** or **Open in Internal Terminal**.
4. Hand the context to your AI assistant of choice (Claude Code, Aider, ChatGPT, Claude, Google AI Studio).
5. Paste the updated code back into your vault’s `.mirage-editor/plugins/` directory and click **`🔄 Reload`**.

No compiling. No waiting for inactive maintainers. Your extensions evolve with you.

---

## 📄 License

Extensions contributed to this repository are provided under the [MIT License](LICENSE) unless explicitly specified otherwise within their individual directory.
