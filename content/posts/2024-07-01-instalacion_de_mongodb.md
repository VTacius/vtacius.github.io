---
date: "2024-07-01T00:00:00Z"
tags:
- configuraciones
title: Instalación de MongoDB en Debian Bookworm
---

## Procedimiento:
Como es usual, necesitamos obtener la llave del repositoio, configurar el repositorio, actualizar caché e instalar el paquete:

```bash
wget https://www.mongodb.org/static/pgp/server-7.0.asc -O - -q | gpg --dearmor -o /etc/apt/trusted.gpg.d/mongodb-org.gpg
echo "deb http://repo.mongodb.org/apt/debian bookworm/mongodb-org/7.0 main" > /etc/apt/sources.list.d/mongodb-org.list
apt update
apt install mongodb-org
```
