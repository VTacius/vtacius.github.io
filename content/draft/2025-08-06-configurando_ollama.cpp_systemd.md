---
date: "2025-08-06T00:00:00Z"
draft: true
tags:
- configuraciones
- llm
title: Configurando ollama.cpp para correr con systemd
---
### Preparación del Entorno

Primero, creamos la estructura de directorios siguiendo las mejores prácticas de FHS (Filesystem Hierarchy Standard):

```bash
# Crear usuario dedicado sin shell de login
sudo useradd -r -s /bin/false -m -d /var/lib/llama.cpp llama

# Estructura de directorios
# sudo mkdir -p /var/lib/models # Lo creamos en el post anterior
sudo mkdir -p /etc/llama.cpp/{models.d,configs.d}
sudo mkdir -p /run/llama.cpp

# Permisos adecuados
sudo chown -R llama:llama /var/lib/llama.cpp
sudo chown -R llama:llama /run/llama.cpp
sudo chmod 755 /etc/llama.cpp

# Siempre verificar que los modelos tengan los permisos adecuados 
sudo chown -R llama:llama /var/lib/llama.cpp/models
```

### Configuración del Servicio Principal

#### 1. Archivo de configuración del modelo

Crear `/etc/llama.cpp/models.d/qwen3-0.6b.conf`:

```bash
# Configuración para Qwen3-0.6B-Q8_0
MODEL_PATH="/var/lib/llama.cpp/models/Qwen3-0.6B-Q8_0.gguf"
MODEL_ALIAS="qwen3-small"
PORT="8080"

# Parámetros críticos del modelo
ROPE_FREQ_BASE="1000000"
CTX_SIZE="4096"
N_PREDICT="512"

# Parámetros de sampling
TEMPERATURE="0.6"
TOP_P="0.95"
TOP_K="20"
MIN_P="0"
PRESENCE_PENALTY="1.5"

# Configuración de rendimiento
THREADS="4"
BATCH_SIZE="512"
N_GPU_LAYERS="0"  # Cambiar si tienes GPU

# Configuración del servidor
HOST="127.0.0.1"
PARALLEL="2"  # Número de slots paralelos
CONT_BATCHING="1"  # Habilitar continuous batching

# Flags adicionales
USE_MLOCK="true"
USE_JINJA="true"
NO_CONTEXT_SHIFT="true"

# Logging
LOG_LEVEL="info"
LOG_FORMAT="json"
```

#### 2. Servicio systemd principal

Crear `/usr/lib/systemd/system/llama-server.service`:

```ini
[Unit]
Description=llama.cpp Server - AI Model Inference Service
Documentation=https://github.com/ggml-org/llama.cpp
After=network.target
Wants=network-online.target

[Service]
Type=normal
User=llama
Group=llama

# Archivo de configuración
EnvironmentFile=/etc/llama.cpp/models.d/qwen3-0.6b.conf

# Comando de ejecución
ExecStart=/usr/local/bin/llama-server \
    --model ${MODEL_PATH} \
    --alias ${MODEL_ALIAS} \
    --host ${HOST} \
    --port ${PORT} \
    --ctx-size ${CTX_SIZE} \
    --n-predict ${N_PREDICT} \
    --temp ${TEMPERATURE} \
    --top-p ${TOP_P} \
    --top-k ${TOP_K} \
    --min-p ${MIN_P} \
    --presence-penalty ${PRESENCE_PENALTY} \
    --threads ${THREADS} \
    --batch-size ${BATCH_SIZE} \
    --n-gpu-layers ${N_GPU_LAYERS} \
    --rope-freq-base ${ROPE_FREQ_BASE} \
    --parallel ${PARALLEL} \
    --cont-batching \
    --log-format ${LOG_FORMAT} \
    --metrics
    --no-display-prompt

# Reinicio y límites
Restart=on-failure
RestartSec=10
TimeoutStartSec=300
TimeoutStopSec=30

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=llama-server

# Variables de entorno adicionales
Environment="LLAMA_LOG_PREFIX=1"
Environment="LLAMA_LOG_TIMESTAMPS=1"

[Install]
WantedBy=multi-user.target
```

#### 4. Target Unit para Gestión Grupal

Crear `/etc/systemd/system/llama-server.target`:

```ini
[Unit]
Description=llama.cpp Server Instances Target
Documentation=https://github.com/ggml-org/llama.cpp
After=network-online.target

[Install]
WantedBy=multi-user.target
```

### Uso del Servicio

#### Comandos Básicos

```bash
sudo systemctl enable llama-server.target
sudo systemctl enable --now llama-server@qwen3-0.6b.service
```


```bash
# Agregar un nuevo modelo para código
sudo llama-add-model -n codegen -m /path/to/codellama.gguf -p 8081 -t 0.2

# Iniciar la nueva instancia
sudo systemctl enable --now llama-server@codegen

# Verificar todas las instancias
sudo llama-health-check

# Iniciar todos los servicios llama
sudo systemctl start llama-server.target

# Detener todos los servicios llama
sudo systemctl stop llama-server.target
```

#### Ejemplos de Uso con curl

```bash
# Health check
curl http://localhost:8080/health

# Obtener información del modelo
curl http://localhost:8080/v1/models

# Generar completación
curl -X POST http://localhost:8080/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Hola, ¿cómo estás?",
    "max_tokens": 100,
    "temperature": 0.7
  }'

# Chat completion (compatible con OpenAI API)
curl -X POST http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3-small",
    "messages": [
      {"role": "system", "content": "Eres un asistente útil."},
      {"role": "user", "content": "Explica qué es Elixir"}
    ]
  }'
```

### Troubleshooting de Servicios

#### Problema: El servicio no inicia

```bash
# Verificar logs detallados
sudo journalctl -xeu llama-server

# Verificar permisos
ls -la /var/lib/llama.cpp/models/
ls -la /usr/local/bin/llama-server

# Probar comando manualmente
sudo -u llama /usr/local/bin/llama-server --model /var/lib/llama.cpp/models/Qwen3-0.6B-Q8_0.gguf --port 8080
```

#### Problema: Error de memoria

```bash
# Ajustar límites en el archivo de configuración
sudo nano /etc/llama.cpp/models.d/qwen3-0.6b.conf
# Reducir: CTX_SIZE, BATCH_SIZE, PARALLEL

# O ajustar límites del sistema
sudo nano /etc/systemd/system/llama-server.service
# Aumentar: MemoryMax=4G
```


## TODO:
```
Faltan estas opciones al ejecutar el comando
${USE_MLOCK:+--mlock} \
${USE_JINJA:+--jinja} \
${NO_CONTEXT_SHIFT:+--no-context-shift} \
```

```
Estas configuraciones podrían ser importantes para systemd
# Límites de recursos
LimitNOFILE=65536
LimitNPROC=512
MemoryMax=2G
MemorySwapMax=0
CPUQuota=200%

# Seguridad y aislamiento
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/llama.cpp /var/log/llama.cpp /run/llama.cpp
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectControlGroups=true
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX
RestrictNamespaces=true
LockPersonality=true
MemoryDenyWriteExecute=true
RestrictRealtime=true
RestrictSUIDSGID=true
RemoveIPC=true

# Health check
ExecStartPost=/bin/bash -c 'until curl -s http://${HOST}:${PORT}/health >/dev/null 2>&1; do sleep 1; done'
```

Para facilitar la gestión de múltiples modelos, crear `/etc/systemd/system/llama-server@.service`:

