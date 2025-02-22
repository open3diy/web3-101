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
- CID de prueba en sub-dominio: `https://QmfQHHLwiUMTgJ5LupnZgtqFQi2oFnpGo9guPpkjvEMqND.ipfs.web3-101.open3diy.org`
- WebUi de prueba: `https://ipfs.web3-101.open3diy.org/webui`

curl -I https://QmfQHHLwiUMTgJ5LupnZgtqFQi2oFnpGo9guPpkjvEMqND.ipfs.web3-101.open3diy.org


https://ipfs.web3-101.open3diy.org/ipfs/bafybeiack3rhfn25rzsvi7s7f3m37zvrwprthspsoitekib3orqxa7ycpi
https://ipfs.web3-101.open3diy.org/ipfs/QmbsnvRzzyxExqDNhZQcin5CrJJDcuP5GTmsxZ4HuvKiFv
https://dweb.link/ipfs/QmU1iddVtBKF96CEkazc7hPNeCczreKbg9uk7BK5g6Xod3
https://QmU1iddVtBKF96CEkazc7hPNeCczreKbg9uk7BK5g6Xod3.dweb.link

https://bafybeiccfclkdtucu6y4yc5cpr6y3yuinr67svmii46v5cfcrkp47ihehy.ipfs.dweb.link/
https://bafybeiccfclkdtucu6y4yc5cpr6y3yuinr67svmii46v5cfcrkp47ihehy.ipfs.web3-101.open3diy.org

https://bafybeiack3rhfn25rzsvi7s7f3m37zvrwprthspsoitekib3orqxa7ycpi.ipfs.dweb.link/
https://bafybeiack3rhfn25rzsvi7s7f3m37zvrwprthspsoitekib3orqxa7ycpi.ipfs.web3-101.open3diy.org/

puede que este CIDv0 QmU1iddVtBKF96CEkazc7hPNeCczreKbg9uk7BK5g6Xod3 sea este en V1 bafybeiack3rhfn25rzsvi7s7f3m37zvrwprthspsoitekib3orqxa7ycpi?


https://ipfs.web3-101.open3diy.org/ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m#x-ipfs-companion-no-redirect

docker-compose -f /etc/appserver/docker/docker-compose.yml exec ipfs-host ipfs config show | jq .Gateway

~/.config/IPFS Desktop/settings.json

https://bafybeiack3rhfn25rzsvi7s7f3m37zvrwprthspsoitekib3orqxa7ycpi.ipfs.infura-ipfs.io
https://infura-ipfs.io

