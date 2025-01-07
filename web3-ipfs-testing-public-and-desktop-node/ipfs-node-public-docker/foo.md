docker run --restart always \
  -d --name ipfs_host \
  --user 1002:1002 \
  -v /home/jesus/docker-ipfs-files/ipfs_staging:/export \
  -v /home/jesus/docker-ipfs-files/ipfs_data:/data/ipfs \
  -p 127.0.0.1:4001:4001 -p 127.0.0.1:4001:4001/udp \
  -p 127.0.0.1:8080:8080 \
  -p 127.0.0.1:5001:5001 \
  ipfs/kubo:latest

Si mapeo puertos tengo que anunciar a mano en docker



Peer Multiaddrs, salida
			/ip4/45.63.110.191/udp/4001/quic-v1/p2p/12D3KooWNWXf7PxXiQBkEKt8tDSmfirh9siRGc7jkbmde61E3uEE/p2p-circuit
			/ip4/57.129.131.125/tcp/4001/p2p/12D3KooWKeidFGYpUquQMXhnvNbPgeSaPfzkszuj4ApQcf8UrsQx/p2p-circuit
			/ip4/57.129.131.125/udp/4001/webrtc-direct/certhash/uEiCOuKwvdVyjW9pfyaal9g0LReQVhloKUgwfXfrksuH4gw/p2p/12D3KooWKeidFGYpUquQMXhnvNbPgeSaPfzkszuj4ApQcf8UrsQx/p2p-circuit
			/ip4/57.129.131.125/udp/4001/quic-v1/p2p/12D3KooWKeidFGYpUquQMXhnvNbPgeSaPfzkszuj4ApQcf8UrsQx/p2p-circuit
			/ip4/57.129.131.125/udp/4001/quic-v1/webtransport/certhash/uEiAkCr2xwpw7bi-qP1SyMJKTcQMA_YQdb4XQoDqme-u_qQ/certhash/uEiCKl-QshIrvyH-hm0IxQymVqqvln984MsUpFAFNYW93CQ/p2p/12D3KooWKeidFGYpUquQMXhnvNbPgeSaPfzkszuj4ApQcf8UrsQx/p2p-circuit

```bash
docker run --restart always \
  -d --name ipfs_host \
  --network host \
  --user 1002:1002 \
  -v /home/jesus/docker-ipfs-files/ipfs_staging:/export \
  -v /home/jesus/docker-ipfs-files/ipfs_data:/data/ipfs \
  ipfs/kubo:latest
```

    "/dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001/p2p/12D3KooWKeidFGYpUquQMXhnvNbPgeSaPfzkszuj4ApQcf8UrsQx/p2p-circuit"


    Ahora ya vemos en "Addresses" la dirección y ya es accesible en otro nodo, pero lo mejor es ir a otro nodo, escribiendo en terminal:

```bash
ipfs swarm connect /ip4/IP_PUBLICA/tcp/4001/p2p/12D3KooWKeidFGYpUquQMXhnvNbPgeSaPfzkszuj4ApQcf8UrsQx
```

ipfs swarm connect /ip4/57.129.131.125/tcp/4001/p2p/12D3KooWKeidFGYpUquQMXhnvNbPgeSaPfzkszuj4ApQcf8UrsQx



Donde:

* `IP_PUBLICA`: es la IP del nodo que estamos instalando
* El valor `12D3KooWKeidFGYpUquQMXhnvNbPgeSaPfzkszuj4ApQcf8UrsQx` es el `PEER_ID` del nodo que estamos instalando.

* **salida:**

    ```plaintext
    connect 12D3KooWKeidFGYpUquQMXhnvNbPgeSaPfzkszuj4ApQcf8UrsQx success
    ```


#### Configurar el nodo detrás del CGNAT

Como nota adicioal, como prologo, para configurar el resto de nodosEn los nodos detrás del `CGNAT` pueden conectarse a tu nodo relay con este comando:

```bash
ipfs swarm connect /ip4/<IP_DEL_RELAY>/tcp/4001/p2p/<PEER_ID>
```
```json
	"Peering": {
		"Peers": [
			{
				"Addrs": [
					"/dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001"
				],
				"ID": "12D3KooWAKx8GuxQchXZazkxDZnjr2sb3StjSNsyyhUvTr9Md5ke"
			}
		]
	},
```

/dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001/

`EnableHolePunching` a `true` y `RelayClient.Enabled` a `true`
```json
"Swarm": {
    "AddrFilters": null,
    "ConnMgr": {},
    "DisableBandwidthMetrics": false,
    "DisableNatPortMap": false,
    "EnableHolePunching": true,
    "RelayClient": {
        "Enabled": true
    },
    "RelayService": {},
    "ResourceMgr": {},
    "Transports": {
        "Multiplexers": {},
        "Network": {},
        "Security": {}
    }
},
```


Autonat
AutoNAT simplemente comprueba si alguna de las direcciones anunciadas del par es marcable

"AppendAnnounce": [
      "/ip4/57.129.131.125/tcp/4001/p2p-circuit"
    ],



Activar `AutoNAT`, poniendo `ServiceMode` a `enabled`:

```json
"AutoNAT": {
      "ServiceMode": "enabled"
  },
```
