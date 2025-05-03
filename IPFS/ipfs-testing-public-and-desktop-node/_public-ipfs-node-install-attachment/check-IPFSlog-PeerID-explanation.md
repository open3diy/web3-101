# Web3 - 101 - IPFS - Probando un Nodo Público y de Escritorio - Instalación de IPFS con docker en un nodo público

## Pasos

### Revisar log de `IPFS`, copiar `PeerID` y explicación

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

> Esto en nuestro caso es un problema, porque queríamos que otros nodos lo puedan acceder, pero luego lo revisaremos...

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
Además indica que es el puerto 8080, pero no es donde queríamos, luego lo cambiaremos al puerto 8081.

Copiar el valor de `PeerID`, lo usaremos para que otros nodos puedan agregarlo como peer. En este ejemplo, sería el valor `12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex`:

```plaintest
PeerID: 12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex
```