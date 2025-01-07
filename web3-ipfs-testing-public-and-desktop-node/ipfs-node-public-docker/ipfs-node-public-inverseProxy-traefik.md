# Web3 - IPFS - Probando un Nodo Público y de Escritorio - Instalación proxy inverso para gateway

Esta es la solucion nombrada como `#ipfs-node-public-inverseProxy-traefik`.

## Contexto

Este es un tutorial que forma parte de [Web3 - IPFS - Probando un Nodo Público y de Escritorio](../README.md)
> Por favor, cualquier referencia o proposito al respecto, te emplazo a leerlo ahí.

Estos son los pasos de configuración e instalación de un proxy inverso para poder publicar un gateway público para IPFS.


## Solución

La solución es la instalación de [Traefic](https://github.com/traefik/traefik) siguiendo las instrucciones de [Enable snaps on Ubuntu and install Traefik](https://snapcraft.io/install/traefik/ubuntu).

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

## Pre-requisitos

- Instalación según se indica en [Enable snaps on Ubuntu and install Traefik](https://snapcraft.io/install/traefik/ubuntu).

## Pasos



## Referencias

- [Traefic](https://github.com/traefik/traefik).
- [Enable snaps on Ubuntu and install Traefik](https://snapcraft.io/install/traefik/ubuntu).
- `chatgpt.com`.
