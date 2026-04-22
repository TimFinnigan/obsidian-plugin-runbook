# Obsidian Plugin Submission Runbook

Step-by-step guide to building, publishing, and submitting an Obsidian community plugin.

---

## 1. Create the plugin files

Every plugin needs at minimum three files. Create a folder at `~/Documents/GitHub/<your-plugin-id>/`.

### `manifest.json`
```json
{
  "id": "your-plugin-id",
  "name": "Your Plugin Name",
  "version": "1.0.0",
  "minAppVersion": "0.15.0",
  "description": "One sentence description — must match exactly everywhere.",
  "author": "Your Name",
  "authorUrl": "https://github.com/YourUsername",
  "isDesktopOnly": false
}
```

> ⚠️ The `description` field must be **identical** in `manifest.json`, `community-plugins.json`, and your PR. Any mismatch will fail the bot check.

### `main.js`
Your compiled plugin code. If writing in TypeScript, compile with:
```bash
npm install
npm run build
```

### `versions.json`
Maps plugin versions to the minimum Obsidian version required:
```json
{
  "1.0.0": "0.15.0"
}
```

### `LICENSE`
Required. MIT is standard. Create a `LICENSE` file — the bot will reject submissions without one.

### `.gitignore`
```
node_modules/
```

---

## 2. Set up the GitHub repo

```bash
cd ~/Documents/GitHub/your-plugin-id

git init
git add .
git commit -m "Initial release: your plugin name v1.0.0"

gh repo create your-plugin-id --public \
  --description "One sentence description" \
  --push --source .
```

---

## 3. Symlink into your Obsidian vault (for development)

Keep the repo in `Documents/GitHub` and symlink it into the vault so Obsidian picks it up:

```bash
ln -s ~/Documents/GitHub/your-plugin-id \
  ~/path/to/vault/.obsidian/plugins/your-plugin-id
```

Enable the plugin in **Obsidian → Settings → Community plugins**.

---

## 4. Set up and run the Obsidian ESLint plugin

The Obsidian team runs [`eslint-plugin-obsidianmd`](https://github.com/obsidianmd/eslint-plugin) on every submission. Run it locally before submitting to catch issues early.

### Install

```bash
npm install --save-dev eslint @typescript-eslint/parser eslint-plugin-obsidianmd
```

Add a `"lint"` script to `package.json`:
```json
"scripts": {
  "lint": "eslint main.ts"
}
```

Create `eslint.config.mjs`:
```javascript
import tsparser from '@typescript-eslint/parser';
import obsidianmd from 'eslint-plugin-obsidianmd';

export default [
    ...obsidianmd.configs.recommended,
    {
        files: ['**/*.ts'],
        languageOptions: {
            parser: tsparser,
            parserOptions: { project: './tsconfig.json' },
        },
    },
];
```

### Run

```bash
npm run lint
```

### Common lint errors

| Error | Fix |
|---|---|
| `Async method 'onload' has no 'await' expression` | Remove `async` from `onload()` — `onLayoutReady` takes a plain callback |
| `Unexpected aliasing of 'this' to local variable` | Replace `const plugin = this` with `.bind(this)` on the methods you need |
| `activeLeaf is deprecated` | Replace `workspace.activeLeaf` with `workspace.getMostRecentLeaf()` |
| `'SomeMethod' requires Obsidian vX.Y.Z, but minAppVersion is ...` | Bump `minAppVersion` in `manifest.json` and `versions.json` to match the highest version required by any API you use |
| `Unsafe assignment of an 'any' value` on prototype access | If you're patching a private prototype (unavoidable for some patterns), wrap the block with `/* eslint-disable @typescript-eslint/no-unsafe-assignment, ... */` and re-enable after |

---

## 5. Create a GitHub release

The bot pulls `main.js` and `manifest.json` directly from the release — they must be attached as individual files.

```bash
gh release create 1.0.0 main.js manifest.json \
  --title "1.0.0" \
  --notes "Initial release."
```

> ⚠️ The release title must be the **bare version number** (`1.0.0`), not `v1.0.0`.

---

## 5. Fork and edit obsidian-releases

```bash
# Clone your fork (run from ~/Documents/GitHub)
gh repo fork obsidianmd/obsidian-releases --clone --default-branch-only
mv obsidian-releases ~/Documents/GitHub/obsidian-releases   # if it cloned somewhere unexpected

cd ~/Documents/GitHub/obsidian-releases
git checkout -b add-your-plugin-id
```

Open `community-plugins.json` and add your entry at the **end of the array** (before the closing `]`):

```json
  },
  {
    "id": "your-plugin-id",
    "name": "Your Plugin Name",
    "author": "Your Name",
    "description": "One sentence description — must match manifest.json exactly.",
    "repo": "YourUsername/your-plugin-id"
  }
]
```

```bash
git add community-plugins.json
git commit -m "Add your-plugin-id plugin"
git push -u origin add-your-plugin-id
```

---

## 6. Open the PR using the exact template

The bot checks for the **exact template structure**. Copy this verbatim and fill in your details:

```
# I am submitting a new Community Plugin

- [x] I attest that I have done my best to deliver a high-quality plugin, am proud of the code I have written, and would recommend it to others. I commit to maintaining the plugin and being responsive to bug reports. If I am no longer able to maintain it, I will make reasonable efforts to find a successor maintainer or withdraw the plugin from the directory.

## Repo URL

<!--- Paste a link to your repo here for easy access -->
Link to my plugin: https://github.com/YourUsername/your-plugin-id

## Release Checklist
- [x] I have tested the plugin on
  - [ ]  Windows
  - [x]  macOS
  - [ ]  Linux
  - [ ]  Android _(if applicable)_
  - [ ]  iOS _(if applicable)_
- [x] My GitHub release contains all required files (as individual files, not just in the source.zip / source.tar.gz)
  - [x] `main.js`
  - [x] `manifest.json`
  - [ ] `styles.css` _(optional)_
- [x] GitHub release name matches the exact version number specified in my manifest.json (_**Note:** Use the exact version number, don't include a prefix `v`_)
- [x] The `id` in my `manifest.json` matches the `id` in the `community-plugins.json` file.
- [x] My README.md describes the plugin's purpose and provides clear usage instructions.
- [x] I have read the developer policies at https://docs.obsidian.md/Developer+policies, and have assessed my plugin's adherence to these policies.
- [x] I have read the tips in https://docs.obsidian.md/Plugins/Releasing/Plugin+guidelines and have self-reviewed my plugin to avoid these common pitfalls.
- [x] I have added a license in the LICENSE file.
- [x] My project respects and is compatible with the original license of any code from other plugins that I'm using.
      I have given proper attribution to these other projects in my `README.md`.
```

Create the PR via CLI (paste your filled-out body in place of `BODY`):

```bash
cd ~/Documents/GitHub/obsidian-releases
gh pr create \
  --repo obsidianmd/obsidian-releases \
  --title "Add plugin: Your Plugin Name" \
  --body "BODY"
```

---

## 7. Triggering re-validation

After fixing any bot errors, push an empty commit to re-trigger the check:

```bash
cd ~/Documents/GitHub/obsidian-releases
git commit --allow-empty -m "Trigger PR validation"
git push
```

---

## Common bot errors and fixes

| Error | Fix |
|---|---|
| "Did not follow PR template" | Use the exact template from step 6 — `Link to my plugin:` must appear on its own line as shown |
| "Description mismatch" | Make `description` identical in `manifest.json`, `community-plugins.json`, and the PR body; then re-cut the release so the attached `manifest.json` is also updated |
| "No license" | Add a `LICENSE` file to the repo (MIT recommended), push, then re-cut the release |

---

## Checklist summary

- [ ] `manifest.json` with all required fields (`minAppVersion` matches highest API used)
- [ ] `main.js` (compiled)
- [ ] `versions.json`
- [ ] `LICENSE`
- [ ] `README.md`
- [ ] `eslint.config.mjs` set up and `npm run lint` passes clean
- [ ] GitHub repo created and pushed
- [ ] GitHub release `1.0.0` with `main.js` and `manifest.json` attached
- [ ] `obsidian-releases` forked and entry added to `community-plugins.json`
- [ ] PR opened using exact template format
- [ ] All descriptions match across `manifest.json`, `community-plugins.json`, and PR
