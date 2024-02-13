---
title: Instalación de certificados CA
comments: true
toc: true
tags:
    - configuraciones
---
## Contexto
Estamos trabajando sobre un sitio que usa un certificado digital auto-firmado, hay muchas razones válidas por las que dicha configuración es válida. Al hacer una petición al sitio desde consola con `curl`, nos da un error `curl: (60) SSL certificate problem: self-signed certificate in certificate chain`:

![Error en curl]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-02-12_13-34-01.png)

Si bien podemos omitir el error con la opción `-k`
![Bypass error en curl]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-02-12_13-34-26.png)

Lo cierto es que podríamos requerir que realmente usar el certificado. Es decir, reconocer a la Autoridad Certificadora auto-firmada. Entre las razones es que necesitemos usar un software de terceros al que no podemos configurar que omita el error, o porque realmente necesitamos usar el certificado en todas su funcionalidad.

## Distribuciones Linux
En principio, tenemos que copiar el certificado CA a una ubicación determinada y correr el comando específicio para integrarlo:

### Debian 11+
La ruta es `/usr/local/share/ca-certificates` y el comando es `update-ca-certificates`
![Agregando certificado CA en Debian 11+]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-02-12_13-40-49.png)

### Fedora 38+
La ruta es `/usr/share/pki/ca-trust-source/anchors/` y el comando `update-ca-trust`

![Agregando certificado CA en Fedora 38+]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-02-12_14-14-26.png)

## Dispositivos móviles emulados
Para referencia, instalé `adb` con DNF en Fedora 38, e instalé `Android Studio` desde `Flatpak`

### Android 14-
Preparamos el certificado:
![Preparando el certificado]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-02-13_08-49-31.png)
La forma más sencilla es iniciar el dispositivo desde consola con la opción `-writable-system`. Además, al momento de crear el dispositivo, debemos usar una imagen que tenga "API" en su nombre. Ni siquiera es necesario activar el modo desarrollado en el dispositivo con estos pasos
```bash
export ANDROID_AVD_HOME=~/.var/app/com.google.AndroidStudio/config/.android/avd/
~/Android/Sdk/emulator/emulator -avd Pixel_7_Pro_API_32 -writable-system
```

Abrimos otra consola y ejecutamos lo siguiente:
```bash
adb root
adb disable-verity
adb reboot
adb root
adb remount
adb push ~/certs/55274267.0 /system/etc/security/cacerts/
```

![Agregando certificado CA en Android]({{site.url}}{{site.baseurl}}/assets/images/Captura de pantalla_2024-02-13_08-47-02.png)

## Fuentes
* [UPDATE-CA-CERTIFICATES(8)](https://manpages.debian.org/buster/ca-certificates/update-ca-certificates.8.en.html)
* [Using Shared System Certificates](https://docs.fedoraproject.org/en-US/quick-docs/using-shared-system-certificates/)
* [Adding a Certificate to Android System Trust Store](https://medium.com/hackers-secrets/adding-a-certificate-to-android-system-trust-store-ae8ca3519a85)
* [Cannot push to android emulator because of Read only file system](https://stackoverflow.com/a/70415820/2608212)
* [adb remount fails - mount: 'system' not in /proc/mounts](https://stackoverflow.com/a/62687021/2608212)
