---
date: "2024-06-06T00:00:00Z"
tags:
- configuraciones
title: Configurando HP ZBook Fury 16 G9 Mobile Workstation PC
toc: false
---

## Configurando HP ZBook Fury 16 G9 Mobile Workstation PC

Esta laptop tiene dos salidas de vídeo: **Intel Corporation Alder Lake-HX GT1 [UHD Graphics 770]** y **NVIDIA Corporation GA107GLM [RTX A1000 Laptop GPU]**. La mejor forma de ser configurada es con ambas activas, con el driver privativo para NVIDIA y con `modeset=1` configurado en el kernel mediante `/etc/default/grub`, y haciendo `grub2-mkconfig`.

Al final me quedé con XFCE en Fedora 40, la mejor combinación posible para este equipo y para mi actual carga de trabajo

### Fuentes:
* [Nvidia Optimus](https://nouveau.freedesktop.org/Optimus.html)
* Como siempre, la guía de [Archlinux](https://wiki.archlinux.org/title/NVIDIA) es una buena fuente de información

### Notas importantes:
* `bbswitch` es un proyecto que parece muerto, pero cuando funcionó, fue la cosa más genial del mundo ([How to turn off nvidia (hybrid graphics)](https://askubuntu.com/questions/551424/how-to-turn-off-nvidia-hybrid-graphics))
* La usual solución de desactivar la NVIDIA [desintalando el driver](https://askubuntu.com/questions/757177/disable-nvidia-optimus-graphics-card) me parece la cosa más cruel del mundo
