# Web3 - 101 - IPFS - Probando un Nodo Público y de Escritorio - Instalación de IPFS con docker en un nodo público

Esta es la solucion nombrada como `#public-ipfs-node-install`.

## Contexto

Este es un tutorial que forma parte de [Web3 - 101 - IPFS - Probando un Nodo Público y de Escritorio](../README.md)
> Por favor, cualquier referencia o proposito al respecto, te emplazo a leerlo ahí.

## Propósito

Estos son los pasos de configuración e instalación de IPFS en un entorno docker, para ser ofrecido en un servicio público.

El servicio ofrece un [gateway](https://docs.ipfs.tech/concepts/how-ipfs-works/#ipfs-http-gateways) e [IPNS](https://docs.ipfs.tech/concepts/ipns/).

Como gateway, de tipo por ruta, `Path-based`, en las URLs: <https://ipfs.web3-101-ipfs.open3diy.org/ipfs/{CID}> / <https://web3-101-ipfs.open3diy.org/ipfs/{CID}>.

> Haciendo pruebas, se ha comprobado que como requisito para ser agregado como gateway en la [webui](https://webui.ipfs.io/#/settings), es necesario que resuelva esas URLs. Viendolo en un ejemplo, si configuras el dominio `dominio.tld`, debe resolverse como `{CID}.ipfs.dominio.tld` y `dominio.tld/ipfs/{CID}`.

Como gateway de tipo subdominio, `Subdomain-based`, en la URL: <https://{CID}.ipfs.web3-101-ipfs.open3diy.org/>.

> Si quieres saber, accede a [IPFS estilo de resolución](https://docs.ipfs.tech/concepts/ipfs-gateway/#resolution-styles).

Como [IPNS](https://docs.ipfs.tech/concepts/ipns) en <https://ipfs.web3-101-ipfs.open3diy.org/ipns/k51qzi5uqu5di56caajjiel546q92pme0hgnh4gofey4tbwlfdr64ur7vu9s9t>.

Para ser usado como nodo un [peer de nodo persistente](https://docs.ipfs.tech/how-to/peering-with-content-providers/) para otros nodos en la [multiaddr](https://github.com/multiformats/multiaddr): </dnsaddr/ipfs.web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex>.

> De esta forma permite tener mejor funcionamiento a nodos que tienen problemas para salir al exterior, como los que están detrás de un CGNAT.

Como caracteristicas:

- Es un gateway [recursivo](https://docs.ipfs.tech/concepts/ipfs-gateway/#gateway-types).
- No requiere [autenticación](https://docs.ipfs.tech/concepts/ipfs-gateway/#authenticated-gateways) y es de [solo lectura](https://docs.ipfs.tech/concepts/ipfs-gateway/#read-only-gateways).
- Como cliente que use el gateway, lo puede usar como [confiable o no confiable](https://docs.ipfs.tech/reference/http/gateway/#trusted-vs-trustless).

## Solución

La solución es la instalación de IPFS con docker teniendo como referencia la documentación [install IPFS Kubo inside Docker](https://docs.ipfs.tech/install/run-ipfs-inside-docker/), pero con configuraciones especificas tras varias pruebas y consultas que verás finalmente en las [referencias](#referencias).

Si estas pensando en crear un servicio público en producción, te recomiendo tener en cuenta que existen ocpiones como:

- [IPFS Infrastructure](https://github.com/ipfs/infra/blob/master/README.md) como solución de la infraestructura de IPFS.
- [Rainbow](https://github.com/ipfs/rainbow/#readme) implementación especifica de gateway.

Esta solución está pensanda para hacer pruebas en lo que considero mí laboratorio de aprendizaje, como habia intentado explicar en [mí propósito](../README.md#propósito).

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

### Permitir a los nodos establecer conexiones directas y autonat

Es importante permitir al nodo realizar conexiones directas cuando se ofrece el servicio relay, para eso la configuracion [`EnableHolePunching`](https://github.com/ipfs/kubo/blob/master/docs/config.md#swarmenableholepunching) indicada a `True` es fundamental. No se indica como paso porque por defecto ya está habilitado, pero por favor, revisa en la configuración que no lo tengas a `False`.

Igualmente, [`Autonat`](https://github.com/ipfs/kubo/blob/master/docs/config.md#autonatservicemode) debe estar establecido a `enabled`, siendo ya la configuración por defecto, motivo por el que no se indica como paso, pero revisa por favor que no lo tengas a `disabled`.

### La configuración indicada en este tutorial provoca error

Siempre tener como referencia [la referencia configuración `IPFS`](https://github.com/ipfs/kubo/blob/master/docs/config.md) porque puede cambiar y afectar a esta explicación.

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

Crear la careptas para IPFS, para los datos y staging, para el usuario `docker-ipfs`; asignarle como propietario y dar permisos:

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

Explicación:

- `restart unless-stopped`: Docker reinicia el contenedor después de un reinicio del sistema o fallo, pero no lo hará si lo detuviste manualmente con comandos como docker stop.
- `user 1012:1014"`: Por seguridad iniciar el contenedor para un usuario concreto creado para este contenedor.
- `container_name ipfs-host`: Asigna el nombre ipfs-host al contenedor para identificarlo.
- `-v ...:/export`: Monta una carpeta local (ipfs_staging) en el contenedor para almacenar archivos que quieras importar/exportar a IPFS.
- `-v ...:/data/ipfs`: Monta una carpeta local para almacenar los datos persistentes de IPFS (como el almacenamiento de bloques).
- En `ports` de `4001:4001` y `4001:4001/udp`: Expone el puerto 4001 (TCP y UDP) para conexiones entre nodos en la red IPFS (intercambio de bloques y descubrimiento de peers) para todas las interfaces disponibles para salir al exterior.
  - Indicamos los puertos, pero es cierto que si hubieramos iniciamos con la opción `--network host` nos habriamos ahorrado todo esto, pero por seguridad, he preferido aplicar esta solución.
- En `ports` `127.0.0.1:5001:5001`: Expone el puerto 5001 para la API de control de IPFS, accesible únicamente desde el host (loopback 127.0.0.1).

Aclaracion:

- En `ports` podría poner `127.0.0.1:8080:8080`para el gateway HTTP de IPFS, accesible únicamente desde el host (loopback 127.0.0.1), pero irá publicado en el proxy inverso y no es necesario.

**¿Qué es el gateway de IPFS?**

El gateway HTTP de IPFS permite acceder a los contenidos almacenados en la red IPFS mediante un navegador web o peticiones HTTP.

Ver el artículo [ipfs gateway](https://docs.ipfs.tech/concepts/ipfs-gateway/#gateway-request-lifecycle).
> A continuación, lo incluiremos en el proxy inverso traefik.

**¿Qué es el API expuesto el puerto 5001?**

El [API de kubo](https://docs.ipfs.tech/reference/kubo/rpc/) es una interfaz que permite interactuar y controlar un nodo IPFS programáticamente.

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

### Revisar el registro de log de `IPFS` y copiar `PeerID`

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml logs ipfs-host
```

- **salida:**

  ```plaintext
  ipfs version 0.32.1
  generating ED25519 keypair...done
  peer identity: 12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex
  initializing IPFS node at /data/ipfs
  Initializing daemon...
  Kubo version: 0.32.1-9017453
  Repo version: 16
  System version: amd64/linux
  Golang version: go1.23.3
  PeerID: 12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex
  Swarm listening on 127.0.0.1:4001 (TCP+UDP)
  Swarm listening on 172.17.0.2:4001 (TCP+UDP)
  Swarm listening on [::1]:4001 (TCP+UDP)
  Run 'ipfs id' to inspect announced and discovered multiaddrs of this node.
  RPC API server listening on /ip4/0.0.0.0/tcp/5001
  WebUI: http://127.0.0.1:5001/webui
  Gateway server listening on /ip4/0.0.0.0/tcp/8080
  Daemon is ready
  ```

Explicación:

- `generating ED25519 keypair...done`:
Se genera un par de claves ED25519 para identificar el nodo IPFS de manera única. Es un mecanismo de criptografía de curva elíptica.

- `peer identity: 12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex` / `PeerID: 12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex`:
Esta es la identidad del nodo (Peer ID), derivada de la clave pública generada. Es única y sirve para identificar tu nodo en la red IPFS.

- `Swarm listening on 127.0.0.1:4001 (TCP+UDP)` / `Swarm listening on 172.17.0.2:4001 (TCP+UDP)`:
El nodo escucha conexiones locales en el puerto 4001 mediante TCP y UDP. Esto permite la comunicación con otros nodos.

> Esto en nuesto caso es un problema, porque queriamos que otros nodos lo puedan acceder, pero luego lo revisaremos...

- `Swarm listening on [::1]:4001 (TCP+UDP)`:
Muestra que el nodo está escuchando en IPv6 localhost ([::1]) en el puerto 4001 en todas las interfaces, saliendo al exterior. Esto es correcto.

- `Run 'ipfs id' to inspect announced and discovered multiaddrs of this node.`:
Sugiere ejecutar el comando `ipfs id` para ver las direcciones multiaddr anunciadas al resto de peers. Es evidente que es necesario revisarlo porque como veremos no se anuncian las multiaddr de en una IP o dominio público al exterior, pero esto ya lo veremos luego...

- `RPC API server listening on /ip4/0.0.0.0/tcp/5001`:
Indica que el servidor de API RPC (Remote Procedure Call) está escuchando en todas las interfaces de red (0.0.0.0) en el puerto TCP 5001 dentro del contenedor. Esto es correcto porque está IPFS en un contenedor, no estás exponiendo nada al exterior porque además ya indicamos al iniciar el contenedor `-p 127.0.0.1:5001:5001`.

- `WebUI: http://127.0.0.1:5001/webui`:
Proporciona la URL para acceder a la interfaz de usuario web (WebUI) del nodo IPFS.

- `Gateway server listening on /ip4/0.0.0.0/tcp/8080`:
El servidor de gateway HTTP está escuchando en todas las interfaces de red (0.0.0.0), esto es correcto porque está IPFS en un contenedor, no estás exponiendo nada al exterior.
Además indica que es el puerto 8080, pero no es donde queriamos, luego lo cambiaremos al puerto 8081.

Copiar el valor de `PeerID`, lo usaremos para que otros nodos puedan agregarlo como peer. En este ejemplo, sería el valor `12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex`:

```plaintest
PeerID: 12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex
```

### Crear configuración de IPFS en el host

Dentro de `/var/docker-ipfs/data` del host dejamos contenido de archivos necesarios para IPFS, pero también está la configuración de inicio del servicio IPFS, en el archivo `/var/docker-ipfs/data/config`, que normalmente se modifica con el API; pero en este tutorial, se considera configuración de solo lectura y solo editable por el propio administrador del servidor de red, por lo tanto, seguiremos estos pasos:

Crear carpeta de configuracion:

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

### Primera varificación de seguridad

Con el objetivo de que sea un servicio seguro, un primer requisito, es que evite ofrezer contenido ilegal.

Como un primer paso, tomamos como referencia la lista de CID denegados dentro del sitio github de infraestructura de IPFS, [ipfs deny list](https://github.com/ipfs/infra/blob/master/ipfs/gateway/denylist.conf). Seguir estos pasos:

Crear directorio de configuración:

```bash
mkdir -p /etc/appserver/nginx/conf.d/gateway-ipfs
```

Bajar el contenido:

```bash
wget -O /etc/appserver/nginx/conf.d/gateway-ipfs/denylist.conf https://raw.githubusercontent.com/ipfs/infra/master/ipfs/gateway/denylist.conf
```

### Anunciar la multiaddr pública al exterior

Previamente, vamos a anunciar nuestro servicio con el DNS `ipfs.web3-101-ipfs.open3diy.org`, paso ya realizado en [la instalación de proxy inverso - configuración TLS](../misc/netServer-reverseProxy-Nginx-install.md#configuración-seguridad-web-tls-con-lets-encrypt).

Como nos indica el issue [Cluster peers in Docker or behind NATs do not advertise their public IP addresses](https://github.com/ipfs-cluster/ipfs-cluster/issues/949), anunciaremos nuestra multiaddr pública para que otros peers puedan acceder, porque al estar en docker no son anunciadas automaticamente.

Para todo esto, editar la configuración de `IPFS` en `/etc/appserver/ipfs/config` y buscar en `Addresses` la lista de `AppendAnnounce` (que inicialmente está vacío).

Partiendo del ejemplo donde tenemos el dominio público `ipfs.web3-101-ipfs.open3diy.org`, debenos agregar esta lista, quedando finalmente como:

```json
    "AppendAnnounce": [
        "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/tcp/4001",
        "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/udp/4001/quic-v1",
        "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/udp/4001/quic-v1/webtransport",
        "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/udp/4001/webrtc-direct/"
    ],
```

> Se ha indicado un dominio, pero podría ser la IP pública, siendo por ejemplo `/ip4/57.129.131.125/tcp/4001`, etc..

### Activar el nodo como realay y anunciarlo

Un [relay](https://docs.ipfs.tech/concepts/nodes/#relay) es la forma en la otros nodos como un `IPFS Desktop` que están detrás de un CGNAT puedan ser accedidos.

Editar la configuración de `IPFS` en `/etc/appserver/ipfs/config` y buscar en `Addresses` la lista de `AppendAnnounce` (que inicialmente está vacío).

Partiendo de nuestro ejemplo, agregar la multiaddr de relay `/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit`, donde:

- `dnsaddr`: se trata de una dirección en un DNS.
- `pfs.web3-101-ipfs.open3diy.org`: es el dominio DNS del nodo IPFS donde estamos instalando.
- `12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex`: es el `PeerID` del nodo instalado.

Debería quedar así:

```json
"AppendAnnounce": [
    Etc...
    "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit"    
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

En este ejemplo, `web3-101.open3diy.org` es el dominio base para `ipfs.web3-101-ipfs.open3diy.org` e `ipns.web3-101-ipfs.open3diy.org`, pero al configurarlo, debemos agregarlo así.

### Revisar limitaciones de disco y ancho de bada

Por defecto el espacio maximo entre caché, temporales, metadatos o contenido agreagado como PIN, tiene como máximo `10GB` como se indica en el archivo `/etc/appserver/ipfs/config` en `Datastore` en entrada `StorageMax`:

```json
    etc..
    "Datastore": {
        etc.. 
        "StorageMax": "10GB"
```

Podemos cambiarlo por otro valor, pero para esta instalación lo dejaré así.

### Revisión final, guardar cambios y verificar

Por si queda alguna duda, `/etc/appserver/ipfs/config` debería quedar así finalmente con todos los cambios aplicados, siguiendo nuestro ejemplo:

```json
{
  "API": {
    "HTTPHeaders": {}
  },
  "Addresses": {
    "API": "/ip4/0.0.0.0/tcp/5001",
    "Announce": [],
    "AppendAnnounce": [
      "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/tcp/4001",
      "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/udp/4001/quic-v1",
      "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/udp/4001/quic-v1/webtransport",
      "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/udp/4001/webrtc-direct/",
      "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit"
    ],
    "Gateway": "/ip4/0.0.0.0/tcp/8080",
    "NoAnnounce": [],
    "Swarm": [
      "/ip4/0.0.0.0/tcp/4001",
      "/ip6/::/tcp/4001",
      "/ip4/0.0.0.0/udp/4001/webrtc-direct",
      "/ip4/0.0.0.0/udp/4001/quic-v1",
      "/ip4/0.0.0.0/udp/4001/quic-v1/webtransport",
      "/ip6/::/udp/4001/webrtc-direct",
      "/ip6/::/udp/4001/quic-v1",
      "/ip6/::/udp/4001/quic-v1/webtransport"
    ]
  },
  "AutoNAT": {},
  etc...
  },
    "Gateway": {
    "DeserializedResponses": null,
    "DisableHTMLErrors": null,
    "ExposeRoutingAPI": null,
    "HTTPHeaders": {},
    "NoDNSLink": false,
    "NoFetch": false,
    "PublicGateways": {
      "web3-101.open3diy.org": {
        "UseSubdomains": true,
        "Paths": ["/ipfs", "/ipns"]
        }
      },
    "RootRedirect": ""
  },
  etc..
  },
  "Swarm": {
    "AddrFilters": null,
    "ConnMgr": {},
    "DisableBandwidthMetrics": false,
    "DisableNatPortMap": false,
    "RelayClient": {},
    "RelayService": {
      "Enabled": true
    },
    "ResourceMgr": {},
    "Transports": {
      "Multiplexers": {},
      "Network": {},
      "Security": {}
    }
  },
  "Version": {}
}
```

Reiniciar el contenedor ipfs-host:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml restart ipfs-host
```

Revisar el log de nuevo y ver que no hay errores:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml logs ipfs-host
```

- **salida:**

  ```plaintext
  ipfs version 0.32.1
  Found IPFS fs-repo at /data/ipfs
  Initializing daemon...
  Kubo version: 0.32.1-9017453
  Repo version: 16
  System version: amd64/linux
  Golang version: go1.23.3
  PeerID: 12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex
  Swarm listening on 127.0.0.1:4001 (TCP+UDP)
  Swarm listening on 172.17.0.2:4001 (TCP+UDP)
  Swarm listening on [::1]:4001 (TCP+UDP)
  Run 'ipfs id' to inspect announced and discovered multiaddrs of this node.
  RPC API server listening on /ip4/0.0.0.0/tcp/5001
  WebUI: http://127.0.0.1:5001/webui
  Gateway server listening on /ip4/0.0.0.0/tcp/8081
  Daemon is ready
  ```

Verificar direcciones de escucha y módulos cargados:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec ipfs-host ipfs id
```

- **salida:**

  ```plaintext
  {
      "ID": "12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
      "PublicKey": "CAESIAeUPT9UWoCFOOkKUfyt+76Ucu5jZ3OLb23w0p4M9lST",
      "Addresses": [
              "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/udp/4001/quic-v1/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/127.0.0.1/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/127.0.0.1/udp/4001/quic-v1/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/127.0.0.1/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/127.0.0.1/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/172.17.0.2/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/172.17.0.2/udp/4001/quic-v1/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/172.17.0.2/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/172.17.0.2/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/57.129.131.125/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip6/::1/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip6/::1/udp/4001/quic-v1/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip6/::1/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip6/::1/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex"
      ],
      "AgentVersion": "kubo/0.32.1/9017453/docker",
      "Protocols": [
              "/ipfs/bitswap",
              "/ipfs/bitswap/1.0.0",
              "/ipfs/bitswap/1.1.0",
              "/ipfs/bitswap/1.2.0",
              "/ipfs/id/1.0.0",
              "/ipfs/id/push/1.0.0",
              "/ipfs/kad/1.0.0",
              "/ipfs/lan/kad/1.0.0",
              "/ipfs/ping/1.0.0",
              "/libp2p/autonat/1.0.0",
              "/libp2p/autonat/2/dial-back",
              "/libp2p/autonat/2/dial-request",
              "/libp2p/circuit/relay/0.2.0/hop",
              "/libp2p/circuit/relay/0.2.0/stop",
              "/libp2p/dcutr",
              "/x/"
      ]
  }
  ```

  Es importante verificar la salida en los siguientes puntos:

  - Revisar que está bien anunciadas la dirección al exterior:
        Partiendo de la IP de escucha `127.0.0.1`, ver como como se anuncian las direcciones:

      ```plaintest
      "/ip4/127.0.0.1/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
      "/ip4/127.0.0.1/udp/4001/quic-v1/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
      "/ip4/127.0.0.1/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
      "/ip4/127.0.0.1/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
      ```

  - Revisar que tenemos los mismos elementos pero para nuestra direción pública, y adicionalmente incluyendo la multiaddr del relay:

    ```plaintest
    "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
    "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
    "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/udp/4001/quic-v1/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
    "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
    "/dnsaddr/ipfs.web3-101-ipfs.open3diy.org/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
    ```

  - Veremos como están los elementos comunes `p2p`, `quic-v1/p2p`, `quic-v1/webtransport/` y `webrtc-direct/`, con el añadido del relay que es la dirección que contiene `p2p-circuit/`.

  - Revisar que está los `Protocols` necesarios para `Autonat`:

    ```plaintest
    "/libp2p/autonat/1.0.0",
    "/libp2p/autonat/2/dial-back",
    "/libp2p/autonat/2/dial-request",
    ```

  - Revisar que están los `Protocols` necesarios para relay como servidor:

    ```plaintest
    "/libp2p/circuit/relay/0.2.0/hop",
    "/libp2p/circuit/relay/0.2.0/stop",
    ```

### Administrar el nodo remoto en nuestro equipo con SSH

Por comodidad, para administrar el nodo VPS remoto desde nuestro equipo de habitudal, podemos re-dirigir en una conexion SSH el puerto 5001.

> Tener activo SSH no sería lo más seguro para un nodo de producción, pero inicialmente y para este nodo de pruebas, para revisar que todo funciona correctamente, es la opción más rápida. Luego podremos desactivar SSH si queremos.

Abre un túnel SSH desde un terminal del cliente:

```bash
ssh -L 5001:127.0.0.1:5001 usuario@ip_del_vps
```

> Remplaza `usaurio` e `ip_del_vps` por los valores correspondientes a tu conexión SSH.

Accede a la WebUI desde tu navegador local, escribe la direccion: `http://127.0.0.1:5001/webui`.

Aprobecha a subir un documento, haz PIN y copia el CID porque nos vendrá vien para la prueba que haremos a continuación.

En el terminal donde iniciaste, para cerrar escribe `logout` en la conexión remota.

### Crear configuración de Ngins para IPFS

Crear archivo de configuración en `/etc/appserver/nginx/conf.d/ipfs.conf`:

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name *.ipfs.web3-101-ipfs.open3diy.org *.ipns.web3-101-ipfs.open3diy.org;
    
    ssl_certificate /etc/letsencrypt/live/web3-101-ipfs.open3diy.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/web3-101-ipfs.open3diy.org/privkey.pem;

    include conf.d/gateway-ipfs/denylist.conf;

    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 1800s;
        proxy_pass http://ipfs-host:8080;
    }

}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name web3-101-ipfs.open3diy.org;
    
    ssl_certificate /etc/letsencrypt/live/web3-101-ipfs.open3diy.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/web3-101-ipfs.open3diy.org/privkey.pem;

    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    location ~ "^/(ipfs|ipns)(/|$)" {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 1800s;
        proxy_pass http://ipfs-host:8080;
    }
}
```

> Recuerda revisar donde aparezca `web3-101-ipfs.open3diy.org` indicando el dominio que corresponda a tú caso.

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

Prueba con las cabeceras Http necesarias, para la petición como ruta:

```bash
curl -v "https://web3-101-ipfs.open3diy.org/ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m"
```

- **salida:**

  ```plaintext  
  * Host web3-101-ipfs.open3diy.org:443 was resolved.
  * IPv6: (none)
  * IPv4: 57.129.131.125
  *   Trying 57.129.131.125:443...
  * Connected to web3-101-ipfs.open3diy.org (57.129.131.125) port 443
  * ALPN: curl offers h2,http/1.1
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  *  CAfile: /usr/lib/ssl/cert.pem
  *  CApath: /usr/lib/ssl/certs
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
  * TLSv1.3 (IN), TLS handshake, Certificate (11):
  * TLSv1.3 (IN), TLS handshake, CERT verify (15):
  * TLSv1.3 (IN), TLS handshake, Finished (20):
  * TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.3 (OUT), TLS handshake, Finished (20):
  * SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / X25519 / id-ecPublicKey
  * ALPN: server accepted http/1.1
  * Server certificate:
  *  subject: CN=web3-101-ipfs.open3diy.org
  *  start date: Feb 27 16:18:26 2025 GMT
  *  expire date: May 28 16:18:25 2025 GMT
  *  subjectAltName: host "web3-101-ipfs.open3diy.org" matched cert's "web3-101-ipfs.open3diy.org"
  *  issuer: C=US; O=Let's Encrypt; CN=E6
  *  SSL certificate verify ok.
  *   Certificate level 0: Public key type EC/prime256v1 (256/128 Bits/secBits), signed using ecdsa-with-SHA384
  *   Certificate level 1: Public key type EC/secp384r1 (384/192 Bits/secBits), signed using sha256WithRSAEncryption
  *   Certificate level 2: Public key type RSA (4096/152 Bits/secBits), signed using sha256WithRSAEncryption
  * using HTTP/1.x
  > GET /ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m HTTP/1.1
  > Host: web3-101-ipfs.open3diy.org
  > User-Agent: curl/8.5.0
  > Accept: */*
  > 
  * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
  * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
  * old SSL session ID is stale, removing
  < HTTP/1.1 301 Moved Permanently
  < Server: nginx/1.27.4
  < Date: Sun, 02 Mar 2025 18:18:58 GMT
  < Content-Type: text/html; charset=utf-8
  < Content-Length: 135
  < Connection: keep-alive
  < Access-Control-Allow-Headers: Content-Type
  < Access-Control-Allow-Headers: Range
  < Access-Control-Allow-Headers: User-Agent
  < Access-Control-Allow-Headers: X-Requested-With
  < Access-Control-Allow-Methods: GET
  < Access-Control-Allow-Methods: HEAD
  < Access-Control-Allow-Methods: OPTIONS
  < Access-Control-Allow-Origin: *
  < Access-Control-Expose-Headers: Content-Length
  < Access-Control-Expose-Headers: Content-Range
  < Access-Control-Expose-Headers: X-Chunked-Output
  < Access-Control-Expose-Headers: X-Ipfs-Path
  < Access-Control-Expose-Headers: X-Ipfs-Roots
  < Access-Control-Expose-Headers: X-Stream-Output
  < Location: https://bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org/
  < Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
  < 
  <a href="https://bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org/">Moved Permanently</a>.

  * Connection #0 to host web3-101-ipfs.open3diy.org left intact
  ```

Prueba con las cabeceras Http necesarias, para la petición como subdominio:

```bash
curl -v "https://bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org"
```

- **salida:**

  ```plaintext  
  * Host bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org:443 was resolved.
  * IPv6: (none)
  * IPv4: 57.129.131.125
  *   Trying 57.129.131.125:443...
  * Connected to bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org (57.129.131.125) port 443
  * ALPN: curl offers h2,http/1.1
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  *  CAfile: /usr/lib/ssl/cert.pem
  *  CApath: /usr/lib/ssl/certs
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
  * TLSv1.3 (IN), TLS handshake, Certificate (11):
  * TLSv1.3 (IN), TLS handshake, CERT verify (15):
  * TLSv1.3 (IN), TLS handshake, Finished (20):
  * TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.3 (OUT), TLS handshake, Finished (20):
  * SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / X25519 / id-ecPublicKey
  * ALPN: server accepted http/1.1
  * Server certificate:
  *  subject: CN=web3-101-ipfs.open3diy.org
  *  start date: Feb 27 16:18:26 2025 GMT
  *  expire date: May 28 16:18:25 2025 GMT
  *  subjectAltName: host "bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org" matched cert's "*.ipfs.web3-101-ipfs.open3diy.org"
  *  issuer: C=US; O=Let's Encrypt; CN=E6
  *  SSL certificate verify ok.
  *   Certificate level 0: Public key type EC/prime256v1 (256/128 Bits/secBits), signed using ecdsa-with-SHA384
  *   Certificate level 1: Public key type EC/secp384r1 (384/192 Bits/secBits), signed using sha256WithRSAEncryption
  *   Certificate level 2: Public key type RSA (4096/152 Bits/secBits), signed using sha256WithRSAEncryption
  * using HTTP/1.x
  > GET / HTTP/1.1
  > Host: bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org
  > User-Agent: curl/8.5.0
  > Accept: */*
  > 
  * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
  * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
  * old SSL session ID is stale, removing
  < HTTP/1.1 200 OK
  < Server: nginx/1.27.4
  < Date: Sun, 02 Mar 2025 18:22:17 GMT
  < Content-Type: text/plain; charset=utf-8
  < Content-Length: 32
  < Connection: keep-alive
  < Accept-Ranges: bytes
  < Access-Control-Allow-Headers: Content-Type
  < Access-Control-Allow-Headers: Range
  < Access-Control-Allow-Headers: User-Agent
  < Access-Control-Allow-Headers: X-Requested-With
  < Access-Control-Allow-Methods: GET
  < Access-Control-Allow-Methods: HEAD
  < Access-Control-Allow-Methods: OPTIONS
  < Access-Control-Allow-Origin: *
  < Access-Control-Expose-Headers: Content-Length
  < Access-Control-Expose-Headers: Content-Range
  < Access-Control-Expose-Headers: X-Chunked-Output
  < Access-Control-Expose-Headers: X-Ipfs-Path
  < Access-Control-Expose-Headers: X-Ipfs-Roots
  < Access-Control-Expose-Headers: X-Stream-Output
  < Cache-Control: public, max-age=29030400, immutable
  < Etag: "bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m"
  < X-Ipfs-Path: /ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m/
  < X-Ipfs-Roots: bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m
  < Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
  < 
  Hello from IPFS Gateway Checker
  * Connection #0 to host bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org left intact
  ```

Prueba el servicio de gateway con la URL <https://ipfs.web3-101.open3diy.org/ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m> y como subdominio <https://bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org>.

> El valor `bafybeiack3rhfn25rzsvi7s7f3m37zvrwprthspsoitekib3orqxa7ycpi` es el CID que creamos anteriormente o uno cualquiera que podamos explorar.

Luego prueba IPNS <https://ipfs.web3-101-ipfs.open3diy.org/ipns/k51qzi5uqu5di56caajjiel546q92pme0hgnh4gofey4tbwlfdr64ur7vu9s9t>.

Revisar los logs de acceso:

```bash
less +G /var/log/nginx/access.log
```

## Referencias

- [Install IPFS Kubo inside Docker](https://docs.ipfs.tech/install/run-ipfs-inside-docker/).
- [Referencia configuración `IPFS`](https://github.com/ipfs/kubo/blob/master/docs/config.md)
- [Configure NAT and port forwarding](https://docs.ipfs.tech/how-to/nat-configuration/#background).
- [Nat traversal](https://docs.libp2p.io/concepts/nat/).
- [libp2p relay](https://docs.libp2p.io/concepts/nat/circuit-relay/).
- [Issue cluster peers in Docker or behind NATs do not advertise their public IP addresses ](https://github.com/ipfs-cluster/ipfs-cluster/issues/949),
- [IPFS](https://ipfs.tech/).
- [IPFS Infrastructure nginx](https://github.com/ipfs/infra/blob/master/ipfs/gateway/nginx.conf).
- [PFS Infrastructure](https://github.com/ipfs/infra/blob/master/README.md).
- <chatgpt.com>.
