# Web3 - 101 - IPFS - Probando un Nodo Público y de Escritorio - Instalación de IPFS con docker en un nodo público

## Pasos

### Revisión final

Por si queda alguna duda, `/etc/appserver/ipfs/config` debería quedar así finalmente con todos los cambios aplicados, siguiendo nuestro ejemplo:

```json
{
  "API": {
    "HTTPHeaders": {}
  },
  "Addresses": {
    "API": "/ip4/0.0.0.0/tcp/5001",
    "Announce": [],
    "AppendAnnounce": [
      "/dnsaddr/web3-101-ipfs.open3diy.org/tcp/4001",
      "/dnsaddr/web3-101-ipfs.open3diy.org/udp/4001/quic-v1",
      "/dnsaddr/web3-101-ipfs.open3diy.org/udp/4001/quic-v1/webtransport",
      "/dnsaddr/web3-101-ipfs.open3diy.org/udp/4001/webrtc-direct/",
      "/dnsaddr/web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit"
    ],
    "Gateway": "/ip4/0.0.0.0/tcp/8080",
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
  },
    "Gateway": {
    "DeserializedResponses": null,
    "DisableHTMLErrors": null,
    "ExposeRoutingAPI": null,
    "HTTPHeaders": {},
    "NoDNSLink": false,
    "NoFetch": false,
    "PublicGateways": {
      "web3-101.open3diy.org": {
        "UseSubdomains": true,
        "Paths": ["/ipfs", "/ipns"]
        }
      },
    "RootRedirect": ""
  },
  etc..
  },
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

Reiniciar el contenedor ipfs-host:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml restart ipfs-host
```

Revisar el log de nuevo y ver que no hay errores:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml logs ipfs-host
```

- **salida:**

  ```plaintext
  ipfs version 0.32.1
  Found IPFS fs-repo at /data/ipfs
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
  Gateway server listening on /ip4/0.0.0.0/tcp/8081
  Daemon is ready
  ```

Verificar direcciones de escucha y módulos cargados:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec ipfs-host ipfs id
```

- **salida:**

  ```plaintext
  {
      "ID": "12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
      "PublicKey": "CAESIAeUPT9UWoCFOOkKUfyt+76Ucu5jZ3OLb23w0p4M9lST",
      "Addresses": [
              "/dnsaddr/web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/dnsaddr/web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/dnsaddr/web3-101-ipfs.open3diy.org/udp/4001/quic-v1/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/dnsaddr/web3-101-ipfs.open3diy.org/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/dnsaddr/web3-101-ipfs.open3diy.org/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/127.0.0.1/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/127.0.0.1/udp/4001/quic-v1/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/127.0.0.1/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/127.0.0.1/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/172.17.0.2/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/172.17.0.2/udp/4001/quic-v1/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/172.17.0.2/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/172.17.0.2/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip4/57.129.131.125/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip6/::1/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip6/::1/udp/4001/quic-v1/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip6/::1/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
              "/ip6/::1/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex"
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
        Partiendo de la IP de escucha `127.0.0.1`, ver como como se anuncian las direcciones:

      ```plaintest
      "/ip4/127.0.0.1/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
      "/ip4/127.0.0.1/udp/4001/quic-v1/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
      "/ip4/127.0.0.1/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
      "/ip4/127.0.0.1/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
      ```

  - Revisar que tenemos los mismos elementos pero para nuestra direción pública, y adicionalmente incluyendo la multiaddr del relay:

    ```plaintest
    "/dnsaddr/web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
    "/dnsaddr/web3-101-ipfs.open3diy.org/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
    "/dnsaddr/web3-101-ipfs.open3diy.org/udp/4001/quic-v1/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
    "/dnsaddr/web3-101-ipfs.open3diy.org/udp/4001/quic-v1/webtransport/certhash/uEiDQvcHyRFvpUhI-_ld7qJcy_LRU0WvoI-ELKH76eLXWMQ/certhash/uEiB8B-v5wmewNjVgNFrMIOICgxV2aFig4vws3xKhRauYXA/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
    "/dnsaddr/web3-101-ipfs.open3diy.org/udp/4001/webrtc-direct/certhash/uEiBZroH3W7SYUiXG8Jbro3s7ui_9yNF_9-2FDGhZX19Dsw/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex",
    ```

  - Veremos como están los elementos comunes `p2p`, `quic-v1/p2p`, `quic-v1/webtransport/` y `webrtc-direct/`, con el añadido del relay que es la dirección que contiene `p2p-circuit/`.

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
