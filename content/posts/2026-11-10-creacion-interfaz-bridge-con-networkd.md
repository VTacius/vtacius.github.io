---
date: "2026-10-11T00:00:00Z"
tags:
- configuraciones
- systemd
title: Creación de interfaz bridge con systemd-networkd 
---

## Definición del bridge
Creamos la nueva interfaz con un archivo `.netdev`, siguiendo la convención, para una bridge de nombre `br0`, el nombre del archivo es `/etc/systemd/network/20-br0.netdev` con el siguiente contenido:

```systemd
[NetDev]
Name=br0
Kind=bridge
```

## Vinculando interfaz física al bridge
Añadimos una interfaz física al bridge con un archivo `.network`. En mi caso, reuso el archivo con el que había configurado la interfaz física antes, `/etc/systemd/network/20-phy.network`, considerando que necesito quitar la configuración IPv4 de esa interfaz
```systemd
[Match]
Name=enp0s13f0u1u2c2

[Network]
Bridge=br0
```

Aunque la mayoría de guías recomienda crear un archivo `.link`, la configuración funciona si él.

## Configuración IPv4 para el bridge
Configuramos el archivo `/etc/systemd/network/20-br0.network` de la siguiente manera:

```systemd
[Match]
Name=br0

[Network]
Address=10.125.66.10/24
Gateway=10.125.66.1
```

En este punto, reiniciar el servicio networkd activa la configuración anterior:
```bash
systemctl restart systemd-networkd.service
```
