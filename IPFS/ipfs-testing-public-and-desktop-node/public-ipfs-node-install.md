# Web3 - 101 - IPFS - Probando un Nodo Público y de Escritorio - Instalación de IPFS con docker en un nodo público

Esta es la solución nombrada como `#public-ipfs-node-install`.

## Contexto

Este es un tutorial que forma parte de [Web3 - 101 - IPFS - Probando un Nodo Público y de Escritorio](../README.md)
> Por favor, cualquier referencia o propósito al respecto, te emplazo a leerlo ahí.

## Propósito

Estos son los pasos de configuración e instalación de IPFS en un entorno docker, para ser ofrecido en un servicio público.

El servicio ofrece un [gateway](https://docs.ipfs.tech/concepts/how-ipfs-works/#ipfs-http-gateways) e [IPNS](https://docs.ipfs.tech/concepts/ipns/).

Como gateway, de tipo por ruta, `Path-based`, en la URL: <https://web3-101-ipfs.open3diy.org/ipfs/{CID}>.

Como gateway de tipo subdominio, `Subdomain-based`, en la URL: <https://{CID}.ipfs.web3-101-ipfs.open3diy.org/>.

> `web3-101-ipfs.open3diy.org` es el `root gateway`. Viéndolo en un ejemplo, si configuras el dominio `dominio.tld`, debe resolverse como `{CID}.ipfs.dominio.tld` y `dominio.tld/ipfs/{CID}`.

Como [IPNS](https://docs.ipfs.tech/concepts/ipns) en <https://web3-101-ipfs.open3diy.org/ipns/k51qzi5uqu5di56caajjiel546q92pme0hgnh4gofey4tbwlfdr64ur7vu9s9t>.

Para ser usado como nodo un [peer de nodo persistente](https://docs.ipfs.tech/how-to/peering-with-content-providers/) para otros nodos en la [multiaddr](https://github.com/multiformats/multiaddr): </dnsaddr/web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex>.

> De esta forma permite tener mejor funcionamiento a nodos que tienen problemas para salir al exterior, como los que están detrás de un CGNAT.

Como características:

- Es un gateway [recursivo](https://docs.ipfs.tech/concepts/ipfs-gateway/#gateway-types).
- No requiere [autenticación](https://docs.ipfs.tech/concepts/ipfs-gateway/#authenticated-gateways) y es de [solo lectura](https://docs.ipfs.tech/concepts/ipfs-gateway/#read-only-gateways).
- Como cliente que use el gateway, lo puede usar como [confiable o no confiable](https://docs.ipfs.tech/reference/http/gateway/#trusted-vs-trustless).

> Si quieres saber más, accede a [IPFS estilo de resolución](https://docs.ipfs.tech/concepts/ipfs-gateway/#resolution-styles).

## Solución

La solución es la instalación de IPFS con docker teniendo como referencia la documentación [install IPFS Kubo inside Docker](https://docs.ipfs.tech/install/run-ipfs-inside-docker/), pero con configuraciones especificas tras varias pruebas y consultas que verás finalmente en las [referencias](#referencias).

Si estas pensando en crear un servicio público en producción, te recomiendo tener en cuenta que existe la opción de [Rainbow](https://github.com/ipfs/rainbow/#readme), que es la implementación especifica de gateway.

Esta solución está pensada para hacer pruebas en lo que considero mí laboratorio de aprendizaje, como había intentado explicar en [mí propósito](../README.md#propósito).

## Configuración

- En servidor VPS.
- En [Ubuntu 24.04 LTS](https://ubuntu.com/blog/tag/ubuntu-24-04-lts).

## [Problemas conocidos que debes saber antes](./_public-ipfs-node-install-attachment/known-issues.md#problemas-conocidos-que-debes-saber-antes)

> Accede al enlace si quieres ver los problemas conocidos.

## Pre-requisitos

- [Configuración inicial del servidor de red](../../misc/initial-netServer-configuration.md).
- [Instalación y configuración de docker](../../misc/docker-install-configuration.md).
- [Instalación de proxy inverso traefik](../../misc/inverseProxy-traefik-install.md)

## Pasos

### Agregar permisos firewall ufw

Agregar permisos puerto 4001, que es el puerto por defecto del `swarm` de IPFS:

```bash
sudo ufw allow 4001
```

### Configuración servicio

Crea el usuario que será usado por Docker:

```bash
sudo useradd -M -s /usr/sbin/nologin docker-ipfs
```

Obtener identificadores para usar mas tarde al crear contenedor:

```bash
id docker-ipfs
```

- **salida:**

    ```plaintext
    uid=1002(docker-ipfs) gid=1002(docker-ipfs) groups=1002(docker-ipfs)
    ```

    > En este ejemplo copiaremos los valores `1002` y `1002` como usuario y grupo.

Crear la carpetas para IPFS, para los datos y staging, para el usuario `docker-ipfs`; asignarle como propietario y dar permisos:

```bash
sudo mkdir -p /var/docker-ipfs
# asignar por defecto a las carpetas
sudo setfacl -d -m u::rwX /var/docker-ipfs    # Propietario: rwx
sudo setfacl -d -m g::rwX /var/docker-ipfs     # Grupo: rwx
sudo setfacl -d -m o::--- /var/docker-ipfs     # Otros: sin permisos
# crear resto de directorios
sudo mkdir -p /var/docker-ipfs/data /var/docker-ipfs/staging
sudo chown docker-ipfs:docker-ipfs /var/docker-ipfs -R
sudo chmod u=rwx,g=rwx,o=--- /var/docker-ipfs -R
sudo chmod g+s /var/docker-ipfs
```

Comprueba los permisos finales:

```bash
sudo ls -ll /var/docker-ipfs
```

- **salida:**

  ```plaintext
  drwx------ 2 docker-ipfs docker-ipfs 4096 Feb  1 20:11 data
  drwx------ 2 docker-ipfs docker-ipfs 4096 Feb  1 20:11 staging
  ```

### Crear servicio en `docker-compose`

Edita en `/etc/appserver/docker/docker-compose.yml` la siguiente configuración:

```yaml
services:
  ipfs-host:
    image: ipfs/kubo:latest
    container_name: ipfs-host
    user: "1012:1014"
    volumes:
      - /var/docker-ipfs/staging:/export
      - /var/docker-ipfs/data:/data/ipfs
    ports:
      - "4001:4001"
      - "4001:4001/udp"
      - "127.0.0.1:5001:5001"
    restart: unless-stopped
```

> [Accede a ver la explicación](./_public-ipfs-node-install-attachment/create-service-docker-compose-explanation.md#crear-servicio-en-docker-compose---explicación).

### Iniciar cambios

Verificar la configuración inicial:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml config
```

> Veremos la salida si no hay fallos de sintaxis, pero sino, veremos un error que nos avise.

Ejecutar nuevo servicio sin detener servicios en ejecución:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml up -d ipfs-host
```

Verificar procesos en ejecución:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml ps
```

- **salida:**

  ```plaintext
  ipfs-host          ipfs/kubo:latest         "/sbin/tini -- /usr/…"   ipfs-host       10 seconds ago   Up 9 seconds (healthy)   0.0.0.0:4001->4001/tcp, 0.0.0.0:4001->4001/udp, :::4001->4001/tcp, :::4001->4001/udp, 127.0.0.1:5001->5001/tcp, 8080-8081/tcp
  ```

### [Revisar log de `IPFS`, copiar `PeerID` y explicación](./_public-ipfs-node-install-attachment/check-IPFSlog-PeerID-explanation.md#revisar-log-de-ipfs-copiar-peerid-y-explicación)

> Accede al enlace para ver el paso necesario.

### Crear configuración de IPFS en el host

Dentro de `/var/docker-ipfs/data` del host dejamos contenido de archivos necesarios para IPFS, pero también está la configuración de inicio del servicio IPFS, en el archivo `/var/docker-ipfs/data/config`, que normalmente se modifica con el API; pero en este tutorial, se considera configuración de solo lectura y solo editable por el propio administrador del servidor de red, por lo tanto, seguiremos estos pasos:

Crear carpeta de configuración:

```bash
mkdir -p /etc/appserver/ipfs
sudo chown nobody:infrastructure /etc/appserver/ipfs -R
sudo chmod u=rwx,g=rwx,o=--- /etc/appserver/ipfs
```

Enlazar:

```bash
sudo ln -s /var/docker-ipfs/data/config /etc/appserver/ipfs/config
```

Dar permiso a usuarios del grupo `infrastructure`:

```bash
sudo setfacl -m g:infrastructure:rX /var/docker-ipfs
sudo setfacl -m g:infrastructure:rX /var/docker-ipfs/data
sudo setfacl -m g:infrastructure:rw /var/docker-ipfs/data/config
```

Quitar permiso de escritura a `docker-ipfs`:

```bash
sudo setfacl -m u:docker-ipfs:r-- /var/docker-ipfs/data/config
```

Comprobar:

```bash
getfacl /var/docker-ipfs/data/config
```

- **salida:**

  ```plaintext
  group:infrastructure:rw-
  other::---
  ```
  
  > Solo se indica lo que destaca revisar.

```bash
getfacl /etc/appserver/ipfs/config
```

- **salida:**

  ```plaintext
  user:docker-ipfs:rw-
  other::---
  ```

  > Solo se indica lo que destaca revisar.

### Primera verificación de seguridad

> ⚠️ Pendiente de completar, error en el plugin. He creado la issue: <https://github.com/ipfs-shipyard/nopfs/issues/45>

Con el objetivo de que sea un servicio seguro, un primer requisito, es que evite ofrecer contenido ilegal, usando el plugin [noPFS](https://github.com/ipfs-shipyard/nopfs/) y usando la lista <https://badbits.dwebops.pub/badbits.deny>.

En sitio de `noPFS` se explican los pasos, siendo en nuestro caso:

Crear el directorio `denylists`:

```bash
sudo mkdir /var/docker-ipfs/data/denylists
```

Descarga la lista inicial:

```bash
sudo wget -O /var/docker-ipfs/data/denylists/badbits.deny https://badbits.dwebops.pub/badbits.deny
```

Descargar el plugin:

```bash
wget https://github.com/ipfs-shipyard/nopfs/releases/download/nopfs-kubo-plugin%2Fv0.23.0/nopfs-kubo-plugin_v0.23.0_linux_amd64.tar.gz
tar -xvzf nopfs-kubo-plugin_v0.23.0_linux_amd64.tar.gz
```

Prueba Copyright: <https://web3-101-ipfs.open3diy.org/ipfs/QmV3N9Tsk1pZn8hkbgsWdqZVGsfdFfvRC2GeYc3gw77TpU>

### Anunciar la multiaddr pública al exterior

Previamente, vamos a anunciar nuestro servicio con el DNS `web3-101-ipfs.open3diy.org`, paso ya realizado en [la instalación de proxy inverso - configuración TLS](../misc/netServer-reverseProxy-Nginx-install.md#configuración-seguridad-web-tls-con-lets-encrypt).

Como nos indica el issue [Cluster peers in Docker or behind NATs do not advertise their public IP addresses](https://github.com/ipfs-cluster/ipfs-cluster/issues/949), anunciaremos nuestra multiaddr pública para que otros peers puedan acceder, porque al estar en docker no son anunciadas automáticamente.

Para todo esto, editar la configuración de `IPFS` en `/etc/appserver/ipfs/config` y buscar en `Addresses` la lista de `AppendAnnounce` (que inicialmente está vacío).

Partiendo del ejemplo donde tenemos el dominio público `web3-101-ipfs.open3diy.org`, debemos agregar esta lista, quedando finalmente como:

```json
    "AppendAnnounce": [
        "/dnsaddr/web3-101-ipfs.open3diy.org/tcp/4001",
        "/dnsaddr/web3-101-ipfs.open3diy.org/udp/4001/quic-v1",
        "/dnsaddr/web3-101-ipfs.open3diy.org/udp/4001/quic-v1/webtransport",
        "/dnsaddr/web3-101-ipfs.open3diy.org/udp/4001/webrtc-direct/"
    ],
```

> Se ha indicado un dominio, pero podría ser la IP pública, siendo por ejemplo `/ip4/57.129.131.125/tcp/4001`, etc..

### Activar el nodo como realay y anunciarlo

Un [relay](https://docs.ipfs.tech/concepts/nodes/#relay) es la forma en la otros nodos como un `IPFS Desktop` que están detrás de un CGNAT puedan ser accedidos.

Editar la configuración de `IPFS` en `/etc/appserver/ipfs/config` y buscar en `Addresses` la lista de `AppendAnnounce` (que inicialmente está vacío).

Partiendo de nuestro ejemplo, agregar la multiaddr de relay `/dnsaddr/web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit`, donde:

- `dnsaddr`: se trata de una dirección en un DNS.
- `pfs.web3-101-ipfs.open3diy.org`: es el dominio DNS del nodo IPFS donde estamos instalando.
- `12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex`: es el `PeerID` del nodo instalado.

Debería quedar así:

```json
"AppendAnnounce": [
    Etc...
    "/dnsaddr/web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit"    
],
```

Luego en `Swarm` (casi al final) en `RelayService`, agregar `Enabled` a `true`:

```json
"RelayService": {
      "Enabled": true
    },
```

### Configurar el nodo como gateway

Para crear el nodo como gateway público, editar la configuración de `IPFS` en `/etc/appserver/ipfs/config` y buscar en `Gateway` la lista de `PublicGateways`, agregar el dominio principal:

```yaml
"Gateway": {
   Etc...
    "PublicGateways": {
      "web3-101.open3diy.org": {
        "UseSubdomains": true,
        "Paths": ["/ipfs", "/ipns"]
        }
      },
    Etc..
  },
```

En este ejemplo, `web3-101.open3diy.org` es el dominio base para los subdominios `ipfs.web3-101-ipfs.open3diy.org` e `ipns.web3-101-ipfs.open3diy.org`.

Igualmente es el dominio base para las rutas `web3-101.open3diy.org/ipfs/{CID}` o `web3-101.open3diy.org/ipns/{peer_id_or_dnslink}`.

Para entenderlo mejor, puedes ir a las URLs de ejemplo del [propósito enunciado](#propósito).

### Revisar limitaciones de disco

Por defecto el espacio máximo entre caché, temporales, metadatos o contenido agregado como PIN, tiene como máximo `10GB` como se indica en el archivo `/etc/appserver/ipfs/config` en `Datastore` en entrada `StorageMax`:

```json
    etc..
    "Datastore": {
        etc.. 
        "StorageMax": "10GB"
```

Podemos cambiarlo por otro valor, pero para esta instalación lo dejaré así.

### [Revisión final](./_public-ipfs-node-install-attachment/final-review.md#revisión-final)

> Accede al enlace si quieres ver la revisión final.

### Administrar el nodo remoto en nuestro equipo con SSH

Por comodidad, para administrar el nodo VPS remoto desde nuestro equipo de habitudal, podemos re-dirigir en una conexion SSH el puerto 5001.

> Tener activo SSH no sería lo más seguro para un nodo de producción, pero inicialmente y para este nodo de pruebas, para revisar que todo funciona correctamente, es la opción más rápida. Luego podremos desactivar SSH si queremos.

Abre un túnel SSH desde un terminal del cliente:

```bash
ssh -L 5001:127.0.0.1:5001 usuario@ip_del_vps
```

> Remplaza `usuario` e `ip_del_vps` por los valores correspondientes a tu conexión SSH.

Accede a la WebUI desde tu navegador local, escribe la direccion: `http://127.0.0.1:5001/webui`.

Aprovecha a subir un documento, haz PIN y copia el CID porque nos vendrá bien para la prueba que haremos a continuación.

En el terminal donde iniciaste, para cerrar escribe `logout` en la conexión remota.

### [Crear configuración de Ngins para IPFS](_public-ipfs-node-install-attachment/nginx_ipfs_config.md#crear-configuración-de-ngins-para-ipfs)

> Accede al enlace si quieres ver como configurar nginx.

### Vincular un nombre de dominio legible con un CID de IPFS con DNSLink

Opcionalmente, si asi lo queremos, para que al acceder a <https:/web3-101-ipfs.open3diy.org> aparezca un contenido de por defecto en base a un CID, podemos usar [DNSLink](https://docs.ipfs.tech/concepts/dnslink/).

Los pasos se realizarán en nuestro proveedor de DNS, en nuestro caso `cloudfare` y los pasos son sencillos:

Crear registro `TXT` con nombre `_dnslink.web3-101-ipfs` y valor `dnslink=/ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m`.

Verificar:

```bash
dig +short TXT _dnslink.web3-101-ipfs.open3diy.org
```

- **salida:**

  ```plaintext
  "dnslink=/ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m"
  ```

## Referencias

- [Install IPFS Kubo inside Docker](https://docs.ipfs.tech/install/run-ipfs-inside-docker/).
- [Referencia de IPFS HTTP Gateway](https://docs.ipfs.tech/reference/http/gateway/).
- [Referencia configuración `IPFS`](https://github.com/ipfs/kubo/blob/master/docs/config.md)
- [Configure NAT and port forwarding](https://docs.ipfs.tech/how-to/nat-configuration/#background).
- [Nat traversal](https://docs.libp2p.io/concepts/nat/).
- [libp2p relay](https://docs.libp2p.io/concepts/nat/circuit-relay/).
- [Issue cluster peers in Docker or behind NATs do not advertise their public IP addresses](https://github.com/ipfs-cluster/ipfs-cluster/issues/949),
- [IPFS](https://ipfs.tech/).
- [IPFS Infrastructure nginx](https://github.com/ipfs/infra/blob/master/ipfs/gateway/nginx.conf).
- [PFS Infrastructure](https://github.com/ipfs/infra/blob/master/README.md).
- [IPFS Content Blocking in Kubo](https://github.com/ipfs/kubo/blob/master/docs/content-blocking.md).
- <chatgpt.com>.
