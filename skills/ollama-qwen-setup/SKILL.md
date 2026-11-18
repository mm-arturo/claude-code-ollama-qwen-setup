---
name: ollama-qwen-setup
description: Instala un modelo de inteligencia artificial local con Ollama, de principio a fin y a la medida de la
  computadora del usuario - pregunta para qué lo quiere, mide la memoria de la tarjeta gráfica (o la memoria unificada o
  la RAM), ofrece dos o tres modelos que caben (Qwen para pensar con calma, Gemma para una conversación ágil; primero las
  versiones oficiales y las versiones con menos rechazos solamente si las pide), instala Ollama, descarga el modelo
  elegido, ajusta la ventana de contexto, comprueba que de verdad va rápido (no solamente "100% GPU"), enseña a liberar
  la memoria y entrega una tarjeta resumen. Úsala cuando el usuario pida instalar Ollama, correr Qwen o Gemma en su
  computadora, tener una IA local o privada, usar IA sin internet, o escoger un modelo o una cuantización GGUF para su
  equipo. Also use it for the same requests in English.
---

# Un modelo de IA en tu computadora con Ollama (Qwen y Gemma)

Llevar una computadora sin nada instalado a un modelo local que funcione y que el usuario pueda usar desde la terminal,
desde una aplicación o por la API HTTP. Son once pasos. No te saltes el paso 9: un modelo que desborda la tarjeta sin
avisar parece que funciona hasta que la primera respuesta tarda minutos.

**Háblale al usuario en su idioma.** Esta skill está en español; la persona puede hablar otro.
**Cada ruta que le des al usuario es absoluta.** La va a copiar y pegar.

## Reglas básicas: pregunta antes de actuar

- **Pide permiso antes de** instalar Ollama (enseña el comando exacto y explica qué hace), antes de **cada** descarga
  (nombre del modelo, de dónde viene y cuánto pesa), antes de crear o cambiar una variable de entorno y antes de borrar
  cualquier cosa. Nunca mandes un script remoto directo a la terminal sin enseñarlo primero.
- **Solamente fuentes oficiales:** la biblioteca de Ollama (`ollama.com/library`) o la página del autor del modelo en
  Hugging Face. Una copia de un tercero es otro archivo, hecho por otra persona.
- **Lee la licencia** en la ficha del modelo antes de que el usuario lo use para trabajar. Los modelos de texto de Qwen
  que se usan aquí son Apache-2.0; otras familias tienen sus propias condiciones.
- **No cambies ajustes del controlador ni del sistema** (por ejemplo «CUDA - Sysmem Fallback Policy» de NVIDIA), salvo
  que el usuario lo pida después de que le expliques lo que gana y lo que arriesga.
- Di qué mediste y qué solamente leíste. Nunca escribas una velocidad que no cronometraste.

## 1. Pregunta para qué lo quiere

Tres preguntas cortas, y espera las respuestas:

1. **¿Para qué lo vas a usar?** ¿Tareas cortas (reescribir, resumir, traducir, sacar una lista de tareas de un correo) o
   razonamiento largo (análisis, código, documentos largos)?
2. **¿Necesitas que se niegue menos?** Solamente si el modelo oficial rechaza trabajo legítimo (por ejemplo, encuestas
   médicas anónimas o investigación sobre adicciones). Por defecto, versiones oficiales. Las versiones modificadas solo
   se ofrecen si el usuario las pide (ver `reference/models.md`).
3. **¿En qué carpeta van los modelos?** Pide una ruta absoluta en un disco con espacio. No la decidas tú.

## 2. Revisa lo que ya está instalado

```bash
ollama --version          # en Windows, si no responde: & "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" --version
ollama list
```

Si ya hay un modelo que sirve, dilo y pregunta si lo reutiliza. No vuelvas a descargar muchos gigabytes sin avisar. Los
nombres cambian según cómo se instaló: uno de la biblioteca aparece como `qwen3.8:27b` y uno registrado a mano puede
aparecer como `qwen3.8-27b:latest`. Usa siempre el nombre que imprime el `ollama list` del usuario.

## 3. Mide la computadora

NVIDIA (Windows o Linux): memoria total de la tarjeta y la que **ya está en uso**:

```bash
nvidia-smi --query-gpu=name,memory.total,memory.used --format=csv,noheader
```

El `nvidia-smi` normal también lista los procesos con sus rutas completas: no pegues esa parte en nada público.

- AMD: `rocm-smi --showmeminfo vram`.
- Apple Silicon: `sysctl -n hw.memsize` y `system_profiler SPHardwareDataType | grep Chip`. Memoria unificada: calcula
  que el modelo más el contexto usen como mucho un 70 % de ella.
- Sin tarjeta gráfica: también funciona con el procesador, pero mucho más lento. Dilo y ofrece solamente las opciones más
  pequeñas.

Lee también el espacio libre en el disco de destino. Resume en una línea: equipo, memoria total, memoria ya en uso y
disco libre. Cuando lo expliques, no mezcles la RAM, la VRAM y la memoria unificada: son cosas distintas.

## 4. Ofrece dos o tres opciones y deja que el usuario elija

Usa `reference/models.md`: filas por VRAM (8, 16 y 24 GB), columnas por uso (pensar con calma = Qwen, conversación ágil
= Gemma), primero las oficiales. Para cada opción da el nombre exacto, el tamaño de la descarga y una línea con lo que
gana y lo que pierde. Después pregunta cuál quiere: nunca decidas tú solo.

Dilo en voz alta:

- **El tamaño del archivo no es toda la memoria que necesita.** El contexto y el programa ocupan más, y otras
  aplicaciones ya usan parte de la tarjeta.
- Una opción **justa** no deja espacio para un segundo modelo y obliga a usar un contexto pequeño.
- Por debajo de Q3 la calidad cae mucho en razonamiento y en código; un modelo más pequeño en Q4 suele ganarle a uno
  grande en Q2.
- Si eligió una versión modificada (con menos rechazos): puede contestar lo que la oficial bloqueaba, y también puede
  equivocarse más en todo lo demás; las descargas no dicen cuál es mejor (evidencia y límites en `reference/models.md`).

Para una cuantización concreta de Unsloth con entrada de imágenes, usa `reference/unsloth-custom-quant.md` en lugar de
los pasos 7 y 8.

## 5. Instala Ollama si no está

Los comandos oficiales (<https://ollama.com/download>):

- **Windows (PowerShell):** `irm https://ollama.com/install.ps1 | iex`, o el instalador `OllamaSetup.exe` de la misma
  página. Se instala en la cuenta del usuario sin permisos de administrador, en
  `C:\Users\<usuario>\AppData\Local\Programs\Ollama\`, y se añade al PATH: hay que abrir una terminal **nueva** para que
  `ollama` funcione.
- **macOS:** `curl -fsSL https://ollama.com/install.sh | sh`, o `Ollama.dmg` de la misma página.
- **Linux:** `curl -fsSL https://ollama.com/install.sh | sh` (instala un servicio de systemd).

Enseña el comando, explica qué hace y córrelo solamente cuando el usuario esté de acuerdo (o deja que lo corra él).
Comprueba con `ollama --version` en una terminal nueva.

## 6. Opcional: carpeta de modelos y modo solo local

Los dos cambios necesitan reiniciar Ollama (cerrar la aplicación de la bandeja y abrirla otra vez, o reiniciar el
servicio). Pregunta antes de hacer cualquiera de los dos: son permanentes.

- **Carpeta de modelos** (por defecto: `C:\Users\<usuario>\.ollama\models`, `~/.ollama/models`, o
  `/usr/share/ollama/.ollama/models` en Linux). Para usar la carpeta del paso 1:

  ```powershell
  [Environment]::SetEnvironmentVariable("OLLAMA_MODELS", "E:\ollama\models", "User")
  ```

  En Linux, el usuario `ollama` necesita permiso de lectura y escritura en esa carpeta.
- **Solo local:** Ollama también ofrece modelos en la nube (etiquetas como `:cloud`). Si el usuario no quiere que nada
  salga de su computadora, desactiva las funciones de nube con `OLLAMA_NO_CLOUD=1` en el proceso del **servidor** (o con
  `"disable_ollama_cloud": true` en `~/.ollama/server.json`), reinicia y comprueba que el registro dice
  `Ollama cloud disabled: true`. Ponerla en una terminal que habla con un servidor que ya estaba corriendo no cambia nada.

## 7. Descarga el modelo elegido

Solamente este paso necesita internet.

```bash
ollama pull qwen3.8:27b                                              # oficial, de la biblioteca de Ollama
ollama pull hf.co/orcarouter/Qwen3.8-27B-Uncensored-GGUF:Q4_K_M     # un GGUF de Hugging Face
```

Usa el nombre exacto de `reference/models.md` o de la página del modelo. Las descargas grandes tardan: córrelas donde un
límite de tiempo no las pueda cortar. Al terminar, `ollama list` enseña el nombre que hay que usar desde ese momento.

## 8. Primera conversación, con el contexto ajustado

```bash
ollama run <nombre> --think=false
```

- **Razonamiento:** Qwen piensa antes de contestar por defecto. Para tareas cortas, `--think=false` contesta
  directamente y más rápido, aunque puede perder precisión en tareas difíciles. Sin esa opción (lo normal) contesta más
  lento y suele contestar mejor. Si el razonamiento se queda encendido, déjale espacio de salida (abajo).
- **Ventana de contexto**: cuánto texto puede tener en cuenta el modelo. Se ajusta dentro del chat (son comandos del
  chat, no de la terminal):

  ```text
  /set parameter num_ctx 4096
  /set parameter num_predict 1024
  ```

  - `num_ctx`: 4096 sobra para un correo o una página. Para documentos largos o razonamiento, prueba 16384 si la memoria
    alcanza (paso 9). Un contexto más pequeño que el texto hace que el modelo no lo vea completo; uno tan grande que
    llena la tarjeta hace que todo vaya lentísimo.
  - `num_predict`: el límite de salida. Con el razonamiento encendido, el razonamiento también lo gasta y la respuesta se
    puede cortar antes de empezar: súbelo (por ejemplo a 8192) para razonamientos largos.
  - Por la API, lo mismo va en `"options": {"num_ctx": 4096}`. Para todo el servidor: `OLLAMA_CONTEXT_LENGTH` (necesita
    reiniciar). Las versiones recientes de Ollama ya eligen un valor según la VRAM (unos 4 mil tokens con menos de 24
    GiB, 32 mil entre 24 y 48 GiB y 256 mil por encima), y el Modelfile de un modelo puede traer el suyo: comprueba en el
    paso 9 lo que de verdad quedó cargado.

## 9. Comprueba: este paso no es opcional

En una segunda terminal, con el modelo cargado:

```bash
ollama ps
nvidia-smi --query-gpu=memory.total,memory.used --format=csv,noheader
```

Después cronometra una respuesta real:

```bash
ollama run <nombre> --think=false --verbose "Reescribe en dos frases: <un párrafo corto>"
```

Pasa solamente si se cumplen **las tres** cosas:

1. `ollama ps` dice `100% GPU` en PROCESSOR y el contexto que pusiste en CONTEXT;
2. la tarjeta no está casi llena: deja margen (en un caso medido, 1.3 GB libres fueron muy poco y 2.6 GB fueron
   suficientes en una tarjeta de laptop de 24 GB);
3. la velocidad (`eval rate`) es razonable para el equipo (referencia: 26 tokens/s con Qwen3.8-27B Q4_K_S, contexto
   4096, razonamiento apagado, RTX 5090 Laptop de 24 GB, medido el 2026-11-15).

**`100% GPU` por sí solo no prueba nada.** En Windows, el controlador de NVIDIA puede pasar sin avisar a la memoria del
sistema cuando la tarjeta se llena (<https://nvidia.custhelp.com/app/answers/detail/a_id/5490>), y Ollama no lo ve: en
esa misma laptop, con el contexto en 32768 y otras aplicaciones ocupando unos 5.7 GB de la tarjeta, `ollama ps` decía
`100% GPU` y el modelo escribía a 1.7 tokens/s, quince veces más lento. Al bajar el contexto a 4096 volvió a 26.

Si no pasa, corrige en este orden: baja `num_ctx` (32768 → 16384 → 8192 → 4096); cierra lo que esté usando la tarjeta
(juegos, renders, navegadores con aceleración por hardware) y vuelve a mirar `nvidia-smi`; baja un tamaño o una
cuantización.

## 10. Libera la memoria al terminar

```text
/bye
```

```bash
ollama ps                 # muchas veces sigue cargado: se descarga solo tras unos 5 min sin uso
ollama stop <nombre>      # lo descarga ya
ollama ps                 # vacío = la tarjeta vuelve a estar libre
```

Los archivos se quedan en el disco para la próxima vez. `ollama rm <nombre>` los borra: pregunta antes.

## 11. Entrega la tarjeta resumen

Termina con un bloque corto que el usuario pueda pegar en sus notas. Con los números reales que mediste; nada más, sin
repetir los pasos.

```
Modelo     <nombre de ollama list>   (<familia, tamaño, cuantización>, oficial | modificado por <autor>)
Licencia   <licencia de la ficha del modelo>
Memoria    tarjeta de <total> GB, libre con el modelo cargado <medido> GB, contexto <num_ctx>
Velocidad  <medida> tokens/s  (razonamiento apagado, contexto <num_ctx>)

Chat       ollama run <nombre> --think=false
           /set parameter num_ctx 4096
Una vez    ollama run <nombre> "tu instrucción"
API        curl http://localhost:11434/api/generate -d '{"model":"<nombre>","prompt":"hola","stream":false,"options":{"num_ctx":4096}}'
SDK OpenAI base_url="http://localhost:11434/v1", api_key="ollama"   (cualquier valor; Ollama lo ignora)
Descargar  ollama stop <nombre>        (se descarga solo tras ~5 min sin uso)
Borrar     ollama rm <nombre>
```

## Trampas conocidas

- **En Windows, `ollama` sin argumentos abre la aplicación de escritorio.** `ollama <argumentos>` es la terminal. No es
  un error.
- **Ollama tiene que estar corriendo** para que funcione cualquier comando. Windows y macOS: la aplicación de la bandeja.
  Linux: `systemctl status ollama`. Con el servidor apagado sale `connection refused` en el puerto 11434.
- **Salir del chat no libera la tarjeta en ese momento.** Usa `ollama stop`.
- **La memoria de la tarjeta se comparte con todo lo demás.** Revisa `nvidia-smi` en el momento del problema, no de
  memoria.
- **El `num_ctx` de un Modelfile es solamente un valor por defecto.** Cambiar el Modelfile no hace nada hasta volver a
  correr `ollama create`, pero una sesión (`/set parameter num_ctx`) o una petición (`options.num_ctx`) lo cambian.
- **Los nombres cambian entre instalaciones.** Usa el que imprime `ollama list` en esa computadora.
