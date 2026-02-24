# annotaid

annotaid is a browser-based, no-code tool for LLM-assisted qualitative coding of open text. It runs entirely locally via LM Studio or Ollama — no programming skills, no cloud, no data leaves your machine.

## Getting started

### Prerequisites

Before using annotaid, you need a local LLM service running on your machine:

- **<a href="https://lmstudio.ai" target="_blank">LM Studio</a>** — download, load a model, start the local server, and enable CORS under *Settings → Local Server → Enable CORS*
- **<a href="https://ollama.com" target="_blank">Ollama</a>** — install, pull a model (`ollama pull <model>`), and set `OLLAMA_ORIGINS=*` before starting (`ollama serve`)

### Option 1 — Firefox (online, no download needed)

Open the app directly in your browser:

👉 **<a href="https://sprengholz.github.io/annotaid" target="_blank">https://sprengholz.github.io/annotaid</a>**

> **Firefox only.** Due to a browser security policy introduced in Chrome 142+, the online version does not work in Chrome, Edge, or Safari. Use Option 2 if you are on one of those browsers.

### Option 2 — All browsers (run locally)

Download and serve the app on your own machine. This works in all browsers and requires only Python, which is pre-installed on macOS and most Linux systems.

**macOS / Linux:**
```bash
git clone https://github.com/sprengholz/annotaid.git
cd annotaid
python3 -m http.server 8000
```

**Windows:**
```bash
git clone https://github.com/sprengholz/annotaid.git
cd annotaid
python -m http.server 8000
```

Then open **<a href="http://localhost:8000" target="_blank">http://localhost:8000</a>** in any browser.

To stop the server, press `Ctrl+C` in the terminal. Next time, just run the `python` command again from the `annotaid` folder — no need to clone again.

## Developer documentation

<a href="https://github.com/sprengholz/annotaid/blob/main/DEVELOPER.md" target="_blank">https://github.com/sprengholz/annotaid/blob/main/DEVELOPER.md</a>
