# Samsung Galaxy M21 (SM-M215F) — OS Migration Progress

## Goal
Replace stock OneUI with a lightweight OS to run a small quantized language model (Gemma 4 2B, LFM 2.5 1.2B, etc.) via llama.cpp in Termux as a subagent for an AI harness.

## Device
- **Model:** Samsung Galaxy M21 (SM-M215F)
- **Baseband:** M215FXXU3CWL1
- **SoC:** Exynos 9611
- **RAM:** 4 GB
- **Stock OS:** OneUI (idle ~1.6 GB RAM used)

## Done
| Step | Status |
|---|---|
| Unlocked bootloader / enabled OEM unlock | ✅ |
| Resolved KG State "Prenormal" via date trick | ✅ |
| Flashed TWRP 3.7.1 via Odin (official `.img.tar`) | ✅ |
| Dual-partition TWRP workaround (boot+recovery both = TWRP) | ✅ |
| Flashed LineageOS 23.2 (2026-02-14, unofficial) | ✅ |
| Boots successfully to LineageOS | ✅ |
| SSH access via Termux + OpenSSH (port 8022) | ✅ |
| tmux for persistent sessions | ✅ |
| Built llama.cpp from source | ✅ |
| Downloaded LFM 2.5 1.2B Q4_K_M via `-hf` flag | ✅ |
| Serving on LAN via llama-server (`--host 0.0.0.0:8080`) | ✅ |

## Current Status
- **LineageOS 23.2** running on the M21 with llama.cpp serving LFM 2.5 1.2B Q4_K_M.
- TWRP still accessible via the dual-partition trick (boot partition holds TWRP, recovery partition holds LineageOS kernel).
- Boot behavior: Volume Up + Power → LineageOS; Volume Down or no keys → TWRP.
- SSH via `ssh -p 8022 u0_aXXX@<phone-ip>`.
- Inference server accessible at `http://<phone-ip>:8080/` (web UI) or `/v1/chat/completions` (OpenAI API).
- Boot partition holds TWRP, so the custom kernel from LineageOS is on recovery. The phone must use the Volume Up + Power key combo to boot properly into Android.
- **Consider using `--host 0.0.0.0` always when starting llama-server to bind to all network interfaces.**

## Key Details
- Flashing TWRP to both `recovery.img` and `boot.img` in a single `.tar` was needed because the M21 boots its OS from the recovery partition — the dual flash ensures TWRP boots even if the stock bootloader targets the wrong partition.
- LineageOS 20.0 (older) was incompatible with Binary 3 firmware (CWL1). LineageOS 22.1 and 23.2 builds target newer firmware bases.
- Formatting data (`Wipe → Format Data → type "yes"`) in TWRP was required to clear encryption metadata after flashing the new ROM.
- Stock OneUI idle RAM: ~1.6 GB; LineageOS idle RAM: expected ~0.8 GB, freeing ~0.8–0.9 GB additional for inference.
- Droidian (Debian native, ~300–400 MB idle) is a future option if more RAM is needed.

## Firmware Sources
- **SamFrew:** M215FXXU3CXJ2 (XEO, Binary 3, Android 12) — used as base for compatibility
- **LineageOS 23.2 XDA thread:** `xdaforums.com/t/unofficial-lineageos-XX-XX-m21.4720105/`
- **TWRP:** `dl.twrp.me/m21/twrp-3.7.1_12-0-m21.img.tar`

## Next Steps
1. Test harness integration against `http://<phone-ip>:8080/v1/chat/completions`
2. Optionally try other models (Qwen 2.5 1.5B, Gemma 4 2B, SmolLM 1.7B) for comparison
3. Tune context size (`-c`) and thread count (`-t`) for best perf
4. Optionally revisit Droidian later if more RAM is needed for larger models
5. Define an opencode subagent as an example.
6. Create launcher scripts and a systemd service to run server on boot and relaunch if it crashes.