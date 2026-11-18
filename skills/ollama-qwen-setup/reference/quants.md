# Cuantizaciones GGUF de Qwen3.8-27B: tamaños y cómo elegir

Solamente para la ruta avanzada ([`unsloth-custom-quant.md`](unsloth-custom-quant.md)). Desde la v2, lo normal es una
etiqueta de la biblioteca o un GGUF listo descargado con `ollama pull`; las opciones están en [`models.md`](models.md).

Repositorio de origen: <https://huggingface.co/unsloth/Qwen3.8-27B-GGUF> (Apache-2.0).
Tamaños leídos de la API de Hugging Face el **2026-09-03**. El repositorio se vuelve a cuantizar de vez en cuando:
confirma los tamaños actuales antes de anunciar una descarga (el comando está en `unsloth-custom-quant.md`).

## La fórmula para saber qué cabe

```
VRAM utilizable      = VRAM total − VRAM ya en uso − 1.0 GiB   (pantalla, compositor, sistema)
espacio para el GGUF = VRAM utilizable − 0.9 GiB (mmproj) − caché del contexto
```

La caché del contexto, según `num_ctx`, ronda:

| num_ctx | Caché del contexto |
|---|---|
| 8192 | ~0.8 GiB |
| 16384 | ~1.6 GiB |
| 32768 | ~3.0 GiB |
| 65536 | ~6.0 GiB |

Son estimaciones de trabajo, no una especificación. `ollama ps` imprime el reparto real entre CPU y GPU del modelo
cargado: si dice algo distinto de `100% GPU`, el cálculo estaba mal, diga lo que diga la aritmética. Al revés no vale: en
Windows la tarjeta se puede desbordar a la memoria del sistema mientras `ollama ps` sigue diciendo `100% GPU`; revisa
también la memoria libre y cronometra una respuesta (`SKILL.md`, paso 9).

## Tabla de cuantizaciones

Los archivos `UD-` son cuantizaciones dinámicas de Unsloth: la precisión se reparte por tensor en lugar de ser uniforme,
así que le ganan a una cuantización simple del mismo tamaño. Prefiérelos a `Q4_0` / `Q4_1` / `Q8_0`.

| Archivo | GiB | ¿Cabe? (tarjeta de 24 GiB, contexto 32k) | Notas |
|---|---:|---|---|
| `Qwen3.8-27B-UD-IQ1_S.gguf` | 5.77 | sí | pérdida de calidad severa; último recurso en 8 GiB |
| `Qwen3.8-27B-UD-IQ1_M.gguf` | 6.27 | sí | igual, un poco mejor |
| `Qwen3.8-27B-UD-IQ2_XXS.gguf` | 6.77 | sí | tarjetas de 8 GiB |
| `Qwen3.8-27B-UD-IQ2_S.gguf` | 7.80 | sí | tarjetas de 10 GiB |
| `Qwen3.8-27B-UD-Q2_K_XL.gguf` | 9.15 | sí | piso cómodo para 12 GiB; el razonamiento se degrada |
| `Qwen3.8-27B-UD-IQ3_XXS.gguf` | 10.18 | sí | 12 GiB, justo |
| `Qwen3.8-27B-UD-IQ3_S.gguf` | 11.21 | sí | 16 GiB, seguro |
| `Qwen3.8-27B-UD-Q3_K_XL.gguf` | 12.24 | sí | **recomendado para 16 GiB** |
| `Qwen3.8-27B-UD-IQ4_XS.gguf` | 13.28 | sí | 16 GiB, justo; calidad cercana a Q4 |
| `Qwen3.8-27B-UD-Q4_K_S.gguf` | 14.30 | sí | tarjetas de 20 GiB |
| `Qwen3.8-27B-Q4_0.gguf` | 14.95 | sí | cuantización uniforme antigua; mejor `UD-Q4_K_S` |
| `Qwen3.8-27B-UD-Q4_K_M.gguf` | 15.33 | sí | 20 GiB, cómodo |
| `Qwen3.8-27B-Q4_1.gguf` | 16.33 | sí | antigua; mejor `UD-Q4_K_XL` |
| `Qwen3.8-27B-UD-Q4_K_XL.gguf` | 16.35 | sí | **recomendado para 24 GiB**: la mejor calidad por GB aquí |
| `Qwen3.8-27B-UD-Q5_K_S.gguf` | 17.38 | sí | 24 GiB, con espacio para contexto de 32k |
| `Qwen3.8-27B-UD-Q5_K_M.gguf` | 18.41 | justo | 24 GiB bajando el contexto a 16k |
| `Qwen3.8-27B-UD-Q5_K_XL.gguf` | 19.44 | justo | 24 GiB solamente si la tarjeta no tiene nada más |
| `Qwen3.8-27B-UD-Q6_K.gguf` | 20.47 | no | 32 GiB |
| `Qwen3.8-27B-UD-Q6_K_M.gguf` | 21.50 | no | 32 GiB |
| `Qwen3.8-27B-UD-Q6_K_L.gguf` | 22.53 | no | 32 GiB, justo |
| `Qwen3.8-27B-UD-Q6_K_XL.gguf` | 23.56 | no | **recomendado para 48 GiB** |
| `Qwen3.8-27B-UD-Q8_K_L.gguf` | 26.12 | no | 48 GiB |
| `Qwen3.8-27B-Q8_0.gguf` | 27.05 | no | cuantización uniforme antigua |
| `Qwen3.8-27B-UD-Q8_K_XL.gguf` | 29.30 | no | 48 GiB; poca mejora sobre Q6 |
| `BF16/` (2 partes) | 50.90 | no | sin cuantizar, de referencia; clase de 80 GiB |

Archivos que lo acompañan:

| Archivo | GiB | Qué es |
|---|---:|---|
| `mmproj-BF16.gguf` | 0.87 | **proyector de visión: descarga este**, necesario para imágenes |
| `mmproj-F16.gguf` | 0.86 | otra precisión; cualquiera de los dos sirve, BF16 es el normal |
| `imatrix_unsloth.gguf` | 0.01 | matriz de importancia con la que se hicieron las cuantizaciones; no hace falta para usarlo |
| `MTP/mtp-Qwen3.8-27B-Q4_0.gguf` | 1.28 | modelo borrador para decodificación especulativa en llama.cpp; **no sirve desde Ollama**, sáltalo |

## Cómo elegir entre dos que caben

Toma el más grande solamente si se cumplen las tres cosas: el paso 9 de `SKILL.md` pasa entero (no solamente
`100% GPU`), el usuario no necesita un segundo modelo cargado, y no va a subir `num_ctx` después. Si no, toma el más
pequeño: la diferencia de calidad entre cuantizaciones vecinas por encima de Q4 es pequeña, y un modelo que se desborda
es de 10 a 20 veces más lento, que no es poco.

Por debajo de Q3, la calidad cae mucho en razonamiento y en código. Un modelo más pequeño en Q4 suele ser mejor trato que
Qwen3.8-27B en Q2.
