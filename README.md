# colab-llm-cloudflare-api

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/callmenguyen/colab-llm-cloudflare-api/blob/main/colab-llm-api.ipynb)

A Colab notebook that runs an open model with [Ollama](https://ollama.com) on the free T4 GPU and
gives you a **temporary, key-protected, OpenAI-compatible API URL** through a Cloudflare quick
tunnel. No credit card, no Cloudflare account, no ngrok token.

## Quick start

1. Click **Open in Colab** above.
2. `Runtime → Change runtime type → T4 GPU`.
3. Pick a model and a time limit in cell 1, then `Runtime → Run all`.
4. After a few minutes, cell 2 prints a base URL and an API key:

```python
from openai import OpenAI

client = OpenAI(base_url="https://<your-url>.trycloudflare.com/v1", api_key="sk-...")
reply = client.chat.completions.create(
    model="qwen2.5:7b-instruct-q4_K_M",
    messages=[{"role": "user", "content": "Hello"}],
)
print(reply.choices[0].message.content)
```

Cell 3 keeps the API alive and shuts it down at the time limit. Stopping cell 3 shuts down early.

## What it does

- Installs Ollama (and `zstd`, which its installer needs but the Colab image lacks), downloads the
  model and loads it into GPU memory before handing out the URL.
- Puts a small auth proxy in front of Ollama: every request needs `Authorization: Bearer <key>`,
  otherwise `401`. The key is random for each run and passed to the proxy through the
  environment, not the command line.
- Streams responses chunk by chunk, so `stream=True` works through the tunnel.
- Waits until the new `trycloudflare.com` URL actually answers (about 30 seconds after it's
  issued) before printing it.
- Keeps the kernel busy so the session isn't dropped as idle, stops everything at the time limit,
  and by default releases the GPU runtime to save your Colab quota.

Both APIs are served at the same URL: OpenAI-compatible under `/v1`, Ollama's native API under
`/api`. Logs are in `/content/llm-api/` on the runtime.

## Limits

- The URL changes every run and dies with the runtime. This is for testing and short jobs, not
  for anything that needs a stable endpoint.
- Free Colab can disconnect you, ask "Are you still there?", or refuse a GPU after heavy use.
  Keep the tab open.
- The API key appears in the notebook output. Clear outputs before sharing or saving the notebook
  anywhere public.
- Colab's terms forbid hosting a web service for others and using several accounts to get around
  its limits. Use this only for your own short, interactive testing.
