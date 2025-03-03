# Configuración en un servidor de red de cuotas de disco por usuario

Esta es la solucion nombrada como `#netServer-security-quotaDisc`.

## Contexto

Este es un tutorial que forma parte de [Web3 - 101](../README.md)

Al crear un servidor público, que estará en un VPS, por seguridad se puede crear una cuota maxima de espacio en disco para el usuario que ejecutará un servicio.

## Proposito

Limitar al usuario de ejecución de un contenedor en docker el espacio en disco, debido a que puede acceder a directorios del `host` y ante ataque, el espacio del `host` estaría expuesto.

Este tutorial parte del ejemplo de configuración del servicio de IPFS en docker, limitandolo a `11GB`.

## Solución

Aunque deprecado, la solución es [debian Linux Disk Quotas](https://www.debian.org/doc/manuals/debian-handbook/sect.quotas.es.html).

Entiendo que existen mejores soluciones, como simplemente configurar un nuevo disco dedicado a IPFS o usar `Internal Quotas (cuotas internas)`, pero en mí caso, y debido al bajo presupuesto, está solución fue la más fácil y barata.

## Configuración

- En servidor VPS.
- En [Ubuntu 24.04 LTS](https://ubuntu.com/blog/tag/ubuntu-24-04-lts).
- Sistema de archivos `ext4`.

## Problemas conocidos que debes saber antes

### No olvidar actualizar e instalar los paquetes debian

```bash
sudo apt update
sudo apt upgrade -y
```

> Hacer casos a los avisos, no ignorarlos o luego todo será peor.

### Problema al agregar el módulo `quota_v1`

En el paso [Habilitación de cuotas](#habilitación-de-cuotas) el comando:

```bash
 sudo modprobe quota_v1 -S 6.8.0-51-generic
```

Provocó el error:

```plaintext
modprobe: ERROR: could not insert 'quota_v1': Invalid argument
```

Para `Ubuntu 24` instalar el `Hardware Enablement Stack (HWE)` para la versión de Ubuntu (24.04) solucionó el problema:

```bash
sudo apt install --install-recommends linux-generic-hwe-24.04
sudo reboot
```

## Pasos

### Instalar herramientas necesarias

```bash
sudo apt update
sudo apt install quota quotatool
```

### Instalación del módulo de kernel de cuota

Si está en un servidor virtual basado en la nube, es posible que su instalación predeterminada de `Ubuntu Linux` no tenga los módulos de kernel. Pasos para comprobar:

Buscar módulos `quota_v1` y `quota_v2`.

```bash
find /lib/modules/ -type f -name '*quota_v*.ko*'
```

> Nota, no te preocupes si no están, sigue leyendo...

- **salida:**

    ```plaintext
    /lib/modules/6.8.0-51-generic/kernel/fs/quota/quota_v2.ko.zst
    /lib/modules/6.8.0-51-generic/kernel/fs/quota/quota_v1.ko.zst
    ```

    > Toma nota de la versión, en este ejemplo, sería copiar los valores `6.8.0-51-generic` y `6.8.0-51-generic`.

*Si no obtiene ningún resultado* del comando anterior, instala el paquete `linux-image-extra-virtual`:

```bash
sudo apt install linux-image-extra-virtual
```

> Ahora vuelve a buscar los módulos `quota_v1` y `quota_v2` y verás como todo es correcto.

### Configurar cuotas en la partición (si es un VPS)

**Si estas usando un VPS sigue estos pasos**, de lo contrario, sigue el siguiente paso.

Edita `/etc/fstab` y agrega las opciones `usrquota` y `grpquota` para la partición con etiqueta `cloudimg-rootfs`. Por ejemplo:

```plaintext
/dev/sda1  /home  ext4  defaults,usrquota,grpquota  0  2
LABEL=cloudimg-rootfs   /        ext4   discard,commit=30,errors=remount-ro,usrquota,grpquota     0 1
```

> Dependerá de cada caso, si hay varias opciones, simplemente separado por `,` agregamos estas nuevas opciones

### Configurar cuotas en la partición (si NO es un VPS)

**Si no estas usando un VPS sigue estos pasos**.

Usa el comando para identificar la partición que contiene la ruta donde estarán los archivos, en este ejemplo `/home/jesus/docker-ipfs-files`:

```bash
df -h /home/jesus/docker-ipfs-files
```

> Supongamos que la partición es `/dev/sda1`.

Edita `/etc/fstab` y agrega las opciones `usrquota` y `grpquota` para la partición `/dev/sda1`. Por ejemplo:

```plaintext
/dev/sda1  /home  ext4  defaults,usrquota,grpquota  0  2
```

> **Aviso**: Probablemte tengas algo diferente, la cuestión es agregar las opciones sea cual sea la partición.

### Remonta la partición para aplicar los cambios:

```bash
sudo mount -o remount /
```

- **salida:**

    ```plaintext
    mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
    ```

    Si ves esto, simplemente recarga el servicio

    ```bash
    sudo systemctl daemon-reload
    ```

Puedes verificar que se usaron las nuevas opciones:

```bash
cat /proc/mounts | grep ' / '
```

- **salida:**

    ```plaintext
    /dev/sda1 / ext4 rw,relatime,discard,quota,usrquota,grpquota,errors=remount-ro,commit=30 0 0
    ```

### Habilitación de cuotas

Antes de activar definitivamente el sistema de cuotas, debes ejecutar manualmente el `quotacheck` una vez:

```bash
sudo quotacheck -ugm /
```

> Este comando crea los archivos `/aquota.usery` y `/aquota.group`

Verificar que se crearon los archivos apropiados:

```bash
ls /aquota.*
```

Agregar los módulos de cuota al kernel de Linux (*reemplazar* `6.8.0-51-generic` y `56.8.0-51-generic` por los valores del paso [Instalación del módulo de kernel de cuota](#instalación-del-módulo-de-kernel-de-cuota).

```bash
sudo modprobe quota_v1 -S 6.8.0-51-generic
sudo modprobe quota_v2 -S 6.8.0-51-generic
```

Activar módulo de cuotas:

```bash
sudo quotaon -v /
```

- **salida:**

    ```plaintext
    /dev/vda1 [/]: group quotas turned on
    /dev/vda1 [/]: user quotas turned on
    ```

**Resultado anterior no esperado**.

Si en su lugar ves:

```plaintext
quotaon: Your kernel probably supports ext4 quota feature but you are using external quota files. Please switch your filesystem to use ext4 quota feature as external quota files on ext4 are deprecated.
quotaon: using //aquota.group on /dev/sda1 [/]: Device or resource busy
quotaon: using //aquota.user on /dev/sda1 [/]: Device or resource busy
```

Desactiva las cuotas:

```bash
sudo quotaoff -v /
```

Ejecuta quotacheck:

```bash
sudo quotacheck -cumf /
```

> No genera mensaje de éxito, si todo es correcto no saldrá nada.

Reactiva las cuotas:

```bash
sudo quotaon -v /
```

- **salida:**

    ```plaintext
    quotaon: Your kernel probably supports ext4 quota feature but you are using external quota files. Please switch your filesystem to use ext4 quota feature as external quota files on ext4 are deprecated.
    /dev/sda1 [/]: group quotas turned on
    /dev/sda1 [/]: user quotas turned on
    ```

### Crear el usuario dedicado para Docker

Crea el usuario que será usado por Docker:

```bash
sudo useradd -m docker-ipfs
```

### Asignar el límite de 11GB al usuario

Ejecuta la configuración para indicar en este caso `11 GB`:

```bash
sudo setquota -u docker-ipfs 11264000 11264000 0 0 /
```

Veifica las cuotas del usuario `docker-ipfs`:

```bash
sudo quota -u docker-ipfs
```

- **salida:**

    ```plaintext
    Disk quotas for user docker-ipfs (uid 1002): 
        Filesystem  blocks   quota   limit   grace   files   quota   limit   grace
        /dev/sda1   60972  11264000 11264000             236       0       0        
    ```

### Crear y configurar la carpeta del usuario `docker-ipfs`

Crear la carpeta para el usuario `docker-ipfs` y asignarle como propietario:

```bash
sudo mkdir -p /home/jesus/docker-ipfs-files
sudo chown docker-ipfs:docker-ipfs /home/jesus/docker-ipfs-files
```

Establece los permisos para el propietario `docker-ipfs` solo con permisos de lectura y escritura:

```bash
sudo chmod 750 /home/jesus/docker-ipfs-files
```

Dar permisos al usuario operador `jesus` para examinar, pero no para leer y mucho menos escribir:

```bash
sudo chmod o+x+r /home/jesus/docker-ipfs-files
```

Comprueba los permisos finales:

```bash
ls -ld /home/jesus/docker-ipfs-files
```

- **salida:**

    ```plaintext
    drwxr-xr-x 2 docker-ipfs docker-ipfs 4096 Dec 18 12:39 /home/jesus/docker-ipfs-files
    ```

### Test final

Comprobar estado quotas

```bash
sudo quotaon -p /
```

- **salida:**

    ```plaintext
    group quota on / (/dev/sda1) is on
    user quota on / (/dev/sda1) is on
    project quota on / (/dev/sda1) is off
    ```

Consultar cuotas usuario:

```bash
sudo quota docker-ipfs
```

- **salida:**

    ```plaintext
    Disk quotas for user docker-ipfs (uid 1002): 
    Filesystem  blocks   quota   limit   grace   files   quota   limit   grace
     /dev/sda1   60972  2816000 2816000             236       0       0        
    ```

Accede a la carpeta de la cuota:

```bash
cd ~/docker-ipfs-files
```

Situate como el usuario `docker-ipfs`:

```bash
sudo su docker-ipfs
```

Crea un archivo de 10GB sin llenar contenido

```bash
fallocate -l 10G test10G
```

> Ningún error debes ver.

Intenta crear otro archivo de 1GB:

```bash
fallocate -l 1G test1G
```

- **salida:**

    ```plaintext
    fallocate: fallocate failed: Disk quota exceeded
    ```

> Esto es la prueba exitiosa de la cuota.

Puedes borrar ahora los archivos de test con `rm test*`

### Configurar Docker para usar el usuario `docker-ipfs`

**Como nota adicional** se indica cómo iniciar docker para aplicar la cuota del usuario:

Obtener uid y guid del usuario:

```bash
id docker-ipfs
```

- **salida:**

    ```plaintext
    uid=1002(docker-ipfs) gid=1002(docker-ipfs) groups=1002(docker-ipfs)
    ```

Ejecutar docker con el usuario:

```bash
docker run --user 1002:1002 -v /home/jesus/docker-ipfs-files:/data ipfs/go-ipfs
```

## Referencias

- [Cómo establecer cuotas del sistema de archivos en Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-set-filesystem-quotas-on-ubuntu-20-04).
- `chatgpt.com`.
