# Instalación proxy inverso traefik

Esta es la solucion nombrada como `#inverseProxy-traefik-install`.

## Contexto

Este es un tutorial de [Web3 - IPFS](../README.md).

## Propósito

Estos son los pasos de configuración e instalación de un proxy inverso para poder publicar en un servidor púiblico aplicaciones Web.

## Solución

La solución es la instalación de [Traefic](https://github.com/traefik/traefik) como proxy inverso y adicionalmente [busybox](https://www.busybox.net/) como servidor simple de contenido HTML, todo esto dentro del contexto docker y `docker-compose`.
Ver las [Referencias](#referencias).

## Configuración

- En servidor VPS.
- Con [docker](https://www.docker.com/) y [`docker-compose`](https://docs.docker.com/compose/).
- En [Ubuntu 24.04 LTS](https://ubuntu.com/blog/tag/ubuntu-24-04-lts).

## Datos de entrada

- Para traefik, el servicio que hace de proxy
  - Carpeta para las configuraciones de traefik: `/etc/appserver/traefik`.
  - Archivo de configuración de traefik: `/etc/appserver/traefik/traefik.yml`.
  - Archivo de solicitudes certificado `Let's Encrypt:`: `/etc/appserver/traefik/vps-a1bdd53d.vps.ovh.net.json`
  - Usuario que iniciará el contenedor: `docker-traefik`.
  - Dominio de ejemplo que se publicará: `vps-a1bdd53d.vps.ovh.net`.
- Para busybox, servicio de pruebas para ver contenido publicado
  - Carpeta de contenido: `/serv/wwww/busybox_http`.
  - Usuario que iniciará el contenedor: `docker-busybox`.

## Problemas conocidos que debes saber antes

### No olvidar actualizar e instalar los paquetes debian

```bash
sudo apt update
sudo apt upgrade -y
```

> Hacer casos a los avisos, no ignorarlos o luego todo será peor.

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
sudo setfacl -m u:docker-traefik:X /etc/appserver/traefik
sudo setfacl -m u:docker-traefik:r /etc/appserver/traefik/traefik.yml
```

Crear el archivo de solicitudes certificado `Let's Encrypt` en `/etc/appserver/traefik/resolver/vps-a1bdd53d.vps.ovh.net.json` y darle permiso de lectura y ecritura unicamente al usuario `docker-traefik`:

```bash
mkdir /etc/appserver/traefik/resolver
touch /etc/appserver/traefik/resolver/vps-a1bdd53d.vps.ovh.net.json
sudo chown docker-traefik:docker-traefik /etc/appserver/traefik/resolver -R
sudo chmod o=rw,g=,o= /etc/appserver/traefik/resolver
```

> Por requisito de traefik, solo el propietario podrá tener acceso, asi que si queremos examinar o copiar el contenido usar sudo.

Crear carpeta para configuraciones dnimáicas de `traefic` y asignar permiso al usurio `docker-traefik`:
> Traefik usará esta carpeta para leer cada archivo `.yml` que contenga de forma automatica para configurar cada servicio.

```bash
mkdir /etc/appserver/traefik/dynamic
sudo setfacl -d -m u::docker-traefik:rX /etc/appserver/traefik/dynamic
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

Para luego realizar pruebas, crear carpeta, asignar propietario y permisos, para el contenido wwww para la app `busybox`:

```bash
mkdir /srv/www /srv/www
sudo chown nobody:infrastructure /srv/www -R
sudo chmod g=rwx,o=,g+s /srv/www/busybox_http

sudo mkdir -p /srv/www
sudo chown nobody:infrastructure /srv/www -R
sudo chmod u=rwx,g=rwx,o=--- /srv/www
sudo chmod g+s /srv/www
sudo setfacl -d -m u::rwx /srv/www     # Propietario: rwx
sudo setfacl -d -m g::rwx /srv/www     # Grupo: rwx
sudo setfacl -d -m o::--- /srv/www     # Otros: sin permisos
```

Dar permiso lectura y ejecución (navegar en capertas) al usuario `docker-busybox` en toda la carpeta y sub-carpetas de forma heredada:

```bash
sudo setfacl -d -m d:u:app1:rX /srv/www
```

Crear página simple de prueba para `busybox` en `/srv/www/index.html`:
                                                 
```html
<!DOCTYPE html>
<html>
<head>
    <title>Prueba Traefik</title>
</head>
<body>
    <h1>¡Traefik funciona con BusyBox!</h1>
</body>
</html>
```

Comprueba los permisos finales:

```bash
sudo getfacl /etc/appserver/traefik
```

* **salida:**

    ```plaintext
   
    ```

### Crear configuración estática de traefik

Edita `/etc/appserver/traefik/traefik.yml` con el contenido:

```yaml
entryPoints:
  web:
    address: ":80" # Define el punto de entrada para tráfico HTTP (puerto 80).
  websecure:
    address: ":443" # Define el punto de entrada para tráfico HTTPS (puerto 443).

certificatesResolvers:
  letsencrypt-resolver: # Nombre personalizado del resolver para certificados SSL.
    acme:
      email: demovoidgan@gmail.com # Email para registrar la cuenta en Let's Encrypt.
      storage: /etc/traefik/acme.json # Archivo dentro del contenedor para guardar los certificados.
      httpChallenge:
        entryPoint: web # Usa el punto de entrada HTTP (puerto 80) para validar el desafío HTTP-01.

http:              # Configuración para manejar peticiones HTTP.
  middlewares:     # Definición de middlewares (transformaciones o redirecciones).
    redirect-to-https: # Nombre del middleware para redirigir tráfico HTTP a HTTPS.
      redirectScheme:
        scheme: https # Especifica que todo el tráfico debe redirigirse al esquema HTTPS.
        permanent: true # Define que la redirección es permanente (código HTTP 301).

providers:
  file:
    directory: "/etc/traefik/dynamic" # Ruta donde estarán los archivos de configuración dinámica.

log:
  level: ERROR # Nivel de detalle de los logs (ERROR, WARN, INFO, DEBUG).
  filePath: "/var/log/traefik.log" # Ruta dentro del contenedor donde se almacenarán los logs generales.

accessLog:
  filePath: "/var/log/traefik-access.log" # Ruta dentro del contenedor para guardar los logs de acceso HTTP.
  format: json # Especifica el formato de los logs de acceso: 'json' o 'common' (texto plano).
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
          - url: "http://localhost:81" 
  routers:
    test-web:
      rule: "Host(`vps-a1bdd53d.vps.ovh.net`) && PathPrefix(`/test`)"
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt-resolver
      service: test-web
 
```

### Crear configuración de traefik en `docker-compose`

Edita en `/etc/appserver/docker/docker-compose.yml` la siguiente configuración:

```yaml
services:
  reverse-proxy:
    image: traefik:latest
    user: "1008:1010"               # UID:GID del usuario en el host
    container_name: reverse-proxy   # Nombra el contenedor como 'traefik' para facilitar su identificación
    ports:
      - 80:80                       
      - 443:443                     
    volumes:
      - /etc/appserver/traefik/traefik.yml:/etc/traefik/traefik.yml:ro # Monta el archivo de configuración estática en el contenedor
      - /etc/appserver/traefik/resolver/vps-a1bdd53d.vps.ovh.net.json:/etc/traefik/acme.json # Almacena los certificados SSL en el host
      - /etc/appserver/traefik/dynamic:/etc/traefik/dynamic # Monta la configuración dinamica para los servicios
      - /var/log/traefik:/var/log              # Monta los logs generales del contenedor en el host
    restart: unless-stopped                 # Reinicia el contenedor automáticamente si se detiene inesperadamente

  test-web:
    image: busybox:latest
    user: "1009:1011"              # UID:GID del usuario en el host
    container_name: test-web
    ports:
      - 81:81 # Exponer el puerto 81 para acceso desde Traefik
    command: httpd -f -v -p 81
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

Ejecutar sin detener servicios en ejecución:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml up -d
```

Verificar:
```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml ps
```

* **salida:**

    ```plaintext
    ```

Revisar logs:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml logs docker-traefik
docker-compose -f /etc/appserver/docker/docker-compose.yml logs test-web
```

* **salida:**

    ```plaintext
    ```

### Prueba final

Acceder a la URL de prueba en `https://vps-a1bdd53d.vps.ovh.net/test` y ver el contenido.

Revisar el log en `/var/log/traefik`.


## Referencias

- [Traefic](https://github.com/traefik/traefik).
- [BUSYBOX](https://www.busybox.net/).
- [Instalación de traefik de atareao](https://atareao.es/tutorial/traefik/instalacion-de-traefik/).
- [Enable snaps on Ubuntu and install Traefik](https://snapcraft.io/install/traefik/ubuntu).
- `chatgpt.com`.
- https://github.com/Tecnativa/docker-socket-proxy/issues/58