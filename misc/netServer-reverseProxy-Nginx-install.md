# Instalación proxy inverso Nginx en un servidor de red

Esta es la solucion nombrada como `#netServer-reverseProxy-Nginx-install`.

## Contexto

Este es un tutorial de [Web3 - IPFS](../README.md) aplicable a cualquier servico, pero que usa como ejemplo la [configuración del contenedor de IPFS](../IPFS/ipfs-testing-public-and-desktop-node/publicNode-docker-install.md)

## Propósito

Estos son los pasos de configuración e instalación de un proxy inverso para poder publicar en un servidor púiblico aplicaciones Web.

## Solución

La solución es la instalación de [nginx](https://nginx.org/en/) como proxy inverso y adicionalmente [busybox](https://www.busybox.net/) como servidor simple de contenido HTML, todo esto dentro del contexto docker y `docker-compose`.

Ver las [Referencias](#referencias).

## Configuración

- En servidor VPS.
- En [Ubuntu 24.04 LTS](https://ubuntu.com/blog/tag/ubuntu-24-04-lts).

## Problemas conocidos que debes saber antes

### No olvidar actualizar e instalar los paquetes debian

```bash
sudo apt update
sudo apt upgrade -y
```

> Hacer casos a los avisos, no ignorarlos o luego todo será peor.

### Algunos comandos cuando hay problemas

Si tienes dudas y quieres forzar que se apliquen cambios en la configuración, puedes usar:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml up -d --force-recreate
```

Puedes probar en el contenedor de nginx del proxy inverso, si funciona correctamente la comunicación con el contenedor busybox de pruebas:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec reverse-proxy curl -s http://test-web:80/test/index.html
```

Puedes comprobar igualmente, que el contenedor de nginx del proxy inverso, está en la misma red que el contenedor busybox de pruebas:

```bash
docker inspect reverse-proxy | grep NetworkMode
docker inspect test-web | grep NetworkMode
```

> Por el hecho de que ambos contenedores estén en el mismo `docker-compose.yml` ya están en la misma red.

### Verificando la renovación de certificados con certbot

Verifica manualmente cuándo expiran los certificados:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec certbot certbot certificates
```

> Si faltan pocos días para la expiración y no se ha renovado, hay un problema.

Si quieres ver si Certbot intentó renovar certificados, usa:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec certbot certbot renew --dry-run
```

Si quieres forzar renovar el certificado Certbot, usa:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec certbot certbot renew --force-renewal
docker-compose -f /etc/appserver/docker/docker-compose.yml exec reverse-proxy nginx -s reload
```

## Pre-requisitos

- [Configuración inicial del servidor de red](./netServer-initial-configuration.md).
- [Instalación y configuración de docker](./netServer-docker-install-configuration.md).

## Pasos

### Agregar permisos firewall ufw

Agregar permisos http y https:

```bash
sudo ufw allow http
sudo ufw allow http
```

### Configuración servicios

Crea usuarios que serán usados por docker para nginx, certbot y busybox:

```bash
sudo useradd -M -s /usr/sbin/nologin docker-reverse-proxy
sudo useradd -M -s /usr/sbin/nologin docker-test-web
```

Crear la carpeta de configuración para el servicio `reverse-proxy`, asignar permisos, propietario y heredarlos:

```bash
mkdir -p /etc/appserver/nginx /etc/appserver/nginx/conf.d
```

Crear el archivo de configuración `/etc/appserver/nginx/nginx.conf` y darle permiso al usuario `docker-reverse-proxy`, incluido directorio `conf.d`:

```bash
touch /etc/appserver/nginx/nginx.conf
sudo setfacl -m u:docker-reverse-proxy:rX /etc/appserver/nginx /etc/appserver/nginx/conf.d
sudo setfacl -m u:docker-reverse-proxy:r /etc/appserver/nginx/nginx.conf
sudo setfacl -d -m u:docker-reverse-proxy:r /etc/appserver/nginx/conf.d
```

Crear el archivo `/etc/appserver/nginx/conf.d/default.conf`, que se usa como el sitio web por defecto del servidor,  darle permiso de escritura al usurio porque lo necesitará al iniciar:

```bash
touch /etc/appserver/nginx/conf.d/default.conf
sudo setfacl -m u:docker-reverse-proxy:rw /etc/appserver/nginx/conf.d/default.conf
```

Para el primer servicio de prueba con `busybox`, crear el archivo de configuración:

```bash
touch /etc/appserver/nginx/conf.d/test-web.conf
```

Crear carpeta de logs de certbot y nginx, asignar propietario y permisos:

```bash
sudo mkdir /var/log/nginx /var/log/certbot
sudo chown docker-reverse-proxy:infrastructure /var/log/nginx -R
sudo chown docker-reverse-proxy:infrastructure /var/log/certbot -R

sudo chmod u=rwx,g=rx,o=--- /var/log/nginx /var/log/certbot
sudo chmod g+s /var/log/nginx /var/log/certbot

sudo setfacl -d -m u::rwX /var/log/nginx /var/log/certbot          # Propietario: rwx
sudo setfacl -d -m g::rX /var/log/nginx /var/log/certbot           # Grupo: rX (navegar)
sudo setfacl -d -m o::--- /var/log/nginx /var/log/certbot          # Otros: sin permisos
```

Para exponer contenido estático de la web, crear carepta y permisos, para el contenido wwww:

```bash
mkdir /srv/www
sudo chown nobody:infrastructure /srv/www -R
sudo chmod u=rwx,g=rwx,o=--- /srv/www
sudo chmod g+s /srv/www
sudo setfacl -d -m u::rwx /srv/www     # Propietario: rwx
sudo setfacl -d -m g::rwx /srv/www     # Grupo: rwx
sudo setfacl -d -m o::--- /srv/www     # Otros: sin permisos
```

Además, busybox necesita que creemos el mismo nombre de directorio que la ruta indicada, esto quiere decir, si indicamos la URL dentro de nginx <http://test-web:80/test/>, para la ruta `test`, debemos crear el directorio `test`, es decir:

```bash
mkdir /srv/www/test
```

Dar permiso lectura y ejecución (navegar en capertas) al usuario del servicio `test-web` en toda la carpeta y sub-carpetas de forma heredada:

```bash
sudo setfacl -m u:docker-test-web:rX /srv/www
sudo setfacl -m u:docker-test-web:rX /srv/www/test
sudo setfacl -d -m u:docker-test-web:r /srv/www/test
```

Crear página simple de prueba para `busybox` en `/srv/www/test/index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Prueba nginx</title>
</head>
<body>
    <h1>Nginx funciona con BusyBox</h1>
</body>
</html>
```

### Configuración seguridad web TLS con let's encrypt

Para poder usar servicir el servicio seguro, debemos disponer de un certificado y para ello usaremos certbot y let's encrypt.

En este proposito surgen dos necesidades, acceder a una web normal, con un subdominio, como puede ser a la Web de pruebas <https://test.web3-101.open3diy.org> y el otro caso más complejo para al gateway de IPFS, con subdominio para `*.ipfs.web3-101-ipfs.open3diy.org` / `*.ipns.web3-101-ipds.open3diy.org`.

> Sobre IPFS, tenemos que aclarar, que lo que veremos a continuación son los pasos necesarios para la solución [`#web3-ipfs-101-publicNode-docker-install`](../web3-101-ipfs-testing-public-and-desktop-node/web3-ipfs-101-publicNode-docker-install.md), que están además en este tutorial a modo de ejemplo, para explicar cómo crear una configuración TLS.

Para el primer caso tenemos una resolución normal `webroot`, para el segundo utilizaremos el método `DNS-01 Challenge`, mediante el proveedor `cloudfare` usando certbot y su plugin.

> No tengo un apego especial a `cloudfare`, pero es el que se usa normalmente en IPFS y además no hay tantas opciones gratuitas, o por lo menos no las conozco, que ofrezcan tanta facilidades para un subdominio wildcard, pero si sabes otra opción, estaré encantado que me lo hagas saber.

Todos los dominios podrían resolverse por el método `DNS-01 Challenge`, pero voy a usar `webroot` para la URL de pruebas `https://test.web3-101.open3diy.org`, principalmente para no perder la perspectiva didactica de este tutorial y asi mostrar las dos opciones.

> Asi además vemos que se pueden usar las dos formas a la vez.

**Para configuración `webroot`**

> Es la que usaremos para resolver la URL `https://test.web3-101.open3diy.org`.

Asegurarte que has configurado el DNS para que resuelva el nombre del servidor, en este ejemplo, para mí dominio `open3diy.org` he agreado el subdominio `web3-101`. Hay muchos tutoriales en la web, puedes seguir este de [jesús en dongee.com](https://www.dongee.com/tutoriales/como-crear-un-subdominio-en-cloudflare).

Comprueba que es correcto el dominio:

```bash
nslookup web3-101.open3diy.org
```

- **salida:**

  ```plaintext
  Server:         127.0.0.53
  Address:        127.0.0.53#53

  Non-authoritative answer:
  Name:   web3-101.open3diy.org
  Address: 57.129.131.125
  ```

Crear la carpeta de configuración para `certbot` de desafios ACME, configuración y temporales; asignar permisos, propietario y heredarlos:

```bash
mkdir -p /etc/appserver/certbot/www/.well-known/acme-challenge /etc/appserver/certbot/conf /etc/appserver/certbot/letsencrypt-lib
```

> `/etc/appserver/certbot/letsencrypt-lib` son archivos temporales, lo montaremos como volumen mas adelante porque no vamos a darle permisos al contenedor como root y necesita tener un sitio donde escribir.
> `/etc/appserver/certbot/www/.well-known/acme-challenge` en principio, el servicio certbot es el que debería crear el subdirectorio `well-known/acme-challenge`, pero no se crea, así que lo creamos previamente.

Daremos permiso:

```bash
sudo chown docker-reverse-proxy:docker-reverse-proxy /etc/appserver/certbot -R
```

> ¿por qué no es propietario el grupo `infraestructure`? por seguridad, es contenido de certbot que nadie debería poder ver, a excepción de un usuario con permisos elevados.

**Para configuración `DNS-01 Challenge`**

Igualmente, asegurarte que has configurado el DNS para que resuelva el nombre del servidor, en este ejemplo, para mí dominio `open3diy.org` he agreado los subdominios `web3-101-ipfs` e `ipfs.web3-101-ipfs` e `ipns.web3-101-ipfs`. Hay muchos tutoriales en la web, puedes seguir este de [jesús en dongee.com](https://www.dongee.com/tutoriales/como-crear-un-subdominio-en-cloudflare).

> Si te preguntas porque he creado estos subdominios para IPFS, lo explico en [tutorial de IPFS](../web3-101-ipfs-testing-public-and-desktop-node/web3-ipfs-101-publicNode-docker-install.md#propósito)

Necesitas un token para el dominio segurizado, y en nuestro caso, para eso, puedes seguir este tutorial de [Burp.es - crear subdominio segurizado](https://burp.es/crear-subdominios-segurizados-con-cloudflare/). Luego para configurar el token seguir los pasos de:

Comprueba que es correcto el dominio:

```bash
nslookup hola.ipfs.web3-101.open3diy.org
```

- **salida:**

  ```plaintext
  Server:         127.0.0.53
  Address:        127.0.0.53#53

  Non-authoritative answer:
  Name:   hola.ipfs.web3-101.open3diy.org
  Address: 57.129.131.125
  ```

```bash
nslookup hola.ipns.web3-101.open3diy.org
```

- **salida:**

  ```plaintext
  Server:         127.0.0.53
  Address:        127.0.0.53#53

  Non-authoritative answer:
  Name:   hola.ipns.web3-101.open3diy.org
  Address: 57.129.131.125
  ```

Crear directorio de configuración:

```bash
mkdir -p /etc/appserver/certbot-cloudfare
```

Crear el archivo de configuración para el token:

```bash
touch /etc/appserver/certbot-cloudfare/cloudflare.ini
```

Editar el archivo e incluir el token copiado (reemplazar el valor `{token}`):

```plaintest
dns_cloudflare_api_token = {token}
```

Asegurar los permisos para el servicio de certbot:

```bash
chown docker-reverse-proxy:docker-reverse-proxy /etc/appserver/certbot-cloudfare -R
```

### Prueba de los permisos finales de los directorios creados

Comprueba los permisos finales de los directorios creados y sus permisos.

> Voy a omitir en la salida lo que es irrelevante para hacer foco en lo que tenemos que revisar.

```bash
getfacl /etc/appserver/nginx/
```

- **salida:**

  ```plaintext
  user:docker-reverse-proxy:r-x
  other::---
  ```

```bash
getfacl /etc/appserver/nginx/nginx.conf
```

- **salida:**

  ```plaintext
  user:docker-reverse-proxy:r--
  other::---
  ```

```bash
getfacl /etc/appserver/nginx/conf.d/test-web.conf
```

- **salida:**

  ```plaintext
  userdocker-reverse-proxy:r--
  other::---
  ```

```bash
sudo ls -all /etc/appserver/certbot/www /etc/appserver/certbot/conf
```

- **salida:**

  ```plaintext
  /etc/appserver/certbot/conf:
  total 8
  drwxr-s---+ 2 docker-certbot docker-certbot 4096 Feb 12 20:57 .
  drwxr-s---+ 4 docker-certbot docker-certbot 4096 Feb 12 20:57 ..

  /etc/appserver/certbot/www:
  total 8
  drwxr-s---+ 2 docker-certbot docker-certbot 4096 Feb 12 20:57 .
  drwxr-s---+ 4 docker-certbot docker-certbot 4096 Feb 12 20:57 ..
  ```

```bash
getfacl /srv/www/test
```

- **salida:**

  ```plaintext
  user:docker-test-webr-x
  default:other::---
  ```

```bash
getfacl /srv/www/test/index.html 
```

- **salida:**

  ```plaintext
  user:docker-test-webr--
  other::---
   ```

### Crear imagen personalizada para el usuario

Como vamos a iniciar el contenedor con un usuario concreto, este no tendrá permisos dentro de la imagen, así que tenemos que crear una imagen personalizada con estos pasos:

Obtener el id del usuario `docker-reverse-proxy` que ejecutará la imagen:

```bash
id docker-reverse-proxy
```

- **salida:**

  ```plaintext
  uid=1015(docker-reverse-proxy) gid=1018(docker-reverse-proxy) groups=1018(docker-reverse-proxy)
  ```

> En este ejemplo, son los valores 1015:1018.

Crear en `/etc/appserver/docker` directorio `docker-nginx` y archico `Dockerfile` con contenido, teniendo en cuenta que en nuestro ejemplo el id de usuario es 1015 y el de grupo 1018, que obtenimos con anterioridad:

```Dockerfile
FROM nginxinc/nginx-unprivileged:latest

# Cambiar temporalmente a root para configurar permisos
USER root

# Asegurar que el usuario 1013:1015 tenga permisos sobre /var/cache/nginx
RUN chown -R 1015:1018 /var/cache/nginx && \
    chmod -R 755 /var/cache/nginx

# Volver al usuario "nginx" para no afectar el inicio del servidor
USER nginx
```

> Recuerda que 1015 y 1018 son id de este ejemplo, pon los que corresponda.

Construir la imagen:

```bash
docker build -f /etc/appserver/docker/docker-nginx/Dockerfile -t custom-docker-nginx /etc/appserver/docker/docker-nginx
```

### Crear configuración global de nginx

Es configuración que elegimos para todas las aplicaciones.

Inicialmente solo definimos el log, pero posteriormente se puede aplicar cualquier configuración que consideremos global al servidor de red.

Edita `/etc/appserver/nginx/nginx.conf` con el contenido:

```nginx
error_log /var/log/nginx/nginx.log;
# Posibles niveles de log (descomentar según necesidad)
# error_log /var/log/nginx/error.log crit;
# error_log /var/log/nginx/error.log error;
# error_log /var/log/nginx/error.log warn;
# error_log /var/log/nginx/error.log notice;
# error_log /var/log/nginx/error.log info;
# error_log /var/log/nginx/error.log debug;

pid /tmp/nginx.pid;  # Evita el error de permisos en /var/run/nginx.pid

events {
    worker_connections 1024;
}

http {
    access_log /var/log/nginx/access.log;
}
```

En `/var/log/nginx` se crearan los archivos de log `error.log`, `nginx.log` y `access.log`.

- `access.log`: tiene los accesos.
- `error.log`: son errores del servicio en la inicialización. Si hay errores de configuración, estarán aquí, pero no verás errores o log habitual del servicio.
- `nginx.log`: los logs habituales del servicio.

### Crear configuración en `docker-compose`

Obtener los id del usuario de `docker-reverse-proxy`, `docker-test-web` para configurar en los servicios:

```bash
id docker-reverse-proxy
id docker-test-web
```

- **salida:**

  ```plaintext
  uid=1015(docker-reverse-proxy) gid=1018(docker-reverse-proxy) groups=1018(docker-reverse-proxy)
  uid=1016(docker-test-web) gid=1019(docker-test-web) groups=1019(docker-test-web)
  ```

> En este ejemplo, son los valores 1014:1017, 1013:1017 y 1009:1011.

Edita en `/etc/appserver/docker/docker-compose.yml` la siguiente configuración:

```yaml
services:
  certbot:
    image: certbot/dns-cloudflare
    user: "1015:1018"
    entrypoint: /bin/sh -c "while :; do 
      certbot renew --quiet --webroot -w /var/www/certbot \
      --dns-cloudflare --dns-cloudflare-credentials /cloudflare.ini;
      sleep 12h; done"
    volumes:
      - /etc/appserver/certbot/www:/var/www/certbot/:rw
      - /etc/appserver/certbot/conf/:/etc/letsencrypt/:rw
      - /etc/appserver/certbot/letsencrypt-lib/:/var/lib/letsencrypt/:rw
      - /var/log/certbot:/var/log/letsencrypt:rw
      - /etc/appserver/certbot-cloudfare/cloudflare.ini:/cloudflare.ini:ro
    restart: unless-stopped
  reverse-proxy:
    image: custom-docker-nginx
    user: "1015:1018"
    container_name: reverse-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /etc/appserver/nginx:/etc/nginx:rw
      - /etc/appserver/certbot/www:/var/www/certbot:ro
      - /etc/appserver/certbot/conf:/etc/letsencrypt:ro
      - /var/log/nginx:/var/log/nginx:rw
    restart: unless-stopped
  test-web:
    image: busybox:latest
    user: "1016:1019"
    container_name: test-web
    command: httpd -f -p 80
    volumes:
      - /srv/www/:/www:ro
    working_dir: /www
    restart: unless-stopped
```

Explicacion de los servicios:

- `certbot`: Servicio para renovar certificados con Certbot y DNS Cloudflare.
  - `image: certbot/dns-cloudflare`: usa la imgen de certbot con el plugin de cloudfare.
  - `user: "1015:1018`: Inicia el contenedor con el usuario especifico creado en configuración. Se debe indicar los identificadores.
  - `entrypoint: /bin/sh -c "while erc..`: Ejecuta un bucle que intenta renovar certificados TLS cada 12h.
  - `restart: unless-stopped`: Significa que el contenedor se reiniciará automáticamente si se detiene inesperadamente, excepto si lo detienes manualmente.
- `reverse-proxy`: Proxy inverso basado en Nginx.
  - `image: custom-docker-nginx`: usa la imagen creada con anterioriad con nombre `custom-docker-nginx`.
  - `ports: 80 y 443`: Expone el puerto 80 para el desafio `webroot`de let's encrypt y el puerto 443 para la comunicación https.
- `test-web`: Servidor web mínimo con busybox.
  - `image: busybox:latest`: Última imagen de busybox.
  - `command: httpd -f -p 80`: Sirve el servicio simple, en segundo plano, en el puerto 80.
  - `working_dir: /www`: Sirve archivos de la ruta `/srv/www/` del host, en el contenedor en `/www`.

### Iniciar cambios

Verificar la configuración inicial:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml config
```

> Veremos la salida si no hay fallos de sintaxis, pero sino, veremos un error que nos avise.

Ejecutar sin detener servicios en ejecución:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml up -d
```

Verificar procesos en ejecución:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml ps
```

- **salida:**

  ```plaintext
  NAME               IMAGE                                COMMAND                  SERVICE         CREATED         STATUS         PORTS
  docker-certbot-1   certbot/certbot:latest               "/bin/sh -c 'while :…"   certbot         5 minutes ago   Up 5 minutes   80/tcp, 443/tcp
  reverse-proxy      custom-docker-nginx                  "/docker-entrypoint.…"   reverse-proxy   5 minutes ago   Up 5 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp, 8080/tcp
  test-web           busybox:latest                       "httpd -f -p 80"         test-web        5 minutes ago   Up 5 minutes   
  ```

Revisar logs en docker de nginx:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml logs reverse-proxy
```

- **salida:**

  ```plaintext
  reverse-proxy  | /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
  reverse-proxy  | /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
  reverse-proxy  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
  reverse-proxy  | 10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
  reverse-proxy  | 10-listen-on-ipv6-by-default.sh: info: /etc/nginx/conf.d/default.conf differs from the packaged version
  reverse-proxy  | /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
  reverse-proxy  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
  reverse-proxy  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
  reverse-proxy  | /docker-entrypoint.sh: Configuration complete; ready for start up
  ```

> Los logs también están en `/var/log/nginx/nginx.log`.

Revisar logs en docker de certbot:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml logs certbot
```

> Ninguna salida debes ver si todo está bien.

Revisar los logs propios de certbot:

```bash
less +G /var/log/certbot/letsencrypt.log
```

> Ningún mensaje de error debes ver.

Revisar logs en docker de busybox:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml logs test-web
```

> Ninguna salida veremos si no hay errores.

Verificar que la configuración es válida en el contenedo:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec reverse-proxy nginx -t
```

- **salida:**

  ```plaintext
  nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
  nginx: configuration file /etc/nginx/nginx.conf test is successful
  ```

### Configuración de Nginx para validación de Let's Encrypt método `webroot`

En `/etc/appserver/nginx/conf.d/default.conf`, configuramos Nginx para permitir que Certbot resuelva los desafíos `webroot`. Debemos incluir cada dominio o subdominio que necesitemos validar.

Inicialmente, para nuestro dominio de pruebas `web3-101.open3diy.org`, agregamos:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name test.web3-101.open3diy.org;

    # Para leer archivos de desafio
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 444; # cierra la conexión sin responder, evitando registros innecesarios
    }
}
```

Verificar que la configuración es válida en el contenedo:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec reverse-proxy nginx -t
```

- **salida:**

  ```plaintext
  nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
  nginx: configuration file /etc/nginx/nginx.conf test is successful
  ```

Recarga configuracion nginx sin reiniciar el contenedor:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec reverse-proxy nginx -s reload
```

Revisar logs:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml logs reverse-proxy
```

> Misma salida anterior con `reverse-proxy  | /docker-entrypoint.sh: Configuration complete; ready for start up`. Asegurarse que no hay errores, quizás te has dejado configuración en un archivo `.conf` de nginx que tiene configurado `ssl_certificate` que depende precisamente de este paso previo, en ese caso, simplemente borrala y ya la pondrás luego cuando ya existe los archivos `.pem`.

Verificar que se podria guardar el desafio, primero creando un archivo de prueba:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec certbot sh -c "echo 'test' > /var/www/certbot/.well-known/acme-challenge/testfile"
```

Verificar que sería resolver el desafio:

```bash
curl -I http://test.web3-101.open3diy.org/.well-known/acme-challenge/testfile
```

- **salida:**

  ```plaintext
  HTTP/1.1 200 OK
  Etc..
  ```

Ejecutar certbot como `webroot` para el sitio web:
> Esto solo lo haremos una vez por cada dominio o subdominio que se tenga que resolver, el resto de veces se renovará antes de la caducidad del certificado.

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec certbot certbot certonly --webroot -w /var/www/certbot -d test.web3-101.open3diy.org --email demovoidgan@gmail.com --agree-tos --non-interactive --force-renewal --debug
```

> `exec certbot` es ejecutar el contenedor con nombre `certbot`, luego ejecutamos el comando `certonly --webroot ...`.
> En este ejemplo `web3-101.open3diy.org` es el dominio propio, con el email relacionado `demovoidgan@gmail.com`. Por favor, revisa esto para tú caso...

- **salida:**

  ```plaintext
  Saving debug log to /var/log/letsencrypt/letsencrypt.log
  Account registered.
  Requesting a certificate for web3-101.open3diy.org

  Successfully received certificate.
  Certificate is saved at: /etc/letsencrypt/live/web3-101.open3diy.org/fullchain.pem
  Key is saved at:         /etc/letsencrypt/live/web3-101.open3diy.org/privkey.pem
  This certificate expires on 2025-05-16.
  These files will be updated when the certificate renews.
  ```

Revisar logs:

```bash
less -G /var/log/certbot/letsencrypt.log
```

Esto genera los certificados en `/etc/letsencrypt/live/web3-101.open3diy.org/`, revisarlo:

```bash
sudo ls -all /etc/appserver/certbot/conf/live/web3-101.open3diy.org
```

### Configuración de Nginx par IPFS para validación de Let's Encrypt método `DNS-01 Challenge`

Como otro ejemplo, tenemos los dominios de IPFS que usaremos más adelante...

Para generar los cerificados del sitio, ejecutar certbot como con el plugin de cloudfare para el método `DNS-01 Challenge` para los dominios `ipfs.web3-101-ipfs.open3diy.org`, `*.ipfs.web3-101-ipfs.open3diy.org` y `*.ipns.web3-101-ipfs.open3diy.org`:
> Esto solo lo haremos la primera vez, luego se renovará para la caducidad del certificado.

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec certbot certbot certonly --dns-cloudflare --dns-cloudflare-credentials /cloudflare.ini --dns-cloudflare-propagation-seconds 30 -d "web3-101-ipfs.open3diy.org" -d "ipfs.web3-101-ipfs.open3diy.org" -d "*.ipfs.web3-101-ipfs.open3diy.org" -d "*.ipns.web3-101-ipfs.open3diy.org" --email demovoidgan@gmail.com --agree-tos --non-interactive --force-renewal --debug
```

> En este ejemplo son los dominios de nuestro ejemplo, con el email relacionado `demovoidgan@gmail.com`. Por favor, revisa esto para tú caso...

- **salida:**

  ```plaintextSaving debug log to /var/log/letsencrypt/letsencrypt.log
  Requesting a certificate for ipfs.web3-101.open3diy.org and *.ipfs.web3-101.open3diy.org
  Waiting 30 seconds for DNS changes to propagate

  Successfully received certificate.
  Certificate is saved at: /etc/letsencrypt/live/ipfs.web3-101.open3diy.org/fullchain.pem
  Key is saved at:         /etc/letsencrypt/live/ipfs.web3-101.open3diy.org/privkey.pem
  This certificate expires on 2025-05-20.
  These files will be updated when the certificate renews.
  ```

Revisar logs:

```bash
less -G /var/log/certbot/letsencrypt.log
```

Esto genera los certificados en `/etc/letsencrypt/live/web3-101.open3diy.org/`, revisarlo:

```bash
sudo ls -all /etc/appserver/certbot/conf/live/ipfs.web3-101.open3diy.org
```

### Crear configuración de la aplicación de pruebas `test-web`

La configuración de cada aplicación, está en a `/etc/appserver/nginx/conf.d` y es una forma de organizar mejor la configuración de cada servicio.

Crearemos en primer lugar la configuración de la app de prueba `test-web` de `busybox` que lo  veremos en la URL `https://web3-101.open3diy.org/test`.

Crear el contenido de `/etc/appserver/nginx/conf.d/test-web.conf`:

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name test.web3-101.open3diy.org;

    ssl_certificate /etc/letsencrypt/live/test.web3-101.open3diy.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/test.web3-101.open3diy.org/privkey.pem;

    location = / {
        proxy_pass http://test-web:80/test/;
        index index.html;
    }
}
```

> Recuerda revisar `test.web3-101.open3diy.org` indicando el dominio que corresponda a tú caso.

Verificar que la configuración es válida en el contenedo:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec reverse-proxy nginx -t
```

- **salida:**

  ```plaintext
  nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
  nginx: configuration file /etc/nginx/nginx.conf test is successful
  ```

Recarga configuracion nginx sin reiniciar el contenedor:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec reverse-proxy nginx -s reload
```

Revisar logs:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml logs reverse-proxy
```

> Misma salida anterior con `reverse-proxy  | /docker-entrypoint.sh: Configuration complete; ready for start up`.

Acceder a la URL de prueba en `https://web3-101.open3diy.org/test` y ver el contenido.

Revisar los logs de acceso:

```bash
less +G /var/log/nginx/access.log
```

### Crear configuración para IPFS

Lo veremos en su debido tutorial de [`#web3-ipfs-101-publicNode-docker-install`](../web3-101-ipfs-testing-public-and-desktop-node/web3-ipfs-101-publicNode-docker-install.md).

## Referencias

- [nginx](https://nginx.org/en/).
- [jesús Tutoriales Dongee](https://www.dongee.com/tutoriales/como-crear-un-subdominio-en-cloudflare/) - Cómo crear un subdominio en cloudfare.
- [BUSYBOX](https://www.busybox.net/).
- `chatgpt.com`.
