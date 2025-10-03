---
date: "2025-08-05T00:00:00Z"
draft: true
tags:
- configuraciones
- llm
title: Mejorando la ejecución de modelos en llama.cpp
---

## Introdución
Pues nada, será el primer acercamiento a como configurar adecuadamente la ejecución de un modelo tal para según que tareas:

## Procedmiento
Obtenemos el modelo en [Hugginface](https://huggingface.co/Qwen/Qwen3-0.6B-GGUF?show_file_info=Qwen3-0.6B-Q8_0.gguf). Lo copiamos en un directorio destinado a tal fin (En nuestro caso, `/var/lib/llama.cpp/`)

Instalamos una herramienta para mejorar el trabajo con `.gguf`:

```bash
pip install "gguf"
# Si estamos en zsh, es posible que ejecutar rehash sea necesario
```

Ahora podemos ejecutar
```bash
gguf-dump /var/lib/llama.cpp/Qwen3-0.6B-Q8_0.gguf
```

Qué nos devuelve información un montón de información, de la cual usaremos alguna para obtener valores más adecuados en su ejecución:

## ⚙️Primer acercamiento a una optimización en la ejecución de Qwen3-0.6B-Q8_0 en llama.cpp 

## 1. `--ctx-size` (Context Size, default: 512)

**Define**: Ventana de memoria del modelo - cuántos tokens puede "recordar" en una conversación

**Valor en GGUF**: `qwen3.context_length = 40960` - El modelo soporta hasta 40,960 tokens nativamente

**Configuración típica**: `--ctx-size 4096`  
**Impacto**: Mayor contexto = más memoria RAM y procesamiento más lento, pero mejor comprensión de conversaciones largas

**Valores típicos**:
- `512-2048`: Uso ligero, respuestas rápidas, conversaciones cortas
- `2048-8192`: Balance óptimo para la mayoría de casos
- `8192-16384`: Conversaciones largas, análisis de código extenso
- `16384-40960`: Casos especiales, documentos completos

**Rango posible**: 512-40960  
**Valor recomendado**: `4096`

**Lecturas recomendadas**:
- [Understanding llama.cpp internals - Context Management](https://sidshome.wordpress.com/2023/12/24/understanding-internals-of-llama-cpp/)
- [llama.cpp Context Size Discussion](https://github.com/ggml-org/llama.cpp/discussions/1838)
- [Context Length Documentation Issues](https://github.com/ggml-org/llama.cpp/issues/5732)

---

## 2. `--n-predict` (Prediction Length, default: -1)

**Define**: Límite máximo de tokens que generará en cada respuesta (-1 = infinito hasta token de fin)

**Valor en GGUF**: Relacionado con `qwen3.context_length` pero independiente - controla solo la generación

**Configuración típica**: `--n-predict 512`  
**Impacto**: Controla la longitud de las respuestas y previene generación infinita

**Valores típicos**:
- `-1`: Generación infinita (peligroso sin presence-penalty)
- `128-256`: Respuestas muy cortas y concisas
- `256-512`: Respuestas estándar balanceadas
- `512-1024`: Explicaciones detalladas
- `1024-2048`: Código complejo o documentación

**Rango posible**: -2 a 32768  
**Valor recomendado**: `512`

**Lecturas recomendadas**:
- [llama.cpp Guide - Generation Parameters](https://blog.steelph0enix.dev/posts/llama-cpp-guide/)
- [llama.cpp Tutorial DataCamp](https://www.datacamp.com/tutorial/llama-cpp-tutorial)

---

## 3. `--temp` (Temperature, default: 0.8)

**Define**: Controla el balance entre creatividad y coherencia

**Valor en GGUF**: No especificado directamente, pero Qwen3 está optimizado para 0.6

**Configuración típica**: `--temp 0.6`  
**Impacto**: Valores bajos = respuestas más predecibles; valores altos = más creatividad

**Valores típicos**:
- `0.0-0.3`: Modo determinista, ideal para matemáticas y lógica
- `0.3-0.6`: Código y respuestas técnicas precisas
- `0.6-0.8`: Balance general, conversación natural
- `0.8-1.2`: Escritura creativa, brainstorming
- `1.2-2.0`: Experimental, alta aleatoriedad

**Rango posible**: 0.0-2.0  
**Valor recomendado**: `0.6` (documentación oficial Qwen3)

**Lecturas recomendadas**:
- [What is LLM Temperature? - Hopsworks](https://www.hopsworks.ai/dictionary/llm-temperature)
- [LLM Temperature Explained - IBM](https://www.ibm.com/think/topics/llm-temperature)
- [Temperature in LLMs - Mathematical Explanation](https://medium.com/@amansinghalml_33304/temperature-llms-b41d75870510)
- [Effect of Sampling Temperature on Problem Solving](https://arxiv.org/html/2402.05201v1)

---

## 4. `--top-p` (Nucleus Sampling, default: 0.95)

**Define**: Filtra tokens considerando solo los que suman hasta P% de probabilidad acumulativa

**Valor en GGUF**: No especificado, pero calibrado para 0.95 según documentación

**Configuración típica**: `--top-p 0.95`  
**Impacto**: Controla la diversidad del vocabulario utilizado

**Valores típicos**:
- `0.1-0.5`: Muy conservador, vocabulario limitado
- `0.5-0.8`: Moderadamente conservador
- `0.8-0.95`: Balance óptimo (recomendado)
- `0.95-0.99`: Permite mayor diversidad
- `0.99-1.0`: Considera casi todos los tokens

**Rango posible**: 0.0-1.0  
**Valor recomendado**: `0.95`

**Lecturas recomendadas**:
- [Top-p Sampling Wikipedia](https://en.wikipedia.org/wiki/Top-p_sampling)
- [Nucleus Sampling Explained](https://www.promptlayer.com/glossary/top-p-nucleus-sampling)
- [Generation Configurations - Chip Huyen](https://huyenchip.com/2024/01/16/sampling.html)
- [How to Use Top_P Parameter](https://www.vellum.ai/llm-parameters/top-p)

---

## 5. `--top-k` (Top-K Sampling, default: 40)

**Define**: Limita la selección a los K tokens más probables en cada paso

**Valor en GGUF**: No especificado, optimizado para k=20 según pruebas

**Configuración típica**: `--top-k 20`  
**Impacto**: Controla cuántas opciones considera el modelo

**Valores típicos**:
- `0`: Deshabilitado (solo usa top-p)
- `1-10`: Muy restrictivo, respuestas predecibles
- `10-20`: Óptimo para Qwen3, balance precisión
- `20-40`: Estándar llama.cpp
- `40-100`: Mayor variedad, riesgo de incoherencia

**Rango posible**: 0-100  
**Valor recomendado**: `20` (específico Qwen3)

**Lecturas recomendadas**:
- [Guide to Top-k, Top-p Parameters](https://ivibudh.medium.com/a-guide-to-controlling-llm-model-output-exploring-top-k-top-p-and-temperature-parameters-ed6a31313910)
- [Setting Top-K, Top-P in LLMs](https://rumn.medium.com/setting-top-k-top-p-and-temperature-in-llms-3da3a8f74832)
- [Top-K and Top-P Guide for Investors](https://www.alphanome.ai/post/top-k-and-top-p-in-large-language-models-a-guide-for-investors)

---

## 6. `--presence-penalty` (Presence Penalty, no default en llama-cli)

**Define**: Penaliza la repetición de tokens ya usados para evitar loops

**Valor en GGUF**: No especificado, pero crítico para modelos cuantizados Q8_0

**Configuración típica**: `--presence-penalty 1.5`  
**Impacto**: Esencial para evitar repeticiones en modelos cuantizados

**Valores típicos**:
- `0.0`: Sin penalización (riesgo de loops)
- `0.5-1.0`: Penalización ligera
- `1.0-1.5`: Óptimo para cuantizados (Q8_0)
- `1.5-1.8`: Anti-repetición fuerte
- `1.8-2.0`: Máximo, puede causar incoherencia

**Rango posible**: 0.0-2.0  
**Valor recomendado**: `1.5` (específico para Q8_0)

> ⚠️ **Nota**: Aumenta hasta 2.0 si experimentas loops

**Lecturas recomendadas**:
- [LLM Settings - Prompt Engineering Guide](https://www.promptingguide.ai/introduction/settings)
- [Comprehensive Guide to LLM Sampling](https://smcleod.net/2025/04/comprehensive-guide-to-llm-sampling-parameters/)

---

## 7. `--threads` (CPU Threads, default: detecta automáticamente)

**Define**: Número de hilos de CPU para procesamiento paralelo

**Valor en GGUF**: No aplica - depende del hardware

**Configuración típica**: `--threads 4`  
**Impacto**: Más hilos = procesamiento más rápido hasta saturación

**Valores típicos**:
- `1-2`: Sistemas muy limitados
- `2-4`: Laptops básicas, equilibrio energía
- `4-8`: Desktop estándar
- `8-16`: Workstations
- `16+`: Servidores

**Rango posible**: 1-[núcleos físicos]  
**Valor recomendado**: `[núcleos físicos / 2]`

**Lecturas recomendadas**:
- [llama.cpp Performance Guide](https://blog.steelph0enix.dev/posts/llama-cpp-guide/)
- [llama.cpp PyImageSearch Tutorial](https://pyimagesearch.com/2024/08/26/llama-cpp-the-ultimate-guide-to-efficient-llm-inference-and-applications/)

---

## 8. `--batch-size` (Batch Size, default: 2048)

**Define**: Número de tokens procesados simultáneamente

**Valor en GGUF**: Relacionado con `qwen3.embedding_length = 1024`

**Configuración típica**: `--batch-size 512`  
**Impacto**: Balance entre velocidad y uso de memoria

**Valores típicos**:
- `32-128`: Sistemas con poca RAM
- `128-512`: Balance conservador (Q8_0)
- `512-1024`: Estándar para modelos pequeños
- `1024-2048`: Default llama.cpp
- `2048+`: Solo con mucha RAM disponible

**Rango posible**: 32-2048  
**Valor recomendado**: `512` (óptimo para 0.6B Q8_0)

**Lecturas recomendadas**:
- [llama.cpp Documentation](https://github.com/ggml-org/llama.cpp)

---

## 9. `--rope-freq-base` (RoPE Frequency Base, default: 10000.0)

**Define**: Frecuencia base para codificación posicional rotativa

**Valor en GGUF**: `qwen3.rope.freq_base = 1000000.0` - **CRÍTICO: debe coincidir exactamente**

**Configuración típica**: `--rope-freq-base 1000000`  
**Impacto**: Valor incorrecto = salida corrupta o sin sentido

**Valores típicos**:
- `10000`: Default LLaMA (NO usar para Qwen3)
- `1000000`: **OBLIGATORIO para Qwen3**

**Rango posible**: Debe ser exactamente 1000000  
**Valor recomendado**: `1000000` (no negociable)

> ⚠️ **CRÍTICO**: Este valor DEBE coincidir con el modelo

**Lecturas recomendadas**:
- [RoPE Scaling Documentation Issue](https://github.com/ggml-org/llama.cpp/issues/2402)
- [Llama 3 RoPE Configuration Discussion](https://github.com/ggml-org/llama.cpp/discussions/6890)
- [Qwen 72B RoPE Issue](https://github.com/abetlen/llama-cpp-python/issues/1271)

---

## 10. `--min-p` (Min-P Sampling, default: 0.05)

**Define**: Umbral mínimo de probabilidad relativa para considerar tokens

**Valor en GGUF**: No especificado, calibrado para 0

**Configuración típica**: `--min-p 0`  
**Impacto**: Filtra tokens muy improbables

**Valores típicos**:
- `0`: Deshabilitado (recomendado Qwen3)
- `0.01-0.05`: Filtrado muy ligero
- `0.05-0.1`: Filtrado moderado
- `0.1-0.2`: Filtrado agresivo
- `0.2+`: Muy restrictivo

**Rango posible**: 0.0-1.0  
**Valor recomendado**: `0`

**Lecturas recomendadas**:
- [Comprehensive Guide to LLM Sampling](https://smcleod.net/2025/04/comprehensive-guide-to-llm-sampling-parameters/)

---

## 11. `--no-context-shift` (Context Shift, default: habilitado)

**Define**: Desactiva la rotación automática del contexto cuando se llena

**Valor en GGUF**: Relacionado con `qwen3.context_length` para manejo de overflow

**Configuración típica**: `--no-context-shift`  
**Impacto**: Evita comportamiento impredecible al llenar contexto

**Valores típicos**:
- Habilitado: Rotación automática (puede causar incoherencia)
- **Deshabilitado**: Detiene generación al límite (recomendado)

**Rango posible**: Flag booleano  
**Valor recomendado**: Usar el flag

**Lecturas recomendadas**:
- [Context Management in llama.cpp](https://github.com/ggml-org/llama.cpp/discussions/1838)

---

## 12. `--mlock` (Memory Lock, default: deshabilitado)

**Define**: Fuerza mantener el modelo en RAM física

**Valor en GGUF**: El modelo Q8_0 ocupa ~650MB según `general.file_type = 7`

**Configuración típica**: `--mlock`  
**Impacto**: Evita swap pero bloquea RAM

**Valores típicos**:
- Deshabilitado: Permite swap (más lento)
- **Habilitado**: Máximo rendimiento (si hay RAM)

**Rango posible**: Flag booleano  
**Valor recomendado**: Usar si tienes >2GB RAM libre

**Lecturas recomendadas**:
- [llama.cpp Performance Documentation](https://github.com/ggml-org/llama.cpp)

---

## 13. `--jinja` (Jinja Templates, default: deshabilitado)

**Define**: Usa la plantilla de chat Jinja2 incrustada

**Valor en GGUF**: `tokenizer.chat_template` presente en el modelo

**Configuración típica**: `--jinja`  
**Impacto**: Formateo correcto de mensajes sistema/usuario

**Valores típicos**:
- Deshabilitado: Formato manual
- **Habilitado**: Usa template del modelo (recomendado)

**Rango posible**: Flag booleano  
**Valor recomendado**: Siempre usar

**Lecturas recomendadas**:
- [Qwen3 Documentation - Chat Templates](https://qwen.readthedocs.io/en/latest/run_locally/llama.cpp.html)

---

## 14. `--color` (Colored Output, default: deshabilitado)

**Define**: Colorea la salida del terminal

**Valor en GGUF**: No aplica - preferencia visual

**Configuración típica**: `--color`  
**Impacto**: Solo visual, sin impacto en rendimiento

**Valores típicos**:
- Deshabilitado: Texto plano
- **Habilitado**: Mejor legibilidad

**Rango posible**: Flag booleano  
**Valor recomendado**: Usar en terminal interactivo

---

## 📋 Comando Completo Optimizado

```bash
llama-cli \
  -m /var/lib/llama.cpp/Qwen3-0.6B-Q8_0.gguf \
  --ctx-size 4096 \
  --n-predict 512 \
  --temp 0.6 \
  --top-p 0.95 \
  --top-k 20 \
  --min-p 0 \
  --presence-penalty 1.5 \
  --threads 4 \
  --batch-size 512 \
  --rope-freq-base 1000000 \
  --no-context-shift \
  --mlock \
  --jinja \
  --color \
  -p "Tu prompt aquí"
```

## 📊 Tabla de Referencia Rápida

| Parámetro | GGUF Key | Valor GGUF | Config Recomendada | Crítico |
|-----------|----------|------------|-------------------|---------|
| `--ctx-size` | qwen3.context_length | 40960 | 4096 | No |
| `--rope-freq-base` | qwen3.rope.freq_base | 1000000.0 | 1000000 | **SÍ** |
| `--temp` | - | - | 0.6 | Sí |
| `--top-k` | - | - | 20 | Sí |
| `--presence-penalty` | general.file_type | 7 (Q8_0) | 1.5 | Sí |
| `--batch-size` | qwen3.embedding_length | 1024 | 512 | No |
| `--jinja` | tokenizer.chat_template | {presente} | usar | Sí |

## 🎯 Configuraciones por Escenario de Uso

### 🔤 **Autocompletado de Código**

```bash
llama-cli -m /var/lib/llama.cpp/Qwen3-0.6B-Q8_0.gguf \
  --rope-freq-base 1000000 \
  --ctx-size 2048 \
  --n-predict 128 \
  --temp 0.2 \
  --top-p 0.9 \
  --top-k 10 \
  --min-p 0.05 \
  --presence-penalty 1.2 \
  --jinja \
  -p "Tu código aquí"
```

**Justificación**:
- `temp 0.2`: Muy determinista para código preciso
- `top-k 10`: Solo las opciones más probables
- `n-predict 128`: Completados cortos y rápidos
- `ctx-size 2048`: Suficiente contexto para el código actual

---

### 💬 **Chat Conversacional**

```bash
llama-cli -m /var/lib/llama.cpp/Qwen3-0.6B-Q8_0.gguf \
  --rope-freq-base 1000000 \
  --ctx-size 8192 \
  --n-predict 512 \
  --temp 0.7 \
  --top-p 0.95 \
  --top-k 30 \
  --min-p 0 \
  --presence-penalty 1.5 \
  --jinja \
  --color \
  --interactive \
  -p "Sistema: Eres un asistente amigable y útil."
```

**Justificación**:
- `temp 0.7`: Balance entre coherencia y naturalidad
- `top-k 30`: Más opciones para conversación natural
- `ctx-size 8192`: Mantener historial de conversación largo
- `--interactive`: Modo chat interactivo

---

### ✏️ **Editor de Código (Refactoring/Review)**

```bash
llama-cli -m /var/lib/llama.cpp/Qwen3-0.6B-Q8_0.gguf \
  --rope-freq-base 1000000 \
  --ctx-size 16384 \
  --n-predict 1024 \
  --temp 0.3 \
  --top-p 0.85 \
  --top-k 15 \
  --min-p 0.02 \
  --presence-penalty 1.3 \
  --batch-size 1024 \
  --jinja \
  -p "Revisa el siguiente código"
```

**Justificación**:
- `temp 0.3`: Preciso pero con algo de variación
- `ctx-size 16384`: Para archivos de código grandes
- `n-predict 1024`: Explicaciones detalladas
- `batch-size 1024`: Procesamiento más rápido

---

### 🤖 **Tareas como Asistente (Q&A, Análisis)**

```bash
llama-cli -m /var/lib/llama.cpp/Qwen3-0.6B-Q8_0.gguf \
  --rope-freq-base 1000000 \
  --ctx-size 4096 \
  --n-predict 768 \
  --temp 0.5 \
  --top-p 0.9 \
  --top-k 20 \
  --min-p 0.01 \
  --presence-penalty 1.4 \
  --jinja \
  -p "Analiza lo siguiente"
```

**Justificación**:
- `temp 0.5`: Respuestas precisas pero no robóticas
- `n-predict 768`: Respuestas completas pero concisas
- `top-p 0.9`: Vocabulario técnico preciso

---

### ✍️ **Asistente de Escritura Creativa**

```bash
llama-cli -m /var/lib/llama.cpp/Qwen3-0.6B-Q8_0.gguf \
  --rope-freq-base 1000000 \
  --ctx-size 8192 \
  --n-predict 2048 \
  --temp 0.9 \
  --top-p 0.98 \
  --top-k 50 \
  --min-p 0 \
  --presence-penalty 1.6 \
  --repeat-penalty 1.1 \
  --jinja \
  -p "Escribe una historia sobre"
```

**Justificación**:
- `temp 0.9`: Alta creatividad
- `top-k 50`: Amplio vocabulario creativo
- `n-predict 2048`: Textos largos
- `repeat-penalty 1.1`: Evitar frases repetitivas
- `presence-penalty 1.6`: Maximizar variedad

---

## 🚀 Comando Mínimo Funcional

```bash
llama-cli -m /var/lib/llama.cpp/Qwen3-0.6B-Q8_0.gguf \
  --rope-freq-base 1000000 --temp 0.6 --top-k 20 \
  --presence-penalty 1.5 --jinja -p "Tu prompt"
```

## 🔧 Ajustes para Casos Específicos

### Para desarrollo en Elixir
- Mantén `--temp 0.6` para código preciso
- Considera `--top-k 10` para respuestas más deterministas
- Aumenta `--n-predict` a 1024 para funciones complejas

### Si experimentas repeticiones
- Aumenta `--presence-penalty` gradualmente hasta 2.0
- Prueba con `--repeat-penalty 1.1` como alternativa
- Verifica que `--rope-freq-base` sea exactamente 1000000

### Para contextos largos (YaRN)
```bash
-c 131072 --rope-scaling yarn --rope-scale 4 --yarn-orig-ctx 32768
```
Nota: Requiere significativamente más RAM

### Para uso en servidor
```bash
./llama-server -m /var/lib/llama.cpp/Qwen3-0.6B-Q8_0.gguf \
  --rope-freq-base 1000000 --temp 0.6 --top-k 20 \
  --presence-penalty 1.5 --jinja --port 8080
```

