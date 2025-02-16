# Web3 - IPFS - 101 - Probando un Nodo Público y de Escritorio - Cheat Sheet

Los pasos explicados están bien, pero al final una [chuleta](https://en.wikipedia.org/wiki/Cheat_sheet) a modo resumen de los comandos o accesos mas usados, es lo más práctico para mí.

## Contexto

Esta es una chuleta que forma parte de [Web3 - IPFS - Probando un Nodo Público y de Escritorio](../README.md)
> Por favor, cualquier referencia o proposito al respecto, te emplazo a leerlo ahí.

## Cheat Sheet

Reiniciando todo y viendo errores:

```bash
docker images -q | xargs -r docker rmi -f
docker ps -aq | xargs -r docker rm -f
sudo rm /etc/appserver/certbot -rf

docker build -f /etc/appserver/docker/docker-nginx/Dockerfile -t custom-docker-nginx /etc/appserver/docker/docker-nginx

docker-compose -f /etc/appserver/docker/docker-compose.yml down --remove-orphans
docker-compose -f /etc/appserver/docker/docker-compose.yml up -d --force-recreate
docker-compose -f /etc/appserver/docker/docker-compose.yml up -d ipfs-host --force-recreate
docker-compose -f /etc/appserver/docker/docker-compose.yml ps

docker-compose -f /etc/appserver/docker/docker-compose.yml exec certbot certbot certonly --webroot -w /var/www/certbot -d web3-101.open3diy.org --email demovoidgan@gmail.com --agree-tos --non-interactive --force-renewal --debug

docker-compose -f /etc/appserver/docker/docker-compose.yml logs reverse-proxy

# revisar la configuracion
docker-compose -f /etc/appserver/docker/docker-compose.yml exec reverse-proxy nginx -t 
docker-compose -f /etc/appserver/docker/docker-compose.yml exec reverse-proxy nginx -s reload

docker-compose -f /etc/appserver/docker/docker-compose.yml logs certbot
docker-compose -f /etc/appserver/docker/docker-compose.yml logs test-web
docker-compose -f /etc/appserver/docker/docker-compose.yml logs ipfs-host

docker-compose -f /etc/appserver/docker/docker-compose.yml restart ipfs-host
docker-compose -f /etc/appserver/docker/docker-compose.yml restart reverse-proxy


less +G /var/log/nginx/nginx.log
less +G /var/log/certbot/letsencrypt.log
less +G /var/log/docker/docker-events.log
journalclt -u monitor-docker
```

Probando tc en `/etc/appserver/docker/post-up.sh` con diferentes limitaciones:

Con 50Mbs:

```bash
sudo nsenter -t $IPFS_HOST_ID -n tc qdisc add dev eth0 root tbf rate 50mbit burst 32kbit latency 400ms
```

Con 500Kbs:

```bash
sudo nsenter -t $IPFS_HOST_ID -n tc qdisc add dev eth0 root tbf rate 500kbit burst 32kbit latency 400ms
```

Accediendo a URLs de prueba: `https://web3-101.open3diy.org/test`

Accediendo a URLs para probar IPFS:
- CID de prueba: `https://ipfs.web3-101.open3diy.org/ipfs/QmfQHHLwiUMTgJ5LupnZgtqFQi2oFnpGo9guPpkjvEMqND`
- CID de prueba en sub-dominio: `https://mfQHHLwiUMTgJ5LupnZgtqFQi2oFnpGo9guPpkjvEMqND.ipfs.web3-101.open3diy.org`
- WebUi de prueba: `https://ipfs.web3-101.open3diy.org/webui`
