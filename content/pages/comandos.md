---
title: Comandos útiles 
permalink: /comandos/
date: 2024-04-12T11:20:00+06:00
toc: true
---

## Tranferir ficheros entre dos equipos con `nc`
Pudiendo garantizar la privacidad de la red subyacente, y con la intención de que la transferencia sea *tan rápida como le sea posible*, puede usarse `nc`, `pv` (Solo para mostrar el avance) y `tar` (Sólo para empaquetar, si se comprime le metemo presión al CPU)  de la siguiente forma:

**Equipo origen**
```bash
cd /var/lib/libvirt/images/
tar cf - \*qcow2 | pv | nc -l -p 5555
```

**Equipo destino**
```bash
cd /var/lib/libvirt/images/
nc 10.10.100.10 5555  | pv | tar xf -
```

## Agregar ruta estática con `nmcli`
```bash
nmcli connection modify Conexión\ cableada\ 1 +ipv4.routes "10.10.100.0/24 10.168.247.251"
nmcli connection down Conexión\ cableada\ 1
nmcli connection up Conexión\ cableada\ 1
```

## Vincular USB a Host Virtual en KVM
Usamos el comando `lsusb -v`. Nos dará un resultado como el siguiente, en el cual buscamos `idVendor` e `idProduct`:

![lsbusb -v]({{site.url}}{{site.baseurl}}/assets/images/Screenshot_20240412_133733.png)

Creamos un fichero xml para configuración `adaptador_realtek.xml`:

```bash
cat adaptador_realtek.xml 
<hostdev mode='subsystem' type='usb' managed='yes'>
    <source>
        <vendor id='0x0bda'/>
        <product id='0x818b'/>
    </source>
</hostdev>
```

Y ejecutamos el comando propiamente dicho:
```bash
virsh attach-device network-gw-sanidad --file adaptador_realtek.xml --persistent
```
