# claude-code-ollama-qwen-setup

A [Claude Code](https://claude.com/claude-code) skill that sets up a local AI model with Ollama,
sized to your machine: it asks what you need it for, measures your graphics card, offers two or
three models that fit (Qwen or Gemma), installs Ollama, downloads the model you pick, sets the
context window, checks that it really runs fast, and shows you how to free the memory again.

**En español:** esta skill le da a tu agente (Claude Code, o cualquier agente que lea el archivo)
las instrucciones para instalar Ollama y un modelo de parámetros abiertos según tu computadora. Te
pregunta para qué lo quieres, mide tu tarjeta, te da dos o tres opciones con su tamaño, y te pide
permiso antes de instalar o descargar cualquier cosa. Te habla en tu idioma. Léela antes de usarla:
está en [`skills/ollama-qwen-setup/SKILL.md`](skills/ollama-qwen-setup/SKILL.md).

A skill is a folder with a `SKILL.md` in it. Claude reads the front matter, decides when the skill
applies, and follows the procedure inside. No plugin, no build step, no dependencies.

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
say "install ollama", "run qwen locally" or "instala un modelo local" — the `description` in the
front matter is what Claude matches against.

**Another agent** (Codex or others): ask it to read
`skills/ollama-qwen-setup/SKILL.md` and its `reference/` folder and follow them step by step.

## What it does

1. Asks what the model is for, whether you need fewer refusals, and where to store models.
2. Measures VRAM (or unified memory on a Mac), what is already in use, and free disk.
3. Offers two or three options from [`reference/models.md`](skills/ollama-qwen-setup/reference/models.md):
   rows for 8, 16 and 24 GB cards, Qwen to think things through, Gemma for quick conversation,
   official versions first. Modified ("uncensored") versions only if you ask, with the published
   evidence of what they gain and lose.
4. Installs Ollama with the official command, after showing it to you.
5. Downloads the model you chose (`ollama pull`), sets the context window and the output limit.
6. Verifies three things, not one: `100% GPU` in `ollama ps`, enough free memory on the card, and a
   timed answer.
7. Shows how to unload the model (`ollama stop`) and hands you a card with the commands and the
   numbers it measured.

## Why the verification has three checks

Everyone gets the same step wrong: the model loads, answers, and is ten times slower than it should
be. On Windows it can even happen while `ollama ps` says `100% GPU`: with the context at 32768 and
other apps open, a 24 GB laptop card filled up and Qwen3.8-27B wrote at 1.7 tokens/s. The same model
with the context at 4096 wrote at 26 tokens/s. So the skill measures instead of trusting one line.

## Changelog

- **v2 (2026-11-18):** asks what you need first; Qwen and Gemma by VRAM, official first, modified
  versions on request with evidence; official install commands (`install.ps1`, `install.sh`);
  `ollama pull` as the default route; context and output limit explained; local-only mode;
  three-check verification; how to free the memory. The hand-registered Unsloth quant with vision
  moved to `reference/unsloth-custom-quant.md`, with the Qwen3.8 card's current sampling values.
- **v1 (2026-09-05):** Qwen3.8-27B from Unsloth GGUF quants, registered with `ollama create`.

## See also

[**claude-code-session-skills**](https://github.com/mm-arturo/claude-code-session-skills) —
`wrap-up` and `resume-work`, so context survives a restart.

## License

MIT — see [LICENSE](LICENSE). Take it, change it, ship it. The models you download have their own
licenses: read the model card.
