# Terminal Setup + Local AI Inference on Samsung Galaxy M21 (SM-M215F)

## SSH Access

**Install & start SSH server (on phone):**
```bash
pkg install openssh
passwd                          # set a password for SSH
sshd                            # daemon runs on port 8022
```

**Connect from PC:**
```bash
ssh -p 8022 u0_aXXX@<phone-ip>
```
Username is shown in the Termux prompt (e.g., `u0_a387`). Find IP with `ip a | grep inet`.

## Persistent Sessions with tmux

Keep long-running processes alive after disconnect:
```bash
pkg install tmux
tmux                            # start session
# run your server here
# Ctrl+B then d    → detach (server continues)
# tmux attach       → reattach later
```

## Build llama.cpp

```bash
pkg install cmake ninja build-essential git
git clone --depth 1 https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build -DGGML_OPENMP=OFF -DCMAKE_BUILD_TYPE=Release
cmake --build build -j4
```

Primary binary: `build/bin/llama-server`

## Download a Model via Hugging Face

Use the `-hf` flag to download directly from Hugging Face:
```bash
llama-server -hf LiquidAI/LFM2.5-1.2B-Instruct-GGUF:Q4_K_M
```
This downloads the model to `~/.cache/huggingface/` and loads it automatically.

Other compatible models:
- **Qwen 2.5 1.5B** — best quality-per-RAM, ~12–15 tok/s
- **SmolLM 1.7B** — solid alternative
- **Gemma 4 2B** — fits at Q4_K_M with small context
- **LFM 2.5 1.2B** — fastest, ~14–18 tok/s

## Serve on Local Network

Start llama-server bound to all interfaces:
```bash
llama-server \
  -hf LiquidAI/LFM2.5-1.2B-Instruct-GGUF:Q4_K_M \
  -c 4096 \
  -t 4 \
  -fit off \
  --host 0.0.0.0 \
  --port 8080
```

Access from any device on your network:
- **Web UI:** `http://<phone-ip>:8080/`
- **API:** `http://<phone-ip>:8080/v1/chat/completions` (OpenAI-compatible, streaming supported.)

## One-liner Remote Start

From PC, start server inside a new tmux session:
```bash
ssh -p 8022 u0_aXXX@<phone-ip> \
  "tmux new-session -d -s llm 'llama-server -hf LiquidAI/LFM2.5-1.2B-Instruct-GGUF:Q4_K_M -c 4096 -t 4 -fit off --host 0.0.0.0 --port 8080'"
```

## Performance Notes

- Exynos 9611 (4 GB RAM), CPU-only inference
- `-t 4` avoids thermal throttling; `-t 6` may be faster but heats up
- After LineageOS idle (~0.8 GB), ~3.2 GB free for model + context
- LFM 2.5 1.2B @ Q4_K_M: ~0.8 GB model, ~2380 MiB projected total RAM usage with 4k context, 14–18 tok/s
- Set `-c` (context size) based on remaining free RAM
- Use `-fit off` to prevent OOM kills during model loading (peak allocation spike)
- The `--host 0.0.0.0` flag is required to bind to all network interfaces; without it, the server only listens on localhost

## Testing from PC

```powershell
# Web UI (open in browser):
# http://<phone-ip>:8080/

# API test (PowerShell):
Invoke-RestMethod -Uri http://<phone-ip>:8080/v1/chat/completions -Method Post -ContentType "application/json" -Body '{"model":"LFM2.5-1.2B","messages":[{"role":"user","content":"Say hello"}]}'

# Using real curl (not PowerShell alias):
curl.exe http://<phone-ip>:8080/v1/chat/completions -H "Content-Type: application/json" -d '{"model":"LFM2.5-1.2B","messages":[{"role":"user","content":"Say hello"}]}'
```
