---
title: "Running llama-server on Android: A Complete Walkthrough"
author: ''
date: '2026-06-08'
slug: []
categories:
  - Blog
tags:
  - AI
  - Android
  - Hardware
  - Tutorial
draft: true
---

This post continues from [AI llama-server on a phone](/posts/ai-llama-server-on-a-phone/) with a detailed walkthrough of every step needed to flash LineageOS, set up Termux, build llama.cpp, and serve a small language model over your local network.

*Not interested in the whole walkthrough? Maybe you're interested in jailbreaking phones but not AI; perhaps you want to explore hosting local AI models but have no interest in the messy world of Android and phone hardware? I've deliberately separated out the elements so you can pick and choose what you want to read.*

## Prerequisites

To follow along from end to end you'll need:
- An older Android phone (this guide uses a Samsung Galaxy M21, but works for other devices - practical limit around 4GB RAM)
- A PC with ADB (Android Debug Bridge) and Odin (for Samsung devices)
- USB cable to connect your phone
- A few hours of patience
- Backups of anything you care about (this **will** wipe your phone)

The Samsung Galaxy M21 is a good candidate because it's affordable, has reasonable specs (4GB RAM, Exynos 9611), and is supported by the LineageOS community (if not LineageOS directly - see [https://wiki.lineageos.org/devices/](https://wiki.lineageos.org/devices/) for officially supported hardware). 

## Part 1: Preparing Your Phone

If you just want to run the AI server on your stock Android phone (or any other hardware) then ignore this and go straight to [part 2](#part-2-flashing-lineageos).

### Step 1.1: Enable Developer Options and USB Debugging

1. Go to **Settings → About Phone**
2. Tap **Build Number** 7 times until you see "Developer mode enabled"
3. Go to **Settings → Developer Options**
4. Enable **USB Debugging**
5. Connect your phone to your PC with USB

### Step 1.2: Unlock the Bootloader

This step varies by manufacturer. For Samsung:

```bash
adb reboot bootloader
# Your phone enters Download Mode (large text at top)
# Hold Volume Up to enter Fastboot (if available) or proceed to Odin
```

**Warning:** Unlocking the bootloader will wipe your device.

1. Download **Odin** from a trusted source (search "Samsung Odin")
2. Put your phone in **Download Mode**: Power Off → Hold Volume Down + Power + Home
3. Open Odin, select your phone in the device list
4. In the **Option** tab, uncheck "Auto Reboot"
5. In **Download Mode**, click **Unlock** or use ADB:
   ```bash
   adb reboot download
   # Then use Odin to send unlock command
   ```

I've flashed a few phones and tablets in the past (both Samsung and non-Samsung devices) but for this phone in particular I was surprised by a security feature I hadn't encountered before: **"KG State Prenormal"**. This is a security feature that prevents flashing unsigned ROMs. If you encounter this:
1. In the stock Android OS (called the stock "ROM"), ensure the phone is on your WiFi network and connected to the internet, with the correct date set. This will register your device on Samsung's servers.
1. Turn off WiFi and sever your connection to the internet.
1. Temporarily set your phone's date to at least 3 months before (I set to January 1, 2026).
1. Leave Android and power off. Then boot into Download Mode (varies by phone but mine needed Power Down + Power Up and then connection by USB to my PC)
1. Finally, you should see the words "KG STATE: Checking" and you are able to install other ROMs (like LineageOS) and recovery utilities (like TWRP).

After bootloader unlock and bypassing KG STATE, your phone reboots with stock ROM but is now unlocked.

### Step 1.3: Flash TWRP (TeamWin Recovery Project)

TWRP is a custom recovery that lets you flash custom ROMs like LineageOS.

1. Download the appropriate TWRP image for your device from [dl.twrp.me](https://dl.twrp.me/)
   - For M21: `twrp-3.7.1_12-0-m21.img.tar`
2. Extract the `.img.tar` file to get `recovery.img`
3. Boot into Download Mode again
4. In Odin:
   - Load `recovery.img` into the **Recovery** slot
   - Click **Start**

5. Once flashed, reboot into recovery:
   ```bash
   adb reboot recovery
   ```

You should see the TWRP interface (blue/orange theme, touch controls).

## Part 2: Flashing LineageOS

### Step 2.1: Download LineageOS

1. Go to [lineageos.org](https://lineageos.org/) and find your device
2. Download the latest stable build (or use XDA Forums for older devices with community support)
   - For M21: Look for unofficial LineageOS 23.2 builds on XDA. The version will be a delicate balance: recent enough to maximise support; old enough to be frugal on memory and compute.
3. Transfer the `.zip` file to your phone via USB or download it directly using TWRP

### Step 2.2: Wipe and Flash

1. Boot into TWRP recovery (Power + Volume Up or Volume Down, depending on your device)
2. Go to **Wipe → Format Data** and type `yes` to clear encryption (you did backup your files, right?)
3. Go to **Install** and select the LineageOS `.zip` file
4. Swipe to confirm flash
5. Go to **Wipe → Dalvik/ART Cache** (optional but recommended)
6. **Reboot System** — LineageOS will boot for the first time (may take a few minutes)

Congratulations! You now have a lightweight OS with much less background RAM usage than stock Android.

> Top tip if you end up flashing the wrong ROM image (like I did initially) and soft brick your phone. Simply flash the TWRP recovery image to both the recovery and  


### Step 2.3: Verify RAM Usage

LineageOS on an M21 idles at roughly 0.8 GB RAM compared to stock OneUI at 1.6 GB. This gives you about 0.8 GB extra RAM for running inference.

```bash
adb shell
free -h
# Should show ~3.2 GB free out of 4 GB total
```

## Part 3: Setting Up Termux

### Step 3.1: Install Termux

1. Install **F-Droid** (an open-source app store) from [f-droid.org](https://f-droid.org/)
2. Open F-Droid and search for **Termux**
3. Install it

Alternatively, download the Termux `.apk` directly from [termux.com](https://termux.com/).

### Step 3.2: Initial Setup

Open Termux and run:

```bash
termux-setup-storage
# Grant permission when prompted
apt update && apt upgrade -y
```

### Step 3.3: Install SSH Server

So you can manage the phone remotely:

```bash
pkg install openssh
passwd
# Set a password for SSH (you'll use this to connect remotely)
sshd
# SSH server now runs on port 8022
```

Find your phone's IP address:

```bash
ip a | grep inet
# Look for an IP like 192.168.x.x or 10.0.x.x
```

From your PC, test SSH access:

```bash
ssh -p 8022 u0_aXXX@<phone-ip>
# Username shown in your Termux prompt (e.g., u0_a387)
# Enter the password you set
```

### Step 3.4: Install tmux

For keeping long-running processes alive after SSH disconnect:

```bash
pkg install tmux
```

## Part 4: Building llama.cpp

### Step 4.1: Install Build Dependencies

```bash
pkg install cmake ninja build-essential git
```

### Step 4.2: Clone and Build

```bash
git clone --depth 1 https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build -DGGML_OPENMP=OFF -DCMAKE_BUILD_TYPE=Release
cmake --build build -j4
```

This takes 10–30 minutes depending on your phone's speed. The `-j4` flag uses 4 threads; adjust based on your CPU core count.

Once complete, the main binary is at `llama.cpp/build/bin/llama-server`.

### Step 4.3: Verify the Build

```bash
./build/bin/llama-server --version
# Should print version info
```

## Part 5: Downloading a Model

### Step 5.1: Understanding Model Quantization

Models are stored in **GGUF format** with different quantization levels:
- **Q4_K_M**: Good quality, moderate RAM (~0.8 GB for 1.2B model)
- **Q3_K_M**: Lower quality, smaller RAM (~0.6 GB for 1.2B model)
- **Q8_0**: Larger, better quality (~1.2+ GB for 1.2B model)

For a 4 GB phone, **Q4_K_M is optimal** for 1–2B parameter models.

### Step 5.2: Download via llama-server

The easiest way is to use llama-server's built-in Hugging Face support:

```bash
./build/bin/llama-server -hf LiquidAI/LFM2.5-1.2B-Instruct-GGUF:Q4_K_M
```

This downloads to `~/.cache/huggingface/` and loads automatically. First run takes a minute to download (~600 MB).

### Step 5.3: Model Recommendations

For a 4 GB phone:
- **LFM 2.5 1.2B**: ~14–18 tok/s (fastest, lower quality)
- **Qwen 2.5 1.5B**: ~12–15 tok/s (best quality-per-token)
- **SmolLM 1.7B**: Solid middle ground
- **Gemma 4 2B**: Requires careful tuning, larger context risky

## Part 6: Running the Server

### Step 6.1: Manual Start (for Testing)

From your phone's Termux:

```bash
cd ~/llama.cpp
./build/bin/llama-server \
  -hf LiquidAI/LFM2.5-1.2B-Instruct-GGUF:Q4_K_M \
  -c 4096 \
  -t 4 \
  -fit off \
  --host 0.0.0.0 \
  --port 8080
```

**Key flags:**
- `-c 4096`: Context window (adjust down if RAM-constrained)
- `-t 4`: Thread count (use 4 to avoid thermal throttling; 6 is faster but hotter)
- `-fit off`: Disable GPU-like fitting to prevent OOM during loading
- `--host 0.0.0.0`: Bind to all network interfaces (required for network access)
- `--port 8080`: API port

The server starts and prints: `Server listening on http://0.0.0.0:8080`

### Step 6.2: Remote Start via SSH (Using tmux)

From your PC, start the server in a persistent tmux session:

```bash
ssh -p 8022 u0_aXXX@<phone-ip> \
  "tmux new-session -d -s llm 'cd ~/llama.cpp && ./build/bin/llama-server -hf LiquidAI/LFM2.5-1.2B-Instruct-GGUF:Q4_K_M -c 4096 -t 4 -fit off --host 0.0.0.0 --port 8080'"
```

The server now runs even after you disconnect. To check on it:

```bash
ssh -p 8022 u0_aXXX@<phone-ip> tmux attach -t llm
# Press Ctrl+B then D to detach (server keeps running)
```

### Step 6.3: Verify Network Accessibility

From your PC:

```bash
# Open web UI in browser:
http://<phone-ip>:8080/

# Test API with curl:
curl http://<phone-ip>:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"LFM2.5-1.2B","messages":[{"role":"user","content":"Hello"}]}'
```

You should get a JSON response with the model's reply.

## Part 7: Integration with OpenCode

Now that you have an OpenAI-compatible API running on your phone, you can use it as a local subagent in **OpenCode** or other tools that support OpenAI-compatible endpoints.

In your OpenCode configuration, add:

```json
{
  "model": "llama-phone",
  "provider": "openai",
  "base_url": "http://<phone-ip>:8080/v1",
  "api_key": "not-needed"
}
```

This lets you spawn lightweight subagents on your phone for tasks like:
- Code linting and formatting checks
- Web searches (with an MCP server)
- Recipe retrieval (like the `recipe_recall` MCP server)
- Any deterministic task that doesn't need a large context or complex reasoning

## Performance Expectations

On a Samsung M21 (Exynos 9611, 4 GB RAM, CPU-only):

| Model | Quantization | RAM | Speed | Quality |
|-------|--------------|-----|-------|---------|
| LFM 2.5 1.2B | Q4_K_M | ~0.8 GB | 14–18 tok/s | Medium |
| Qwen 2.5 1.5B | Q4_K_M | ~1.0 GB | 12–15 tok/s | High |
| SmolLM 1.7B | Q4_K_M | ~1.1 GB | ~12 tok/s | Medium-High |
| Gemma 4 2B | Q4_K_M | ~1.2 GB | ~10 tok/s | High (risky) |

**Tips for optimization:**
- Reduce `-c` (context) if you hit OOM errors
- Increase `-t` (threads) carefully; 4–6 is sweet spot for M21
- Use `-fit off` to avoid unexpected crashes during model loading
- Monitor thermal throttling with `watch -n 1 'grep cpu /proc/cpuinfo'`

## Troubleshooting

### Server crashes after a few minutes
- **Cause:** Thermal throttling or OOM
- **Fix:** Reduce `-t` to 2–3, or reduce `-c` to 2048

### `bind: Address already in use`
- **Cause:** Port 8080 already in use
- **Fix:** Kill the old process or use a different port: `--port 8081`

### Model won't load / OOM kill
- **Cause:** Not enough free RAM
- **Fix:** Use `--host 0.0.0.0 -fit off` or reduce context size

### SSH connection refused
- **Cause:** SSH daemon not running
- **Fix:** Open Termux and run `sshd` again, or add to startup script

## What's Next?

Now that you have a working inference server, consider:
1. **Testing different models** to find the best quality/speed tradeoff
2. **Integrating with your AI harness** (like OpenCode) to spawn subagents
3. **Building MCP servers** for specialized tasks (e.g., web search, recipe lookup)
4. **Exploring Droidian** (Debian native OS) if you need more RAM for larger models

The cost perspective is compelling: your phone is already paid for, the electricity cost is minimal, and you get a capable inference endpoint for tasks that don't need a powerful model. As cloud AI pricing climbs, this kind of setup will become increasingly attractive.

## References

- [LineageOS Official](https://lineageos.org/)
- [TWRP Project](https://twrp.me/)
- [llama.cpp Repository](https://github.com/ggml-org/llama.cpp)
- [Termux Documentation](https://termux.dev/)
- [Hugging Face Model Hub](https://huggingface.co/)
- [OpenCode Project](https://opencode.ai/)
