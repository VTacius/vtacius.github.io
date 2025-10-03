---
date: "2024-02-27T00:00:00Z"
tags:
- sso
- seguridad
- configuraciones
title: Introducción e instalación de Janssen
---

## Introducción
A ver, que esto puede ser un poco confuso:

En un principio, existía Gluu. En un punto, se vuelve *un poco* comercial. Ahora a eso la compañía (Que también se llama Gluu) le llama [Gluu 4](https://gluu.org/gluu-4/), que es opensource (Pero necesita suscripción) y seguirá funcionando hasta 2028.

Eso de *un poco* comercial siempre es motivo de disputa, tanto como para los desarrolladores/colaboradores como para los usuarios/clientes y usuarios/beneficiarios. Así que la compañía decide hacer bien las cosas, y deja que la Linux Foundation tome [el proyecto como opensource bajo el nombre de Janssens](https://www.linuxfoundation.org/press/press-release/the-janssen-project-takes-on-worlds-most-demanding-digital-trust-challenges-at-linux-foundation). Ahora la compañía vende [flex](https://gluu.org/flex/), que no es más que *la versión comercial del Linux Foundation Janssen Project*, según sus propias palabras.

Hay demasiado material para darnos la confianza de que tal proyecto siempre será opensource. La mejor lectura al respecto es [Ten Impacts of the Janssen Project on Gluu](https://gluu.org/gluu-articles/ten-impacts-of-the-janssen-project-on-gluu/), que de hecho explica mucho mejor lo acá escrito.

## Instalación
Pues nada, ahora vamos a lo que seguramente fue lo que te trajo acá: Una guía en español para instalar Janssen Project en una Máquina Virtual/Baremetal.

El sistema objetivo es un Debian 12 que corre sobre KVM. Tiene 150GB de disco, 2 vcpu y 4Gb, que parece ser la configuración mínima necesaria para pruebas.

Creamos un directorio para trabajar. Asignamos permisos y nos movemos a él:
```bash
mkdir /tmp/paquete
chown -R _apt /tmp/paquete
cd /tmp/paquete/
```
A continuación, buscamos un [lanzamiento apropiado](https://github.com/JanssenProject/jans/releases). Escojo el de Ubuntu 22:
```bash
wget https://github.com/JanssenProject/jans/releases/download/v1.0.22/jans_1.0.22.ubuntu22.04_amd64.deb
apt install --no-install-recommends ./jans_1.0.22.ubuntu22.04_amd64.deb
```

## Configuración inicial
Es bastante sencilla, pero antes hay que agregar el soporte para Debian 12. Luego, el siguiente comando empieza un asistente con un montón de preguntas que por lo pronto puedan dejarse con las respuestas que tiene por defecto.
```bash
cat package_list.json | jq '. += {"debian 12": ."debian 11"'} | tee package_list.json
python3 /opt/jans/jans-setup/setup.py
```

Para comprobar la instalación, hay un par de formas, todas involucran hacer peticiones a lo endpoint para ello habilitados para la aplicación. Por ejemplo:
```bash
curl -s -k https://$(hostname -f)/jans-auth/sys/health-check | jq
```

## Fuentes
* [jq documentation](https://devdocs.io/jq/)
* [Overview - Janssen Documentation](https://docs.jans.io/v1.0.22/admin/install/vm-install/)
