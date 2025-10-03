---
date: "2024-02-09T00:00:00Z"
tags:
- herramientas
title: Uso básico de mitmproxy
---
## Contexto
`mitmproxy` es una herramienta para crear un proxy MITM. Allí donde el código no este disponible o sea muy confuso, por ejemplo, puede usarse para examinar el uso que tal aplicación haga de una API.

## Instalación
Lo estoy usando desde Debian, y esta en los repositorios, así que bastó con
```bash
apt install mitmproxy
```

## Uso
Para correrlo, basta con usar el comando `mitmproxy`, pero de esta forma, la instancia sólo escucha en `localhost`, lo cual puede cambiarse usando la opción `--listen-host`. 

También puede especificarse el puerto con `-p`, lo cual es útil si va a usarse varias instancias. Por defecto, la instancia corre en el puerto `8080`
```bash
mitmproxy --listen-host 0.0.0.0 -p 8080
```

Al ejecutarse, es probable que se presente una ventana vacía. Después de haber configurado el navegador para que lo use como proxy y se configure el certificado, la ventana se verá como la siguiente:

![Pantalla inicial de mitmproxy]({{ site.url }}{{ site.baseurl }}/assets/images/Captura de pantalla_2024-02-09_15-33-17.png)

Al hacer click (La aplicación es así de intuitiva) en alguna de las peticiones listadas, podemos ver más información de ella:

![Detalles de petición]({{ site.url }}{{ site.baseurl }}/assets/images/Captura de pantalla_2024-02-09_15-33-53.png)
