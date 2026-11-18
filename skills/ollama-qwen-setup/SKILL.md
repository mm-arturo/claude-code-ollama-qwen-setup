---
name: ollama-qwen-setup
description: Set up a local LLM with Ollama, end to end, sized to the user's machine - ask what it is
  for, measure VRAM (or unified memory / RAM), offer two or three models that fit (Qwen to think
  things through, Gemma for quick conversation; official versions first, reduced-refusal versions
  only on request), install Ollama, download the chosen model, set the context window, verify it
  really runs fast (not only "100% GPU"), show how to free the memory, and hand back a one-card
  summary. Use when the user asks to install Ollama, run Qwen or Gemma locally, set up a local or
  private AI, pick a model or GGUF quant for their hardware, or "use AI without internet".
---

# Local LLM with Ollama, sized to your machine (Qwen and Gemma)

Take a machine with nothing installed to a working local model the user can call from a terminal,
an app or the HTTP API. Eleven steps. Do not skip step 9 - a model that silently overflows the
card looks like success until the first answer takes minutes.

**Talk to the user in their language.** This skill is in English; the person may not be.
**Every path you show the user is absolute.** They will copy-paste it.

## Ground rules - ask before you act

- **Ask before** installing Ollama (show the exact command and what it does), before **each**
  download (model name, source, size), before setting any environment variable, and before
  removing anything. Never pipe a remote script into a shell without showing it first.
- **Official sources only:** the Ollama library (`ollama.com/library`) or the model author's page
  on Hugging Face. Third-party copies are a different file from a different person.
- **Read the license** on the model card before the user relies on it for work. Qwen's text models
  used here are Apache-2.0; other families have their own terms.
- **Do not change driver or system settings** (for example NVIDIA's "CUDA - Sysmem Fallback Policy")
  unless the user asks for it after you explained the trade-off.
- Say what you measured and what you only read. Never write a speed you did not time.

## 1. Ask what it is for

Three short questions, then wait for the answers:

1. **What will you use it for?** Short tasks (rewrite, summarize, translate, extract a list from an
   email) or long reasoning (analysis, code, long documents)?
2. **Do you need fewer refusals?** Only if the official model refuses legitimate work (for example
   anonymised medical surveys or addiction research). Default: official versions. Modified
   versions are an explicit opt-in (see `reference/models.md`).
3. **Where should the models live?** Ask for an absolute folder on a drive with room. Do not
   default it.

## 2. Check what is already there

```bash
ollama --version          # Windows fallback: & "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" --version
ollama list
```

If a suitable model is already installed, say so and ask whether to reuse it. Do not silently
re-download many gigabytes. Note that names differ: a library download appears as
`qwen3.8:27b`, a hand-registered one may appear as `qwen3.8-27b:latest`. Always use the name the
user's own `ollama list` prints.

## 3. Measure the machine

NVIDIA (Windows or Linux) - total and **already used** VRAM:

```bash
nvidia-smi --query-gpu=name,memory.total,memory.used --format=csv,noheader
```

The plain `nvidia-smi` also lists processes with their full paths; do not paste that part into
anything public.

- AMD: `rocm-smi --showmeminfo vram`.
- Apple Silicon: `sysctl -n hw.memsize` and `system_profiler SPHardwareDataType | grep Chip`. Unified
  memory: plan for model + context to use at most about 70% of it.
- No GPU: it still works on the CPU, much slower. Say so and offer only the smallest options.

Also read free disk on the target drive. Report one line: device, total memory, memory already in
use, free disk. Keep RAM, VRAM and unified memory apart when you explain it: they are different
things.

## 4. Offer two or three options, then let the user choose

Use `reference/models.md`: rows by VRAM (8, 16, 24 GB), columns by use (think it through = Qwen,
quick conversation = Gemma), official versions first. For each option give the exact name, the
download size and one line of trade-off. Then ask which one - never decide alone.

Say out loud:

- **File size is not total memory.** The context and the runtime need room on top, and other
  applications already hold part of the card.
- A **tight** option leaves no room for a second model and forces a small context.
- Below Q3, quality drops sharply on reasoning and code; a smaller model at Q4 usually beats a big
  one at Q2.
- If they chose a modified (reduced-refusal) version: it may answer what the official one blocked,
  and it may also be wrong more often on everything else; downloads are not quality (evidence and
  caveats in `reference/models.md`).

For a specific Unsloth quant with image input, use `reference/unsloth-custom-quant.md` instead of
steps 7-8.

## 5. Install Ollama if missing

The official commands (<https://ollama.com/download>):

- **Windows (PowerShell):** `irm https://ollama.com/install.ps1 | iex` - or the installer
  `OllamaSetup.exe` from the same page. It installs in the user's account without administrator
  rights, to `C:\Users\<user>\AppData\Local\Programs\Ollama\`, and adds itself to PATH: open a
  **new** terminal before `ollama` resolves.
- **macOS:** `curl -fsSL https://ollama.com/install.sh | sh` - or `Ollama.dmg` from the same page.
- **Linux:** `curl -fsSL https://ollama.com/install.sh | sh` (installs a systemd service).

Show the command, say what it does, and run it only after the user agrees (or let them run it).
Confirm with `ollama --version` in a fresh terminal.

## 6. Optional: model folder and local-only mode

Both need a restart of Ollama (quit the tray app and start it again, or restart the service).
Ask before setting either; they are persistent.

- **Model folder** (default: `C:\Users\<user>\.ollama\models`, `~/.ollama/models`, or
  `/usr/share/ollama/.ollama/models` on Linux). To use the folder from step 1:

  ```powershell
  [Environment]::SetEnvironmentVariable("OLLAMA_MODELS", "E:\ollama\models", "User")
  ```

  On Linux the `ollama` user needs read and write access to it.
- **Local only:** Ollama also offers cloud models (tags like `:cloud`). If the user wants nothing
  sent out, disable cloud features with `OLLAMA_NO_CLOUD=1` for the **server** process (or
  `"disable_ollama_cloud": true` in `~/.ollama/server.json`), restart, and check the log says
  `Ollama cloud disabled: true`. Setting it in a terminal that talks to an already running server
  changes nothing.

## 7. Download the chosen model

Internet is needed only for this step.

```bash
ollama pull qwen3.8:27b                                              # official, from the library
ollama pull hf.co/orcarouter/Qwen3.8-27B-Uncensored-GGUF:Q4_K_M     # a GGUF from Hugging Face
```

Use the exact name from `reference/models.md` or the model page. Large downloads take a while;
run them where a timeout cannot kill them. Afterwards `ollama list` shows the name to use from now
on.

## 8. First run, with the context set

```bash
ollama run <name> --think=false
```

- **Thinking:** Qwen reasons before answering by default. For short tasks `--think=false` answers
  directly and faster, and can lose precision on hard tasks. Without the flag (the default) answers
  are slower and usually better. If thinking stays on, give it enough output room (below).
- **Context window** - how much text the model can keep in view. Set it inside the chat (these are
  chat commands, not shell commands):

  ```text
  /set parameter num_ctx 4096
  /set parameter num_predict 1024
  ```

  - `num_ctx`: 4096 is plenty for an email or a page. For long documents or reasoning try 16384, if
    the memory allows (step 9). A context smaller than the text means the model does not see all of
    it; a context so large that the card fills up makes everything very slow.
  - `num_predict`: the output limit. With thinking on, the reasoning spends it too and the answer can
    be cut before it starts - raise it (for example 8192) for long reasoning.
  - From the API the same goes in `"options": {"num_ctx": 4096}`. For the whole server:
    `OLLAMA_CONTEXT_LENGTH` (needs a restart). Recent Ollama versions already pick a default by VRAM
    (about 4k under 24 GiB, 32k from 24 to 48 GiB, 256k above) and a model's Modelfile can set its
    own; check what is really loaded in step 9.

## 9. Verify - this step is not optional

In a second terminal, while the model is loaded:

```bash
ollama ps
nvidia-smi --query-gpu=memory.total,memory.used --format=csv,noheader
```

Then time a real answer:

```bash
ollama run <name> --think=false --verbose "Rewrite in two sentences: <a short paragraph>"
```

Pass only if **all three** hold:

1. `ollama ps` shows `100% GPU` in PROCESSOR and the context you set in CONTEXT;
2. the card is not nearly full - leave headroom (in one measured case, 1.3 GB free was too little and
   2.6 GB was fine on a 24 GB laptop card);
3. the `eval rate` is reasonable for the hardware (reference: 26 tokens/s for Qwen3.8-27B Q4_K_S,
   context 4096, thinking off, RTX 5090 Laptop 24 GB, measured 2026-11-15).

**`100% GPU` alone is not proof.** On Windows the NVIDIA driver can quietly fall back to system
memory when the card fills up (<https://nvidia.custhelp.com/app/answers/detail/a_id/5490>), and
Ollama does not see it: on that same laptop, with the context at 32768 and other apps holding about
5.7 GB of the card, `ollama ps` said `100% GPU` and the model wrote at 1.7 tokens/s - fifteen times
slower. Dropping the context to 4096 brought it back to 26.

If it fails, fix in this order: lower `num_ctx` (32768 → 16384 → 8192 → 4096); close what else holds
the card (games, renders, browsers with hardware acceleration) and re-check `nvidia-smi`; go one
size or one quant down.

## 10. Free the memory when done

```text
/bye
```

```bash
ollama ps                 # often still loaded: it unloads after ~5 min idle by default
ollama stop <name>        # unload now
ollama ps                 # empty = the card is free again
```

The files stay on disk for next time. `ollama rm <name>` deletes them - ask first.

## 11. Hand back the card

Close with a short block the user can paste into their notes. Fill in the real numbers you
measured; nothing else, no recap of the steps.

```
Model      <name as in ollama list>   (<family, size, quant>, official | modified by <author>)
License    <license from the model card>
Memory     card <total> GB, free with the model loaded <measured> GB, context <num_ctx>
Speed      <measured> tok/s  (thinking off, context <num_ctx>)

Chat       ollama run <name> --think=false
           /set parameter num_ctx 4096
One-shot   ollama run <name> "your prompt"
API        curl http://localhost:11434/api/generate -d '{"model":"<name>","prompt":"hi","stream":false,"options":{"num_ctx":4096}}'
OpenAI SDK base_url="http://localhost:11434/v1", api_key="ollama"   (any value; Ollama ignores it)
Unload     ollama stop <name>        (auto-unloads after ~5 min idle)
Remove     ollama rm <name>
```

## Known traps

- **`ollama` on Windows opens the desktop app when run with no arguments.** `ollama <args>` is the
  CLI. Not a failure.
- **Ollama must be running** for any command to work. Windows/macOS: the tray app. Linux:
  `systemctl status ollama`. A dead server gives `connection refused` on port 11434.
- **Exiting the chat does not free the card at once.** Use `ollama stop`.
- **The memory is shared with everything else on the card.** Check `nvidia-smi` at the moment of
  the complaint, not from memory.
- **A Modelfile's `num_ctx` is only a default.** Changing the Modelfile does nothing until
  `ollama create` runs again, but a session (`/set parameter num_ctx`) or a request
  (`options.num_ctx`) overrides it.
- **Names differ between installs.** Use the one `ollama list` prints on this machine.
