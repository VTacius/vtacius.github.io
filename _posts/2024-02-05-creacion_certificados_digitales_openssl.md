---
title: Creación de certificados digitales con OpenSSL
tags:
    - configuraciones
---
## Creación sencilla
Creamos primero nuestro CA. El parametro `-sub` nos permite ahorrarnos el responder al cuestionario que usualmente aparece 

```bash
openssl genrsa -out ca.pem.key 4096
openssl req -new -x509 -nodes -days 3650 -subj "/C=SV/ST=El Salvador/L=San Salvador/O=sanidad/CN=DTIC" -key ca.pem.key -out ca.pem.crt
```

Ahora creamos el certificado propiamente a usar. Una de las claves es usar en `CN` el nombre del servicio a usar
```bash
openssl req -newkey rsa:2048 -nodes -subj "/C=SV/ST=El Salvador/L=San Salvador/O=sanidad/CN=www.sanidad.gob.sv" -keyout www.sanidad.gob.sv.pem.key -out www.sanidad.gob.sv.pem.csr
openssl x509 -req -days 3650 -in www.sanidad.gob.sv.pem.csr -out www.sanidad.gob.sv.pem.crt -CA ca.pem.crt -CAkey ca.pem.key
```

## Fuentes
* [Create Self-Signed Certificates and Keys with OpenSSL](https://mariadb.com/docs/server/security/data-in-transit-encryption/create-self-signed-certificates-keys-openssl/)
