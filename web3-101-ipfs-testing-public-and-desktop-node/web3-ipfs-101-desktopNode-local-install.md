# Web3 - IPFS-101 - Probando un Nodo Público y de Escritorio - Instalación de IPFS Desktop en local

Esta es la solucion nombrada como `#web3-ipfs-101-desktopNode-local-install`.

## Contexto

Este es un tutorial que forma parte de [Web3 - IPFS - Probando un Nodo Público y de Escritorio](../README.md)
> Por favor, cualquier referencia o proposito al respecto, te emplazo a leerlo ahí.

Estos son los pasos de configuración e instalación de IPFS en un entorno local, usando IPFS Desktop cuando tú nodo es inaccesible, bien sea porque tu ISP no te provee una IPv6 de salida o estás detrás de un CGNAT.

## Solución

La solución es la instalación de IPFS con docker teniendo como referencia la documentación [install IPFS Desktop App](https://docs.ipfs.tech/install/ipfs-desktop/), pero con configuraciones especificas tras varias pruebas y consultas en la documentación de [referencia de IPFS](https://github.com/ipfs/kubo/blob/master/docs/config.md).

Igualmente tuve que tener en cuenta cosas que finalmente están en [Referencias](#referencias).

## Configuración

- En PC de sobremesa.
- En [Ubuntu 24.04 LTS](https://ubuntu.com/blog/tag/ubuntu-24-04-lts).

### Problemas conocidos que debes saber antes

#### No olvidar actualizar e instalar los paquetes debian

```bash
sudo apt update
sudo apt upgrade -y
```

> Hacer casos a los avisos, no ignorarlos o luego todo será peor.

### Permitir a los nodos establecer conexiones directas y autonat

Es importante permitir al nodo realizar conexiones directas cuando se requiere el servicio relay, para eso la configuracion [`EnableHolePunching`](https://github.com/ipfs/kubo/blob/master/docs/config.md#swarmenableholepunching) indicada a `True` es fundamental. No se indica como paso porque por defecto ya está habilitado, pero por favor, revisa en la configuración que no lo tengas a `False`.

Igualmente, [`Autonat`](https://github.com/ipfs/kubo/blob/master/docs/config.md#autonatservicemode) debe estar establecido a `enabled`, siendo ya la configuración por defecto, motivo por el que no se indica como paso, pero revisa por favor que no lo tengas a `disabled`.

#### La configuración indicada en config no es reconocida

Siempre tener como referencia [la referencia configuración `IPFS`](https://github.com/ipfs/kubo/blob/master/docs/config.md) porque puede cambiar y afectar a esta explicación.

#### No aparece en terminal el comando `ipfs`

En ubuntu al instalar el `.deb`, el `CLI` no es facil encontrar el comando `ipfs`, está en `/opt/'IPFS Desktop'/resources/app.asar.unpacked/node_modules/kubo/kubo`
> Es una ruta complicada de escribir, está en comillas simples el nombre de la ruta: `'IPFS Desktop'`
    
Lo mejor es agregar esa la ruta al `PATH`:
    
```
echo 'export PATH=$PATH:/opt/"IPFS Desktop"/resources/app.asar.unpacked/node_modules/kubo/kubo' >> ~/.bashrc
source ~/.bashrc
```

Probar en terminal que ya tenemos acceso a los comandos `ipfs`:

```
ipfs --version
```

#### Lo mejor ante problemas es salir y entrar

Lo cierto es que muchas veces, al ver que no conecta correctamante al nodo persistente (como veremos más adelante), lo mejor es apagar IPFS Desktop y voler a entrar, no solo reiniciar.

## Pre-requisitos

- [Instalación de IPFS Desktop App](https://docs.ipfs.tech/install/ipfs-desktop/#install-with-deb) como es indicado en la guía para Ubuntu en una instalación `.deb`.

## Pasos

### Habilitar relay como cliente

Editar la configuración en `~/.ipfs/config` o abrir la propia app, en apartado `Configuración` y agregar en `Swarm` (al final de la configuración), `RelayClient` la opción `Enabled` a `true`:

```json
"Swarm": {
    Etc..
    "RelayClient": {
      "Enabled": true
    },
```

Guardar los cambios y reiniciar el servicio. En Ubuntu, verrás el icono arriba a la derecha de IPFS, tienes que ir a `Restart`.

### Revisión de los protocolos cargados

Verificar inicialmente que los protocolos necesarios se han cargado, verificando desde un terminal:

```bash
ipfs id
```

* **salida:**

    ```plaintext
	"ID": "12D3KooWBmvCob83mHZh99XApRmAW6G1WKP2teSFznu3wXe7zDmZ",
	"PublicKey": "CAESIB0W2qXS7LhOqnm3ufpjGYJHE4BNg17k3dOghDtxsqGw",
	"Addresses": [
		Etc...
	],
	"AgentVersion": "kubo/0.32.1/desktop",
	"Protocols": [
		"/ipfs/bitswap",
		"/ipfs/bitswap/1.0.0",
		"/ipfs/bitswap/1.1.0",
		"/ipfs/bitswap/1.2.0",
		"/ipfs/id/1.0.0",
		"/ipfs/id/push/1.0.0",
		"/ipfs/lan/kad/1.0.0",
		"/ipfs/ping/1.0.0",
		"/libp2p/autonat/2/dial-back",
		"/libp2p/autonat/2/dial-request",
		"/libp2p/circuit/relay/0.2.0/stop",
		"/x/"
	]
    ```

    Es importante verificar la salida en los siguientes puntos:

    - Revisar que está los `Protocols` necesarios para `Autonat`:

        ```plaintest
        "/libp2p/autonat/1.0.0",
        "/libp2p/autonat/2/dial-back",
        "/libp2p/autonat/2/dial-request",
        ```
    
    - Revisar que están los `Protocols` necesarios para relay como cliente:

        ```plaintest
        "/libp2p/circuit/relay/0.2.0/stop"
        ```

#### Pruebas de correcto funcionamiento

Es crucial realizar estas pruebas para asegurarnos que todo es correcto.

Realizaremos las pruebas desde un terminal aunque podrias revisarlo también desde la propia App de escritorio.

**Pasos**:

Abrir la aplicación y pasado un tiempo, tener algo de paciencia porque no es inmediato, verificar que otros nodos nos anuncian como relay:

```bash
ipfs id | grep p2p-circuit
```

* **salida:**

    ```plaintext
    "/ip4/57.129.131.125/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit/p2p/12D3KooWBmvCob83mHZh99XApRmAW6G1WKP2teSFznu3wXe7zDmZ",
    "/ip4/84.155.126.229/tcp/4004/p2p/12D3KooWNbJU9dn1dmubZhuWXiqQodZic22cJCcp5H7Jwm6qJj84/p2p-circuit/p2p/12D3KooWBmvCob83mHZh99XApRmAW6G1WKP2teSFznu3wXe7zDmZ",
    "/ip4/84.155.126.229/udp/4004/quic-v1/p2p/12D3KooWNbJU9dn1dmubZhuWXiqQodZic22cJCcp5H7Jwm6qJj84/p2p-circuit/p2p/12D3KooWBmvCob83mHZh99XApRmAW6G1WKP2teSFznu3wXe7zDmZ",
    "/ip4/84.155.126.229/udp/4004/quic-v1/webtransport/certhash/uEiDXtXYR3C537HYkb-kCHu-7g7SQjKfbdKXhp7JkfvDfAA/certhash/uEiAOush0h1Zx-b202Zp9o7Cuhl5kzSzECt2ZJA1UZ1LV-w/p2p/12D3KooWNbJU9dn1dmubZhuWXiqQodZic22cJCcp5H7Jwm6qJj84/p2p-circuit/p2p/12D3KooWBmvCob83mHZh99XApRmAW6G1WKP2teSFznu3wXe7zDmZ",
    "/ip6/2003:d5:af00:c300:be24:11ff:fea7:4830/tcp/4004/p2p/12D3KooWNbJU9dn1dmubZhuWXiqQodZic22cJCcp5H7Jwm6qJj84/p2p-circuit/p2p/12D3KooWBmvCob83mHZh99XApRmAW6G1WKP2teSFznu3wXe7zDmZ",
    "/ip6/2003:d5:af00:c300:be24:11ff:fea7:4830/udp/4004/quic-v1/p2p/12D3KooWNbJU9dn1dmubZhuWXiqQodZic22cJCcp5H7Jwm6qJj84/p2p-circuit/p2p/12D3KooWBmvCob83mHZh99XApRmAW6G1WKP2teSFznu3wXe7zDmZ",
    "/ip6/2003:d5:af00:c300:be24:11ff:fea7:4830/udp/4004/quic-v1/webtransport/certhash/uEiDXtXYR3C537HYkb-kCHu-7g7SQjKfbdKXhp7JkfvDfAA/certhash/uEiAOush0h1Zx-b202Zp9o7Cuhl5kzSzECt2ZJA1UZ1LV-w/p2p/12D3KooWNbJU9dn1dmubZhuWXiqQodZic22cJCcp5H7Jwm6qJj84/p2p-circuit/p2p/12D3KooWBmvCob83mHZh99XApRmAW6G1WKP2teSFznu3wXe7zDmZ",
    ```

En este ejemplo, vemos que varios peers te dan servicio como IPv4 `57.129.131.125`, `84.155.126.229`, IPv6, etc..

Con esto ya sería todo suficiente, ahora puedes agregar tú contenido y lo veras en el gateway pública de IPFS...

Si todo te funciona correctamente, ya puedes acabar aquí, pero sino, sigue leyendo porque esto te podrá ayudar...

### Agregar como nodo persistente `vps-a1bdd53d.vps.ovh.net`

Este nodo persistente, que es un relay, te ayudará a poder acceder a tú contenido desde el exterior.
> **Nota**: este nodo `vps-a1bdd53d.vps.ovh.net` lo hemos creado según los pasos de [`#ipfs-node-public-docker-install` Instalación de IPFS en docker en un VPS público](../ipfs-node-public-docker/ipfs-node-public-docker-install.md).

**Pasos**:

Verificar que el nodo está disponible para conectarse:

```bash
ipfs swarm connect /dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex
```

* **salida:**

    ```plaintext
    p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex
    connect 12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex success
    ```

En la configuración en `~/.ipfs/config` o abrir la propia app, en apartado `Configuración`, agregar en `Peering`:

```json
	"Peering": {
		"Peers": [
			{
				"Addrs": [
					"/dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001"
				],
				"ID": "12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex"
			}
		]
	},
```

Reiniciar la aplicación, tener algo de paciencia, verificar que está conectado al nodo por la IP:

```bash
ipfs swarm peers | grep 12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex
```

* **salida:**

    ```plaintext
    /ip4/57.129.131.125/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex
    ```

> Aunque conectamos por DNS a `vps-a1bdd53d.vps.ovh.net`, siempre lo tranforma a la IP de `57.129.131.125`.

Comprobar luego que nos anuncia como relay:

```bash
ipfs id | grep 12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex
```

* **salida:**

    ```plaintext
    "/dnsaddr/vps-a1bdd53d.vps.ovh.net/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit/p2p/12D3KooWBmvCob83mHZh99XApRmAW6G1WKP2teSFznu3wXe7zDmZ",
	"/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/quic-v1/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit/p2p/12D3KooWBmvCob83mHZh99XApRmAW6G1WKP2teSFznu3wXe7zDmZ",
	"/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/quic-v1/webtransport/certhash/uEiD7qSzudDQme09i0vjsIwzSrAMdU7sFvoEvVVlgWaJP5g/certhash/uEiA2Edv0pkrKpVoqrAM6XvOFIdgf8pcH3EBS8V939U8g4A/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit/p2p/12D3KooWBmvCob83mHZh99XApRmAW6G1WKP2teSFznu3wXe7zDmZ",
	"/dnsaddr/vps-a1bdd53d.vps.ovh.net/udp/4001/webrtc-direct/certhash/uEiAsCA7PGleuvDGH3mT6L2shLLDHO1cEQLzyfl_zOW-OgA/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit/p2p/12D3KooWBmvCob83mHZh99XApRmAW6G1WKP2teSFznu3wXe7zDmZ",
	"/ip4/57.129.131.125/tcp/4001/p2p/12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex/p2p-circuit/p2p/12D3KooWBmvCob83mHZh99XApRmAW6G1WKP2teSFznu3wXe7zDmZ"
    ```

### Configuración en IPFS Desktop y prueba

En IPFS Desktop ir a "Configuración", última pestaña del panel izquiero, y en "Select a fallback Path Gateway for generating shareable links for CIDs that exceed the 63-character DNS limit.", donde pone la URL `https://ipfs.io`, probar a poner `

Agrega un documento de prueba, hazle PIN para fijarlo y realizar una prueba rápida de que aparece en el gateway público que tienes agregado como nodo persistente: pendiente...


ipfs ping 12D3KooWF7TUbY8NWCcLsPUhWMFVCGGvB9mKdEmU4bQaWy9Wkqex


## Referencias

- [Install IPFS Desktop App](https://docs.ipfs.tech/install/ipfs-desktop/).
- [Referencia configuración `IPFS`](https://github.com/ipfs/kubo/blob/master/docs/config.md)
- [Configure NAT and port forwarding](https://docs.ipfs.tech/how-to/nat-configuration/#background).
- [Nat traversal](https://docs.libp2p.io/concepts/nat/).
- [libp2p relay](https://docs.libp2p.io/concepts/nat/circuit-relay/).
- [IPFS](https://ipfs.tech/).
- `chatgpt.com`.
