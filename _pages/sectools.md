---
title: SecTools
permalink: /sectools/
date: 2024-03-13T14:23:52+06:00
toc: true
gallery:
  - url: /assets/images/Captura de pantalla_2024-03-14_13-33-52.png
    image_path: /assets/images/Captura de pantalla_2024-03-14_13-33-52.png
    alt: "Pantalla Inicial"
    title: "Pantalla Inicial"
  - url: /assets/images/Captura de pantalla_2024-03-14_13-34-25.png
    image_path: /assets/images/Captura de pantalla_2024-03-14_13-34-25.png
    alt: "Se abre el navegador escogido, ya configurado con el proxy"
    title: "Se abre el navegador escogido, ya configurado con el proxy"
  - url: /assets/images/Captura de pantalla_2024-03-14_13-35-27.png
    image_path: /assets/images/Captura de pantalla_2024-03-14_13-35-27.png
    alt: "Esto es parte de lo que se ve cuando se usa con juice-shop"
    title: "Esto es parte de lo que se ve cuando se usa con juice-shop"
  - url: /assets/images/Captura de pantalla_2024-03-14_13-35-52.png
    image_path: /assets/images/Captura de pantalla_2024-03-14_13-35-52.png
    alt: "Este es parte del detalle para una petición cualquiera"
    title: "Este es parte del detalle para una petición cualquiera"

---

Pequeña recopilación de algunas herramientas probadas. Va por orden de recomendación

## katana 
### Instalación
Es un paquete `go` con instalación simple:
```bash
go install github.com/projectdiscovery/katana/cmd/katana@latest
``` 
Se recomienda instalar Google Chrome, pero es un adicional en su uso, no lo predeterminado
### Opinión
Es una de las mejores adicciones a mi conjunto de herramientas. Con juice-shop, fue capaz de encontrar casi 79 url, hasta donde me puse a revisar, todas ellas tienen sentido. Como referencia, `ZAP` encontró 39 (Encontró otras URL, pero esa de assets). Así que no solo es un reemplazo, sino una mejora. En comparativa, parece solo no encontró algunos URL relacionadas con la capacidades de socket de la aplicación, y un endpoint hijo.
![Resultado de katana]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-20_09-05-59.png)

## Nuclei
### Instalación
Tienen su [página de releases](https://github.com/projectdiscovery/nuclei/releases), pero se puede instalar en Fedora 39 con las herramientas de `go`: 
```bash
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latestugo install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
```
### Opinión
Me pareció una buena adicción a las herramientas que uso. Básicamente, puede dejar fuera a `nikto`, avisando de algunas librerías completas. Especial mención a que supo detectar que el sitio es el Juice Shop
![Resultado de nuclei]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-14_10-36-52.png)

## fallparams
### Instalación
No falta su página de [releases](https://github.com/ImAyrix/fallparams/releases), pero al estar escrito en Go, su instalación es más sencilla:
```bash
go install github.com/ImAyrix/fallparams@latest
```
### Opinión
Lo que hace es crear una lista de parametros que la aplicación usa. En juice-shop, listo 848, pero de algunos parecen ser falsos positivos. Como sea, empezar a fuzzear con una lista así parece mucha mejor idea que usar una lista como `trickest/wordlists`, al menos en un primer intento. 
```bash
fallparams -url http://juice-shop.sanidad.gob.sv/#/ --crawl --output parametros.lst
```
![Resultado de fallparams]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-20_11-06-36.png)

## trickest/wordlists
### Instalación
Con git, yo use `--depth 1` para reducir un poco el tamaño del repositorio. Lo dejé a propósito en $HOME, así será más sencillo de referir
```bash
git clone --depth 1 https://github.com/trickest/wordlists
```
### Opinión
A la fecha de escribir la reseña (marzo/2024), parece que el proyecto esta activo y recibe actualizaciones. Especial mención a que tiene un directorio `technologies` que en principio ayudaría a encontrar algunas CMS (Con las rutas por defecto) con solo un fuzzer. Acá esta una muestra parcial del contenido:
![Resultado de nuclei]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-18_13-21-34.png)

## gobuster
### Instalación:
Aparte de su [página de releases](https://github.com/OJ/gobuster/releases), tienen [imagen docker](https://github.com/OJ/gobuster/pkgs/container/gobuster). Yo usé golang
```bash
go install github.com/OJ/gobuster/v3@latest
```
### Opinión
La mayoría de modos tienen la opción `-t` para limitar el número de threads usados.

`gobuster` tiene más formas de enumeración disponibles que las basadas en consultas HTTP. Por ejemplo, `dns`, para encontrar subdominios:
```bash
gobuster dns -d sanidad.gob.sv -w ~/wordlists/inventory/subdomains.txt
```
![gobuster dns]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-18_14-48-48.png)

Para encontrar sudirectorios, esta vez con HTTP:
```bash
gobuster dir -u http://juice-shop.sanidad.gob.sv -w ~/wordlists/robots/top-100.txt -b 200
```
![gobuster dns]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-18_15-01-50.png)

No entendí porque fue necesario poner `-b 200`, pero de no usarlo la aplicación termina con la segunda petición hecha con un mensaje `Error: the server returns a status code that matches the provided options for non existing urls. http://juice-shop.sanidad.gob.sv/ae1b1c19-5bc4-4c47-b640-4f4fa8367376 => 200 (Length: 3748). To continue please exclude the status code or the length`

Nota: En este caso, la aplicación `juice-shop` retorna un 200 falso para todas las peticiones, y son aquellas rutas que realmente existen las que marcan el error 500. Funciona, pero no como se esperaba

Y tiene el modo FUZZ propiamente, en general, para buscar posibles parametros:
```bash
gobuster fuzz -u http://juice-shop.sanidad.gob.sv?FUZZ=test -w ~/wordlists/inventory/parameters.txt
```
![gobuster fuzz]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-18_15-15-11.png)

Estas herramientas suelen ser tan útiles como la listas que se usen. Por si mismas, la aplicación es increíble. Me dejó más sorprendido de lo que `ffuf` me dejó

## jwt-hack
### Instalación
Pues el método que ya se esta volviendo común con todas estas herramientas:
```bash
go install github.com/hahwul/jwt-hack@latest
```
### Opinión
No es precisamente perfecta, pero está bastante cerca a lo que andaba buscando. Puede decodificar, craquear en base a listas, y sobre todo, ya hace algunos payloads interesantes para trabajar con ellos:
![gobuster fuzz]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-20_15-24-41.png)

## ffuf
### Instalación
Aunque tiene su [página de releases](https://github.com/ffuf/ffuf/releases), gracias a que esta escrito en go, podemos instalar fácilmente con las herramientas:
```bash
go install github.com/ffuf/ffuf/v2@latest
```
### Opinión
Es una herramienta bastante completa en su tipo. Es posible configurar proxy (`-x`), cambiar el verbo HTTP (`-X`) y enviar contenido POST con `-d`. Agregando que tenemos `-H` (`"Content-Type: application/json"`) para agregar cabeceras personalizadas, básicamente tenemos un cliente HTTP completo

En mi caso, use las opciones `-rate` (Configurado a 0 por defecto, es decir, sin límite) y `-t` (threads) para evitar que mi pequeño servidor de prueba colapsara.

Por defecto, `FUZZ` es el parametro que será cambiado desde la URL base.

Para buscar, por ejemplo, subdominios:
```bash
ffuf -rate 20 -t 20 -c -w $HOME/wordlists/inventory/levels/level1.txt -u http://FUZZ.sanidad.gob.sv/
```
![Resultado de ffuf]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-18_14-11-24.png)

## jaeles
### Instalación
Aunque esta escrito en go, la instalacion con `go install` me falló (Marzo/2024) y tuve que buscar [los releases](https://github.com/jaeles-project/jaeles/releases). A la fecha, todos son Beta:
```bash
unzip /home/alortiz/Descargas/jaeles-beta-v0.17.1-linux.zip -d /home/alortiz/bin/
jaeles config init
```
### Opinión
Es un desarrollo aún muy temprano, pero promete mucho. En sí, el proyecto siempre será tan fuerte como lo sean sus signatures. Por defecto, se guardan en `~/.jaeles/base-signatures/`, y de allí se pueden escoger la signature a usar.
Lo mejor es se podrían crear signatures propias, lo cual abre muchas posibilidades
```bash
jaeles scan -s 'nodejs' -s 'nginx' -u http://juice-shop.sanidad.gob.sv --debug
```
Use la opción `--debug` porque en realidad no emite NADA, y es un poco chocante
![Resultado de jaeles]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-20_10-15-47.png)

## feroxbuster
### Instalación
Como aplicación escrita en Rust y publicada en cargo, puede usarse cargo, pero esta vez lo hice con un script que ellos disponen para tal cosa:
```bash
curl -sL https://raw.githubusercontent.com/epi052/feroxbuster/master/install-nix.sh | bash -s $HOME/bin
```

### Opinión
Otro fuzzer, lo veo un poco al menos nivel de `ffuf`, aunque tiene cosas como `--extract-links`...
![Resultado de feroxbuster]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-19_14-15-45.png)

Para revisar subdirectorios:
```bash
ffuf -rate 20 -t 20 -c -w $HOME/wordlists/robots/top-100.txt -u http://juice-shop.sanidad.gob.sv/FUZZ
```
Nota: Resulta que juice-shop devuelve un falso error 200 en casi todos, excepto en algunas rutas que si existen pero que requieren más parametros. Básicamente, acá funciona al revés de como se supone que la herramienta funciona, pero al final funciona y eso ya es importante

Para revisar posibles parametros:
```bash
ffuf -rate 20 -c -w $HOME/wordlists/inventory/parameters.txt -u http://juice-shop.sanidad.gob.sv?FUZZ=valor
```

## HTTP Toolkit 
### Instalación
Tiene una [página de descarga](https://httptoolkit.com/docs/getting-started/installing/) con instrucciones detalladas. Para Fedora 39, hay que [descargar un zip](https://httptoolkit.com/download/linux-standalone/) y luego descomprimirlo en un lugar apropiado (Como root):
```bash
mkdir /opt/httptoolkit/
unzip /home/alortiz/Descargas/HttpToolkit-linux-x64-1.14.10.zip -d /opt/httptoolkit/
ln -s /opt/httptoolkit/httptoolkit /usr/local/bin/
```
Habría que abrirlo desde consola, o hacerle el entrada en el menú
### Opinión
{% include gallery caption="Uso de HTTP Toolkit" %}

Me parece que es solo una interfaz gráfica bonita, al menos en el modo gratuito, la mayoría de la funcionalidad de encuentra en [mitmproxy]({{site.url}}{{site.baseurl}}/uso_basico_mitmproxy/), y el uso más avanzado, que yo mismo pensé que tenía, se encuentra en Burp. Sin embargo, la facilidad de uso es impresionante, sobre todo para [Android](https://httptoolkit.com/blog/android-14-install-system-ca-certificate/)

## cariddi
### Instalación
Punto para el hecho que esta en `snap`, pero aprovechando que lo que tengo más a la mano es `golang`:
```bash
go install -v github.com/edoardottt/cariddi/cmd/cariddi@latest
```
### Opinión
Me parece que dió más información de `hakrawler`, y tiene más opciones interesantes, como almacenar las peticiones HTTP `-sr`
![Resultado de cariddi]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-18_10-53-33.png)

## hakrawler 
### Instalación
Otra herramienta hecha en go:
```bash
go install github.com/hakluke/hakrawler@latest
```
### Opinión
Es una herramienta bastante simple debería ser usada en conjunto a otras herramientas del creador, [hakluke](https://github.com/hakluke), que tiene contenido bastante intersante y digno de revisar. La herramienta es demasiado simple, pero efectiva, sobre todo a la hora de precisamente hacer Crawling
![Resultado de hakrawler]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-18_08-50-21.png)

## Osmedus
### Instalación
Escrito en Go, pero falló la instalación por otros medios que no sean su script oficial de instalación:
```bash
bash <(curl -fsSL https://raw.githubusercontent.com/osmedeus/osmedeus-base/master/install.sh)
```
### Opinión
Al final, internamente usa muchas más herramientas, así que quizá valga la pena revisar esas herramientas. Por alguna extraña razón, no es capaz de crear un buen reporte, quizá sea necesario darle otra oportunidad, pero algo me dice que no es algo que este buscando
```bash
osmedeus scan -f vuln -t juice-shop.sanidad.gob.sv
```
![Resultado de osmedeus]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-20_11-49-24.png)

## RustScan
### Instalación
Tiene un [artículo entero](https://github.com/RustScan/RustScan/wiki/Installation-Guide), yo lo [construí desde cero](https://github.com/RustScan/RustScan/wiki/Installation-Guide#-building-it-yourself)
### Opinión
Se supone que usa nmap para operar, así que no le ví mayor avance a usarlo en lugar del mismo. No le veo utilidad, en realidad, supongo que es un trabajo en proceso que despúes verá frutos
