# Qwen3.8-27B GGUF quants — sizes and how to choose

Source repo: <https://huggingface.co/unsloth/Qwen3.8-27B-GGUF> (Apache-2.0).
Sizes read from the Hub API on **2026-09-03**. The repo is re-quantized occasionally, so
confirm live sizes before quoting a download; the command is in `SKILL.md` step 3.

## The sizing formula

```
usable VRAM   = total VRAM − VRAM already in use − 1.0 GiB   (display, compositor, OS)
weight budget = usable VRAM − 0.9 GiB (mmproj) − KV cache
```

KV cache, measured against `num_ctx`, is roughly:

| num_ctx | KV cache |
|---|---|
| 8192 | ~0.8 GiB |
| 16384 | ~1.6 GiB |
| 32768 | ~3.0 GiB |
| 65536 | ~6.0 GiB |

These are working estimates, not a spec. **`ollama ps` is the ground truth** — it prints the
real CPU/GPU split for the model as loaded. If it says anything other than `100% GPU`, the
budget was wrong, whatever the arithmetic said.

## Quant table

`UD-` files are Unsloth Dynamic quants: precision is allocated per tensor rather than uniformly,
so they beat a plain quant of the same size. Prefer them over `Q4_0` / `Q4_1` / `Q8_0`.

| File | GiB | Fits (24 GiB card, 32k ctx) | Notes |
|---|---:|---|---|
| `Qwen3.8-27B-UD-IQ1_S.gguf` | 5.77 | yes | severe quality loss; last resort on 8 GiB |
| `Qwen3.8-27B-UD-IQ1_M.gguf` | 6.27 | yes | as above, slightly better |
| `Qwen3.8-27B-UD-IQ2_XXS.gguf` | 6.77 | yes | 8 GiB cards |
| `Qwen3.8-27B-UD-IQ2_S.gguf` | 7.80 | yes | 10 GiB cards |
| `Qwen3.8-27B-UD-Q2_K_XL.gguf` | 9.15 | yes | 12 GiB comfortable floor; reasoning degrades |
| `Qwen3.8-27B-UD-IQ3_XXS.gguf` | 10.18 | yes | 12 GiB tight |
| `Qwen3.8-27B-UD-IQ3_S.gguf` | 11.21 | yes | 16 GiB safe |
| `Qwen3.8-27B-UD-Q3_K_XL.gguf` | 12.24 | yes | **16 GiB recommended** |
| `Qwen3.8-27B-UD-IQ4_XS.gguf` | 13.28 | yes | 16 GiB tight; near-Q4 quality |
| `Qwen3.8-27B-UD-Q4_K_S.gguf` | 14.30 | yes | 20 GiB cards |
| `Qwen3.8-27B-Q4_0.gguf` | 14.95 | yes | legacy uniform quant; prefer `UD-Q4_K_S` |
| `Qwen3.8-27B-UD-Q4_K_M.gguf` | 15.33 | yes | 20 GiB comfortable |
| `Qwen3.8-27B-Q4_1.gguf` | 16.33 | yes | legacy; prefer `UD-Q4_K_XL` |
| `Qwen3.8-27B-UD-Q4_K_XL.gguf` | 16.35 | yes | **24 GiB recommended** — best quality/GB here |
| `Qwen3.8-27B-UD-Q5_K_S.gguf` | 17.38 | yes | 24 GiB, room for 32k ctx |
| `Qwen3.8-27B-UD-Q5_K_M.gguf` | 18.41 | tight | 24 GiB with ctx dropped to 16k |
| `Qwen3.8-27B-UD-Q5_K_XL.gguf` | 19.44 | tight | 24 GiB only if the card is otherwise idle |
| `Qwen3.8-27B-UD-Q6_K.gguf` | 20.47 | no | 32 GiB |
| `Qwen3.8-27B-UD-Q6_K_M.gguf` | 21.50 | no | 32 GiB |
| `Qwen3.8-27B-UD-Q6_K_L.gguf` | 22.53 | no | 32 GiB tight |
| `Qwen3.8-27B-UD-Q6_K_XL.gguf` | 23.56 | no | **48 GiB recommended** |
| `Qwen3.8-27B-UD-Q8_K_L.gguf` | 26.12 | no | 48 GiB |
| `Qwen3.8-27B-Q8_0.gguf` | 27.05 | no | legacy uniform quant |
| `Qwen3.8-27B-UD-Q8_K_XL.gguf` | 29.30 | no | 48 GiB; diminishing returns over Q6 |
| `BF16/` (2 shards) | 50.90 | no | unquantized reference; 80 GiB class |

Companion files:

| File | GiB | What it is |
|---|---:|---|
| `mmproj-BF16.gguf` | 0.87 | **vision projector — download this one**, required for images |
| `mmproj-F16.gguf` | 0.86 | alternative precision; either works, BF16 is the default |
| `imatrix_unsloth.gguf` | 0.01 | importance matrix used to build the quants; not needed at runtime |
| `MTP/mtp-Qwen3.8-27B-Q4_0.gguf` | 1.28 | multi-token-prediction draft model for speculative decoding in llama.cpp — **not usable from Ollama**; skip it |

## Choosing between two that both fit

Take the larger one only if all three hold: `ollama ps` still reads 100% GPU, the user does not
need a second model resident, and they will not raise `num_ctx` later. Otherwise take the
smaller one — the quality difference between adjacent quants above Q4 is small, and a model
that spills to CPU is 10-20x slower, which is not small.

Below Q3, quality falls off sharply on reasoning and code. A smaller model at Q4 is usually the
better trade than Qwen3.8-27B at Q2.
