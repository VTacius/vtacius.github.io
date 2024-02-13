---
title: Content Security Policy (CSP)
tags:
    - comentarios
    - configuraciones
---
## Contexto
Content Security Policy es una estándar de ciberseguridad que permite controlar el origen del contenido que se carga en una página web, con lo cual es posible mitigar muchos ataques basados en inyección de código o contenido; entre los cuales puede mencionarse XSS y clickjackin.

Su configuración puede realizarse mediante una etiqueta meta o mediante una cabecera HTTP, siendo este último método el más recomendado porque habilita todo el potencial de la herramienta

## Configuración

Básicamente, la forma más simple y completa de configurarlo es la siguiente:
```
Content-Security-Policy" "default-src 'self'; frame-ancestors 'none'; form-action 'none'
```

La directiva así configurada, hace que el sitio solo puede cargar contenido del mismo dominio (Excluye implícitamente subdominios). es `default-src` del que se configuran la mayoría de las demás directivas, excepto las otras dos que le acompañan: `frame-ancestor`, que vuelve obsoleta a la cabecera `X-Frame-Options` y `form-action` (TODO, una investigación más exhaustiva es necesaria)

Otras directivas que podrían necesitar un configuración diferente a la por defecto podrían ser:
* `font-src`: Define fuentes válidas para fuentes de letras que son cargadas mediante @font-face
* `connect-src`: Se aplica a `XMLHttpRequest`, `WebSocket`, `fetch()`, `<a ping>` y a `EventSource`. TODO: Es necesario una investigación más exhaustiva

### NGINX
```
add_header "Content-Security-Policy" "default-src 'self'; frame-ancestors 'none'; form-action 'none'";
```

## Resultado
Se ha configurado la cabecera anterior en un sitio `objetivo.sanidad.gob.sv` en nginx, en el cual hay un página `contenido.php` con lo mínimo:
![contenido.php]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-02-13_14-26-48.png)

Ahora puede verse como, de las dos imagenes, la segunda no se ha cargado, y en las herramientas de desarrollador, aparece un mensaje de bloqueo
![Resultado]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-02-13_15-01-07.png)

Además, no es posible cargarla desde otro dominio `colector.sanidad.gob.sv`, que tiene una página `fake.php` con el siguiente contenido mínimo:
![fake.php]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-02-13_15-05-36.png)

El mensaje de error ha variado un poco, pero es la misma idea:
![resultado]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-02-13_15-08-55.png)

## Fuentes
* [Content Security Policy (CSP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
* [Content Security Policy Reference](https://content-security-policy.com/)
* [Cross Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)
* [How XSS Payloads Work with Code Examples, and How to Prevent Them](https://www.hackerone.com/knowledge-center/how-xss-payloads-work-code-examples-preventing-them)
