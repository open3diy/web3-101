# Configuración segura de `docker proxy`

Esta es la solucion errada, nombrada como `#docker-proxy-secureConfiguration`.

## Contexto

Este es un paso de configuración de [Web3 - Instalación inicial de `docker` y su configuración inicial](../../misc/netServer-docker-install-configuration.md).

## Propósito

Inicialmente al instalar `traefik`, para no dar acceso al socket de docker en el host, determine usar `docker-proxy`.

[`Docker proxy`](https://github.com/Tecnativa/docker-socket-proxy) es un contenedor que actúa como intermediario para acceder al socket de Docker.

En este paso se intentó crear un contenedor con privilegios de usuario limitados para cumplir con el compliance descrito en la [conifiguración inicial de un servidor de red](../../misc/netServer-initial-configuration.md).

Después de buscar en foros y ayuda, parece que no es posible que dentro del propio contenedor de `docker-proxy` se pueda iniciar con un usuario que no sea root (ID = 0).

Por este motivo `docker-proxy` está descartado por ser un gran abujero de seguridad, es decir, prefiero no usarlo y configurar manualmente `traefik`.

Pero los pasos que quedaron, con errores porque no termine de revisarlo, aquí están..

## Pasos

### Configuración previa de `docker proxy`

Inicialmente y para ejecutar su contendor, crearemos el usuario especifico `docker-socket-proxy`:

```bash
sudo useradd -M -s /usr/sbin/nologin docker-socket-proxy
```

Como necesitará permisos para `/var/run/docker.sock` después de hacer pruebas, el usuario se tiene que incluir en el grupo `docker`:

```bash
sudo usermod -aG docker docker-socket-proxy
```

```bash
sudo mkdir -p /var/lib/haproxy
sudo touch /var/lib/haproxy/server-state
sudo chown docker-socket-proxy:infrastructure /var/lib/haproxy -R
sudo chmod u=rwx,g=rx,g+s,o= /var/lib/haproxy
```

### Construir imagen de `docker proxy` para el usuario permitido (no root)

Obtener el id de usuario y grupo del usuario `docker-socket-proxy:`:

```bash
id docker-socket-proxy
```

- **salida:**

    ```plaintext
    uid=1003(docker-socket-proxy) gid=1004(docker-socket-proxy) groups=1004(docker-socket-proxy),988(docker)
    ```

> Copiar valores 1003 y 1004, que son ejemplo en los siguientes ejemplos, pero por favor, revisa estos valores para tú caso concreto.

Crear en `/etc/appserver/docker` directorio `docker-proxy` y archico `Dockerfile` con contenido:

```Dockerfile
FROM tecnativa/docker-socket-proxy:latest

# Crear el usuario dentro del contenedor
RUN addgroup -g 1004 customgroup && \
    adduser -D -u 1003 -G customgroup customuser

# Cambiar temporalmente a root para configurar permisos
USER root

# Dar permisos completos a la carpeta
RUN mkdir -p /usr/local/etc/haproxy && \
    chmod -R 777 /usr/local/etc/haproxy && \
    chmod -R 777 /run

# Cambiar al usuario creado
USER customuser

# Establecer el directorio de trabajo
WORKDIR /home/customuser
```

> Recuerda que 1003 y 1004 son id de este ejemplo, pon los que corresponda en tú caso.

Construir la imagen:

```bash
docker build -f /etc/appserver/docker/docker-proxy/Dockerfile -t custom-docker-socket-proxy /etc/appserver/docker/docker-proxy
```

### Incluir imagen en archivo de `docker-compose`

Editar en `/etc/appserver/docker/docker-compose.yml`:

```yaml
services:
  docker-proxy:
    image: custom-docker-socket-proxy
    container_name: docker-proxy
    environment:
      CONTAINERS: 1 # Permitir acceso a contenedores
      SERVICES: 1   # Permitir acceso a servicios
      TASKS: 1      # Permitir acceso a tareas
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/haproxy:/var/lib/haproxy  # Persistir estado del servidor
    ports:
      - 2375:2375 # Puerto expuesto para el proxy
```

### Iniciar cambios

Verificar la configuración inicial:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml config
```

> Si falla, evisar desde un editor que todos los elementos están bien tabulados.

Ejecutar sin detener servicios en ejecución:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml up -d
```

- **salida:**

    ```plaintext
    [+] Running 1/1
     ✔ Container docker-proxy  Started   
    ```

Verificar:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml ps
```

- **salida:**

    ```plaintext
    NAME           IMAGE                           COMMAND                  SERVICE        CREATED         STATUS         PORTS
    docker-proxy   tecnativa/docker-socket-proxy   "docker-entrypoint.s…"   docker-proxy   7 seconds ago   Up 6 seconds   0.0.0.0:2375->2375/tcp, :::2375->2375/tcp
    ```

Revisar logs:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml logs docker-proxy
```

- **salida:**

    ```plaintext
    docker-proxy  | [WARNING] 013/220444 (1) : config : missing timeouts for backend 'docker-events'.
    docker-proxy  |    | While not properly invalid, you will certainly encounter various problems
    docker-proxy  |    | with such a configuration. To fix this, please ensure that all following
    docker-proxy  |    | timeouts are set to a non-zero value: 'client', 'connect', 'server'.
    docker-proxy  | Proxy dockerbackend started.
    docker-proxy  | Proxy docker-events started.
    docker-proxy  | Proxy dockerfrontend started.
    docker-proxy  | [NOTICE] 013/220444 (1) : New worker #1 (12) forked
    ```

Probar en host:

```bash
curl http://localhost:2375/version
```

- **salida esperda (pero que no fue):**

    ```json
    {"Platform":{"Name":"Docker Engine - Community"},"Components":[{"Name":"Engine","Version":"27.4.1","Details":{"ApiVersion":"1.47","Arch":"amd64","BuildTime":"2024-12-17T15:45:46.000000000+00:00","Experimental":"false","GitCommit":"c710b88","GoVersion":"go1.22.10","KernelVersion":"6.8.0-51-generic","MinAPIVersion":"1.24","Os":"linux"}},{"Name":"containerd","Version":"1.7.24","Details":{"GitCommit":"88bf19b2105c8b17560993bee28a01ddc2f97182"}},{"Name":"runc","Version":"1.2.2","Details":{"GitCommit":"v1.2.2-0-g7cb3632"}},{"Name":"docker-init","Version":"0.19.0","Details":{"GitCommit":"de40ad0"}}],"Version":"27.4.1","ApiVersion":"1.47","MinAPIVersion":"1.24","GitCommit":"c710b88","GoVersion":"go1.22.10","Os":"linux","Arch":"amd64","KernelVersion":"6.8.0-51-generic","BuildTime":"2024-12-17T15:45:46.000000000+00:00"}
    ```

- **salida real del error mostrado** 

  ```html
  <html><body><h1>503 Service Unavailable</h1>
  No server is available to handle this request.
  </body></html>
  ```

Ni en foros ni en `chatgpt` aparece una solución coherente y que funcione, por lo que finalmente decidí dejar este paso en el cementerio :(
