# Probando un Nodo Público y de Escritorio - configuración del API de IPDS

Esta es la solucion nombrada como `#web3-ipfs-101-publicNode-docker-traefik_and_API`.

## Contexto

Este es un solución no forma parte de [Web3 - IPFS - 101 - Probando un Nodo Público y de Escritorio - Instalación en docker](../web3-101-ipfs-testing-public-and-desktop-node/web3-ipfs-101-publicNode-docker-install.md)

## Propósito

Dentro de la solución, al instalar IPFS, tuve que descargar traefik porque no soporta usar subdominios para enrutarlo al servicio de IPDS.

Ademas realizando pruebas, publique el API de docker al exterior para usarlo  desde internet. 
> Obviamente no puedes publicarlo sin controlar el acceso, esto era una prueba.

No está incluido en la solución, pero aquí están los pasos realizados.

## Configuración

- En servidor VPS.
- En [Ubuntu 24.04 LTS](https://ubuntu.com/blog/tag/ubuntu-24-04-lts).


## Pasos

### Configuración dinámica de traefik para el API

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

### Configuración del proxy para traefik

El [gateway](https://docs.ipfs.tech/concepts/how-ipfs-works/#ipfs-http-gateways) lo expondremos en las URL `https://vps-a1bdd53d.vps.ovh.net/ipfs/{CID}` y el [IPNS](https://docs.ipfs.tech/concepts/ipns) en `https://vps-a1bdd53d.vps.ovh.net/ipns/{PeerID}`.

Crear y editar archivo de configuración dinámica de traefik en `/etc/appserver/traefik/dynamic/ipfs-host.yml`

```yaml
http:
  services:
    ipfs-host-web:
      loadBalancer:
        servers:
          - url: "http://ipfs-host:8080"
  middlewares:
    ipfs-path-rewrite:
     redirectRegex:
      regex: "^https?://([a-zA-Z0-9]+).ipfs.web3-101.open3diy.org(/.*)?$"
      replacement: "https://ipfs.web3-101.open3diy.org/ipfs/$1$2"
      permanent: true
  routers:
    ipfs-gateway:
      rule: "Host(`ipfs.web3-101.open3diy.org`) && (PathPrefix(`/ipfs`) || PathPrefix(`/ipns`))"
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt-resolver
      service: ipfs-host-web
    ipfs-subdomain-gateway:
      rule: "HostRegexp(`^[a-zA-Z0-9]+.ipfs.web3-101.open3diy.org$`)"
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt-resolver
      service: ipfs-host-web
      middlewares:
        - ipfs-path-rewrite
```

Valida la sintaxis con `yq`:

```bash
yq eval /etc/appserver/traefik/dynamic/ipfs-host.yml
```

> Debe mostrar el contenido de la configuración si todo es correcto.

Revisar logs de traefik por si existen errores:

```bash
less +G /var/log/traefik/traefik.log 
```

Siguiendo este ejemplo donde el PEER_ID es `12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex` acceder a `https://vps-a1bdd53d.vps.ovh.net/ipns/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex`

> Verás la paǵina de IPFS inicialmente sin contenido si todo es correcto.