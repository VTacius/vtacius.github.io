---
date: "2025-06-05T00:00:00Z"
tags:
- configuraciones
- desarrollo
title: Configuración de Laravel para API Rest
---

Ya que en el sitio oficial de Lumen nos dicen directamente que *we recommend always beginning new projects with Laravel.*, supongo que habrá que aprender a usar Laravel a modo de micro-framework, sin ser micro-framework y sin querer realmente usarlo como micro-framework. Si Google te trajó acá, sé que me entenderás.

# Configuraciones previas
Trataremos de usar las mismas versiones de software en nuestro entorno de desarrollo y producción. En base de datos, mi objetivo será la versión por defecto de Postgres en Debian. Para ahorrarme la virtualización, usaré podman, que viene por defecto en Fedora 42.

La configuración irá de la siguiente forma:

## Base de datos
* Cómo usuario normal de toda la vida, creamos un volumen
```bash
podman volume create notas-data-db
```

* Luego, definimos el contenedor y lo iniciamos:
```bash
podman create --name notas-data-db -e POSTGRES_PASSWORD=password -p 5432:5432 -v notas-data-db:/var/lib/postgresql/data postgres:15-bookworm
podman start notas-data-db
```

* Si necesitamos revisar los logs, pues usamos el comando `logs`:
```bash
podman logs notas-data-db
```

## Laravel
* Creamos el proyecto 
```bash
composer create-project laravel/laravel --prefer-dist prueba-02
```

* Eliminamos los archivos que no vamos a usar:
```bash
cd prueba-02
rm -rf vite.config.js resources/{css,js,views} database/database.sqlite
```

* Apuntamos a la base de datos configurando el archivo `.env`:
```bash
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=laravel
DB_USERNAME=postgres
DB_PASSWORD=password
```

* Añadimos el soporte de lumen para trabajar con API. De hecho, en este paso se hará la migración a nuestra base de datos
```bash
php artisan install:api
```

De acá, ya podemos crear rutas en el archivo `routes/api.php`, y no tendrán verificación CSRF o sesión, con lo que se supone que nos vamos a ahorrar un par de ciclos de procesamiento

El último cambio queda al gusto. Yo quiero que la API responda en `/`, por lo que cambio el `apiPrefix` en `bootstrap/app.php`:
```diff
diff --git a/bootstrap/app.php b/bootstrap/app.php
index d654276..625d677 100644
--- a/bootstrap/app.php
+++ b/bootstrap/app.php
@@ -8,6 +8,7 @@
     ->withRouting(
         web: __DIR__.'/../routes/web.php',
         api: __DIR__.'/../routes/api.php',
+        apiPrefix: '/',
         commands: __DIR__.'/../routes/console.php',
         health: '/up',
     )
```

## Primera ruta
Solo muevo el contenido de `routes/web.php` en `routes/api.php`, cambiado el retorno a una string

## FrankenPHP

```bash 
curl https://frankenphp.dev/install.sh | sh
mv frankenphp /usr/local/bin/
```

Situados como estamos dentro del proyecto, basta correr el comando
```bash
frankenphp php-server -r public/
```

Y ahora ya podemos hacer un curl desde consola:
```bash
curl localhost
```

## Siguientes pasos
De acá solo sigue programar. Retornar un montón de JSON y esas cosas