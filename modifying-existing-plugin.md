# Modifying an Existing Obsidian Plugin

How to fork, run locally, and modify a community plugin.

---

## 1. Fork and clone

```bash
gh repo fork <author>/<plugin-repo> --clone
mv <plugin-repo> ~/Documents/GitHub/<plugin-repo>
cd ~/Documents/GitHub/<plugin-repo>
```

---

## 2. Install dependencies and build

```bash
npm install
npm run build
```

This produces a fresh `main.js` from the TypeScript source.

---

## 3. Swap the vault plugin for your fork

Remove the installed version and symlink your local fork in its place:

```bash
rm -rf ~/vaults/Vault/.obsidian/plugins/<plugin-id>
ln -s ~/Documents/GitHub/<plugin-repo> ~/vaults/Vault/.obsidian/plugins/<plugin-id>
```

> The `plugin-id` is the `id` field in the plugin's `manifest.json` — it may differ from the repo name.

Reload the plugin in Obsidian: **Settings → Community plugins → toggle off, then on**.

---

## 4. Make changes

Edit `main.ts` (or whichever source files are relevant), then rebuild:

```bash
npm run build
```

Reload the plugin in Obsidian after each build to pick up changes.

> **Quick reload:** `Cmd+P` → "Reload app" is faster than toggling via Settings and picks up all plugin changes.

### For live rebuilds while editing

```bash
npm run dev
```

This watches for file changes and rebuilds automatically. You still need to reload the plugin in Obsidian after each rebuild.

---

## 5. Reverting to the original

To go back to the unmodified community plugin, disable and re-enable it from the Community plugins browser — Obsidian will restore the original files.

Or manually:

```bash
rm ~/vaults/Vault/.obsidian/plugins/<plugin-id>   # removes the symlink
```

Then re-install from **Settings → Community plugins → Browse**.

---

## Tips

- Check `package.json` for the available scripts — some plugins use `esbuild`, others use `rollup` or a custom build script. The `build` and `dev` commands may differ.
- If the plugin has no `dev` script, you can usually add one by appending `--watch` to the build command in `package.json`.
- Keep your fork's `main` branch in sync with upstream so you can pull in updates:
  ```bash
  git remote add upstream https://github.com/<author>/<plugin-repo>
  git fetch upstream
  git merge upstream/main
  npm run build
  ```
