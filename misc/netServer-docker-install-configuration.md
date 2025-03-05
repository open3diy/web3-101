# Instalación y configuración inicial de `docker` en un servidor de red

Esta es la solución nombrada como `#netServer-docker-install-configuration`.

## Contexto

Este es un tutorial de [Web3 - 101](../README.md).

## Propósito

Estos son los pasos de instalación de `docker` incluyendo las herramientas y configuraciones iniciales necesarias.

Adicionalmente, se crea el entorno que permita crear hooks o extensión de los contenedores.
Esto es porque ciertos contenedores pueden necesitar modificaciones o simplemente necesitamos ejecutar un script concreto en el `host` al iniciar un contenedor.

## Solución

En esta instalación, se incluirá la herramienta [`docker-componse`]([https://docs.docker.com/compose/) para iniciar contenedores.
> ¿por qué no Kubernetes? Estos proyectos son de prueba para entornos web3, descentralizados, realmente toda la complejidad de Kubernetes no la considero necesaria.

A modo de hooks de los contenedores, mediante un servicio systemd, ejecutará un script de shell para escuchar los eventos de inicio de cada contenedor.

Ir a [Referencias](#referencias).

## Configuración

- En servidor VPS.
- Con [docker](https://www.docker.com/).
- En [Ubuntu 24.04 LTS](https://ubuntu.com/blog/tag/ubuntu-24-04-lts).

## Problemas conocidos que debes saber antes

### No olvidar actualizar e instalar los paquetes debian

```bash
sudo apt update
sudo apt upgrade -y
```

> Hacer casos a los avisos, no ignorarlos o luego todo será peor.

### Contenedores huérfanos por hacer pruebas

Si llegas a ver el error:

```plaintest
etc/appserver/docker/docker-compose.yml down
[+] Running 1/0
 ! Network docker_default  Resource is still in use
```

Apaga removiendo contenedores huérfanos:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml down --remove-orphans
```

## Pre-requisitos

- [Configuración inicial del servidor de red](./netServer-initial-configuration.md).
- [Instalación de docker](https://voidnull.es/instalacion-de-docker-en-ubuntu-24-04/).

## Pasos

### Instalación de `docker-compose`

Puedes instalarlo así:

```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

### Configuración de `docker-compose`

Puedes organizar donde dejar el archivo [`docker-compose.yml`](https://docs.docker.com/compose/intro/compose-application-model/) donde quieras y organizarlo cómo quieras, pero en estos tutoriales siempre partiremos de un único archivo en `/etc/appserver/docker/docker-compose.yml`, así que los pasos son:

Crear carpeta `/etc/appserver/docker`:

```bash
mkdir /etc/appserver/docker
```

Crear el archivo de configuración:

```bash
touch /etc/appserver/docker/docker-compose.yml
```

### Hooks de los contenedores

Crear archivo de logs para docker y permisos:

```bash
sudo mkdir /var/log/docker
sudo chown nobody:infrastructure /var/log/docker -R
sudo chmod u=rwx,g=rwx,o=--- /var/log/docker -R
sudo chmod g+s /etc/appserver /var/log/docker -R
sudo setfacl -d -m u::rwX /var/log/docker    # Propietario: rwx
sudo setfacl -d -m g::rwX /var/log/docker    # Grupo: rwx
sudo setfacl -d -m o::--- /var/log/docker    # Otros: sin permisos
```

Crear los scripts de entrada y salida principales con permiso para ejecutarlo:

```bash
touch /etc/appserver/docker/post-up.sh
touch /etc/appserver/docker/post-down.sh
chmod +x /etc/appserver/docker/post-up.sh
chmod +x /etc/appserver/docker/post-down.sh
```

Edita `/etc/appserver/docker/post-up.sh`:

```sh
CONTAINER_ID="$1"
CONTAINER_NAME="$2"
echo "$(date '+%Y-%m-%d %H:%M:%S') - Contenedor $CONTAINER_NAME ($CONTAINER_ID) iniciado" >> /var/log/docker/docker-events.log
# Indicar aquí script de extension al iniciar (se aplicará al iniciar el sistema también)
```

> Este script se ejecutará al iniciar o reiniciar un contenedor, eso incluye al iniciar el sistema.

Edita `/etc/appserver/docker/post-down.sh`:

```sh
CONTAINER_ID="$1"
CONTAINER_NAME="$2"
echo "$(date '+%Y-%m-%d %H:%M:%S') - Contenedor $CONTAINER_NAME ($CONTAINER_ID) detenido" >> /var/log/docker/docker-events.log
# Indicar aquí script de extension al detener (no se aplica al parar el sistema)
```

> Este script se ejecutará al parar o reiniciar un contenedor, pero **no** al parar o reiniciar el sistema.

Crear script para monitorizar eventos de Docker y permitir ejecutarlo:

```bash
touch /etc/appserver/docker/monitor-docker.sh
chmod +x /etc/appserver/docker/monitor-docker.sh
```

Edita `/etc/appserver/docker/monitor-docker.sh`:

```sh
#!/bin/bash

# Ejecutar post-up.sh para contenedores ya en ejecución al arrancar el servicio
docker ps --format '{{.ID}} {{.Names}}' | while read CONTAINER_ID CONTAINER_NAME; do
    /etc/appserver/docker/post-up.sh "$CONTAINER_ID" "$CONTAINER_NAME"
done

# Escuchar eventos en tiempo real después de haber procesado los contenedores ya en ejecución
docker events --filter event=start --filter event=stop --format '{{.ID}} {{.Actor.Attributes.name}} {{.Type}} {{.Action}}' | while read CONTAINER_ID CONTAINER_NAME TYPE ACTION; do
    if [ "$ACTION" == "start" ]; then
        /etc/appserver/docker/post-up.sh "$CONTAINER_ID" "$CONTAINER_NAME"
    elif [ "$ACTION" == "stop" ]; then
        /etc/appserver/docker/post-down.sh "$CONTAINER_ID" "$CONTAINER_NAME"
    fi
done
```

**Hacerlo persistente con systemd**.

Crear el archivo de servicio:

```bash
sudo nano /etc/systemd/system/monitor-docker.service
```

Agregar el contenido:

```ini
[Unit]
Description=Monitor de eventos de Docker
After=docker.service
Requires=docker.service

[Service]
ExecStart=/etc/appserver/docker/monitor-docker.sh
Restart=always
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

Recargar systemd y habilitar el servicio:

```bash
sudo systemctl daemon-reload
sudo systemctl enable monitor-docker.service
sudo systemctl start monitor-docker.service
```

Verificar estado:

```bash
sudo systemctl status monitor-docker.service
```

Revisar inicio en log:

```bash
less /var/log/docker/docker-events.log
```

## Notas adicionales sobre `docker-compose`

Cuando ejecutes `docker-compose` en el terminal, situate en la carpeta donde esté el archivo `docker-compose.yml` o indica siempre el parámetro `-f`, por ejemplo, `docker-compose -f /etc/docker/docker-compose.yml down`.

Para ejecutar `docker-compose` los comandos puedes encontrarlos en la [referencia dockerdocs de docker compose](https://docs.docker.com/reference/cli/docker/compose/), pero si quieres conocer algunos comunes, son los siguientes:

- `docker-compose down`: Para limpiar el entorno antes de reiniciar servicios, detiene y elimina los contenedores, redes, y volúmenes anónimos creados por docker-compose up.
- `docker-compose down --remove-orphans`: Igualmente detiene y elimina pero incluso los que no están en el archivo `docker-compose-yml` en uso.
- `docker-compose pull`: Descarga (o actualiza) las imágenes necesarias desde el repositorio remoto (Docker Hub o privado) sin levantar los contenedores.
- `docker-compose up -d`: Construye, inicia y monta los contenedores definidos en `docker-compose.yml` que han cambiado, con parámetro `-d` (detached) que ejecuta los contenedores en segundo plano.
  - Opciones útiles:
    - `--build`: Fuerza la reconstrucción de las imágenes locales antes de iniciar.
    - `--force-recreate`: Elimina y vuelve a crear los contenedores, aunque no haya cambios en ellos.
- `docker-compose ps`: Lista los contenedores gestionados por el archivo actual.
- `docker-compose logs`: Muestra los logs de los contenedores.
- `docker-compose logs -f`: Sigue los logs en tiempo real.
- `docker-compose restart`: Reinicia los servicios definidos en el archivo.
- `docker-compose exec <servicio> <comando>`: Ejecuta un comando dentro de un contenedor.
- `docker-compose stop`: Detiene los contenedores sin eliminarlos.
- `docker-compose rm`: Elimina los contenedores detenidos.

### Algunos comandos cuando hay problemas

Entrar en el contenedor:

```bash
docker run -it --entrypoint /bin/sh [nombre_contenedor]
```

Hacer limpieza y borrar todas las imágenes de docker:

```bash
docker images -q | xargs -r docker rmi -f
```

Hacer limpieza y borrar todos los contenedores

```bash
docker ps -aq | xargs -r docker rm -f
```

## Referencias

- [`dokcer-componse`]([https://docs.docker.com/compose/).
- [voidnull.es - instalación de docker](https://voidnull.es/instalacion-de-docker-en-ubuntu-24-04/).
- [docker events](https://docs.docker.com/reference/cli/docker/system/events/).
- `chatgpt.com`.
