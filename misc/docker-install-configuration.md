# Instalación inicial de `docker` y su configuración inicial

Esta es la solucion nombrada como `#docker-install-configuration`.

## Contexto

Este es un tutorial de [Web3 - IPFS](../README.md).

## Propósito

Estos son los pasos de instalación de `docker` incluyendo las herramientas y configuraciones iniciales necesarias.

En esta instalación, se incluirá la herramienta [`dokcer-componse`]([https://docs.docker.com/compose/) para iniciar contenedores y [`docker-proxy`](https://github.com/Tecnativa/docker-socket-proxy) para acceder al socket de Docker para otros contenedores y mejorar en la seguridad.
> ¿por qué no Kubernetes? Estos proyectos son de prueba para entornos web3, descentralizados, realmente toda la complejidad de Kubernetes no la considero necesaria.

## Configuración

- En servidor VPS.
- Con [docker](https://www.docker.com/).
- En [Ubuntu 24.04 LTS](https://ubuntu.com/blog/tag/ubuntu-24-04-lts).

## Datos de entrada

- Carpeta para las configuraciones de docker: `/etc/appserver/docker`.
- Archivo de configuración de `docker-compose`: `/etc/appserver/docker/docker-compose.yml`.

## Problemas conocidos que debes saber antes

### No olvidar actualizar e instalar los paquetes debian

```bash
sudo apt update
sudo apt upgrade -y
```

> Hacer casos a los avisos, no ignorarlos o luego todo será peor.

### Contenedores huefanos por hacer pruebas

Si llegas a ver el error:

```plaintest
etc/appserver/docker/docker-compose.yml down
[+] Running 1/0
 ! Network docker_default  Resource is still in use
```

Apaga removiendo contenedores huefanos:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml down --remove-orphans
```

### Algunos comandos cuando hay problemas...

Entrar en el contenedor:

```bash
docker run -it --entrypoint /bin/sh custom-docker-socket-proxy
```

Hacer limpieza y borrar todas las imagenes de docker:

```bash
docker rmi $(docker images -q) -f
```

> Si no hay imagenes, fallará este comando...

Hacer limpieza y borrar todos los contenedores

```bash
docker rm -f $(docker ps -aq)
```

> Si no hay contenedores, fallará este comando

## Pre-requisitos

- [Configuración inicial del servidor de red](./initial-netServer-configuration.md).
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

Crear el archivo de configuracion:

```bash
touch /etc/appserver/docker/docker-compose.yml
```

### Notas adicionales sobre `docker-compose`

Cuando ejecutes `docker-compose` en el terminal, situate en la carpeta donde esté el archivo `docker-compose.yml` o indica siempre el parámetro `-f`, por ejemplo, `docker-compose -f /etc/docker/docker-compose.yml down`.

Para ejecutar `docker-compose` los comandos puedes encontrarlos en la [referencia dockerdocs de docker compose](https://docs.docker.com/reference/cli/docker/compose/), pero si quieres conocer algunos comunes, son los siguientes:

- `docker-compose down`: Para limpiar el entorno antes de reiniciar servicios, detiene y elimina los contenedores, redes, y volúmenes anónimos creados por docker-compose up.
- `docker-compose pull`: Descarga (o actualiza) las imágenes necesarias desde el repositorio remoto (Docker Hub o privado) sin levantar los contenedores.
- `docker-compose up -d`: Construye, inicia y monta los contenedores definidos en docker-compose.yml, con parámetro `-d` (detached) que ejecuta los contenedores en segundo plano.
    - Opciones útiles:
        - `--build`: Fuerza la reconstrucción de las imágenes locales antes de iniciar.
        - `--remove-orphans`: Elimina contenedores no definidos en el archivo actual.
- `docker-compose ps`: Lista los contenedores gestionados por el archivo actual.
- `docker-compose logs`: Muestra los logs de los contenedores.
- `docker-compose logs -f`: Sigue los logs en tiempo real.
- `docker-compose restart`: Reinicia los servicios definidos en el archivo.
- `docker-compose exec <servicio> <comando>`: Ejecuta un comando dentro de un contenedor.
- `docker-compose stop`: Detiene los contenedores sin eliminarlos.
- `docker-compose rm`: Elimina los contenedores detenidos.

## Referencias

- [`dokcer-componse`]([https://docs.docker.com/compose/)
- `chatgpt.com`.
