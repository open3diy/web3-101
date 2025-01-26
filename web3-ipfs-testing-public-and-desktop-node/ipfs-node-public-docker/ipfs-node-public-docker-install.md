# Web3 - IPFS - Probando un Nodo Público y de Escritorio - Instalación en docker

Esta es la solucion nombrada como `#ipfs-node-public-docker-install`.

## Contexto

Este es un tutorial que forma parte de [Web3 - IPFS - Probando un Nodo Público y de Escritorio](../README.md)
> Por favor, cualquier referencia o proposito al respecto, te emplazo a leerlo ahí.

Estos son los pasos de configuración e instalación de IPFS en un entorno docker, para que haga de servidor público.

Ofrece:
    - Un [gateway](https://docs.ipfs.tech/concepts/how-ipfs-works/#ipfs-http-gateways) en la URL:.
    - Un [peer de nodo persistente](https://docs.ipfs.tech/how-to/peering-with-content-providers/) para otros nodos en la [multiaddr](https://github.com/multiformats/multiaddr):.

## Solución

La solución es la instalación de IPFS con docker teniendo como referencia la documentación [install IPFS Kubo inside Docker](https://docs.ipfs.tech/install/run-ipfs-inside-docker/), pero con configuraciones especificas tras varias pruebas y consultas en la documentación de [referencia de IPFS](https://github.com/ipfs/kubo/blob/master/docs/config.md).

Igualmente tuve que tener en cuenta muchas cosas que finalmente están en [Referencias](#referencias).

## Configuración

- En servidor VPS.
- En [Ubuntu 24.04 LTS](https://ubuntu.com/blog/tag/ubuntu-24-04-lts).
- Con instalación previa de [Docker](https://www.docker.com/).

## Datos de entrada

- Usuario que iniciará el contenedor: `docker-ipfs`.
- Rutas de espacio en disco requeridas por `IFPS` en el host:
    - ipfs_staging: `/home/jesus/docker-ipfs-files/ipfs_staging`.
    - ipfs_data: `/home/jesus/docker-ipfs-files/ipfs_data`.

## Pre-requisitos

- [Instalación de docker](https://voidnull.es/instalacion-de-docker-en-ubuntu-24-04/).

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

### He creado mal contenedor

En el paso de [crear e iniciar](#crear-e-iniciar-el-contenedor-de-forma-persistente) podemos tener errores, y si queremos empezar de nuevo, lor mejor es seguir estos pasos...

Listas los contenedores ejecutados y ver si está el que hemos creado:

```bash
docker ps
```

Si está mal creado, listar si existe aunque no este iniciado:

```bash
docker ps -a
```

Borrar el contenedor para empezar de nuevo:

```bash
docker rm ipfs_host
```

### Revisar la configuración y estado del contenedor

En el paso de [crear e iniciar](#crear-e-iniciar-el-contenedor-de-forma-persistente) si queremos revisar el estado de la configuración, estos comandos podrán ayudar.

Revisar la configuración de `IPFS`:

```bash
docker exec -it ipfs_host ipfs config show
```

Revisar direcciones multiaddr, sobre detalle sobre cómo otros nodos pueden conectarse al tuyo

```bash
docker exec -it ipfs_host ipfs id
```

## Pasos

### Crear usuario para docker y carpeta en host para archivos IPFS

Crea el usuario que será usado por Docker:

```bash
sudo useradd -m docker-ipfs
```

Crear la carpeta para el usuario `docker-ipfs` y asignarle como propietario:

```bash
sudo mkdir -p /home/jesus/docker-ipfs-files
sudo chown docker-ipfs:docker-ipfs /home/jesus/docker-ipfs-files
```

Establece los permisos para el propietario `docker-ipfs` con solo permisos de lectura y escritura:

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

* **salida:**

    ```plaintext
    drwxr-xr-x 2 docker-ipfs docker-ipfs 4096 Dec 18 12:39 /home/jesus/docker-ipfs-files
    ```
    
### Crear carpeta staging y data para IPFS

Crear carpetas:

```bash
sudo mkdir -p /home/jesus/docker-ipfs-files/ipfs_data /home/jesus/docker-ipfs-files/ipfs_staging
```

Dar como propietario al usuario `docker-ipfs` a cualquier subcarpeta:

```bash
sudo chown -R docker-ipfs:docker-ipfs /home/jesus/docker-ipfs-files
```

Asegurar permisos heredados de la carpeta principal `/home/jesus/docker-ipfs-files` a las 2 supcarpetas:

```bash
sudo chmod g+s /home/jesus/docker-ipfs-files
```

### Obtener uid y gid del usuario que ejcutará el contenedor

Obtener los identificadores con:

```bash
id docker-ipfs
```

* **salida:**

    ```plaintext
    uid=1002(docker-ipfs) gid=1002(docker-ipfs) groups=1002(docker-ipfs)
    ```

    > En este ejemplo copiaremos los valores `1002` y `1002` como usuario y grupo.

### Crear e iniciar el contenedor de forma persistente

Ejecutar el comando:

```bash
docker run --restart unless-stopped \
  -d --name ipfs_host \
  --user 1002:1002 \
  -v /home/jesus/docker-ipfs-files/ipfs_staging:/export \
  -v /home/jesus/docker-ipfs-files/ipfs_data:/data/ipfs \
  -p 4001:4001 -p 4001:4001/udp \
  -p 127.0.0.1:8081:8081 \
  -p 127.0.0.1:5001:5001 \
  ipfs/kubo:latest
```

Explicación:

* `--restart unless-stopped`: Docker reinicia el contenedor después de un reinicio del sistema o fallo, pero no lo hará si lo detuviste manualmente con comandos como docker stop.
* `--user docker-ipfs`: Por seguridad y para aplicar la cuota de espacio en disco, iniciar el contenedor para un usuario concreto creado para este contenedor.
* `-d`: Ejecuta el contenedor en modo "desconectado" (background).
* `--name ipfs_host`: Asigna el nombre ipfs_host al contenedor para identificarlo.
* `-v ...:/export`: Monta una carpeta local (ipfs_staging) en el contenedor para almacenar archivos que quieras importar/exportar a IPFS.
* `-v ...:/data/ipfs`: Monta una carpeta local para almacenar los datos persistentes de IPFS (como el almacenamiento de bloques).
* `-p 4001:4001` y '-p `4001:4001/udp`: Expone el puerto 4001 (TCP y UDP) para conexiones entre nodos en la red IPFS (intercambio de bloques y descubrimiento de peers) para todas las interfaces disponibles para salir al exterior.
* `-p 127.0.0.1:8081:8081`: Expone el puerto 8081, el gateway HTTP de IPFS, accesible únicamente desde el host (loopback 127.0.0.1). 
> He elegido el puerto 8081 porque el puerto 8080 es muy usado como puerto de servidor de aplicaciones web.
* `-p 127.0.0.1:5001:5001`: Expone el puerto 5001 para la API de control de IPFS, accesible únicamente desde el host (loopback 127.0.0.1).


**¿Qué es el gateway de IPFS?**

El gateway HTTP de IPFS (puerto 8081 en este ejemplo) permite acceder a los contenidos almacenados en la red IPFS mediante un navegador web o peticiones HTTP. 
Ver el artículo [ipfs gateway](https://docs.ipfs.tech/concepts/ipfs-gateway/#gateway-request-lifecycle).

**¿Qué es el API expuesto el puerto 5001?**

Es una interfaz que permite interactuar y controlar un nodo IPFS programáticamente.

### Revisar el registro de log de `IPFS` y copiar `PeerID`

```bash
docker logs -f ipfs_host
```

* **salida:**

    ```plaintext
    ipfs version 0.32.1
    generating ED25519 keypair...done
    peer identity: 12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke
    initializing IPFS node at /data/ipfs
    Initializing daemon...
    Kubo version: 0.32.1-9017453
    Repo version: 16
    System version: amd64/linux
    Golang version: go1.23.3
    PeerID: 12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke
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
    * `generating ED25519 keypair...done`:
    Se genera un par de claves ED25519 para identificar el nodo IPFS de manera única. Es un mecanismo de criptografía de curva elíptica.
    * `peer identity: 12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke` / `PeerID: 12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke`:
    Esta es la identidad del nodo (Peer ID), derivada de la clave pública generada. Es única y sirve para identificar tu nodo en la red IPFS.
    * `Swarm listening on 127.0.0.1:4001 (TCP+UDP)` / `Swarm listening on 172.17.0.2:4001 (TCP+UDP)`:
    El nodo escucha conexiones locales en el puerto 4001 mediante TCP y UDP. Esto permite la comunicación con otros nodos.
    > Esto en nuesto caso es un problema, porque queriamos que otros nodos lo puedan acceder, pero luego lo revisaremos...
    * `Swarm listening on [::1]:4001 (TCP+UDP)`:
    Muestra que el nodo está escuchando en IPv6 localhost ([::1]) en el puerto 4001 en todas las interfaces, saliendo al exterior. Esto es correcto.
    * `Run 'ipfs id' to inspect announced and discovered multiaddrs of this node.`:
    Sugiere ejecutar el comando `ipfs id` para ver las direcciones multiaddr anunciadas al resto de peers. Es evidente que es necesario revisarlo porque como veremos no se anuncian las multiaddr de en una IP o dominio público al exterior, pero esto ya lo veremos luego...
    * `RPC API server listening on /ip4/0.0.0.0/tcp/5001`:
    Indica que el servidor de API RPC (Remote Procedure Call) está escuchando en todas las interfaces de red (0.0.0.0) en el puerto TCP 5001 dentro del contenedor. Esto es correcto porque está IPFS en un contenedor, no estás exponiendo nada al exterior porque además ya indicamos al iniciar el contenedor `-p 127.0.0.1:5001:5001`.
    * `WebUI: http://127.0.0.1:5001/webui`:
    Proporciona la URL para acceder a la interfaz de usuario web (WebUI) del nodo IPFS.
    * `Gateway server listening on /ip4/0.0.0.0/tcp/8080`:
    El servidor de gateway HTTP está escuchando en todas las interfaces de red (0.0.0.0), esto es correcto porque está IPFS en un contenedor, no estás exponiendo nada al exterior.
    Además indica que es el puerto 8080, pero no es donde queriamos, luego lo cambiaremos al puerto 8081.

Copiar el valor de `PeerID`, lo usaremos para que otros nodos puedan agregarlo como peer. En este ejemplo, sería el valor `12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke`:

```plaintest
PeerID: 12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke
```

### Cambiar puerto por defecto del gateway

El gateway JTTP de IPFS se expone por defecto en el puerto 8080, pero es un puerto típico para servicios, asi que lo cambio por 8081, al [iniciar el contenedor](#crear-e-iniciar-el-contenedor-de-forma-persistente) y en la configuración.

Editar la configuración de `IPFS` en `/home/jesus/docker-ipfs-files/ipfs_data/config` y buscar en `Addresses` la entrada `Gateway` y cambiarla:

```json
"Gateway": "/ip4/0.0.0.0/tcp/8081",
```

### Anunciar la multiaddr pública al exterior

Como nos indica el issue [Cluster peers in Docker or behind NATs do not advertise their public IP addresses](https://github.com/ipfs-cluster/ipfs-cluster/issues/949), anunciaremos nuestra multiaddr pública para que otros peers puedan acceder, porque al estar en docker no son anunciadas automaticamente.
- **Nota**: al [iniciar el contenedor](#crear-e-iniciar-el-contenedor-de-forma-persistente) indicamos los puertos, pero es cierto que si hubieramos iniciamos con la opción `--network host` nos habriamos ahorrado todo esto, pero por seguridad, he preferido aplicar esta solución.

Para todo esto, editar la configuración de `IPFS` en `/home/jesus/docker-ipfs-files/ipfs_data/config` y buscar en `Addresses` la lista de `AppendAnnounce` (que inicialmente está vacío).

Partiendo del ejemplo donde tenemos el dominio público `vps-a1bdd53d.vps.ovh.net`, debenos agregar esta lista, quedando finalmente como:

```json
    "AppendAnnounce": [
        "/dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001",
        "/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/quic-v1",
        "/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/quic-v1/webtransport",
        "/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/webrtc-direct/"
    ],
```

> Se ha indicado un dominio, pero podría ser la IP pública, siendo por ejemplo `/ip4/57.129.131.125/tcp/4001`, etc..

### Activar el nodo como realay y anunciarlo

Un [relay](https://docs.ipfs.tech/concepts/nodes/#relay) es la forma en la otros nodos como un `IPFS Desktop` que están detrás de un CGNAT puedan ser accedidos.

Editar la configuración de `IPFS` en `/home/jesus/docker-ipfs-files/ipfs_data/config` y buscar en `Addresses` la lista de `AppendAnnounce` (que inicialmente está vacío).

Partiendo de nuestro ejemplo, agregar la multiaddr de relay `/dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke/p2p-circuit`, donde:
- `dnsaddr`: se trata de una dirección en un DNS.
- `vps-a1bdd53d.vps.ovh.net`: es el dominio DNS del nodo IPFS donde estamos instalando.
- `12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke`: es el `PeerID` del nodo instalado.

Debería quedar así:

```json
"AppendAnnounce": [
    Etc...
    "/dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke/p2p-circuit"    
],
```

Luego en `Swarm` (casi al final) en `RelayService`, agregar `Enabled` a `true`:

```json
"RelayService": {
      "Enabled": true
    },
```

### Revisar limitaciones de disco y ancho de bada

Por defecto el espacio maximo entre caché, temporales, metadatos o contenido agreagado como pin, tiene como máximo `10GB` como se indica en el archivo `/home/jesus/docker-ipfs-files/ipfs_data/config` en `Datastore` en entrada `StorageMax`:

```json
    etc..
    "Datastore": {
        etc.. 
        "StorageMax": "10GB"
```

Podemos cambiarlo por otro valor, pero para esta instalación está bien de espacio.

Luego podemos limitar velocidad... pendiente.

### Revisión final, guardar cambios y verificar

Por si queda alguna duda, debería quedar así finalmente con todos los cambios aplicados, siguiendo nuestro ejemplo:

 ```json
{
  "API": {
    "HTTPHeaders": {}
  },
  "Addresses": {
    "API": "/ip4/0.0.0.0/tcp/5001",
    "Announce": [],
    "AppendAnnounce": [
      "/dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001",
      "/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/quic-v1",
      "/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/quic-v1/webtransport",
      "/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/webrtc-direct/",
      "/dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001/p2p/12D3KooWKeidFGYpUquQMXhnvNbPgeSaPfzkszuj4ApQcf8UrsQx/p2p-circuit"
    ],
    "Gateway": "/ip4/0.0.0.0/tcp/8081",
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

Reiniciar el contenedor:

```bash
docker restart ipfs_host
```

Revisar el log de nuevo y ver que no hay errores:

```bash
docker logs ipfs_host
```

* **salida:**

    ```plaintext
    ipfs version 0.32.1
    Found IPFS fs-repo at /data/ipfs
    Initializing daemon...
    Kubo version: 0.32.1-9017453
    Repo version: 16
    System version: amd64/linux
    Golang version: go1.23.3
    PeerID: 12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke
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
docker exec -it ipfs_host ipfs id
```

* **salida:**

    ```plaintext
    {
        "ID": "12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
        "PublicKey": "CAESIAeUPT9UWoCFOOkKUfyt+76Ucu5jZ3OLb23w0p4M9lST",
        "Addresses": [
                "/dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke/p2p-circuit/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/quic-v1/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/ip4/127.0.0.1/tcp/4001/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/ip4/127.0.0.1/udp/4001/quic-v1/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/ip4/127.0.0.1/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/ip4/127.0.0.1/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/ip4/172.17.0.2/tcp/4001/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/ip4/172.17.0.2/udp/4001/quic-v1/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/ip4/172.17.0.2/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/ip4/172.17.0.2/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/ip4/57.129.131.125/tcp/4001/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/ip6/::1/tcp/4001/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/ip6/::1/udp/4001/quic-v1/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/ip6/::1/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
                "/ip6/::1/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke"
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
        
        Partiendo de la IP de escucha `127.0.0.1`, ver como referencia como se anuncian las direcciones:

        ```plaintest
        "/ip4/127.0.0.1/tcp/4001/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
        "/ip4/127.0.0.1/udp/4001/quic-v1/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
        "/ip4/127.0.0.1/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
        "/ip4/127.0.0.1/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
        ```

        Revisar que tenemos los mismos elementos pero para nuestra direción pública, y adicionalmente incluyendo la multiaddr del relay:

        ```plaintest
        "/dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
        "/dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke/p2p-circuit/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
        "/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/quic-v1/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
        "/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
        "/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke",
        ```

        Veremos como están los elementos comunes `p2p`, `quic-v1/p2p`, `quic-v1/webtransport/` y `webrtc-direct/`, con el añadido del relay que es la dirección que contiene `p2p-circuit/`.
        
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

En el terminal donde iniciaste, para cerrar escribe `logout` en la conexión remota.


## Referencias

- [Install IPFS Kubo inside Docker](https://docs.ipfs.tech/install/run-ipfs-inside-docker/).
- [Referencia configuración `IPFS`](https://github.com/ipfs/kubo/blob/master/docs/config.md)
- [Configure NAT and port forwarding](https://docs.ipfs.tech/how-to/nat-configuration/#background).
- [Nat traversal](https://docs.libp2p.io/concepts/nat/).
- [libp2p relay](https://docs.libp2p.io/concepts/nat/circuit-relay/).
- [Issue cluster peers in Docker or behind NATs do not advertise their public IP addresses ](https://github.com/ipfs-cluster/ipfs-cluster/issues/949),
- [IPFS](https://ipfs.tech/).
- `chatgpt.com`.
