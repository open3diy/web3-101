# Web3 - IPFS - 101 - Actualizar versión de IPFS Kubo

Esta es la solución nombrada como `#web3-ipfs-101-IPFS-Kubo-version-update`.

## Contexto

Este es un tutorial de [Web3 - 101 - IPFS](../README.md).

Tras instalar IPFS Kubo como contenedor Docker, se observó en los logs que hay una versión más reciente disponible. Para mantener el nodo actualizado y seguro, se procede a realizar la actualización manualmente.

Queda pendiente investigar cómo monitorizar automáticamente la disponibilidad de nuevas versiones y evaluar la viabilidad de automatizar el proceso de actualización en el futuro.

## Propósito

**El propósito principal de este tutorial** es garantizar que el nodo IPFS esté siempre actualizado, minimizando riesgos de seguridad y asegurando la compatibilidad con las últimas mejoras y correcciones. Mantener el nodo en la versión más reciente permite aprovechar nuevas funcionalidades, mejorar el rendimiento y proteger la infraestructura frente a vulnerabilidades conocidas.

## Solución

El nodo publico se llama `#public-ipfs-node`, es un nodo en un VPS, con `ubuntu 24` con la instalación en docker.

## Pasos

### Revisar que hay nueva versión

Al revisar el log:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml logs ipfs-host
```

- **salida:**

    ```plaintext
    ipfs-host  | Daemon is ready
    ipfs-host  | 2025-08-18T02:13:56.744Z   ERROR   cmd/ipfs        kubo/daemon.go:1165
    ipfs-host  | ⚠️ A NEW VERSION OF KUBO DETECTED
    ipfs-host  |
    ipfs-host  | This Kubo node is running an outdated version (0.33.2).
    ipfs-host  | 34% of the sampled Kubo peers are running a higher version.
    ipfs-host  | Visit https://github.com/ipfs/kubo/releases or https://dist.ipfs.tech/#kubo and update to version 0.36.0 or later.
    ```

### Parar contenedor

Para detener el contenedor Docker, asegurando que no queden contenedores huérfanos y evitando problemas de caché, ejecuta los siguientes comandos:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml stop ipfs-host
docker-compose -f /etc/appserver/docker/docker-compose.yml rm -f ipfs-host
```

### Actualizar a última versión y arrancar

Bajar nueva versión:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml pull ipfs-host
docker-compose -f /etc/appserver/docker/docker-compose.yml up -d ipfs-host --force-recreate
```

### Revisar actualización

Revisar el log de nuevo:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml logs ipfs-host
```

- **salida:**

    ```plaintext
    ipfs-host  | ipfs version 0.36.0
    ipfs-host  | Found IPFS fs-repo at /data/ipfs
    ipfs-host  | Initializing daemon...
    ipfs-host  | Kubo version: 0.36.0-37b8411
    ipfs-host  | Repo version: 16
    ipfs-host  | System version: amd64/linux
    ipfs-host  | Golang version: go1.24.5
    ipfs-host  | PeerID: 12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex
    ipfs-host  | Swarm listening on 127.0.0.1:4001 (TCP+UDP)
    ipfs-host  | Swarm listening on 172.19.0.2:4001 (TCP+UDP)
    ipfs-host  | Swarm listening on [::1]:4001 (TCP+UDP)
    ipfs-host  | Run 'ipfs id' to inspect announced and discovered multiaddrs of this node.
    ipfs-host  | RPC API server listening on /ip4/0.0.0.0/tcp/5001
    ipfs-host  | WebUI: http://127.0.0.1:5001/webui
    ipfs-host  | Gateway server listening on /ip4/0.0.0.0/tcp/8080
    ipfs-host  | Daemon is ready
    ```

Revisar que sigue funcionando consultando un CID de pruebas: <https://web3-101-ipfs.open3diy.org/ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m>.

## Referencias

- Principalmente la documentación que ya tenía con anterioridad y chat-gpt.
