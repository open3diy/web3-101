# Web3 - IPFS - 101 - Probando un Nodo Público y de Escritorio - Cheat Sheet

Los pasos explicados están bien, pero al final una [chuleta](https://en.wikipedia.org/wiki/Cheat_sheet) a modo resumen de los comandos o accesos mas usados, es lo más práctico para mí.

## Contexto

Esta es una chuleta que forma parte de [Web3 - IPFS - Probando un Nodo Público y de Escritorio](../README.md).
> Por favor, cualquier referencia o propósito al respecto, te emplazo a leerlo ahí.

## Cheat Sheet

Borrando todas las imágenes y contenedores docker:

```bash
docker images -q | xargs -r docker rmi -f
docker ps -aq | xargs -r docker rm -f
```

Construyendo la imagen personalizada para IPFS:

```bash
docker build -f /etc/appserver/docker/docker-nginx/Dockerfile -t custom-docker-nginx /etc/appserver/docker/docker-nginx
```

Detener / borrar contenedores docker, asegurando que se borran sin usar e iniciar forzando a crearlos:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml down --remove-orphans
docker-compose -f /etc/appserver/docker/docker-compose.yml up -d --force-recreate
```

Igualmente que la instrucción anterior, pero para un servicio concreto como `ipfs-host`:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml stop ipfs-host
docker-compose -f /etc/appserver/docker/docker-compose.yml rm -f ipfs-host
docker-compose -f /etc/appserver/docker/docker-compose.yml up -d ipfs-host --force-recreate
```

Reiniciar todos los contenedores:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml restart
```

Solo reiniciar los servicios de docker `ipfs-host` y `reverse-proxy`:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml restart ipfs-host
docker-compose -f /etc/appserver/docker/docker-compose.yml restart reverse-proxy
```

Listar procesos en ejecución de docker y revisar logs de los servicios principales:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml ps

docker-compose -f /etc/appserver/docker/docker-compose.yml logs certbot
docker-compose -f /etc/appserver/docker/docker-compose.yml logs test-web
docker-compose -f /etc/appserver/docker/docker-compose.yml logs ipfs-host
docker-compose -f /etc/appserver/docker/docker-compose.yml logs reverse-proxy
```

Generar certificados con certbot para el sitio `web3-101.open3diy.org`:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec certbot certbot certonly --webroot -w /var/www/certbot -d web3-101.open3diy.org --email demovoidgan@gmail.com --agree-tos --non-interactive --force-renewal --debug
```

Verificar los logs de certbot:

```bash
less +G /var/log/nginx/nginx.log
less +G /var/log/certbot/letsencrypt.log
less +G /var/log/docker/docker-events.log
```

Después de cambiar la configuración de nginx, verificar si es correcto y re-cargarlo:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec reverse-proxy nginx -t 
docker-compose -f /etc/appserver/docker/docker-compose.yml exec reverse-proxy nginx -s reload
```

Revisar la configuración de IPFS:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec ipfs-host ipfs id
```

Configuración de IPFS Kubo: <https://github.com/ipfs/kubo/blob/master/docs/config.md>.

Revisar el estado del servicio `monitor-docker` encargado de aplicar configuraciones al iniciar y parar contenedores:

```bash
journalclt -u monitor-docker
```

**Probando tc** en `/etc/appserver/docker/post-up.sh` con diferentes limitaciones:

Con 50Mbs:

```bash
sudo nsenter -t $IPFS_HOST_ID -n tc qdisc add dev eth0 root tbf rate 50mbit burst 32kbit latency 400ms
```

Con 500Kbs:

```bash
sudo nsenter -t $IPFS_HOST_ID -n tc qdisc add dev eth0 root tbf rate 500kbit burst 32kbit latency 400ms
```

Accediendo a URLs de prueba: <https://test.web3-101.open3diy.org>.

Accediendo a URLs para probar IPFS:

- Acceder a contenido CID de prueba en base a la ruta: <https://web3-101-ipfs.open3diy.org/ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m>.
- Acceder a contenido CID de prueba en base al subdominio: <https://bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m.ipfs.web3-101-ipfs.open3diy.org>.

Pruebas de acceso a contenido:

- Válidos:
  - <https://ipfs.io/ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m>.
  - <https://web3-101-ipfs.open3diy.org/ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m>.

- Copyright:
  - <https://ipfs.io/ipfs/QmcvyefkqQX3PpjpY5L8B2yMd47XrVwAipr6cxUt2zvYU8>.
  - <https://web3-101-ipfs.open3diy.org/ipfs/QmcvyefkqQX3PpjpY5L8B2yMd47XrVwAipr6cxUt2zvYU8>.
