# claude-code-ollama-qwen-setup

Una skill para [Claude Code](https://claude.com/claude-code) (o cualquier agente que lea el archivo) que instala un
modelo de inteligencia artificial de parámetros abiertos en tu computadora con Ollama, a la medida de tu equipo: te
pregunta para qué lo quieres, mide tu tarjeta gráfica, te ofrece dos o tres modelos que caben (Qwen o Gemma), instala
Ollama, descarga el que elijas, ajusta la ventana de contexto, comprueba que de verdad va rápido y te enseña a liberar la
memoria.

Te pide permiso antes de instalar o descargar cualquier cosa, y te habla en tu idioma. **Léela antes de usarla**, no la
copies y pegues sin más: está en [`skills/ollama-qwen-setup/SKILL.md`](skills/ollama-qwen-setup/SKILL.md).

*English summary at the end.*

## Qué es una skill

Una skill es una carpeta con un archivo `SKILL.md` dentro. Claude lee la cabecera, decide cuándo aplica y sigue el
procedimiento que trae. No es un plugin, no hay que compilar nada y no tiene dependencias.

## Instalación

**Para todos tus proyectos:** en `~/.claude/skills/` (en Windows, `C:\Users\<tu usuario>\.claude\skills\`).

En PowerShell (Windows):

```powershell
git clone https://github.com/mm-arturo/claude-code-ollama-qwen-setup.git
Copy-Item -Recurse claude-code-ollama-qwen-setup\skills\* "$env:USERPROFILE\.claude\skills\"
```

En la terminal (Mac o Linux):

```bash
git clone https://github.com/mm-arturo/claude-code-ollama-qwen-setup.git
cp -r claude-code-ollama-qwen-setup/skills/* ~/.claude/skills/
```

**Solamente para un proyecto:** ponla en `<tu proyecto>/.claude/skills/`, y viaja con el proyecto.

Reinicia Claude Code y compruébalo con el comando `/skills`. Úsala por su nombre (`/ollama-qwen-setup`) o simplemente
pide «instala un modelo local», «quiero usar Qwen en mi computadora» o «instala Ollama»: Claude la reconoce por la
descripción de la cabecera.

**Con otro agente** (Codex u otros): pídele que lea `skills/ollama-qwen-setup/SKILL.md` y su carpeta `reference/` y que
los siga paso a paso.

## Qué hace

1. Te pregunta para qué vas a usar el modelo, si necesitas que se niegue menos y en qué carpeta guardar los modelos.
2. Mide la memoria de tu tarjeta (o la memoria unificada de una Mac), cuánta está ya en uso y el espacio libre en disco.
3. Te ofrece dos o tres opciones de [`reference/models.md`](skills/ollama-qwen-setup/reference/models.md): filas para
   tarjetas de 8, 16 y 24 GB, Qwen para pensar con calma y Gemma para una conversación ágil, primero las versiones
   oficiales. Las versiones modificadas («sin censura») solamente si las pides, con la evidencia publicada de lo que ganan
   y lo que pierden.
4. Instala Ollama con el comando oficial, después de enseñártelo.
5. Descarga el modelo que elegiste (`ollama pull`) y ajusta la ventana de contexto y el límite de salida.
6. Comprueba tres cosas, no una: `100% GPU` en `ollama ps`, suficiente memoria libre en la tarjeta y una respuesta
   cronometrada.
7. Te enseña a descargar el modelo de la memoria (`ollama stop`) y te deja una tarjeta con los comandos y los números
   que midió.

## Por qué comprueba tres cosas

Es el paso en el que casi todos se equivocan: el modelo carga, contesta, y va diez veces más lento de lo que debería. En
Windows puede pasar inclusive cuando `ollama ps` dice `100% GPU`: con el contexto en 32768 y otras aplicaciones abiertas,
una tarjeta de laptop de 24 GB se llenó y Qwen3.8-27B escribía a 1.7 tokens por segundo. El mismo modelo con el contexto
en 4096 escribía a 26. Por eso la skill mide en lugar de fiarse de una sola línea.

## Cambios

- **v2.1 (2026-11-18):** la skill, sus referencias y este README, en español. Una sección corta en inglés al final.
- **v2.0 (2026-11-18):** pregunta primero para qué lo quieres; Qwen y Gemma según la VRAM, primero las oficiales y las
  modificadas solamente si las pides, con su evidencia; comandos oficiales de instalación (`install.ps1`, `install.sh`);
  `ollama pull` como ruta normal; contexto y límite de salida explicados; modo solo local; comprobación en tres partes;
  cómo liberar la memoria. La cuantización de Unsloth registrada a mano, con imágenes, pasó a
  `reference/unsloth-custom-quant.md`, con los valores de muestreo actuales de la ficha de Qwen3.8.
- **v1 (2026-09-05):** Qwen3.8-27B con las cuantizaciones GGUF de Unsloth, registrado con `ollama create`.

## Ver también

[**claude-code-session-skills**](https://github.com/mm-arturo/claude-code-session-skills): `wrap-up` y `resume-work`,
para que el contexto sobreviva a un reinicio.

## Licencia

MIT, ver [LICENSE](LICENSE). Tómala, cámbiala y compártela. Los modelos que descargues tienen sus propias licencias:
léelas en la ficha de cada modelo.

## English

A Claude Code skill (or instructions any agent can follow) that sets up a local AI model with Ollama, sized to your
machine: it asks what you need it for, measures your graphics card, offers two or three models that fit (Qwen to think
things through, Gemma for quick conversation; official versions first, reduced-refusal versions only on request),
installs Ollama, downloads the model you pick, sets the context window, checks that it really runs fast (not only
`100% GPU`), and shows you how to free the memory. It asks before installing or downloading anything and talks to you in
your language. The instructions are in Spanish; Claude and other agents follow them in any language.

Install: copy `skills/ollama-qwen-setup` into `~/.claude/skills/` (commands above), restart Claude Code, then run
`/ollama-qwen-setup` or say "install ollama" or "run qwen locally". The English v2.0 is tagged
[`v2.0`](https://github.com/mm-arturo/claude-code-ollama-qwen-setup/tree/v2.0).
