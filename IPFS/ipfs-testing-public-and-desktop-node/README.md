# Web3 - IPFS - 101 - Probando un Nodo Público y de Escritorio

Esta es la solución nombrada como `#web3-ipfs-testing-public-and-desktop-node`.

## Contexto

Este es un tutorial de [Web3 - 101 - IPFS](../README.md).

Inicialmente, intenté instalar `IPFS Desktop` en un ordenador personal para entender su funcionalidad y explorar su uso en un entorno local. Sin embargo, debido a que mí red está detrás de un CGNAT, me encontré con limitaciones para permitir la conexión pública a mí nodo, dificultando mucho compartir archivos con el resto de la red IPFS.

Es cierto que en la teoría, esto está solventado, otros nodos hacían de relay, es decir, actúan como intermediario para anunciar mí par de nodo, pero en la práctica me encontré con muchos problemas y finalmente decidí crear un nodo público en un `VPS` que me sirva como nodo relay y gateway público, que funciona de verdad, por lo menos para mí, para poder exponer al público archivos y hacer pruebas.

## Propósito

**Mi propósito aquí** es aprender a implementar y gestionar nodos IPFS, enfrentándome a desafíos técnicos reales, como la conectividad limitada de un nodo IPFS detrás de un CGNAT (Carrier-Grade NAT).

### Adicional al propósito

Gracias a esto, he llegado a:

- **Crear un nodo IPFS público con docker**: estuve creando mí primer nodo IPFS con docker, siendo completamente funcional y además pude experimentar con la compartición de datos y la interacción con la red IPFS.
- **Aprender a preparar un servidor VPS**: preparé el servidor para que sea lo suficientemente seguro, ya que soy consciente que al ser público estará expuesto.
- **Apoyar a otros**: Al crear un nodo público IPFS, con un gateway y además compartiendo el ID de par, pretendo ayudar a cualquiera que también necesite hacer pruebas y no consiga ver su contenido porque está detrás de un CGNAT y los gateway públicos no le hacen mas que fallar.

#### Aviso a los cyber-delincuentes

Aunque sabrán el ID del nodo, es mí nodo de laboratorio, cualquiera prejuicio será un aprendizaje para mí.

## Solución

La solución para esta practica, fue crear dos nodos: un nodo local `IPFS Desktop` en el que haré las pruebas pertinentes y luego un nodo público en un VPS que permitirá que dicho nodo local pueda ser consultado desde internet.

### Nodo local `IPFS Desktop`

Lo llamaremos `#local-ipfs-node-desktop`, es un instalación normal en un PC con `ubuntu 24` con la configuración necesaria, como veremos más adelante, para que sea su contenido sea consultado desde internet.

### Nodo Público en un VPS

Lo llamaremos `#public-ipfs-node`, es un nodo en un VPS, igualmente con `ubuntu 24` con la instalación en docker, el cual permite a `#local-ipfs-node-desktop` la conectividad en los siguientes aspectos:

- `#local-ipfs-node-desktop` se conecta de [forma persistente](https://docs.ipfs.tech/how-to/peering-with-content-providers/) con el nodo público IPFS `#public-ipfs-node`, de esta forma se garantiza la conectividad.
- El nodo público `#public-ipfs-node` está configurado para ofrecer servicio [relay](https://docs.ipfs.tech/concepts/nodes/#relay) a cualquier nodo, incluido a nuestro nodo local.
- El nodo público `#public-ipfs-node` ofrece un [gateway](https://docs.ipfs.tech/concepts/how-ipfs-works/#ipfs-http-gateways) en la URL <https://ipfs.web3-101-ipfs.open3diy.org> existiendo más éxito para ofrecer el contenido.

    > Las pruebas me demostraron que tengo más éxito para consultar contenido público que tengo en el nodo local, desde este gateway el cual está añadido de forma persistente.

## Pasos el `#public-ipfs-node`

- [Configuración inicial de un servidor de red](../../misc/netServer-initial-configuration.md).
- [Instalación y configuración inicial de docker](../../misc/netServer-docker-install-configuration.md).
- [Instalación proxy inverso Nginx](../../misc/netServer-reverseProxy-Nginx-install.md).
- [Instalación de IPFS con docker en un nodo público](./public-ipfs-node-install.md)

### Adicionalmente

Podemos optimizar la instalación con:

- [Controlar el ancho de banda de red en contenedor IPFS](../../misc/netServer-docker-network-bandwidth.md)
    > De esta forma, podremos evitar que el servicio IPFS consuma todo el ancho de banda de la red.
- [Configurar una cuota en disco para el servicio de IPFS](../../misc/netServer-security-quotaDisc.md)
    > Por seguridad, si el servicio de IPFS en docker fuera comprometido, podríamos evitar que nos consuma todo el disco.

Además, existen otras optimizaciones sobre docker que la web de [industry40.systems](https://industry40.systems/docker,-limitar-los-recursos-de-sistema-de-un-contenedor-368a57b81561427f8b79fb018b18f76d) indica.

> Muy recomendable en cualquier caso, ojear este sitio web y su canal de YouTube.

## Pasos el `#local-ipfs-node-desktop`

- [Instalación de IPFS Desktop en local](./local-ipfs-node-desktop-install.md)

## Referencias

Para más información, visita los siguientes recursos:

- [IPFS Documentation](https://docs.ipfs.tech/) - Guía oficial para instalar, configurar y utilizar IPFS.
- [Relay Nodes in IPFS](https://docs.ipfs.tech/concepts/nodes/#relay) - Explicación sobre el concepto de relay en la red IPFS.
- [IPFS HTTP Gateways](https://docs.ipfs.tech/concepts/how-ipfs-works/#ipfs-http-gateways) - Explicación sobre gateway de IPFS.
- [Understanding CGNAT](https://en.wikipedia.org/wiki/Carrier-grade_NAT) - Detalles técnicos sobre las limitaciones y desafíos del CGNAT.
- [industry40.systems](https://industry40.systems/) - Web de tutoriales muy interesante sobre IoT, Industria 4.0,Informática industrial y Automatización industrial.
