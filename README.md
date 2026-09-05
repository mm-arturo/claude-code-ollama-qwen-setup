# claude-code-ollama-qwen-setup

A [Claude Code](https://claude.com/claude-code) skill that installs Ollama and Qwen3.8-27B from
zero — measures your VRAM, recommends the quant that fits, downloads, registers, and verifies
the model runs 100% on the GPU.

A skill is a folder with a `SKILL.md` in it. Claude reads the front matter, decides when the
skill applies, and follows the procedure inside. No plugin, no build step, no dependencies.

## Install

**All projects** — `~/.claude/skills/` (Windows: `C:\Users\<you>\.claude\skills\`):

```bash
git clone https://github.com/mm-arturo/claude-code-ollama-qwen-setup.git
cp -r claude-code-ollama-qwen-setup/skills/* ~/.claude/skills/
```

```powershell
git clone https://github.com/mm-arturo/claude-code-ollama-qwen-setup.git
Copy-Item -Recurse claude-code-ollama-qwen-setup\skills\* "$env:USERPROFILE\.claude\skills\"
```

**One project only** — put it in `<repo>/.claude/skills/` instead, and it ships with the repo.

Restart Claude Code, then check with `/skills`. Invoke it by name (`/ollama-qwen-setup`) or just
say "install ollama" or "run qwen locally" — the `description` in the front matter is what Claude
matches against.

## Why this exists

Everyone gets the same step wrong.

Picking a GGUF quant is arithmetic — model size, plus the vision projector, plus the KV cache
for your context length, against the VRAM you actually have free right now. Get it slightly
wrong and the model still loads, still answers, and is ten times slower forever, because half of
it quietly went to system RAM.

So the skill measures instead of guessing: reads `nvidia-smi`, computes the budget, offers you
two or three quants with the trade-off spelled out, downloads the GGUF **and** `mmproj-BF16.gguf`
(images do not work without it), registers the model with Qwen's own recommended sampling
parameters, and then verifies with `ollama ps` that the answer says `100% GPU`. It ends with a
paste-ready card: absolute paths, the CLI call, the HTTP call, the Python call, and how to free
the VRAM again.

On a 24 GB card that lands on `UD-Q4_K_XL`. It will tell you why, and what you would give up by
taking `UD-Q5_K_XL` instead.

Full quant table and the sizing formula:
[`skills/ollama-qwen-setup/reference/quants.md`](skills/ollama-qwen-setup/reference/quants.md).

The traps are the valuable part: `ollama create` copies 17 GB instead of linking it; `num_ctx`
is baked in at create time and editing the Modelfile later does nothing. Knowledge somebody paid
for once, written down so nobody pays again.

## See also

[**claude-code-session-skills**](https://github.com/mm-arturo/claude-code-session-skills) —
`wrap-up` and `resume-work`, so context survives a restart.

## License

MIT — see [LICENSE](LICENSE). Take it, change it, ship it.
