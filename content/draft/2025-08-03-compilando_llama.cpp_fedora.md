---
date: "2025-08-03T00:00:00Z"
draft: true
tags:
- configuraciones
- zsh
title: Compilando llama.cpp en Fedora 42
---

## Introducción
Este es un tema tan extenso que para guiarme he tenido que aprovechar mis vacaciones y mi nueva cuenta de Claude Code Pro

Para empezar, en fedora todo esta roto. Resulta que la versión de `CUDA` no soporta la versión actual de gcc en Fedora (Incluso existen guías sobre como [compilar el compilador](https://www.if-not-true-then-false.com/2023/fedora-build-gcc/). Ese es el nivel de desastre en este tema). Luego, parece que las mismas librerías tienen problemas realmente serios de código llamando a otras librerías...

Para no hacer largo el cuento (Acá esta [todo el cuento que necesitan](https://forum.level1techs.com/t/cuda-12-9-on-fedora-42-guide-including-getting-cuda-samples-running/230769)), la guía oficial de configuración dice que lo mejor es ¡[Hacerlo en un contenedor](https://github.com/ggml-org/llama.cpp/blob/1d72c841888b9450916bdd5a9b3274da380f5b36/docs/backend/CUDA-FEDORA.md)!

Por mi parte, intentaré otro enfoque: Usar `Vulkan`, con la adicional ventaja que podré usar la tarjeta de vídeo integrada en conjunto.

## Configuraciones previas
```bash
dnf install intel-gpu-tools vulkan-tools nvtop cmake gcc-c++ vulkan-headers vulkan-loader-devel glslc
```

## Procedimiento

Nos ubicaciones en un directorio
```bash
take ~/espacio/
```

Obtenemos el repositorio en ~[Release](https://github.com/ggml-org/llama.cpp/releases) adecuado con `-b`: 
```bash
git clone -b b6123 --depth 1 https://github.com/ggml-org/llama.cpp.git

cd llama.cpp
```

La compilación se resume así: Configuramos, compilamos e instalamos:
```bash
cmake -B build \
      -DCMAKE_INSTALL_PREFIX=/usr/local \
      -DCMAKE_BUILD_TYPE=Release \
      -DGGML_VULKAN=ON \
      -DGGML_NATIVE=ON \
      -DGGML_AVX2=ON \
      -DGGML_FMA=ON \
      -DGGML_F16C=ON \
      -DGGML_BMI2=ON \
      -DGGML_LTO=ON \
      -DGGML_CCACHE=ON \
      -DCMAKE_CXX_FLAGS="-march=native -mtune=native -O3"

cmake --build build --config Release -j 24

cd build
sudo make install
echo '/usr/local/lib64' | sudo tee /etc/ld.so.conf.d/llamacpp.conf
sudo ldconfig
```

Creamos un directorio con nuestros archivos `gguf`:
```bash 
sudo mkdir /var/lib/llama.cpp/
```

Y echamos a correr nuestro primer chat con:
```bash
GGML_VK_VISIBLE_DEVICES=0 llama-cli -m /var/lib/llama.cpp/Qwen3-0.6B-Q8_0.gguf -cnv -ngl 99 -c 4096 -t 8
```

Antes de hechar a correr, necesitamos configurar la variable `GGML_VK_VISIBLE_DEVICES=0,1` en `~/.zhrc` o `~/.bashrc`

```bash
GGML_VK_VISIBLE_DEVICES=0 llama-cli -m /var/lib/llama.cpp/Qwen3-0.6B-Q8_0.gguf -cnv -ngl 99 -c 4096 -t 8 --temp 0.7 --top-p 0.9 --repeat-penalty 1.1
```
