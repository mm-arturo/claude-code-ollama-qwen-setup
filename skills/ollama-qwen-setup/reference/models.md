# Qué modelo para qué tarjeta: las opciones que ofrece esta skill

Revisado el **2026-11-16/18**. Las bibliotecas y los repositorios cambian: antes de anunciar una descarga, confirma la
etiqueta y su tamaño en su página (`ollama.com/library/<nombre>` o el repositorio de Hugging Face).

Las filas son **VRAM dedicada** de la tarjeta gráfica. No son la RAM del sistema ni la memoria unificada de una Mac (ver
«Macs» abajo). El tamaño del archivo no es toda la memoria: el contexto y el programa ocupan más, y otras aplicaciones ya
usan parte de la tarjeta.

## Dos columnas, dos usos

- **Pensar con calma (Qwen):** Qwen razona antes de contestar por defecto y suele pensar mucho. Bueno cuando el usuario
  puede esperar una mejor respuesta.
- **Conversación ágil (Gemma):** para tareas cortas y sencillas donde importa la rapidez.

Esta división es una orientación, no una medición: las dos familias tienen modo de razonamiento. Dilo así.

## Versiones oficiales: las de siempre

Ofrécelas primero. Usa una versión modificada solamente si el usuario dice expresamente que la oficial rechaza tareas que
necesita (paso 1 de `SKILL.md`).

| VRAM | Pensar con calma | Conversación ágil |
| --- | --- | --- |
| 8 GB | `qwen3.5:4b` - 3.3 GB | `gemma4:e2b` - 4.6 GB; `gemma4:e4b` - 6.6 GB queda justo |
| 16 GB | `qwen3.5:9b` - 6.6 GB | `gemma4:12b` - 8.0 GB |
| 24 GB | `qwen3.8:27b` - 18 GB | `gemma4:26b` (26B-A4B) - 18 GB; `gemma4:31b` - 20 GB solamente con la tarjeta libre |

Los tamaños son las descargas Q4_K_M de las páginas de la biblioteca (2026-11-18), todas con entrada de imágenes:
<https://ollama.com/library/qwen3.5/tags>, <https://ollama.com/library/qwen3.8/tags>,
<https://ollama.com/library/gemma4/tags>. Qwen3.8 solamente se publica en 27B y su etiqueta más pequeña en la biblioteca
pesa 18 GB, así que una tarjeta de 16 GB no puede con el 27B oficial de la biblioteca: usa `qwen3.5:9b`, el `UD-Q3_K_XL`
de Unsloth (12.2 GiB, [`unsloth-custom-quant.md`](unsloth-custom-quant.md)) o, si lo pide, el Q3 de OrcaRouter de abajo.

## Versiones modificadas con menos rechazos («sin censura», «abliteradas», «Heretic»)

Solamente si el usuario las pide. Son modificaciones de terceros sobre los parámetros oficiales: alguien buscó el patrón
de actividad asociado a negarse y cambió parte de los parámetros para debilitarlo. No es una instrucción ni un ajuste. La
precisión de esa modificación varía mucho entre autores.

Las opciones que se enseñan en el video (tamaños de archivo en GB decimales, leídos de la API de Hugging Face el
2026-11-16):

| VRAM | Pensar con calma | Conversación ágil |
| --- | --- | --- |
| 8 GB | (sin opción de Qwen: usa el oficial `qwen3.5:4b`) | `hf.co/Abiray/gemma-4-E4B-it-heretic-GGUF:Q4_K_M` - 5.34 GB (la modificación Heretic de coder3101) |
| 16 GB | `hf.co/orcarouter/Qwen3.8-27B-Uncensored-GGUF:Q3_K_M` - 13.50 GB | `hf.co/llmfan46/gemma-4-26B-A4B-it-ultra-uncensored-heretic-GGUF:Q3_K_M` - 13.29 GB |
| 24 GB | `hf.co/orcarouter/Qwen3.8-27B-Uncensored-GGUF:Q4_K_M` - 16.81 GB | `hf.co/mradermacher/gemma-4-31B-it-heretic-GGUF:Q4_K_M` - 18.69 GB (la modificación de coder3101) |

La fila de 16 GB es justa: con el contexto que necesite el usuario puede no caber; entonces baja un tamaño.

Lo que dice la evidencia, y lo que no:

- Nathan Sapwell comparó el Qwen3.8-27B oficial con ocho versiones modificadas en una sola RTX 5090: 13 pruebas de
  capacidad y 400 solicitudes problemáticas, 167 horas de GPU en once días
  (<https://nathan.sapwell.net/posts/qwen38-27b-abliteration/>; la tabla completa y actualizada, con más variantes, está
  en <https://abliterlitics.dev/models/qwen38-27b/>). OrcaRouter salió como el mejor equilibrio. En TruthfulQA MC2 perdió
  1.5 puntos frente al original; Ultra-Heretic perdió 6.4. La pérdida más grande de la tabla completa fue la de
  `obliteratus`.
- OrcaRouter rechazó 2 de las 400 solicitudes, pero **el 38 % de sus respuestas no terminó de pensar** dentro del límite
  de generación de esa prueba. Revisa las dos cosas: si contesta, y si termina.
- Para Gemma 4 E4B, su estudio de 23 variantes no encontró una pérdida de capacidad distinguible en la modificación de
  coder3101 (<https://nathan.sapwell.net/posts/gemma4-e4b-abliteration/>). Las opciones de Gemma 26B y 31B no salen de un
  estudio de Sapwell.
- Las descargas no son calidad. La modificación más popular no es la que mejor salió en las pruebas.
- Ninguna de estas cifras se midió con la cuantización ni con la computadora del usuario.
- La licencia del modelo original sigue valiendo para la versión modificada.

## Modelos que no caben en una computadora normal

Kimi K3.1 Pro o DeepSeek V4.1 Flash se publican abiertos, pero hasta sus versiones más comprimidas necesitan cientos de
GB, o más de un terabyte, de memoria. Dilo si el usuario pregunta; no los ofrezcas.

## Macs (memoria unificada)

Apple Silicon comparte una sola memoria entre el procesador y la tarjeta gráfica. Como regla práctica, que el modelo más
el contexto usen como mucho un 70 % de la memoria total. Puede correr modelos grandes, normalmente más lento que una
tarjeta dedicada del mismo tamaño. Elige en las filas con ese presupuesto y después comprueba con el paso 9 de
`SKILL.md`.

## Sin tarjeta gráfica

Ollama también funciona con el procesador, solamente que mucho más lento. Ofrece las opciones más pequeñas
(`qwen3.5:4b` o menor, `gemma4:e4b`) y cronometra una respuesta antes de que el usuario cuente con él para trabajar.
