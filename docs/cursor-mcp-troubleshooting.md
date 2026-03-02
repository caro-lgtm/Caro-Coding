# Cursor IDE MCP Setup – Diagnostic & Fix (Student Checklist)

## 1. Config location and which one is active

### Where Cursor reads MCP config

| Location | Scope | Path |
|----------|--------|------|
| **Project** | This repo only | `<project-root>/.cursor/mcp.json` |
| **Global** | All projects | `~/.cursor/mcp.json` (e.g. `C:\Users\<You>\.cursor\mcp.json` or `~/.cursor/mcp.json` on Mac/Linux) |

- **If both exist:** project `.cursor/mcp.json` overrides/extends global for that project. Global is used when no project file exists.
- **Settings UI:** When you use **Settings → Tools and MCP** and add or edit servers, Cursor typically writes to the **global** `~/.cursor/mcp.json`. Pasting invalid JSON there can corrupt that file and trigger the error you see.

### How to verify which config is active

1. **Check global file**
   - Open: `~/.cursor/mcp.json` (Mac/Linux) or `%USERPROFILE%\.cursor\mcp.json` (Windows).
   - If the content you pasted is here, that’s what Cursor is using (and why you see the error if the format is wrong).

2. **Check project file (optional)**
   - In your project folder, open `.cursor/mcp.json` if it exists.
   - If it exists, Cursor uses it for that project (possibly in addition to or instead of global, depending on Cursor version).

3. **After fixing**
   - Restart Cursor fully (quit and reopen).
   - In **Settings → Tools and MCP**, the “Figma Desktop” server should appear under Installed MCP servers.

---

## 2. Correct JSON structure Cursor expects

Cursor expects the **top-level key to be `mcpServers`**, not `servers`. The value must be a **single object** (map of server name → config). Anything else (array, string, wrong key) can produce the error you described.

### Wrong (what Figma’s snippet often shows – causes the error)

```json
{
  "servers": {
    "Figma Desktop": {
      "type": "http",
      "url": "http://127.0.0.1:3845/mcp"
    }
  }
}
```

Problems: top-level key is `servers`; Cursor looks for `mcpServers`. Also, for HTTP in Cursor the type is usually `streamableHttp`, not `http`.

### Correct – minimal (recommended to try first)

```json
{
  "mcpServers": {
    "Figma Desktop": {
      "url": "http://127.0.0.1:3845/mcp"
    }
  }
}
```

### Correct – with explicit HTTP type (if your Cursor version requires it)

```json
{
  "mcpServers": {
    "Figma Desktop": {
      "type": "streamableHttp",
      "url": "http://127.0.0.1:3845/mcp"
    }
  }
}
```

Use **one** of the two correct blocks above. Replace the entire content of `~/.cursor/mcp.json` with it (or add the `"Figma Desktop"` entry inside an existing `mcpServers` object).

---

## 3. Common causes of “mcpServers is not valid” / “can’t read entry” type errors

- **Wrong top-level key** – Using `"servers"` instead of `"mcpServers"`. Cursor only reads `mcpServers`.
- **Wrong nesting** – `mcpServers` must be an **object** `{ "Server Name": { ... } }`, not an array `[ ... ]` or a string.
- **Value type mismatch** – One of the server entries is a string or array instead of an object (e.g. `"Figma Desktop": "http://..."` instead of `"Figma Desktop": { "url": "..." }`).
- **Invalid JSON** – Trailing commas, unquoted keys, single quotes, or comments. Use double quotes, no trailing commas, no comments.
- **Duplicate keys** – Two keys named `mcpServers` or two identical server names; JSON allows it but behavior is undefined.
- **Quoting / encoding** – Pasted from a doc with “smart” quotes or hidden characters. Retype the key as `mcpServers` in a plain text editor if unsure.

---

## 4. Quick verification steps

### A. Confirm the local MCP server is running (Figma Desktop)

- Open **Figma Desktop** (the app).
- Go to **Menu → Preferences** (or **Settings**).
- Find the option to **enable the Dev Mode / MCP server** and turn it on.
- Leave Figma Desktop open; the MCP server runs at `http://127.0.0.1:3845/mcp` while it’s enabled.

### B. Confirm the URL is reachable

- In a browser, open: `http://127.0.0.1:3845/mcp`  
  (You may see an error or “method not allowed” – that’s OK; it means the port is open and something is responding.)
- Or from a terminal:
  - **Mac/Linux:** `curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:3845/mcp`
  - **Windows (PowerShell):** `(Invoke-WebRequest -Uri "http://127.0.0.1:3845/mcp" -UseBasicParsing).StatusCode`
- If you get “connection refused” or similar, the Figma MCP server isn’t running; enable it in Figma Desktop and try again.

### C. Cursor logs / diagnostics for MCP

- **Open logs:** Help → Toggle Developer Tools → Console tab, or check Cursor’s log file location for your OS.
- **What to look for:** Messages containing “MCP”, “mcpServers”, or “mcp.json”; any JSON parse errors or “invalid config” messages.
- **After changing config:** Fully quit Cursor, fix `mcp.json`, save, then start Cursor again and check the console for MCP-related errors when opening Settings → Tools and MCP.

---

## 5. Concise checklist to send to the student

- [ ] **1. Fix the config key:** Use `mcpServers` (not `servers`) as the only top-level key.
- [ ] **2. Use the correct file:** Edit `~/.cursor/mcp.json` (global) or the project’s `.cursor/mcp.json`. If you pasted into the UI, the global file was likely updated (and may be invalid).
- [ ] **3. Use valid JSON:** Object only; double quotes; no trailing commas; no comments. Validate at [jsonlint.com](https://jsonlint.com) if needed.
- [ ] **4. Paste the correct block:** Replace the file content with one of the “Correct” examples in section 2 (e.g. the minimal one with just `mcpServers` and `url`).
- [ ] **5. Figma Desktop:** App open, MCP/Dev Mode server enabled in Preferences.
- [ ] **6. Port check:** Confirm `http://127.0.0.1:3845/mcp` is reachable (browser or curl).
- [ ] **7. Restart Cursor:** Full quit and reopen after saving `mcp.json`.
- [ ] **8. Check Settings:** Settings → Tools and MCP → “Figma Desktop” should appear under Installed MCP servers.
- [ ] **9. If it still fails:** Check Developer Tools console and any MCP-related log lines; fix JSON or URL based on errors.

---

## Corrected JSON config example (copy-paste ready)

**Minimal (recommended):**

```json
{
  "mcpServers": {
    "Figma Desktop": {
      "url": "http://127.0.0.1:3845/mcp"
    }
  }
}
```

**With explicit type:**

```json
{
  "mcpServers": {
    "Figma Desktop": {
      "type": "streamableHttp",
      "url": "http://127.0.0.1:3845/mcp"
    }
  }
}
```

Save one of these as the **entire** content of `~/.cursor/mcp.json`, then restart Cursor and verify in Settings → Tools and MCP.
