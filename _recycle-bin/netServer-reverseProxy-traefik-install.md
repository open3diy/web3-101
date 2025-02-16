# Instalación proxy inverso traefik en un servidor de red

Esta es la solucion nombrada como `#netServer-reverseProxy-traefik-install`.

## Contexto

Este es un tutorial de [Web3 - IPFS](../README.md).

## Propósito

Estos son los pasos de configuración e instalación de un proxy inverso para poder publicar en un servidor púiblico aplicaciones Web.

Se descarta la solución porque no tiene funciones para enrutar para IPFS.

En IPFS el valor del CID puede viajar como subdominio, no en la ruta, traefik no trae funciones al respecto :/

## Solución

La solución es la instalación de [Traefic](https://github.com/traefik/traefik) como proxy inverso y adicionalmente [busybox](https://www.busybox.net/) como servidor simple de contenido HTML, todo esto dentro del contexto docker y `docker-compose`.
Ver las [Referencias](#referencias).

## Configuración

- En servidor VPS.
- Con [docker](https://www.docker.com/) y [`docker-compose`](https://docs.docker.com/compose/).
- En [Ubuntu 24.04 LTS](https://ubuntu.com/blog/tag/ubuntu-24-04-lts).

## Problemas conocidos que debes saber antes

### No olvidar actualizar e instalar los paquetes debian

```bash
sudo apt update
sudo apt upgrade -y
```

> Hacer casos a los avisos, no ignorarlos o luego todo será peor.

### Algunos comandos cuando hay problemas...

Si tienes dudas y quieres forzar que se apliquen cambios en la configuración, puedes usar:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml up -d --force-recreate
```

Puedes probar en el contenedor de traefik del proxy inverso, si funciona correctamente la comunicación con el contenedor busybox de pruebas:

```bash
docker exec -it reverse-proxy wget -qO- http://test-web:80
```

Puedes comprobar igualmente, que el contenedor de traefik del proxy inverso, está en la misma red que el contenedor busybox de pruebas:

```bash
docker inspect reverse-proxy | grep NetworkMode
docker inspect test-web | grep NetworkMode
```

> Por el hecho de que ambos contenedores estén en el mismo `docker-compose.yml` ya están en la misma red.

Al reiniciar el contenedor `docker-compose restart` ves en `/var/log/traefik/traefik.log` este error:

```plaintest
2025-01-30T18:00:34Z ERR error="accept tcp [::]:443: use of closed network connection" entryPointName=websecure
2025-01-30T18:00:34Z ERR Error while starting server error="accept tcp [::]:443: use of closed network connection" entryPointName=websecure
2025-01-30T18:00:34Z ERR error="accept tcp [::]:80: use of closed network connection" entryPointName=web
2025-01-30T18:00:34Z ERR Error while starting server error="accept tcp [::]:80: use of closed network connection" entryPointName=web
```

Parece un error sin sentido de traefik, no sé si lo corregiran en versiones posteriores, lo que está claro es que el contenedor funciona correcamente y que solo ocurre al hacer `restart` al limpiar con `down`.

## Pre-requisitos

- [Configuración inicial del servidor de red](./initial-netServer-configuration.md).
- [Instalación y configuración de docker](./docker-install-configuration.md).

## Pasos

### Agregar permisos firewall ufw

Agregar permisos http y https:

```bash
sudo ufw allow http
sudo ufw allow http
```

### Configuración

Crea usuarios que será usados por docker para traefik y busybox:

```bash
sudo useradd -M -s /usr/sbin/nologin docker-traefik
sudo useradd -M -s /usr/sbin/nologin docker-busybox
```

Obtener los id del usuario para configurar luego `docker-traefik` y `docker-busybox`:

```bash
id docker-traefik docker-busybox
```

* **salida:**

  ```plaintext
  uid=1008(docker-traefik) gid=1010(docker-traefik) groups=1010(docker-traefik)
  uid=1009(docker-busybox) gid=1011(docker-busybox) groups=1011(docker-busybox)
  ```

> En este ejemplo, son los valores 1008:1010 y 1009:1011.

Crear la carpeta de configuración para `docker-traefik`, asignar permisos, propietario y heredarlos:

```bash
mkdir -p /etc/appserver/traefik
sudo chown nobody:infrastructure /etc/appserver/traefik -R
sudo chmod u=rwx,g=rwx,o=--- /etc/appserver/traefik
sudo chmod g+s /etc/appserver/traefik
sudo setfacl -d -m u::rwx /etc/appserver/traefik     # Propietario: rwx
sudo setfacl -d -m g::rwx /etc/appserver/traefik     # Grupo: rwx
sudo setfacl -d -m o::--- /etc/appserver/traefik     # Otros: sin permisos
```

Crear el archivo de configuración `/etc/appserver/traefik/traefik.yml` y darle permiso al usuario `docker-traefik`:

```bash
touch /etc/appserver/traefik/traefik.yml
sudo setfacl -m u:docker-traefik:rX /etc/appserver/traefik
sudo setfacl -m u:docker-traefik:r /etc/appserver/traefik/traefik.yml
```

Crear el archivo de solicitudes certificado `Let's Encrypt` en `/etc/appserver/traefik/resolver/vps-a1bdd53d.vps.ovh.net.json` y darle permiso de lectura y ecritura unicamente al usuario `docker-traefik` porque por requisito de traefik, solo el propietario podrá tener acceso:

```bash
mkdir /etc/appserver/traefik/resolver
touch /etc/appserver/traefik/resolver/vps-a1bdd53d.vps.ovh.net.json
sudo chown docker-traefik:docker-traefik /etc/appserver/traefik/resolver -R
sudo chmod u-rwx,u+rw,g-rwx,o-rwx -R /etc/appserver/traefik/resolver
```

Crear carpeta para configuraciones dnimáicas de `traefic` y asignar permiso al usurio `docker-traefik`:
> Traefik usará esta carpeta para leer cada archivo `.yml` que contenga de forma automatica para configurar cada servicio.

```bash
mkdir /etc/appserver/traefik/dynamic
sudo setfacl -m u:docker-traefik:rX /etc/appserver/traefik/dynamic
sudo setfacl -d -m u:docker-traefik:r /etc/appserver/traefik/dynamic
```

> Estamos dando permiso al directorio `/etc/appserver/traefik/dynamic` para `docker-traefik` para poder leer cualquier futuro nuevo archivo que contenga

Para el primer servicio de prueba con `busybox`, crear el archivo de configuración:

```bash
touch /etc/appserver/traefik/dynamic/test-web.yml
```

Crear carpeta de logs de traefik, asignar propietario y permisos:

```bash
sudo mkdir /var/log/traefik
sudo chown docker-traefik:infrastructure /var/log/traefik -R
sudo chmod u=rwx,g=rx,o=--- /var/log/traefik
sudo chmod g+s /var/log/traefik
sudo setfacl -d -m u::rwX /var/log/traefik          # Propietario: rwx
sudo setfacl -d -m g::rX /var/log/traefik           # Grupo: rX (navegar)
sudo setfacl -d -m o::--- /var/log/traefik          # Otros: sin permisos
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

Además, busybox necesita que creemos el mismo nombre de carpeta que la ruta indicada en el navegador, esto quiere decir, como pondremos la url `https://vps-a1bdd53d.vps.ovh.net/test`, para la ruta `test`, debemos crear el directorio `test`, es decir:

```bash
mkdir /srv/www/test
```

Dar permiso lectura y ejecución (navegar en capertas) al usuario `docker-busybox` en toda la carpeta y sub-carpetas de forma heredada:

```bash
sudo setfacl -m u:docker-busybox:rX /srv/www
sudo setfacl -m u:docker-busybox:rX /srv/www/test
sudo setfacl -d -m u:docker-busybox:r /srv/www/test
```

Crear página simple de prueba para `busybox` en `/srv/www/index.html`:
                                                 
```html
<!DOCTYPE html>
<html>
<head>
    <title>Prueba Traefik</title>
</head>
<body>
    <h1>Traefik funciona con BusyBox</h1>
</body>
</html>
```

Comprueba los permisos finales:
> Voy a omitir en la salida lo que es irrelevante para hacer foco en lo que tenemos que revisar.

```bash
getfacl /etc/appserver/traefik/
```

* **salida:**

  ```plaintext
  user:docker-traefik:--x
  default:user::rwx
  default:group::rwx
  default:other::---
  ```

```bash
getfacl /etc/appserver/traefik/traefik.yml
```

* **salida:**

  ```plaintext
  user::rw-
  user:docker-traefik:r--
  ```


```bash
getfacl /etc/appserver/traefik/dynamic/test-web.yml
```

* **salida:**

  ```plaintext
  user:docker-traefik:r--
  other::---
  ```


```bash
sudo ls -ll /etc/appserver/traefik/resolver/
```

* **salida:**

  ```plaintext
  -rw-rw---- 1 docker-traefik docker-traefik 0 Jan 28 20:08 vps-a1bdd53d.vps.ovh.net.json
  ```

```bash
getfacl /srv/www/test
```

* **salida:**

  ```plaintext
  user:docker-busybox:--x
  other::---
  ```

```bash
getfacl /srv/www/test/index.html 
```

* **salida:**

  ```plaintext
  user:docker-busybox:r--
  other::---
   ```

### Crear configuración estática de traefik

Edita `/etc/appserver/traefik/traefik.yml` con el contenido:

```yaml
entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"

certificatesResolvers:
  letsencrypt-resolver: # Nombre personalizado del resolver para certificados SSL.
    acme:
      email: demovoidgan@gmail.com # Email para registrar la cuenta en Let's Encrypt.
      storage: /etc/traefik/acme.json # Archivo dentro del contenedor para guardar los certificados.
      httpChallenge:
        entryPoint: web # Usa el punto de entrada HTTP (puerto 80) para validar el desafío HTTP-01.

http:             
  middlewares:    
    redirect-to-https: # Nombre del middleware para redirigir tráfico HTTP a HTTPS.
      redirectScheme:
        scheme: https
        permanent: true # Define que la redirección es permanente (código HTTP 301).

providers:
  file:
    directory: "/etc/traefik/dynamic" # Ruta donde estarán los archivos de configuración dinámica.
    watch: true # Detectar cambios de archivos de configuracion para no hace restart de docker-traefic

log:
  level: ERROR # Nivel de detalle de los logs (ERROR, WARN, INFO, DEBUG).
  filePath: "/var/log/traefik.log"

accessLog:
  filePath: "/var/log/traefik-access.log"
  format: json
```

### Crear configuración dinámica de traefik

Esta configuración complementa a `/etc/appserver/traefik/traefik.yml` y es una forma de organizar mejor la configuración de cada servicio.

Editar para el servicio `test-web` en `/etc/appserver/traefik/dynamic/test-web.yml` la configuración:
> Para otros servicios crearemos otros archivos diferentes en esta misma carpeta `/etc/appserver/traefik/dynamic/`.

```yaml
http:
  services:
    test-web:
      loadBalancer:
        servers:
          - url: "http://test-web:80"  #test-web es el nombre del contenedor
  routers:
    test-web:
      rule: "Host(`test.web3-101.open3diy.org`)"
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt-resolver
      service: test-web
```

Valida la sintaxis con `yq`:

```bash
yq eval /etc/appserver/traefik/dynamic/test-web.yml
```

> Debe mostrar el contenido de la configuración si todo es correcto.

### Crear configuración de traefik en `docker-compose`

Edita en `/etc/appserver/docker/docker-compose.yml` la siguiente configuración:

```yaml
services:
  reverse-proxy:
    image: traefik:latest
    user: "1008:1010"
    container_name: reverse-proxy
    ports:
      - 80:80                       
      - 443:443                     
    volumes:
      - /etc/appserver/traefik/traefik.yml:/etc/traefik/traefik.yml:ro
      - /etc/appserver/traefik/resolver/vps-a1bdd53d.vps.ovh.net.json:/etc/traefik/acme.json
      - /etc/appserver/traefik/dynamic:/etc/traefik/dynamic
      - /var/log/traefik:/var/log
    restart: unless-stopped                 # Reinicia el contenedor automáticamente si se detiene inesperadamente
  test-web:
    image: busybox:latest
    user: "1009:1011"              # UID:GID del usuario en el host
    container_name: test-web
    command: httpd -f -p 80
    volumes:
      - /srv/www/:/www:ro
    working_dir: /www
    restart: unless-stopped
```

Explicacion:



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

* **salida:**

  ```plaintext
  NAME            IMAGE            COMMAND                  SERVICE         CREATED          STATUS         PORTS
  reverse-proxy   traefik:latest   "/entrypoint.sh trae…"   reverse-proxy   10 seconds ago   Up 9 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp
  test-web        busybox:latest   "httpd -f -v -p 81"      test-web        10 seconds ago   Up 9 seconds   0.0.0.0:81->81/tcp, :::81->81/tcp
  ```

Revisar logs en docker:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml logs reverse-proxy
docker-compose -f /etc/appserver/docker/docker-compose.yml logs test-web
```

> Ninguna salida veremos si no hay errores


Revisar logs de traefik:

```bash
less /var/log/traefik/traefik.log 
```

> Ninguna salida veremos si no hay errores

Revisar si está enrutando correctamente en traefik a test-web:

```bash
docker exec -it reverse-proxy traefik routes
```


### Prueba final

Acceder a la URL de prueba en `https://vps-a1bdd53d.vps.ovh.net/test` y ver el contenido.

Revisar los logs de acceso:

```bash
less /var/log/traefik/traefik-access.log
```

## Referencias

- [Traefic](https://github.com/traefik/traefik).
- [BUSYBOX](https://www.busybox.net/).
- [Instalación de traefik de atareao](https://atareao.es/tutorial/traefik/instalacion-de-traefik/).
- [Enable snaps on Ubuntu and install Traefik](https://snapcraft.io/install/traefik/ubuntu).
- `chatgpt.com`.
- https://github.com/Tecnativa/docker-socket-proxy/issues/58