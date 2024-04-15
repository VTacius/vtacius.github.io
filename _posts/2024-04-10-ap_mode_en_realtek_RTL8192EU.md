---
title: Resolviendo `Mode AP` en Realtek RTL8192EU
tags:
    - configuraciones
---

Si bien es posible configurar un adaptor de red con el chipset `Realtek Semiconductor Corp. RTL8192EU 802.11b/g/n WLAN Adapter` con el uso del driver `firmware-realtek`, no es posible ponerla en Modo AP, lo cual es un requisito para usarla con Hostapd. El error devuelto es ` nl80211: Could not configure driver mode`.

La solución ya ha sido dada por el usuario [Mange](https://github.com/Mange/rtl8192eu-linux-driver), y solo tenemos que compilar su versión del driver:
```bash
# Instalamos los paquetes necesarios
apt update
apt install git linux-headers-generic build-essential dkms

# Eliminamos el driver si ya lo habíamos instalado. Lo más saludable es reiniciar el sistema después de ello
apt purge firmware-realtek 

git clone https://github.com/Mange/rtl8192eu-linux-driver
cd rtl8192eu-linux-driver
dkms add .
dkms install rtl8192eu/1.0

echo -e "8192eu\nloop" >> /etc/modules
update-grub
update-initramfs -u
reboot
```
