# Probando un Nodo Público y de Escritorio - configuración del API de IPDS

Esta es la solucion nombrada como `#web3-ipfs-101-publicNode-docker-API`.

## Contexto

Este es un solución no forma parte de [Web3 - IPFS - 101 - Probando un Nodo Público y de Escritorio - Instalación en docker](../web3-101-ipfs-testing-public-and-desktop-node/web3-ipfs-101-publicNode-docker-install.md)

## Propósito

Dentro de la solución, al instalar IPFS, realizando pruebas, publique el API de docker al exterior para usarlo  desde internet. 
> Obviamente no puedes publicarlo sin controlar el acceso, esto era una prueba.

No está incluido en la solución, pero aquí están los pasos realizados en el proxy inverso traefik para que se pueda acceder.

## Configuración

- En servidor VPS.
- En [Ubuntu 24.04 LTS](https://ubuntu.com/blog/tag/ubuntu-24-04-lts).


## Pasos

### Configuración dinámica de traefik

Configurar el servicio:

```yaml
http:
  services:
    ipfs-host-web:
      loadBalancer:
        servers:
          - url: "http://ipfs-host:8080"
    ipfs-host-api:
      loadBalancer:
        servers:
          - url: "http://ipfs-host:5001"
  routers:
    ipfs-gateway:
      rule: "Host(`ipfs.web3-101.open3diy.org`) && (PathPrefix(`/ipfs`) || PathPrefix(`/ipns`))"
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt-resolver
      service: ipfs-host-web
    ipfs-api:
      rule: "Host(`ipfs.web3-101.open3diy.org`) && PathPrefix(`/api`)"
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt-resolver
      service: ipfs-host-api
    ipfs-api-webui:
      rule: "Host(`ipfs.web3-101.open3diy.org`) && PathPrefix(`/webui`)"
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt-resolver
      service: ipfs-host-api
    ipfs-subdomain-gateway:
      rule: "HostRegexp(`{cid:[a-zA-Z0-9]+}.ipfs.web3-101.open3diy.org`)"
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt-resolver
      service: ipfs-host-web
  middlewares:
    ipfs-path-rewrite:
      replacePathRegex:
        regex: "^/$"
        replacement: "/ipfs/{cid}"
```