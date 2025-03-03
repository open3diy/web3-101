# Configuración de docker `userns-remap`

Esta es la solucion errada, nombrada como `#docker-userns-remap-configuration`.

## Contexto

Este es un paso de configuración de [Web3 - Instalación inicial de `docker` y su configuración inicial](../../misc/netServer-docker-install-configuration.md).

## Propósito

[userns-remap](https://docs.docker.com/engine/security/userns-remap/) es una configuración que mejora la seguridad al remapear los IDs de usuario y grupo dentro de los contenedores (como root, UID 0) a IDs no privilegiados en el host. Esto asegura que los contenedores no tengan permisos de root en el sistema subyacente, incluso si son comprometidos.

En este paso se intenta crear un contenedor con privilegios de usuario limitados para cumplir con el compliance descrito en la [conifiguración inicial de un servidor de red](../../misc/netServer-initial-configuration.md).

El problema y motivo de descartarlo, es que no permite granularidad, sólo un usuario puede ser usado para todos los contenedores y lo que puede ver uno lo ven todos, así que al considerarlo un problema de seguridad lo descarto.

## Problemas conocidos que debes saber antes

### Rangos de usuarios para `userns-remap` 

Cuando agregamos un usuario, ya se crean los rangos de usuarios y grupos en `/etc/subuid` y `/etc/subgid` que se usará para docker, sin embargo, si vemos que tenemos problemas, podemos revisarlo o crearlo.

> IMPORTANTE: este ejemplo es sólo si tienes problemas y ves que **no** aparece el usuario creado... si ya está, no cambies nada.

**Partiendo con el ejemplo de crear el usuario `docker-socket-proxy`**

Edita rango de usuario para docker `userns-remap` en `/etc/subuid`:

```plaintest
docker-socket-proxy:100000:65536
```

**Notas:**

Se inicia con `100000` para no usar IDs del sistema y se indica `65536` es el rango de por defecto.

Igualmente debemos revisar de no usar rangos entre usuarios, es decir, esto es incorrecto:

```plaintest
docker-socket-proxy:100000:65536
usuario1:110000:65536
```

Lo correcto:

```plaintest
docker-socket-proxy:100000:65536
usuario1:165536:65536
```

Luego, edita rango de grupo para docker `userns-remap` en `/etc/subgid`:

```plaintest
docker-socket-proxy:100000:65536
```

Reinicia docker para aplicar cambios en rangos de usuario:

```bash
sudo systemctl restart docker
sudo systemctl daemon-reload
```

## Paso omitido

### Configurar docker con `userns-remap`

`userns-remap` es una configuración que mejora la seguridad al remapear los IDs de usuario y grupo dentro de los contenedores (como root, UID 0) a IDs no privilegiados en el host. Esto asegura que los contenedores no tengan permisos de root en el sistema subyacente, incluso si son comprometidos.

**Los pasos para configurar son:**

Editar `/etc/docker/daemon.json` e incluir:

```bash
sudo nano /etc/docker/daemon.json
``` 

Luego incluir:

```json
{
    "userns-remap": "default"
}
```

Reinicia servicio docker para aplicar cambios:

```bash
sudo systemctl restart docker
```

- **salida:**

    ```plaintext
    Warning: The unit file, source configuration file or drop-ins of docker.service changed on disk. Run 'systemctl daemon-reload' to reload units.
    ```

Al ver el aviso, simplemente ejecutar:

```bash
sudo systemctl daemon-reload'
```

Creará el usuario `dockremap`, comprobarlo:

```bash
id dockremap
```

- **salida:**

    ```plaintext
    uid=111(dockremap) gid=113(dockremap) groups=113(dockremap)
    ```

Docker usa `/etc/subuid` y `/etc/subgid` para asignar rangos de UIDs y GIDs en el host que remapean los IDs de usuario y grupo dentro de los contenedores, garantizando aislamiento y seguridad. 

Lo comprobamos para el ID:

```bash
sudo tail /etc/subuid | grep dockremap
```

- **salida:**

    ```plaintext
    dockremap:493216:65536
    ```

Y para el GID:

```bash
sudo tail /etc/subgid | grep dockremap
```

- **salida:**

    ```plaintext
    dockremap:493216:65536
    ```

## Referencias

- [Jay Schmidt - Understanding the Docker USER Instruction](https://www.docker.com/blog/understanding-the-docker-user-instruction/).
- `chatgpt.com`.
