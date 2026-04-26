# Auditing an Obsidian Plugin

How to evaluate a community plugin for safety and privacy in about 10-15 minutes. You're looking for red flags, not proving it's perfect.

---

## 1. Check repo trust signals

Go to the plugin's GitHub page and scan:

- **Stars / downloads** — rough popularity proxy
- **Contributors** — more reviewers means more eyes on the code
- **Last commit date** — ideally within the last year
- **Issues tab** — look for unresolved security or privacy complaints

A plugin with many users and no security complaints is generally low risk.

---

## 2. Read the manifest

Open `manifest.json` and check:

| Field | What to verify |
|---|---|
| `id` / `name` | Matches what you installed |
| `description` | Accurately describes what it does |
| `author` | Identifiable person or org |

**Red flag:** Vague or misleading description that doesn't match the plugin's actual behavior.

---

## 3. Scan the main source file

Open `main.js` (or `main.ts` if it's a TypeScript source repo).

Use Cmd+F to search for these keywords:

### Network access

```
fetch(
axios
XMLHttpRequest
requestUrl
```

These indicate the plugin makes network requests. That's not automatically bad — a plugin that fetches page titles needs to. Ask: is this expected given what the plugin claims to do?

### Higher-risk APIs

```
eval(
Function(
child_process
```

These are not always malicious, but unexpected usage warrants a closer look.

### File system access

```
fs.
writeFile
readFile
```

All Obsidian plugins can read and write vault files via the Obsidian API. Direct `fs` usage (bypassing the API) is unusual and worth investigating.

---

## 4. Check where data is going

If you found network calls, look at the target URLs:

- `https://api.github.com` — well-known public API, probably fine
- `https://api.some-unknown-service.com` — third-party service, investigate more

Ask: Is the plugin sending your note content to an external server? Is that documented in the README?

**Example tradeoff:** A "fetch page title from URL" plugin has to make external requests by design. That's a documented, acceptable privacy tradeoff. An undocumented call to a proprietary API is not.

---

## 5. Test in a throwaway vault first

Before using a new plugin on your real vault:

1. Create a test vault in Obsidian (**File → Open vault → Create new vault**)
2. Install the plugin there
3. Watch for unexpected behavior — files modified, network activity, errors in the developer console (`Cmd+Option+I`)

Optional: use a firewall like Little Snitch to monitor outbound connections while the plugin is active.

---

## 6. Check community reputation

Search:

- Reddit: `obsidian plugin <name>`
- The plugin's GitHub Issues tab

Real users surface broken vaults and unexpected behavior quickly. A plugin with thousands of users and no complaints is a strong signal.

---

## Quick checklist

Before installing, check all of these:

- [ ] Open source with visible code
- [ ] Actively maintained (commit within the last year)
- [ ] No security or privacy complaints in issues or Reddit
- [ ] Description matches actual functionality
- [ ] Network calls are expected and documented (or absent)
- [ ] Tested in a throwaway vault

---

## Risk by plugin type

| Plugin type | Risk level | Why |
|---|---|---|
| Formatting, themes, UI | Low | No network access needed |
| Local search, graph tools | Low | Reads vault, no external calls |
| Sync, backup plugins | Medium | File access + potential network |
| API integrations, AI tools | Higher | Sends data externally by design |
| Abandoned plugins | Higher | No security patches |

---

## The biggest real risk is privacy, not malware

Plugins that fetch URLs, call APIs, or sync data can quietly expose:

- The content of your notes
- Internal links and structure
- Research topics and metadata

This is usually undocumented rather than intentionally malicious — but the effect on your privacy is the same. Review network calls carefully for any plugin that touches your note content.
