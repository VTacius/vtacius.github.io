---
title: TODO's
permalink: /todos/
date: 2024-02-29T15:38:52+06:00
toc: true
---

## Almacenamiento
* https://docs.ipfs.tech/ - Es usado en proyectos muy interesantes

## Autenticación
* https://w3c.github.io/webauthn/

## Big Data
* https://www.janes.com/ (Sigo sin tener una idea clara de que hace, pero se me hace demasiado interesante)
* https://linkis.apache.org/ Middleware para integración. Uno de esos proyectos de Apache que prometen mucho, pero que solo parecen ser un desarrollo específico de alguna compañía que lo publica para ver si alguien cae y le ayuda con el código
* https://inlong.apache.org/ Framework para integración. Más o menos de la misma idea que linkis.

## Correo electrónico
* https://james.apache.org/ - Por si alguna vez tenemos que volver a tener el servicio de correo electrónico en casa

## Desarrollo
* [Parse Platform](https://parseplatform.org/) - *The Complete Application Stack* - Básicamente, una especie de algo en NodeJS que después sirve para crear aplicaciones en otros lenguajes, pero se oye más interesante que solo eso, que ya es mucho

## Herramientas
* [Ferret](https://www.montferret.dev/) - *a web scraping system aiming to simplify data extraction from the web*
* [Basic Crypto Operations Using step](https://smallstep.com/docs/step-cli/basic-crypto-operations/)

## Cloud Storage
Deberías probarlas, sobre todo los que están relacionados con AWS S3
* https://openebs.io/
* https://github.com/seaweedfs/seaweedfs
* https://cubefs.io/
* https://moosefs.com/
* https://longhorn.io/docs/1.5.1/deploy/important-notes/
* https://juicefs.com/en/
* https://www.gluster.org/
* https://docs.openstack.org/cinder/latest/
* https://www.beegfs.io/c

## DB
* https://dbos-project.github.io/ - Esta cosa es demasiado interesante como para no echarle una ojeada

## DNS
* https://www.knot-resolver.cz/

## Frontend
* https://docs.cotter.app/
* https://alpinejs.dev/ Esta cosa se me hace demasiado simpática como para no usarla en alguna cosa
* https://vega.github.io/vega-lite/ Hace gráficas algo bonitas

## Lecturas
* [HTTP/3: el pasado, el presente y el futuro](https://blog.cloudflare.com/http3-the-past-present-and-future-es-es/)
* [The Hilarious Handbook of Kubernetes Resources: From Pods to Policies](https://medium.com/@s.atmaramani/the-hilarious-handbook-of-kubernetes-resources-from-pods-to-policies-5b8b0a583493)
* [A Crash Course in OAuth2.1](https://jc1175.medium.com/a-crash-course-in-oauth2-1-1b882fb50fc6) - TODO: Agregar a Enlaces para entender OAuth
* [A Crash Course In OAuth2.0](https://jc1175.medium.com/a-crash-course-in-oauth-c4c00e418db0) - TODO: Agregar a Enlaces para entender OAuth
* [OAuth grant types](https://portswigger.net/web-security/oauth/grant-types) - Volvé a repasarlo
* [OAuth 2.0 authentication vulnerabilities](https://portswigger.net/web-security/oauth)

## Seguridad informática

### Apps vulnerablemente diseñadas
* [WebGoat: A deliberately insecure Web Application](https://github.com/WebGoat/WebGoat)
* [Xtreme Vulnerable Web Application (XVWA)](https://github.com/s4n7h0/xvwa)
* [OWASP Mutillidae II](https://github.com/webpwnized/mutillidae)
* [OWASP Security Shepherd](https://github.com/OWASP/SecurityShepherd)
* [owasp-bricks](https://github.com/itsos4devs/owasp-bricks)
* [Server-Side Request Forgery (SSRF) vulnerable Lab](https://github.com/incredibleindishell/SSRF_Vulnerable_Lab)

### Herramientas
* https://github.com/noir-cr/noir - Noir is an attack surface detector that identifies endpoints by static analysis.
* [SX](https://github.com/v-byte-cpu/sx) - Un scanner de red, otro supuesto a reemplazar nmap, con más de dos años sin actualizarse
* [Arjun](https://github.com/s0md3v/Arjun) - Lo empecé a usar un poco, pero no ví nada por defecto
* [dalfox](https://github.com/hahwul/dalfox) - A powerful open-source XSS scanner and utility focused on automation
* [swisskyrepo/SSRFmap](https://github.com/swisskyrepo/SSRFmap) - Hecha en python
* [dreadlocked/SSRFmap](https://github.com/dreadlocked/SSRFmap)- Hecha en ruby, seis años sin actualizaciones
* [SSRFUZZ](https://github.com/ryandamour/ssrfuzz) - Hecha en Go
* https://github.com/spyboy-productions/CloakQuest3r - Dice ser capaz de *"Descubrir la verdadera IP de sitios web tras CloudFlare"*
* https://github.com/httptoolkit/frida-interception-and-unpinning - Mitmt para android, me llama mucho la atención para probar
* [sysdream/ligolo](https://github.com/sysdream/ligolo) - Reverse Tunneling made easy for pentesters, by pentesters https://sysdream.com/

### Contenido
* [Black Hat Rust](https://github.com/skerkour/black-hat-rust) - Si vuelves al camino de Rust, podría ser por acá

### Ingeniería Social
* https://github.com/trustedsec/social-engineer-toolkit - SET, la verdad se ve bastante completo

## Proyectos en curso 
* Revisar [What Is HTTP Request Smuggling?What Is HTTP Request Smuggling?](https://brightsec.com/blog/http-request-smuggling-hrs/), hasta entenderlo y poder crear un POC, luego, buscar/crear herramientas para incluirlo en la pila de pruebas que realizas
* Trabajar en una guía para usar con mitmproxy con los plugins que usaste la vez anterior
* Estoy creando un proyecto, en PHP, que sirva como modelo de buenas prácticas en la creación de API's
* Estoy creando un proyecto, en JS, que sirva como objetivo de pruebas de seguridad y herramientas
* Por ahora, el único proyecto es https://github.com/vtacius/viuda, que ya puede hacer combinaciones posibles de rutas y las peticiona con varios verbos. Es un gran ahorro de cara a examinar API's de forma exhaustiva
* Debo empezar otro proyecto para trabajar sobre los JWT, por ahora la primera fase del proyecto en JS será para trabajar con este otro
* Tenés que crear una herramienta que pueda sintetizar esto: https://jwt.io/ y http://jwtbuilder.jamiekurtz.com/

### OAuth
* [Run your own OAuth2 Server](https://www.ory.sh/run-oauth2-server-open-source-api-security/) - Usa [Ory/Hydra](https://github.com/ory/hydra) y parece ser una buena opción
* [Configuring Keycloak](https://www.keycloak.org/server/configuration) - ¿Es mi imaginación o el proceso se ha simplificado bastante?
* [Configuring NGINX for OAuth/OpenID Connect SSO with Keycloak/Red Hat SSO](https://developers.redhat.com/blog/2018/10/08/configuring-nginx-keycloak-oauth-oidc#)

## SSO
* https://github.com/casdoor/casdoor - Se declara un IAM completo, al mismo nivel que KeyCloak pues
* https://www.hello.coop/#about - Este es más bien un servicio en línea, pero me gustaría revisar algunas ideas
* https://github.com/oauth2-proxy/oauth2-proxy - Es un proxy reverso, pero permite autenticación con OpenID providers

## TPM
* Sigo creyendo que esto solo tiene sentido como un segundo factor de autenticación. Literalmente, es como dejar un auto con la llave adentro
* https://www.monperrus.net/martin/7-things-to-do-with-your-TPM-on-Linux - Sigo sin decidir si lo de acceder por SSH a un equipo remoto es una muy buena o muy mala idea, pero sé que no hay punto intermedio
* https://next.redhat.com/2021/05/13/what-can-you-do-with-a-tpm/

