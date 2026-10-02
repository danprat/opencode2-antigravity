# @danprat/opencode2-antigravity

[![npm version](https://img.shields.io/npm/v/@danprat/opencode2-antigravity?logo=npm)](https://www.npmjs.com/package/@danprat/opencode2-antigravity)
[![license](https://img.shields.io/npm/l/@danprat/opencode2-antigravity)](LICENSE)

**opencode2-antigravity** is an [OpenCode 2.0](https://opencode.ai) plugin that connects OpenCode directly to Google Antigravity / Cloud Code Assist models — Gemini (3.8 Flash, 3.7 Flash, 3.6 Flash, 3.1 Pro), Claude (Sonnet 4.6, Opus 4.6), and GPT-OSS (120b).

It handles Google OAuth login (PKCE), native SSE streaming, thinking levels, model routing, and quota diagnostics internally without external CLI dependencies.

> **Unofficial integration.** This project is not affiliated with or endorsed by Google. Use it only with an account and services you are authorized to access.

---

## Features

- **Native OpenCode 2.0 Architecture**: Implements 2.0 hooks and transforms (`ctx.provider.transform`, `ctx.integration.transform`, `ctx.aisdk.hook`, `ctx.tool.transform`).
- **Complete Model Catalog**: Gemini 3.8/3.7/3.6/3.5, Claude Sonnet/Opus, and GPT-OSS with thinking effort levels (Low, Medium, High).
- **In-Memory Inventory**: Registers models and providers directly in memory without modifying your `opencode.json`.
- **Integrated Authentication**: Seamless OAuth PKCE flow via OpenCode 2.0 `/connect` command.
- **Built-in Tools & Commands**: Image generation (`/antigravity-image`), quota summary (`/antigravity-usage`), and runtime model inspector (`/antigravity-models`).

---

## Installation

Add the plugin to your OpenCode 2.0 configuration (`~/.config/opencode/opencode.json`):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugins": ["@danprat/opencode2-antigravity/plugin/opencode2"]
}
```

OpenCode 2.0 will automatically download and activate the plugin on startup.

---

## Quick Start

1. **Add the plugin** to `~/.config/opencode/opencode.json` as shown above.
2. **Restart OpenCode** or start a session.
3. **Connect Account**:
   - Run `/connect` in OpenCode.
   - Choose **Antigravity**.
   - Complete Google login in your browser.
4. **Pick a Model**:
   - Run `/models` (filter by provider **Antigravity**).
   - Select e.g. `antigravity/gemini-3.8-flash`.
5. **Start coding!** Check your quota at any time using `/antigravity-usage`.

---

## Commands & Tools

| Command / Tool | Description |
| --- | --- |
| `/antigravity-usage` | Displays server-reported quota pools and reset times (`antigravity_usage` tool). |
| `/antigravity-models` | Lists available runtime models, quota, and capabilities (`antigravity_models` tool). |
| `/antigravity-image` | Generates images via Antigravity image models (`generate_image` tool). |

The tools above are also available directly for the model to invoke when needed.

---

## Models & Routing

Antigravity exposes Google Gemini along with Claude and GPT-OSS models served through Vertex backend endpoints under the unified `antigravity` provider:

| Public Model ID | Input | Thinking Levels | Max Output | Request Routing |
| --- | --- | --- | --- | --- |
| `gemini-3.8-flash` | Text, Image | Low, Medium, High | 65,536 | `gemini-3.8-flash-tiered` |
| `gemini-3.7-flash` | Text, Image | Low, Medium, High | 65,536 | `gemini-3.7-flash-tiered` |
| `gemini-3.6-flash` | Text, Image | Low, Medium, High | 65,536 | Per-effort runtime IDs |
| `gemini-3.5-flash` | Text, Image | Low, Medium, High | 65,536 | Per-effort runtime IDs |
| `gemini-3.1-pro` | Text, Image | Low, High | 65,535 | Per-effort runtime IDs |
| `claude-sonnet-4-6` | Text, Image | High | 64,000 | `claude-sonnet-4-6` |
| `claude-opus-4-6` | Text, Image | High | 64,000 | `claude-opus-4-6-thinking` |
| `gpt-oss-120b` | Text | Medium | 32,768 | `gpt-oss-120b-medium` |

*Unspecified thinking effort automatically defaults to **high**.*

---

## Configuration & Environment Variables

Credentials can be supplied via `/connect` or customized using environment variables (lookup order: `OPENCODE_ANTIGRAVITY_*` → `ANTIGRAVITY_*` → `NOAGY_*`):

| Variable | Description |
| --- | --- |
| `ANTIGRAVITY_ACCESS_TOKEN` / `GOOGLE_ACCESS_TOKEN` | Direct OAuth access token override (`ya29...`). |
| `ANTIGRAVITY_PROJECT_ID` | Specify a custom Google Cloud Code Assist project ID. |
| `ANTIGRAVITY_BASE_URL` | Override the Google API base URL (must be HTTPS allowed Google host). |
| `ANTIGRAVITY_CALLBACK_HOST` | Bind OAuth callback to `127.0.0.1`, `::1`, or `localhost` (default: `127.0.0.1`). |
| `ANTIGRAVITY_CLIENT_ID` / `ANTIGRAVITY_CLIENT_SECRET` | Use custom Google OAuth client credentials. |

---

## Development & Local Testing

This project uses [Bun](https://bun.sh) for building and testing:

```bash
# Install dependencies
bun install

# Run typechecking and tests
bun run typecheck
bun test

# Build bundles
bun run build
```

### Loading Local Builds in OpenCode 2.0

To run OpenCode directly against your local checkout without npm:

1. Create a local plugin directory in `~/.config/opencode/plugins/antigravity`:
   ```bash
   mkdir -p ~/.config/opencode/plugins/antigravity
   ```

2. Add `package.json`:
   ```json
   { "name": "antigravity-local", "type": "module", "main": "./index.js" }
   ```

3. Add `index.js` pointing to your local `dist/`:
   ```javascript
   process.env.ANTIGRAVITY_OPENCODE2_DEV_ENTRY ??=
     "/absolute/path/to/opencode2-antigravity/dist/sdk.js";

   export default (
     await import("file:///absolute/path/to/opencode2-antigravity/dist/plugin-opencode2.js")
   ).default;
   ```

---

## License

[MIT](LICENSE)
