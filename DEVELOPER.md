# annotaid — Developer Documentation (v 1.0)

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [File Structure](#file-structure)
4. [Tech Stack](#tech-stack)
5. [Running Locally](#running-locally)
6. [LLM API Integration](#llm-api-integration)
7. [Key Subsystems](#key-subsystems)
   - [Provider Abstraction](#provider-abstraction)
   - [State & Persistence](#state--persistence)
   - [CSV Parser](#csv-parser)
   - [Batch Processor](#batch-processor)
   - [Streaming Response Handler](#streaming-response-handler)
8. [UI Architecture](#ui-architecture)
9. [Design System](#design-system)
10. [Deployment](#deployment)
    - [HTTPS / CORS Constraints](#https--cors-constraints)
11. [Contributing](#contributing)

---

## Project Overview

**annotaid** is a privacy-first, browser-only tool for LLM-assisted qualitative coding of open-text data. It connects to a locally running LLM service (LM Studio, Ollama, or any OpenAI-compatible API) and never transmits user data to external servers.

The application has no build step, no framework dependencies, and no backend. It is a single HTML file + CSS file + font assets.

---

## Architecture

```
┌─────────────────────────────────────────┐
│              Browser (Client)           │
│                                         │
│  ┌───────────┐   ┌────────────────────┐ │
│  │ index.html│   │     style.css      │ │
│  │ (HTML+JS) │   │  (CSS variables +  │ │
│  │           │   │   component styles)│ │
│  └─────┬─────┘   └────────────────────┘ │
│        │                                │
│        ▼                                │
│  ┌───────────────────────────────────┐  │
│  │          JavaScript Core          │  │
│  │                                   │  │
│  │  - Provider abstraction           │  │
│  │  - Fetch / SSE streaming          │  │
│  │  - CSV parser                     │  │
│  │  - Batch processor (concurrency)  │  │
│  │  - localStorage persistence       │  │
│  └──────────────┬────────────────────┘  │
│                 │                       │
└─────────────────┼───────────────────────┘
                  │  HTTP to 127.0.0.1 (loopback exception applies)
                  ▼
┌─────────────────────────────────────────┐
│     Local LLM Service (user-provided)   │
│                                         │
│   LM Studio  │  Ollama  │  Compatible   │
│ :1234        │  :11434  │  endpoint     │
└─────────────────────────────────────────┘
```

The app is a two-panel SPA:

- **TEST** — Send a single text to the LLM and inspect the response in real time.
- **BATCH** — Upload a CSV, pick a column, preview coding on 5 rows, then process the entire dataset with configurable concurrency.

---

## File Structure

```
annotaid/
├── index.html          Main application — HTML markup + all JavaScript
├── style.css           All styles; uses CSS custom properties for theming
├── LICENSE             MIT
├── README.md           End-user readme
├── DEVELOPER.md        This file
└── fonts/
    ├── DMMono-300-latin.woff2
    ├── DMMono-300italic-latin.woff2
    ├── DMMono-400-latin.woff2
    ├── DMSans-latin.woff2
    └── PlayfairDisplay-700-latin.woff2
```

`index.html` is the only file that matters at runtime. Inline `<script>` tags contain all application logic (~1,300 lines).

---

## Tech Stack

| Layer | Technology |
|---|---|
| Markup | HTML5 |
| Styles | CSS3 with custom properties |
| Logic | Vanilla ES2020+ JavaScript |
| HTTP | `fetch()` with `AbortController` |
| Streaming | Server-Sent Events (SSE) via `ReadableStream` |
| Persistence | `localStorage` |
| File I/O | `FileReader` API (upload), `Blob` + `URL.createObjectURL` (download) |
| Icons | Lucide SVG icons (inlined, MIT) |
| Fonts | DM Mono, DM Sans, Playfair Display (self-hosted WOFF2) |

No npm, no bundler, no transpiler. Everything runs as-is in modern browsers.

---

## Running Locally

### Option A — Open as a file

```bash
# Clone and open directly. No server needed.
git clone https://github.com/sprengholz/annotaid.git
open annotaid/index.html
```

> **Note:** Some browsers restrict `fetch()` to same-origin when using `file://`. If model loading fails, use Option B.

### Option B — Serve via HTTP

```bash
# Python (built-in)
python -m http.server 8000

# Node (if available)
npx serve .
```

Then open `http://localhost:8000`.

### Prerequisites

A local LLM service must be running before using the app:

| Service | Default address | Notes |
|---|---|---|
| LM Studio | `http://127.0.0.1:1234` | Enable local server in the app |
| Ollama | `http://127.0.0.1:11434` | `ollama serve` |
| Other | User-configured | Must expose `/v1/chat/completions` |

---

## LLM API Integration

### Model Discovery

On load (and on refresh), the app fetches the list of available models from the configured provider:

| Provider | Endpoint |
|---|---|
| LM Studio | `GET {base}/api/v0/models` |
| Ollama | `GET {base}/api/tags` |
| Generic | `GET {base}/v1/models` |

Each provider entry in `PROVIDERS` includes a `parseModels(data)` function that normalises the response into an array of `{ id, name }` objects.

### Chat Completions

All text generation goes through a single endpoint:

```
POST {base}/v1/chat/completions
Content-Type: application/json

{
  "model": "<selected model id>",
  "messages": [
    { "role": "system", "content": "<context / instructions>" },
    { "role": "user",   "content": "<text to code>" }
  ],
  "stream": true
}
```

Responses are consumed as an SSE stream. Each `data:` line is parsed as JSON; the `choices[0].delta.content` field is appended to the output.

A sentinel `data: [DONE]` ends the stream.

---

## Key Subsystems

### Provider Abstraction

```javascript
const PROVIDERS = {
  lmstudio: {
    defaultBase:  'http://127.0.0.1:1234',
    modelsPath:   '/api/v0/models',
    parseModels:  (data) => data.data.map(m => ({ id: m.id, name: m.id }))
  },
  ollama: {
    defaultBase:  'http://127.0.0.1:11434',
    modelsPath:   '/api/tags',
    parseModels:  (data) => data.models.map(m => ({ id: m.name, name: m.name }))
  },
  generic: {
    defaultBase:  '',
    modelsPath:   '/v1/models',
    parseModels:  (data) => data.data.map(m => ({ id: m.id, name: m.id }))
  }
};
```

To add a new provider: add an entry to `PROVIDERS`, add a `<option>` to the provider `<select>` in the settings modal, and handle any auth headers in the `fetch` call if needed.

---

### State & Persistence

Runtime state lives in plain JavaScript variables. The following keys are persisted to `localStorage` automatically:

| Key | Type | Description |
|---|---|---|
| `rate-lm:context` | string | System prompt / instructions |
| `rate-lm:text` | string | Last test input |
| `rate-lm:server` | string | API base URL |
| `rate-lm:provider` | string | `lmstudio` / `ollama` / `generic` |
| `rate-lm:concurrency` | number (1–16) | Parallel request limit for batch |
| `rate-lm:last_column` | string | Last selected CSV column name |
| `rate-lm:last_tab` | string | `test` / `batch` |
| `rate-lm:gs_dismissed` | boolean | Whether the getting-started banner is hidden |

All keys use the prefix `rate-lm:` (legacy name retained for backwards compatibility).

---

### CSV Parser

The custom parser (`parseCSV`) is RFC 4180-compliant and handles:

- Quoted fields (including embedded commas and literal newlines)
- Automatic separator detection — counts occurrences of `,`, `;`, and `\t` in the first few rows and picks the most frequent
- Inconsistent row lengths (rows shorter than the header are right-padded with empty strings)
- UTF-8 via `FileReader.readAsText`

```
parseCSV(text: string) → { headers: string[], rows: string[][] }
```

Throws a descriptive `Error` if no headers can be detected.

---

### Batch Processor

Batch processing runs in chunks whose size equals the configured concurrency value. The algorithm:

```
for each chunk of rows (size = concurrency):
    await Promise.all(chunk.map(row => codeRow(row)))
    update progress bar
    if paused: break and store resumeFrom
```

**Retry logic:** Each row is attempted twice. On first failure the request is retried after 800 ms. If the retry also fails the cell is marked `ERROR`.

**Pause/resume:** When the user pauses, `resumeFrom` stores the last completed index. On resume, processing skips already-completed rows and continues from where it stopped.

**Output CSV:** After completion a CSV is assembled in memory as a `Blob`, a temporary object URL is created, and a download is triggered programmatically. The output contains all original columns plus a new `annotaid` column with the LLM response.

---

### Streaming Response Handler

```javascript
const reader = response.body.getReader();
const decoder = new TextDecoder();
let buffer = '';

while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  buffer += decoder.decode(value, { stream: true });
  const lines = buffer.split('\n');
  buffer = lines.pop(); // keep incomplete last line

  for (const line of lines) {
    if (!line.startsWith('data:')) continue;
    const payload = line.slice(5).trim();
    if (payload === '[DONE]') return;
    const json = JSON.parse(payload);
    const token = json.choices?.[0]?.delta?.content ?? '';
    appendToken(token); // updates DOM
  }
}
```

The `AbortController` signal is passed to `fetch()`. When the user clicks "Stop", `controller.abort()` is called, which causes the reader to throw an `AbortError` that is caught and handled gracefully.

---

## UI Architecture

The UI has no component framework. Structure is driven by HTML sections and JavaScript that adds/removes CSS classes to toggle visibility and state.

| Section | Purpose |
|---|---|
| `#server-modal` | Settings: provider, URL, concurrency |
| `#getting-started` | Dismissible onboarding banner |
| `#model-row` | Model picker dropdown + refresh button |
| `#context-area` | System prompt textarea (auto-resizes) |
| `#tabs` | TEST / BATCH tab switcher |
| `#test-panel` | Single-text testing interface |
| `#batch-panel` | CSV upload → column picker → preview → batch run |

**Tab switching** updates `rate-lm:last_tab` in localStorage and toggles the `active` class on panels.

**Auto-resizing textareas** use a shared `autoResize(el)` helper that sets `el.style.height` to `el.scrollHeight` on every `input` event.

**Token estimation** is a rough heuristic: `Math.round(chars / 4)`, displayed below the context textarea.

---

## Design System

All design tokens are CSS custom properties on `:root` in `style.css`:

```css
:root {
  --bg:           #f5f5f4;   /* page background */
  --surface:      #ffffff;   /* card / modal background */
  --border:       #d6d3d1;   /* borders and dividers */
  --accent:       #337681;   /* primary teal */
  --accent-dim:   #005461;   /* hover / active state */
  --accent-tint:  #edf5f6;   /* light teal highlight */
  --text:         #1c1917;   /* primary text */
  --text-muted:   #78716c;   /* secondary / placeholder text */
  --radius:       12px;      /* standard border radius */
}
```

**Typefaces** (self-hosted WOFF2, declared with `@font-face`):

| Family | Weight | Usage |
|---|---|---|
| Playfair Display | 700 | Logo / headings |
| DM Sans | 300–500 | Body text |
| DM Mono | 300–400 | Code, LLM output, CSV preview |

---

## Deployment

annotaid is a fully static site. No server-side logic is required.

**GitHub Pages (current):**
Push to `main` → GitHub Pages serves `index.html` at `https://sprengholz.github.io/annotaid`.

**Any static host:**
Copy `index.html`, `style.css`, and `fonts/` to the webroot. No build step required.

**Self-hosted:**
```bash
# Nginx example
server {
  listen 80;
  root /var/www/annotaid;
  index index.html;
}
```

---

### HTTPS / CORS Constraints

#### How `127.0.0.1` is resolved

`127.0.0.1` always resolves to **the machine running the browser**, not the machine serving the HTML file. This is intentional — each user connects to their *own* local LLM.

#### Mixed content, the loopback exception, and Chrome's Local Network Access policy

Browser behaviour here has changed significantly across versions. The table below reflects the current state:

| Browser | `fetch("http://127.0.0.1:…")` from GitHub Pages (HTTPS) | Notes |
|---|---|---|
| Firefox 84+ | ✅ Works | Simple loopback exception; no extra headers needed |
| Chrome ≤ 141 | ✅ Works | Old loopback exception (same as Firefox) |
| **Chrome 142+** | ❌ **Blocked** | New Local Network Access policy — requires `Access-Control-Allow-Private-Network: true` from the LLM server |
| Safari | ❌ Blocked | No loopback exception at all ([WebKit bug #171934](https://bugs.webkit.org/show_bug.cgi?id=171934)) |

**Chrome 142 introduced Local Network Access (LNA)**, which requires local servers to opt in by responding to CORS preflight requests with the header `Access-Control-Allow-Private-Network: true`. Neither LM Studio nor Ollama send this header yet — both have open issues ([LM Studio #392](https://github.com/lmstudio-ai/lmstudio-bug-tracker/issues/392), [Ollama #7000](https://github.com/ollama/ollama/issues/7000)). Until those are fixed upstream, **Chrome 142+ users cannot use the GitHub Pages version**.

#### Required user-side configuration (all browsers)

Users must enable CORS in their LLM service regardless of how the app is served:

| Service | How to enable CORS |
|---|---|
| LM Studio | *Settings → Local Server → Enable CORS* |
| Ollama | Set env variable `OLLAMA_ORIGINS=https://sprengholz.github.io` (or `*`) before starting |

#### Workaround: serve the app locally over HTTP

This is the **recommended approach for all users today**, since it bypasses all of the above restrictions entirely — no LNA policy, no mixed content, no `Access-Control-Allow-Private-Network` header needed:

```bash
git clone https://github.com/sprengholz/annotaid.git
cd annotaid
python -m http.server 8000
# Open http://localhost:8000
```

Both the app (`http://localhost:8000`) and the LLM service (`http://127.0.0.1:…`) are plain HTTP on the same machine. All browsers work, including Safari and Chrome 142+. CORS still needs to be enabled in the LLM service.

#### Future outlook

Two upstream fixes could restore full browser compatibility with the GitHub Pages version:

- **`Access-Control-Allow-Private-Network: true` header** — once LM Studio and Ollama send this header in their CORS preflight responses, Chrome 142+ users will be able to use the GitHub Pages version again (with a one-time browser permission prompt). Monitor the linked issues for progress ([LM Studio #392](https://github.com/lmstudio-ai/lmstudio-bug-tracker/issues/392), [Ollama #7000](https://github.com/ollama/ollama/issues/7000)).
- **Native HTTPS support in LLM services** — if LM Studio or Ollama add an HTTPS server option, users could point the app at an `https://` URL, eliminating the mixed-content problem entirely. The app already detects this scenario and prompts users to enable HTTPS when available.

---

## Contributing

1. Fork the repository and create a feature branch.
2. Make changes to `index.html` (JavaScript) and/or `style.css`.
3. Test manually — open the file in a browser with a local LLM running.
4. Test edge cases: malformed CSVs, network errors, abort during batch, very long prompts.
5. Open a pull request against `main` with a clear description of the change.

There is no linter, formatter, or test suite configured. Keep the zero-dependency principle — do not introduce npm packages or a build step.
