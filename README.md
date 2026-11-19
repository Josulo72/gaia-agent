# Agente GAIA – Hugging Face Agents Course (Unidad 4)

Agente de IA que responde preguntas del benchmark **GAIA** (nivel 1) combinando razonamiento con herramientas: búsqueda web, Wikipedia, lectura de hojas de cálculo, transcripción de audio, análisis de imágenes y ejecución de código.

Proyecto final del [Hugging Face Agents Course](https://huggingface.co/learn/agents-course).

## Cómo funciona

```
Pregunta GAIA (+ archivo adjunto opcional)
        │
        ▼
CodeAgent (smolagents) ── modelo: Claude vía LiteLLM
        │  ciclo ReAct: Pensar → escribir código Python → ejecutar → observar
        ├── DuckDuckGoSearchTool   búsqueda web
        ├── VisitWebpageTool       leer páginas
        ├── WikipediaSearchTool    Wikipedia
        ├── transcribe_audio       Whisper (mp3, wav)
        ├── analyze_image          visión de Claude (ajedrez, gráficos, fotos)
        ├── youtube_transcript     subtítulos de vídeos
        ├── run_python_file        ejecutar archivos .py adjuntos
        └── pandas / openpyxl      Excel y CSV
        │
        ▼
Respuesta normalizada (coincidencia exacta) → API de puntuación
```

### Decisiones de diseño

- **CodeAgent en lugar de ToolCallingAgent**: el agente escribe Python, lo que le permite combinar herramientas, filtrar datos de un Excel o hacer cálculos en un solo paso.
- **Planificación cada 4 pasos** (`planning_interval=4`): el agente revisa su plan periódicamente y evita bucles.
- **Prompt de formato estricto**: GAIA puntúa por coincidencia exacta, así que el prompt obliga a devolver solo el valor (sin unidades, artículos ni explicaciones) y una función `clean()` elimina restos como "Final answer:".
- **Progreso guardado en `answers.json`**: si la ejecución se corta, continúa donde lo dejó sin volver a pagar llamadas al modelo.
- **Tandas de 5 preguntas** (`BATCH_SIZE`): permite revisar aciertos y gasto antes de seguir.
- **Control de tokens**: el texto de cada página visitada se limita a 10.000 caracteres y se reutilizan las respuestas ya obtenidas.
- **Respuestas largas**: si el agente agota pasos y devuelve texto, una llamada extra al modelo extrae solo el valor final.

## Ejecutarlo

1. Abre `gaia_agent.ipynb` en [Google Colab](https://colab.research.google.com/).
2. En el panel 🔑 *Secrets*, crea `ANTHROPIC_API_KEY` con tu clave de Anthropic.
3. Rellena la celda **Configuración** (usuario de Hugging Face y URL de este repositorio).
4. Ejecuta las celdas en orden: prueba con una pregunta, lanza las tandas, revisa y envía.

Coste: depende de cuántos pasos necesite cada pregunta. Con Claude Sonnet puede rondar 1-2 US$ por pregunta en las más complejas; con Haiku es varias veces más barato.

## Resultados

| Modelo | Aciertos | Puntuación |
|---|---|---|
| Claude Sonnet | _pendiente_ | _pendiente_ |

## Limitaciones y mejoras posibles

- DuckDuckGo limita peticiones desde Colab; con una API de búsqueda dedicada (Tavily, Serper) el agente sería más fiable.
- Las preguntas sobre vídeos sin subtítulos no se pueden resolver sin analizar los fotogramas.
- El servidor del curso devuelve 404 para 5 de los archivos adjuntos (imagen, audios, .py y Excel); esas preguntas se responden sin el archivo o quedan sin resolver.
- Siguiente paso: añadir trazas con Langfuse para ver en qué paso falla cada pregunta.

## Stack

Python · smolagents · LiteLLM · Claude (Anthropic) · Whisper · pandas · Google Colab
