# Conifiguración inicial de un servidor de red

Esta es la solucion nombrada como `#initial-netServer-configuration`.

## Contexto

Este es un tutorial de [Web3 - IPFS](../README.md).

## Propósito

El [servidor](https://es.wikipedia.org/wiki/Servidor) donde se alojaran las aplicaciones o componentes que sirvan a la red, será inicialmente configurado. Esta solución agrupa todos estas configuraciones e instalaciones necesarias.

## Configuración

- En [Ubuntu 24.04 LTS](https://ubuntu.com/blog/tag/ubuntu-24-04-lts).

## Problemas conocidos que debes saber antes

### No olvidar actualizar e instalar los paquetes debian

```bash
sudo apt update
sudo apt upgrade -y
```

> Hacer casos a los avisos, no ignorarlos o luego todo será peor.

## Pasos

### Crear configuración del servidor

El usuario que realizará todos los pasos estará en un grupo llamdo `infrastructure` y en principio necesita tener permiso de `sudo` para crear cuentas de usuario o asignar permisos. 
> En todos los ejemplos de este tutorial se llamará `jesus`.

Instalar `acl` Access Control list, para afinar los permisos:

```bash
sudo apt install acl
```

Crear el grupo `infrastructure` el cual se usará para los diferentes servicios:

```bash
sudo groupadd infrastructure
```

Agregar el usuario operador al grupo:

```bash
sudo usermod -aG infrastructure jesus
```

Crear la carpeta de configuración principal para las aplicaciones `/etc/appserver`, para logs en `/var/log` (si no existe) y servir contenido en `/srv`:

```bash
sudo mkdir /etc/appserver /var/log /srv
```

Cambiar el grupo propietario de las carpetas `/etc/appserver` y `/srv`:

```bash
sudo chown nobody:infrastructure /etc/appserver /srv -R
```

> Como `/var/log` puede existir con anterioridad, no cambiamos el propietario porque fallarian logs que ya se escriban.

Configurar los permisos base de escritura a las carpetas para el grupo y ninguno para el resto y establecer el bit setgid:

```bash
sudo chmod u=rwx,g=rwx,o=--- /etc/appserver /srv
sudo chmod g+s /etc/appserver /srv               # Activar el bit setgid
```

Configurar ACLs para garantizar herencias de subdirectorios y archivos

```bash
sudo setfacl -d -m u::rwx /etc/appserver /srv     # Propietario: rwx
sudo setfacl -d -m g::rwx /etc/appserver /srv     # Grupo: rwx
sudo setfacl -d -m o::--- /etc/appserver /srv     # Otros: sin permisos
```

#### Suposición de casos de permisos

**El propietario es la infraestructura**

Cuando es configuración inicial de un servicio, los propietarios son los del grupo `infrastructure` y debemos crear el usuario que inicia el servicio, al que damos permisos minimos. Viendolo en un ejemplo:

Crear configuración:

```bash
sudo mkdir /etc/appserver/app1
```

Asignar permiso solo al grupo `infrastructure` permitiendo heredar estos permisos:

```bash
sudo chown nobody:infrastructure /etc/appserver/app1 -R
sudo chmod u=rwx,g=rwx,o=--- /etc/appserver/app1 -R
sudo chmod g+s /etc/appserver /etc/appserver/app1 -R
sudo setfacl -d -m u::rwX /etc/appserver/app1     # Propietario: rwx
sudo setfacl -d -m g::rwX /etc/appserver/app1     # Grupo: rwx
sudo setfacl -d -m o::--- /etc/appserver/app1     # Otros: sin permisos
```

Crear usuario de la aplicación concreta:

```bash
sudo useradd -M -s /usr/sbin/nologin app1
```

Según su caso, crear el archivo de configuracion para asignar permiso al usuario `app1`:

```bash
touch /etc/appserver/app1/init.config
```

Dar permiso concretos con `setfacl` a la carpeta para que pueda navegar y permiso al archivo de configuración:

```bash
sudo setfacl -m u:app1:X /etc/appserver/app1
sudo setfacl -m u:app1:r /etc/appserver/app1/init.config
```

En el caso que queramos dar permisos mínimos, pero queremos el usuario tenga permiso en toda la carpeta para leer de forma heredada, por ejemplo, si es contenido estático como contenido HTML que vamos creando, podemos usar:

```bash
sudo setfacl -d -m d:u:app1:rX /srv/www
```

**El propietario es la propia aplicación**

Cuando es la propia aplicación la que necesita albergar en el host información variable como logs, o guardar su estado por si la aplicación se renicia, o cualquier otro contenido que en realidad concierte a la aplicación, crearemos un usuario al que le asignaremos como propietario, permitiendo al grupo `infrastructure` ver el contenido. Viendolo con un ejemplo, suponiendo que es la carpeta de logs:

Crearemos una carpeta para la aplicacion:

```bash
sudo mkdir /var/log/app1
```

Cambiar el propietario de las carpetas `/var/log/app1` para el usuario `app1` y el grupo `infrastructure` permitiendo herdar estos permisos:

```bash
sudo chown app1:infrastructure /var/log/app1 -R
sudo chmod u=rwx,g=rx,o=--- /var/log/app1
sudo chmod g+s /var/log/app1
sudo setfacl -d -m u::rwX /var/log/app1           # Propietario: rwX (navegar)
sudo setfacl -d -m g::rX /var/log/app1            # Grupo: rX (navegar)
sudo setfacl -d -m o::--- /var/log/app1           # Otros: sin permisos
```

### Establecer zona horara UTC en el servidor

Se sigue la recomendación de establecer la zona de referencia UTC, con los pasos:

Establecer la zona horaria a UTC:

```bash
sudo timedatectl set-timezone UTC
```

Verificar:

```bash
timedatectl
```

> Ahora debería mostrar `Time zone: UTC`.

### Sincronización horaria con `systemd-timesyncd`

Para tener sincronizada la hora, activar la sincronización de tiempo:

```bash
sudo timedatectl set-ntp true
```

Verificar finalmente

```bash
timedatectl status
```

* **salida:**

    ```plaintext
        Time zone: UTC (UTC, +0000)
        NTP service: active
    ```

## Pasos `Hardening` del servidor de red

> Un aspecto muy mejorable en estos tutoriales DIY, pero que esperemos que vayan mejorando con el tiempo.

El hardening es un conjunto de buenas prácticas, configuraciones, y medidas específicas diseñadas para reducir las vulnerabilidades de un sistema o servidor y protegerlo contra ataques. 

Es más que una simple checklist: es un proceso continuo y adaptativo que consta de:
- Hardening como proceso operativo:
    - Análisis inicial: Identificar servicios, configuraciones, y aplicaciones en el servidor que puedan ser vulnerables.
    - Implementación de medidas: Aplicar configuraciones de seguridad, deshabilitar servicios innecesarios y endurecer accesos.
    - Monitoreo continuo: Revisar regularmente logs y configuraciones para detectar nuevas vulnerabilidades o necesidades de ajuste.
- Hardening como checklist, destacando:
    - Configuración de firewall.
    - Protección de puertos y servicios.
    - Actualizaciones regulares.
    - Políticas de contraseñas.
    - Seguridad de acceso remoto (como SSH).
- Hardening como buenas prácticas:
    - Recomendaciones de organizaciones como CIS (Center for Internet Security) o OWASP.
    - Experiencia operativa y análisis de amenazas comunes.
    - Aplicación de políticas de mínimo privilegio y principios de seguridad por diseño.

Estas medidas de seguridad se aplicarán mediante las siguientes iteraciones:
- [iteración #1](./initial-netServer-hardening/hardering-iteration-1.md).
- [iteración #2](./initial-netServer-hardening/hardering-iteration-2.md).


## Referencias

- [curso web3Mba - seguridad](https://www.web3mba.io/).
- [Ubuntu-Server-Hardening](https://gist.github.com/cybergitt/caf4451ad3f231735d97a1a42a1e88db).
- `chatgpt.com`.

