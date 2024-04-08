---
title: Ideas para crear Laboratorio Virtualizado
tags:
    - configuraciones
---

Llega un momento en que vamos a necesitar crear un laboratorio virtualizado. En mi caso, la necesidad emergió de la necesidad de superar los problemas habituales de red.

Estas son un montón de notas sueltas sobre como he ido creando el mío, en algún momento supongo que pondré todo en orden.

## Instalación inicial
* Se configura el equipo físico con virtualización mediante KVM
* Se instala una máquina virtual

## Resolviendo `nl80211: Could not configure driver mode`
Usando el adaptador USB `Realtek RTL8192EU Wireless LAN 802.11n USB 2.0 Network Adapter` con `hostapd` resulta un problema:
```
nl80211: Could not configure driver mode
```
Esto es un problema con el driver empaquetado en Debian. La solución ya ha sido dada por el usuario [Mange](https://github.com/Mange/rtl8192eu-linux-driver), y solo tenemos que compilar su versión del driver:
```bash
apt update
apt install git linux-headers-generic build-essential dkms

# Eliminamos el driver si ya lo habíamos instalado. Lo más saludable es reiniciar el sistema después de ello
apt purge firmware-realtek 

git clone https://github.com/Mange/rtl8192eu-linux-driver
cd rtl8192eu-linux-driver
dkms add .
dkms install rtl8192eu/1.0

echo -e "8192eu\n\nloop" >> /etc/modules
update-grub
update-initramfs -u
reboot

```

## Configuración de interfaz inalámbrica como salida por defecto para el sistema
Resulta que el equipo físico en el que esta montado este laboratorio esta situado en un lugar sin conexión ethernet disponible, lo mejor ha sido configurarle una interfaz inalámbrica con un adaptador USB `Realtek RTL8192EU Wireless LAN 802.11n USB 2.0 Network Adapter`
* Adjunto la USB a la máquina virtual dada. No quise batallar tanto, así que lo hice en un equipo con interfaz gráfica mediante virt-manager
* En el equipo virtual, `Debian Bookworm` se instala el paquete `firmware-realtek`, teniendo configurado la rama `non-free-firmware` en los repositorios

La configuración para que el equipo pueda usar una red `WPA-PSK` y `WPA2-PSK` es la siguiente:
```bash
wpa_passphrase Prueba 'Prueba$123'
```

Especial atención al psk que se crea, porque es el que se usa en la configuración de `/etc/network/interfaces`:
```
...

allow-hotplug wlan0
iface wlan0 inet dhcp
        wpa-ssid myssid
        wpa-psk ccb290fd4fe6b22935cbae31449e050edd02ad44627b16ce0151668f5f53c01b

...
```

## Configuración de firewall mínimo
Aunque ahora existan formas más avanzadas de hacer una configuración como esta, usaremos la forma más básica posible:

* Creamos el script `/usr/local/sbin/firewall.sh`, con el siguiente contenido:

```bash
#!/bin/bash
iptables -t nat -X
iptables -t nat -Z
iptables -t nat -F
iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o wlx7898e81f45ef -j MASQUERADE
iptables -t nat -A POSTROUTING -s 10.10.20.0/24 -o wlx7898e81f45ef -j MASQUERADE

echo 0
```

Damos permisos de ejecución:
```bash
chmod +x /usr/local/sbin/firewall.sh
```

Y el fichero `/lib/systemd/system/firewall.service`, definición de servicio oneshot, con su contenido correspondiente:
```bash
[Unit]
Description=Configura firewall en el sistema
After=networking.service
Requires=networking.service

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/firewall.sh

[Install]
WantedBy=multi-user.target
```  

Habilitamos e iniciamos (Ejecuta una sola vez y después el servicio se muere) el servicio en un solo comando
```bash
systemctl enable --now firewall.service
```

## Fuentes
* [Mange/rtl8192eu-linux-driver ](https://github.com/Mange/rtl8192eu-linux-driver)
