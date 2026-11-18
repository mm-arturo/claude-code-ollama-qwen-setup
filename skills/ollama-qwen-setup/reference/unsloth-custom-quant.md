# Advanced route: an Unsloth quant of Qwen3.8-27B, with vision, registered by hand

This was the main procedure of v1 (2026-09-05). Since v2 the default route is `ollama pull` of a
library tag (see `SKILL.md`). Use this one only when the user wants a specific Unsloth quant that
the library does not offer **and** wants image input with it. Quant sizes and the sizing formula
are in [`quants.md`](quants.md).

## Facts

- Quants come from **`unsloth/Qwen3.8-27B-GGUF`** on Hugging Face (Apache-2.0). The `UD-` files are
  Unsloth Dynamic quants - mixed precision per tensor, better quality per GB than the plain `Q4_0` /
  `Q8_0` files at the same size.
- Qwen3.8-27B is **multimodal**. Text works from the GGUF alone; images need **`mmproj-BF16.gguf`**
  (~0.87 GB) in the same folder at `ollama create` time. Always download both.
- `ollama create` **copies** the GGUF into Ollama's blob store. Budget **~2.2x the model size** in
  free disk, or move the store first (`OLLAMA_MODELS`, `SKILL.md` step 6).

## Download the two files

Confirm the live size first (the repo is re-quantized from time to time):

```bash
curl -s "https://huggingface.co/api/models/unsloth/Qwen3.8-27B-GGUF?blobs=true" \
  | python -c "import json,sys; [print(f\"{f['rfilename']:44s} {f.get('size',0)/2**30:6.2f} GiB\") for f in json.load(sys.stdin)['siblings'] if f['rfilename'].endswith('.gguf')]"
```

Preferred - resumable, parallel, verifies checksums:

```bash
pip install -U "huggingface_hub[cli]"
hf download unsloth/Qwen3.8-27B-GGUF \
  Qwen3.8-27B-UD-Q4_K_XL.gguf mmproj-BF16.gguf \
  --local-dir "E:\AI_models_opensource\qwen3.8-27b"
```

Fallback without Python:

```bash
curl -L -C - -o "E:\AI_models_opensource\qwen3.8-27b\Qwen3.8-27B-UD-Q4_K_XL.gguf" \
  "https://huggingface.co/unsloth/Qwen3.8-27B-GGUF/resolve/main/Qwen3.8-27B-UD-Q4_K_XL.gguf?download=true"
```

A 17 GB download takes tens of minutes. Run it detached, never in a call that will time out, and
**verify the byte size on disk against the Hub before registering**. A truncated GGUF fails at
`ollama create` with a confusing tensor error, and a proxy that returned an HTML error page leaves a
4 KB "model" behind.

## Register the model

`Modelfile` next to the GGUF, with an absolute `FROM` path:

```
FROM E:\AI_models_opensource\qwen3.8-27b\Qwen3.8-27B-UD-Q4_K_XL.gguf
PARAMETER num_ctx 8192
PARAMETER temperature 1.0
PARAMETER top_p 0.95
PARAMETER top_k 20
PARAMETER min_p 0.0
```

These are the thinking-mode values on the official model card (it also gives `presence_penalty 0.0`,
which is the default)
(<https://huggingface.co/Qwen/Qwen3.8-27B>, read 2026-11-15). For non-thinking use, the card gives
`temperature 0.7`, `top_p 0.80`, `top_k 20`, `min_p 0.0`, `presence_penalty 1.5`. v1 shipped
`temperature 0.6`, which is not what the Qwen3.8 card recommends. The
`num_ctx` in a Modelfile is only the model's **default**; a session can still change it with
`/set parameter num_ctx` and a request with `options.num_ctx`. v1 baked in 32768, and on a 24 GB
laptop card with other apps open that left the card nearly full and generation ran ~15x slower (see
`SKILL.md`, step 9). Start at 8192 and raise it only after step 9 passes.

```bash
ollama create qwen3.8-27b -f "E:\AI_models_opensource\qwen3.8-27b\Modelfile"
```

Running it twice with different source files leaves **both** blobs on disk - `ollama list` will not
show the orphan. Check the store size afterwards if disk is tight.

## Vision smoke test

```bash
ollama run qwen3.8-27b "Describe this image. E:\path\to\image.png"
```

If it answers as if blind, `mmproj-BF16.gguf` was not registered - it must be in the same folder at
`ollama create` time; re-run the registration with both files present.

Deleting the source GGUF afterwards does not break `ollama` (the blob is a copy), but you cannot
re-register without downloading it again.
