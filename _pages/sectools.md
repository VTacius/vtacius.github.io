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

## Nuclei
### Instalación
Tienen su [página de releases](https://github.com/projectdiscovery/nuclei/releases), pero se puede instalar en Fedora 39 con las herramientas de `go`: 
```bash
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latestugo install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
```
### Opinión
Me pareció una buena adicción a las herramientas que uso. Básicamente, puede dejar fuera a `nikto`, avisando de algunas librerías completas. Especial mención a que supo detectar que el sitio es el Juice Shop
![Resultado de nuclei]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-03-14_10-36-52.png)

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
## RustScan
### Instalación
Tiene un [artículo entero](https://github.com/RustScan/RustScan/wiki/Installation-Guide), yo lo [construí desde cero](https://github.com/RustScan/RustScan/wiki/Installation-Guide#-building-it-yourself)
### Opinión
Se supone que usa nmap para operar, así que no le ví mayor avance a usarlo en lugar del mismo. No le veo utilidad, en realidad, supongo que es un trabajo en proceso que despúes verá frutos

