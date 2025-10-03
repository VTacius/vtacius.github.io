---
date: "2024-04-12T00:00:00Z"
tags:
- configuraciones
title: Configuración de OpenVSwitch en Debian Bookworm
---
Por ahora solo lo he usado como parte de una  configuración de virtualzación con KVM, y eso porque necesito configurar una misma red LAN en diferentes nodos de virtualización, es decir, una red hyperconvergente (O al menos algo que se le acerque al significado)
Teniendo configurados los repositorios, la instalación va de la siguiente manera:

```bash
apt install openvswitch-switch
```

## Bridge standalone
Permite crear un puente sin conexión alguna con interfaces físicas, según entiendo, esto permite crear una especie de red privada a usarse en el mismo servidor. Se usa `manual` para evitar ponerle una red

```bash
auto LINK_01
allow-ovs LINK_01
iface LINK_01 inet manual
    ovs_type OVSBridge
```

## Bridge con puerto físico
Permite un acceso directo a la red donde pertenece la interfaz física `enp1s0`. `INTERNET` puede ser cualquier otro nombre. Y si, se puede usar el `manual` para no tener que configurar IP

```bash
# The primary network interface
auto INTERNET
allow-ovs INTERNET
iface INTERNET inet static
    ovs_type OVSBridge
    ovs_ports enp1s0
        address 10.168.247.251/24
        gateway 10.168.247.1

allow-INTERNET enp1s0
iface enp1s0 inet manual
    ovs_bridge INTERNET
    ovs_type OVSPort

```

## Túnel (Una especie de red hyperconvergente)
Esta es la razón por la cual uso KVM. Con esta red, podemos tener la misma red LAN en dos nodos diferentes. El nombre `dmz_eru_02` debe ser único, y en mi caso me gusta que sea un poco referencial.

En un host `eru-01` (10.10.100.10):
```bash
##########
# DMZ
auto DMZ
allow-ovs DMZ
iface DMZ inet manual
    ovs_type OVSBridge
    ovs_ports dmz_eru_02

allow-DMZ dmz_eru_02
iface dmz_eru_02 inet manual
    ovs_bridge DMZ
    ovs_type OVSTunnel
    ovs_tunnel_type gre
    ovs_tunnel_options options:remote_ip=10.10.100.20 options:key=1
```

En el otro host, `eru-02` (10.10.100.20):
```bash
##########
# DMZ
auto DMZ
allow-ovs DMZ
iface DMZ inet manual
    ovs_type OVSBridge
    ovs_ports dmz_eru_01

allow-DMZ dmz_eru_01
iface dmz_eru_01 inet manual
    ovs_bridge DMZ
    ovs_type OVSTunnel
    ovs_tunnel_type gre
    ovs_tunnel_options options:remote_ip=10.10.100.10 options:key=1
```

## Recomendaciones: 
Los nombres de puente no pueden ser mayor a ¿16? caracteres. Es más, no me acuerdo donde este el límite, pero que sirva como referencia a quién lo necesite

`options:key={n}` debe ser diferente para cada una

## Fuentes
* [openvswitch-switch.README.Debian](https://github.com/openvswitch/ovs/blob/main/debian/openvswitch-switch.README.Debian)
