# annotaid

annotaid is a browser-based, no-code tool for LLM-assisted qualitative coding of open text. It runs entirely locally via LM Studio or Ollama — no programming skills, no cloud, no data leaves your machine.

## Getting started

### Prerequisites

Before using annotaid, you need a local LLM service running on your machine:

- **<a href="https://lmstudio.ai" target="_blank">LM Studio</a>** — download and install the app, download a model from the model catalogue, load the model, start the local server (*Local Server* tab → *Start Server*), and enable CORS under *Settings → Local Server → Enable CORS*
- **<a href="https://ollama.com" target="_blank">Ollama</a>** — install the app, pull a model (`ollama pull <model>`), and set `OLLAMA_ORIGINS=*` before starting the server (`ollama serve`)

### Recommended model settings

For consistent, reproducible coding results, configure the following in your LLM service before running annotaid:

- **Temperature = 0** — Temperature controls how random the model's output is. Setting it to 0 makes the model deterministic: given the same input, it will always produce the same output. This is important for coding tasks where you want consistent labels, not random variation.
- **Disable thinking (if available)** — Some models (e.g. DeepSeek-R1, QwQ) have a "thinking" or "reasoning" mode that generates a chain-of-thought before the final answer. This is unnecessary for structured coding tasks and produces noisy output that will end up in your results. Disable it if your model and service offer that option.

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
