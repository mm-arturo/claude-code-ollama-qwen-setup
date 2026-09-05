---
name: ollama-qwen-setup
description: Install Ollama and run Qwen3.8-27B locally, end to end - measure the machine's
  VRAM, recommend the quantization that actually fits, download the GGUF plus the vision
  projector, register the model, verify it runs fully on the GPU, and hand back a one-card
  summary of how to call it. Use when the user asks to install Ollama, run Qwen locally,
  set up a local LLM, or pick a GGUF quant for their hardware.
---

# Install Ollama + Qwen3.8-27B locally

Take a machine with nothing installed to a working local `qwen3.8-27b` the user can call from
a terminal, an app, or the HTTP API. Nine steps. Do not skip the verification step - a model
that silently spills onto the CPU is the single most common bad outcome, and it looks like
success until the first prompt takes two minutes.

**Every path you show the user is absolute.** They will copy-paste it.

## 0. Facts this skill is built on

- Quants come from **`unsloth/Qwen3.8-27B-GGUF`** on Hugging Face (Apache-2.0). The `UD-` files
  are Unsloth Dynamic quants - mixed precision per tensor, better quality per GB than the plain
  `Q4_0` / `Q8_0` files at the same size. Prefer them.
- Qwen3.8-27B is **multimodal**. Text works from the GGUF alone; images need
  **`mmproj-BF16.gguf`** (~0.87 GB) alongside it. Always download both - it is small, and
  retrofitting it later means re-registering the model.
- `ollama create` **copies** the GGUF into Ollama's blob store. Budget **~2.2x the model size**
  in free disk, or move the store first (step 4).

## 1. Check what is already there

```bash
ollama --version          # Windows fallback: & "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" --version
ollama list
```

If a `qwen3.8` model is already registered, say so and ask whether to reuse it, add another
quant, or replace it. Do not silently re-download 17 GB.

## 2. Measure the hardware

NVIDIA (Windows or Linux):

```bash
nvidia-smi --query-gpu=name,memory.total,memory.used --format=csv,noheader
```

AMD: `rocm-smi --showmeminfo vram`. Apple Silicon: `sysctl -n hw.memsize` and
`system_profiler SPHardwareDataType | grep Chip` - unified memory, so usable budget is roughly
**70% of total RAM**.

Also read free disk on the drive the user will download to. If nothing reports a GPU, say
plainly that this model will run on CPU at roughly 1-3 tokens/second and ask whether to
continue or stop.

Report one line: GPU name, total VRAM, VRAM already in use, free disk.

## 3. Recommend a quant, then let the user choose

Compute the budget:

```
usable VRAM = total VRAM - VRAM already in use - 1.0 GB (display/OS)
weight budget = usable VRAM - 0.9 GB (mmproj) - 3.0 GB (KV cache at 32k context)
```

Pick from `reference/quants.md` the largest quant whose size is at or under the weight budget,
and offer it plus the next size down as the safe option. Present **two or three** candidates
with their sizes and a one-line trade-off each, then ask which one - never decide alone.

Reference points, using the whole card:

| VRAM | Comfortable at 32k ctx | Tight but usually fits |
|---|---|---|
| 12 GB | `UD-Q2_K_XL` (9.2 GiB) | `UD-IQ3_XXS` (10.2 GiB) |
| 16 GB | `UD-Q3_K_XL` (12.2 GiB) | `UD-IQ4_XS` (13.3 GiB) |
| **24 GB** | **`UD-Q4_K_XL` (16.4 GiB)** | `UD-Q5_K_XL` (19.4 GiB) |
| 32 GB | `UD-Q5_K_XL` (19.4 GiB) | `UD-Q6_K_L` (22.5 GiB) |
| 48 GB+ | `UD-Q6_K_XL` (23.6 GiB) | `UD-Q8_K_XL` (29.3 GiB) |

Sizes are from the Hub on 2026-09-03; the repo is re-quantized from time to time, so confirm
the live size before quoting a download time:

```bash
curl -s "https://huggingface.co/api/models/unsloth/Qwen3.8-27B-GGUF?blobs=true" \
  | python -c "import json,sys; [print(f\"{f['rfilename']:44s} {f.get('size',0)/2**30:6.2f} GiB\") for f in json.load(sys.stdin)['siblings'] if f['rfilename'].endswith('.gguf')]"
```

Two things to say out loud when the tight option is chosen: it leaves no room for a second
model, and it forces `num_ctx` down. Below `Q3` quality degrades noticeably on reasoning - a
smaller model at `Q4` usually beats this one at `Q2`.

## 4. Ask where the models go

Ask for an absolute folder. Do not default it. Good answers are a data drive with room, e.g.
`E:\AI_models_opensource\qwen3.8-27b` or `/home/<user>/models/qwen3.8-27b`.

If the system drive is short on space, set the blob store to the same drive **before**
`ollama create`, and restart Ollama afterwards so it takes effect:

```powershell
# Windows, persistent
[Environment]::SetEnvironmentVariable("OLLAMA_MODELS", "E:\ollama\models", "User")
```

```bash
# Linux/macOS, persistent - add to the shell profile
export OLLAMA_MODELS=/mnt/data/ollama/models
```

## 5. Install Ollama if missing

- **Windows:** `winget install --id Ollama.Ollama -e`, or the installer from
  <https://ollama.com/download>. Installs to
  `C:\Users\<user>\AppData\Local\Programs\Ollama\ollama.exe` and adds itself to PATH -
  a **new shell** is needed before `ollama` resolves.
- **macOS:** `brew install ollama` then `brew services start ollama`, or the .dmg.
- **Linux:** `curl -fsSL https://ollama.com/install.sh | sh` (installs a systemd service).
  Show the user the command and let them run it; do not pipe a remote script to a shell on
  their behalf without saying what it does.

Confirm with `ollama --version` in a fresh shell before continuing.

## 6. Download the two files

Preferred - resumable, parallel, verifies checksums:

```bash
pip install -U "huggingface_hub[cli]"
hf download unsloth/Qwen3.8-27B-GGUF \
  Qwen3.8-27B-UD-Q4_K_XL.gguf mmproj-BF16.gguf \
  --local-dir "E:\AI_models_opensource\qwen3.8-27b"
```

Substitute the chosen quant filename. No token is needed; the repo is public.

Fallback without Python:

```bash
curl -L -C - -o "E:\AI_models_opensource\qwen3.8-27b\Qwen3.8-27B-UD-Q4_K_XL.gguf" \
  "https://huggingface.co/unsloth/Qwen3.8-27B-GGUF/resolve/main/Qwen3.8-27B-UD-Q4_K_XL.gguf?download=true"
```

This is a 17 GB download - tens of minutes on most connections. Run it in the background or in
a detached process, never in a call that will time out, and **verify the byte size on disk
against the Hub before registering**. A truncated GGUF fails at `ollama create` with a confusing
tensor error, and a proxy that returned an HTML error page leaves a 4 KB "model" behind.

## 7. Register the model

Write `Modelfile` next to the GGUF (absolute `FROM` path):

```
FROM E:\AI_models_opensource\qwen3.8-27b\Qwen3.8-27B-UD-Q4_K_XL.gguf
PARAMETER num_ctx 32768
PARAMETER temperature 0.6
PARAMETER top_p 0.95
PARAMETER top_k 20
PARAMETER min_p 0.0
```

Those sampling values are Qwen's published recommendation for thinking mode - keep them unless
the user asks otherwise. `num_ctx 32768` is the honest default: Ollama's built-in default is far
smaller, and a long document will be silently truncated without it. Raise it only if step 8
still reports 100% GPU.

```bash
ollama create qwen3.8-27b -f "E:\AI_models_opensource\qwen3.8-27b\Modelfile"
```

This copies ~17 GB into the blob store and takes a few minutes. Running it twice with different
source files leaves **both** blobs on disk - `ollama list` will not show the orphan. Check the
store size afterwards if disk is tight.

## 8. Verify - this step is not optional

```bash
ollama run qwen3.8-27b "Reply with exactly: ok"
ollama ps
```

`ollama ps` must show **`100% GPU`** in the PROCESSOR column. Anything else (`48%/52% CPU/GPU`)
means the model does not fit and every prompt will be slow. Fix in this order:

1. lower `num_ctx` (32768 → 16384 → 8192), re-run `ollama create`;
2. drop one quant level;
3. close whatever else holds VRAM (a game, a browser with hardware acceleration, a running
   render) and re-check `nvidia-smi`.

Then time a real generation and report tokens/second, so the user has a baseline:

```bash
ollama run --verbose qwen3.8-27b "Explain a B-tree in 200 words."
```

Vision smoke test, if the user has an image handy:

```bash
ollama run qwen3.8-27b "Describe this image. E:\path\to\image.png"
```

If it answers as if blind, `mmproj-BF16.gguf` is not registered - it must be in the same folder
at `ollama create` time; re-run step 7 with both files present.

## 9. Hand back the card

Close with a short block the user can paste into their own notes. Nothing else - no recap of
the steps.

```
Model      qwen3.8-27b  (Qwen3.8-27B UD-Q4_K_XL, vision enabled)
Files      E:\AI_models_opensource\qwen3.8-27b\Qwen3.8-27B-UD-Q4_K_XL.gguf   16.4 GiB
           E:\AI_models_opensource\qwen3.8-27b\mmproj-BF16.gguf               0.9 GiB
           E:\AI_models_opensource\qwen3.8-27b\Modelfile
Context    32768 tokens      GPU  100%      ~42 tok/s on RTX 5090 Laptop 24 GB

Chat       ollama run qwen3.8-27b
One-shot   ollama run qwen3.8-27b "your prompt"
Image      ollama run qwen3.8-27b "describe this. C:\path\img.png"
API        curl http://localhost:11434/api/generate -d '{"model":"qwen3.8-27b","prompt":"hi","stream":false}'
Python     pip install ollama
           import ollama; ollama.chat(model="qwen3.8-27b", messages=[{"role":"user","content":"hi"}])
OpenAI SDK base_url="http://localhost:11434/v1", api_key="ollama"
Unload     ollama stop qwen3.8-27b        (frees VRAM; auto-unloads after 5 min idle)
Remove     ollama rm qwen3.8-27b          (deletes the blob copy, not the source GGUF)
```

Fill in the real numbers you measured. Never write a tokens/second figure you did not time.

## Known traps

- **`ollama` on Windows opens the desktop app when run with no arguments.** `ollama <args>` is
  the CLI. Not a failure.
- **Ollama must be running** for any command to work. Windows/macOS: the tray app. Linux:
  `systemctl status ollama`. A dead server gives `connection refused` on port 11434.
- **`num_ctx` is per-model, baked in at `ollama create`.** Changing the Modelfile does nothing
  until you re-run `ollama create`.
- **VRAM is shared with everything else on the card.** A 24 GB card running a video render has
  no room for this model. Check `nvidia-smi` at the moment of the complaint, not from memory.
- **Do not delete the source GGUF to save space and then expect `ollama` to break** - it will
  not, the blob is a copy. But you also cannot re-register without re-downloading.
