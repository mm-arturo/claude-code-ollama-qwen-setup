# Which model for which card - the options this skill offers

Checked on **2026-11-16/18**. Libraries and repositories change; confirm the tag and its size on its
page before quoting a download (`ollama.com/library/<name>` or the Hugging Face repo).

Rows are **dedicated VRAM** on the graphics card. They are not system RAM, and not Apple unified
memory (see "Macs" below). File size is not total memory: the context and the runtime need room
on top, and other applications already use part of the card.

## Two columns, two uses

- **Think it through (Qwen):** Qwen reasons before answering by default and tends to think long.
  Good when the user can wait for a better answer.
- **Quick conversation (Gemma):** for short, simple tasks where speed matters.

This split is an orientation, not a measurement: both families have a reasoning mode. Say so.

## Official versions - the default

Offer these first. Use a modified version only if the user explicitly says the official one
refuses tasks they need (step 1 of `SKILL.md`).

| VRAM | Think it through | Quick conversation |
| --- | --- | --- |
| 8 GB | `qwen3.5:4b` - 3.3 GB | `gemma4:e2b` - 4.6 GB; `gemma4:e4b` - 6.6 GB is tight |
| 16 GB | `qwen3.5:9b` - 6.6 GB | `gemma4:12b` - 8.0 GB |
| 24 GB | `qwen3.8:27b` - 18 GB | `gemma4:26b` (26B-A4B) - 18 GB; `gemma4:31b` - 20 GB only on an otherwise idle card |

Sizes are the Q4_K_M downloads on the library pages (2026-11-18), all with image input:
<https://ollama.com/library/qwen3.5/tags>, <https://ollama.com/library/qwen3.8/tags>,
<https://ollama.com/library/gemma4/tags>. Qwen3.8 is published only at 27B and its smallest library
tag is 18 GB, so a 16 GB card cannot take the official 27B from the library: use `qwen3.5:9b`, the
Unsloth `UD-Q3_K_XL` (12.2 GiB, [`unsloth-custom-quant.md`](unsloth-custom-quant.md)) or, on explicit
request, the OrcaRouter Q3 below.

## Modified versions with fewer refusals ("uncensored", "abliterated", "Heretic")

Only on explicit request. These are third-party edits of the official weights: someone found the
activity pattern linked to refusing and changed part of the parameters to weaken it. It is not a
prompt and not a setting. The precision of that edit varies a lot between authors.

The options shown in the video (file sizes in decimal GB, read from the Hugging Face API on
2026-11-16):

| VRAM | Think it through | Quick conversation |
| --- | --- | --- |
| 8 GB | (no Qwen pick: use official `qwen3.5:4b`) | `hf.co/Abiray/gemma-4-E4B-it-heretic-GGUF:Q4_K_M` - 5.34 GB (coder3101's Heretic edit) |
| 16 GB | `hf.co/orcarouter/Qwen3.8-27B-Uncensored-GGUF:Q3_K_M` - 13.50 GB | `hf.co/llmfan46/gemma-4-26B-A4B-it-ultra-uncensored-heretic-GGUF:Q3_K_M` - 13.29 GB |
| 24 GB | `hf.co/orcarouter/Qwen3.8-27B-Uncensored-GGUF:Q4_K_M` - 16.81 GB | `hf.co/mradermacher/gemma-4-31B-it-heretic-GGUF:Q4_K_M` - 18.69 GB (coder3101's edit) |

The 16 GB row is tight: with the context the user needs it may not fit; then go one size down.

What the evidence says, and what it does not:

- Nathan Sapwell compared the official Qwen3.8-27B with eight modified versions on one RTX 5090:
  13 capability tests and 400 problematic requests, 167 GPU-hours over eleven days
  (<https://nathan.sapwell.net/posts/qwen38-27b-abliteration/>; the full, updated table with more
  variants is at <https://abliterlitics.dev/models/qwen38-27b/>). OrcaRouter came out as the best
  balance. On TruthfulQA MC2 it lost 1.5 points against the original; Ultra-Heretic lost 6.4. The
  heaviest loss in the full table was `obliteratus`.
- OrcaRouter refused 2 of 400 requests, but **38% of its answers never finished thinking** within that
  test's generation budget. Check both: does it answer, and does it finish.
- For Gemma 4 E4B, his study of 23 variants found no distinguishable capability loss in coder3101's
  edit (<https://nathan.sapwell.net/posts/gemma4-e4b-abliteration/>). The Gemma 26B and 31B picks are
  not from a Sapwell study.
- Downloads are not quality. The most popular edit is not the one that tested best.
- None of these numbers were measured on the user's quantization or machine.
- The license of the base model still applies to the modified version.

## Models that do not fit a normal computer

Kimi K3.1 Pro or DeepSeek V4.1 Flash are published openly, but even their most compressed versions
need hundreds of GB to over a terabyte of memory. Say so if the user asks; do not offer them.

## Macs (unified memory)

Apple Silicon shares one memory between CPU and GPU. As a working rule, plan for the model plus
context to use at most about 70% of total memory. It can run large models, usually slower than a
dedicated card of the same size. Pick from the rows by that budget, then verify with step 9 of
`SKILL.md`.

## No GPU

Ollama runs on the CPU too, only much slower. Offer the smallest options (`qwen3.5:4b` or smaller,
`gemma4:e4b`) and time a generation before the user commits to a workflow.
