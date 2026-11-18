# Ruta avanzada: una cuantización de Unsloth de Qwen3.8-27B, con imágenes, registrada a mano

Este era el procedimiento principal de la v1 (2026-09-05). Desde la v2, la ruta normal es `ollama pull` de una etiqueta
de la biblioteca (ver `SKILL.md`). Usa esta solamente cuando el usuario quiera una cuantización concreta de Unsloth que la
biblioteca no tiene **y** quiera usarla con imágenes. Los tamaños y la fórmula para calcular cuál cabe están en
[`quants.md`](quants.md).

## Lo que hay que saber

- Las cuantizaciones vienen de **`unsloth/Qwen3.8-27B-GGUF`** en Hugging Face (Apache-2.0). Los archivos `UD-` son
  cuantizaciones dinámicas de Unsloth: precisión distinta por tensor, mejor calidad por GB que los `Q4_0` / `Q8_0` simples
  del mismo tamaño.
- Qwen3.8-27B es **multimodal**. El texto funciona solamente con el GGUF; las imágenes necesitan **`mmproj-BF16.gguf`**
  (~0.87 GB) en la misma carpeta en el momento de `ollama create`. Descarga siempre los dos.
- `ollama create` **copia** el GGUF al almacén de Ollama. Calcula **unas 2.2 veces el tamaño del modelo** de espacio
  libre, o mueve el almacén antes (`OLLAMA_MODELS`, paso 6 de `SKILL.md`).

## Descarga los dos archivos

Confirma primero el tamaño actual (el repositorio se vuelve a cuantizar de vez en cuando):

```bash
curl -s "https://huggingface.co/api/models/unsloth/Qwen3.8-27B-GGUF?blobs=true" \
  | python -c "import json,sys; [print(f\"{f['rfilename']:44s} {f.get('size',0)/2**30:6.2f} GiB\") for f in json.load(sys.stdin)['siblings'] if f['rfilename'].endswith('.gguf')]"
```

La forma preferida (se puede reanudar, descarga en paralelo y comprueba las sumas):

```bash
pip install -U "huggingface_hub[cli]"
hf download unsloth/Qwen3.8-27B-GGUF \
  Qwen3.8-27B-UD-Q4_K_XL.gguf mmproj-BF16.gguf \
  --local-dir "E:\AI_models_opensource\qwen3.8-27b"
```

Sin Python:

```bash
curl -L -C - -o "E:\AI_models_opensource\qwen3.8-27b\Qwen3.8-27B-UD-Q4_K_XL.gguf" \
  "https://huggingface.co/unsloth/Qwen3.8-27B-GGUF/resolve/main/Qwen3.8-27B-UD-Q4_K_XL.gguf?download=true"
```

Una descarga de 17 GB tarda decenas de minutos. Córrela en un proceso independiente, nunca en una llamada que pueda
cortarse por tiempo, y **compara el tamaño en disco con el de Hugging Face antes de registrarlo**. Un GGUF incompleto
falla en `ollama create` con un error de tensores confuso, y un proxy que devolvió una página de error deja un «modelo»
de 4 KB.

## Registra el modelo

Un `Modelfile` junto al GGUF, con la ruta absoluta en `FROM`:

```
FROM E:\AI_models_opensource\qwen3.8-27b\Qwen3.8-27B-UD-Q4_K_XL.gguf
PARAMETER num_ctx 8192
PARAMETER temperature 1.0
PARAMETER top_p 0.95
PARAMETER top_k 20
PARAMETER min_p 0.0
```

Son los valores para el modo de razonamiento de la ficha oficial del modelo (<https://huggingface.co/Qwen/Qwen3.8-27B>,
leída el 2026-11-15; también da `presence_penalty 0.0`, que es el valor por defecto). Para usarlo sin razonamiento, la
ficha da `temperature 0.7`, `top_p 0.80`, `top_k 20`, `min_p 0.0` y `presence_penalty 1.5`. La v1 traía
`temperature 0.6`, que no es lo que recomienda la ficha de Qwen3.8. El `num_ctx` de un Modelfile es solamente el **valor
por defecto** del modelo: una sesión lo cambia con `/set parameter num_ctx` y una petición con `options.num_ctx`. La v1
dejaba 32768 fijo, y en una tarjeta de laptop de 24 GB con otras aplicaciones abiertas eso dejó la tarjeta casi llena y
el modelo escribía unas 15 veces más lento (ver `SKILL.md`, paso 9). Empieza en 8192 y súbelo solamente después de que
pase el paso 9.

```bash
ollama create qwen3.8-27b -f "E:\AI_models_opensource\qwen3.8-27b\Modelfile"
```

Correrlo dos veces con archivos distintos deja **los dos** en el disco, y `ollama list` no enseña el huérfano. Si falta
espacio, revisa después el tamaño del almacén.

## Prueba de imágenes

```bash
ollama run qwen3.8-27b "Describe esta imagen. E:\ruta\a\imagen.png"
```

Si contesta como si no viera nada, `mmproj-BF16.gguf` no quedó registrado: tiene que estar en la misma carpeta al
momento de `ollama create`; vuelve a registrarlo con los dos archivos ahí.

Borrar el GGUF original después no rompe `ollama` (lo que usa es una copia), pero ya no puedes volver a registrarlo sin
descargarlo otra vez.
