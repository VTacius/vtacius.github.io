---
title: Configurado un gateway virtual sencillo para laboratorio virtualizado
tags:
    - configuraciones
---

Esta configuración es la mínima necesaria para tener un gateway disponible a laboratorio virtual. Su verdadera utilidad se presenta a la hora de crear esquemas de red un poco más complejos. Nos ahorra los inconvenientes de tener que configurar switches, y contiene todos en un par de nodos físicos.

A través de todo el ejemplo, estamos usando `wlx7898e81f45ef` como nombre de la interfaz en el sistema

## Instalación de paquetes
```bash
apt install --no-install-suggests dnsmasq hostapd iw lshw
```

### Interfaz
Una vez instalada con el driver adecuado, se configura de la siguiente forma en `/etc/network/interfaces`:
```
auto wlx7898e81f45ef
iface wlx7898e81f45ef inet static
    address 10.10.101.1/24
```

Y reiniciamos la red
```bash
systemctl restart networking.service
```

### dnsmasq
La siguiente parece ser una configuración mínima para `Hostapd`. 
* Pueden desactivarse las opciones de `log-*` si no se requiere observarlos.
* Se usa `except-interface` y `no-dhcp-interface` para no prestar servicios a la red normal, lo cual es algo deseable para un laboratorio.
* Se prefiere el uso de un DNS externo, pero hemos señalado el uso de un interno para un dominio en específico

```conf
# Never forward plain names (without a dot or domain part)
domain-needed
# Never forward addresses in the non-routed address spaces.
bogus-priv

# Uncomment this to filter useless windows-originated DNS requests
filterwin2k

# Configure dnsmasq to don't read /etc/resolv.conf 
no-resolv

# Name servers 
server=8.8.8.8
server=8.8.4.4
server=/interno.com/10.10.20.20
server=/interno.com/10.10.20.21

# Specify which interface _not_ to listen on
except-interface=enp1s0
# Configure dnsmasq to provide only DNS service on an interface,
no-dhcp-interface=enp1s0

# Don't want dnsmasq to read /etc/hosts
no-hosts

# Set this to have a domain automatically added to simple names
expand-hosts

# Set the domain for dnsmasq. this is optional
domain=interno.com

# DHCP range where the netmask is given
dhcp-range= set:wlan, 10.10.101.25, 10.10.101.30, 255.255.255.0, 12h

# Specify an option which will only be sent to the "wlan" network
dhcp-option = tag:wlan, option:router, 10.10.101.1 

# Log each DNS query as it passes through dnsmasq.
log-queries

# Log lots of extra information about DHCP transactions.
log-dhcp

# This fixes a security hole. see CERT Vulnerability VU#598349
dhcp-name-match=set:wpad-ignore,wpad
dhcp-ignore-names=tag:wpad-ignore
```

Aplicamos la configuración:
```bash
systemctl status dnsmasq.service
```

### Hostapd
Configuramos en `/etc/default/hostapd` el archivo de configuración a usar:
```bash
sed -i -E 's|#(DAEMON_CONF=")|\1/etc/hostapd/hostapd.conf|g' /etc/default/hostapd
```

Una configuración mínima luce así en el archivo de configuración:
```bash
cat << CONF > /etc/hostapd/hostapd.conf
interface=wlx7898e81f45ef
ssid=SERVICIOS
country_code=SV
hw_mode=g

# 0 por defecto, con lo cual el buscaría el mejor canal disponible, pero hay drivers que no soportan dicha búsqueda
channel=1 

# La idea será usar únicamente 802.11b/g/n with WPA2-PSK/WPA2-PSK-256 y CCMP
wpa=2
wpa_passphrase=P4ssw0rd
wpa_key_mgmt=WPA-PSK WPA-PSK-SHA256
wpa_pairwise=CCMP
rsn_pairwise=CCMP

# Activamos 80211n, es decir, la n que se agrega a g en g+n. Resulta que probé, y sí, parece haber una mejora en throughput, aunque por otro lado la señal parece disminuir cuando se encuentra otras en el ambiente
ieee80211n=1
wmm_enabled=1

logger_syslog=-1
logger_syslog_level=2
logger_stdout=-1
logger_stdout_level=2

ctrl_interface=/var/run/hostapd

CONF
```

Aplicamos la configuración:
```bash
systemctl unmask hostapd.service
systemctl enable --now hostapd.service
```

### IPTABLES
La configuración es tan sencilla como agregar un script así:
```bash
cat <<MAFI> /usr/local/sbin/firewall.sh 
#!/bin/bash
iptables -t nat -X
iptables -t nat -Z
iptables -t nat -F
iptables -t nat -A POSTROUTING -s 10.10.101.0/24 -o enp1s0 -j MASQUERADE

echo 0
MAFI
```
Configuramos permisos de ejecución:
```bash
chmod u+x /usr/local/sbin/firewall.sh
```

Configuramos el servicios mediante `systemd`

```bash
cat <<MAFI>/lib/systemd/system/firewall.service
[Unit]
Description=Configura firewall en el sistema
After=networking.service
Requires=networking.service

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/firewall.sh

[Install]
WantedBy=multi-user.target
MAFI

```

Lo activamos:

```bash
systemctl enable --now firewall.service
```

Y configuramos el reenvío de paquetes en el sistema, y lo aplicamos:

```bash
sed -i -E 's/#(net.ipv4.ip_forward=1)/\1/g' /etc/sysctl.conf
sysctl -p 
```
