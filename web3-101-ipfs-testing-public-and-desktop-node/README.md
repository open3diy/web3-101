# Web3 - IPFS - 101 - Probando un Nodo Público y de Escritorio

Esta es la solucion nombrada como `#web3-ipfs-testing-public-and-desktop-node`.

## Contexto

Este es un tutorial de [Web3 - IPFS](../README.md).

Inicialmente, intenté instalar `IPFS Desktop` en un ordenador personal para entender su funcionalidad y explorar su uso en un entorno local. Sin embargo, debido a que mi red está detrás de un CGNAT, me encontré con limitaciones significativas para permitir la conexión pública a mí nodo y no podía compartir archivos con el resto de la red IPFS.

Es cierto que en la teoria, esto está solventado, otros nodos hacían de relay, es decir, actúan como intermediario para anunciar mí par de nodo, pero en la práctica me encontré con muchos problemas y finalmente decidí crear un nodo público en un `VPS` que me sirva como nodo relay y gateway público, que funciona de verdad, por lo menos para mí, para poder exponer al público archivos y hacer pruebas.

## Propósito

**Mi propósito aquí** es aprender a implementar y gestionar nodos IPFS, enfrentándome a desafíos técnicos reales, como la conectividad limitada de un nodo IPFS detrás de un CGNAT (Carrier-Grade NAT) 

### Adicional al propósito

Gracias a esto, hemos llegado a:
- **Crear un nodo IPFS público con docker**: estuve creando mí primer nodo IPFS con doker, siendo completamente funcional y además pude experimentar con la compartición de datos y la interacción con la red IPFS.
- **Aprender a preparar un servidor VPS**: preparé el servidor para que sea lo suficientemente seguro, ya que soy consciente que al ser público estará expuesto.
- **Apoyar a otros**: Al crear un nodo público IPFS, también he querido permitir a que cualquier poder usarlo para sus primeras pruebas, quizás tenía el mismo problema, y este servicio le ayudará.

#### Aviso a los delincuentes

No se emocionen, aunque sabrán el ID del nodo, es mí nodo de laboratorio, si lo tumban... mala suerte para mí :( y desde luego que no encontraran la frase semilla de mis wallet de bitcoin :)

## Solución

La solución para esta practica fue crear dos nodos: un nodo local `IPFS Desktop` en el que haré las pruebas pertinentes y luego un nodo público en un VPS que permitirá que dicho nodo local pueda ser consultado desde internet.

### Nodo local `IPFS Desktop`

Lo llamaremos `#ipfs-node-desktop-local`, es un instalación normal en un PC con `ubuntu 24` con la configuración necesario, como veremos más adelante, para que sea su contenido accedido desde internet.

### Nodo Público en un VPS

Lo llamaremos `#ipfs-node-public-docker`, es un nodo en un VPS, igualmente con `ubuntu 24` con la instalación en docker, el cual permite a `#ipfs-node-desktop-local` la conectividad en los siguientes aspectos:
- `#ipfs-node-desktop-local` se conecta de [forma persistente](https://docs.ipfs.tech/how-to/peering-with-content-providers/) con el nodo público IPFS `#ipfs-node-public-docker`, de esta forma se garantiza la conectividad.
- El nodo público `#ipfs-node-public-docker` está configurado para ofrecer servicio [relay](https://docs.ipfs.tech/concepts/nodes/#relay) a cualquier nodo, incluido a nuestro nodo local.
- El nodo público `#ipfs-node-public-docker` ofrece un [gateway](https://docs.ipfs.tech/concepts/how-ipfs-works/#ipfs-http-gateways) en la URL: [] existiendo más éxito para ofrecer el contenido.
    > Las pruebas me demostraron que tengo más exito de poder consultar contenido público que tengo en el nodo local, desde este gateway el cual está añadido de forma persistente. No tendría que ser así, como se menciona nada de esto tendría que ser necesario, pero finalmente esta ha sido la única forma real en la que el nodo local detrás de un CGNAT pudo tener una salida al exterior para hacer pruebas.

## Pasos

### Preparar el servidor VPS `#ipfs-node-public-docker`:

En el nodo publico, que es un VPS, con nombre `#ipfs-node-public-docker`, se preparará inicialmente, realizando los pasos:


### Instalar y configurar en `#ipfs-node-public-docker` IPFS con docker:

`#ipfs-node-public-docker-install`
    

### Preparar  `nodo-ipfs-desktop-local`

## Referencias

Para más información, visita los siguientes recursos:

- [IPFS Documentation](https://docs.ipfs.tech/) - Guía oficial para instalar, configurar y utilizar IPFS.
- [Relay Nodes in IPFS](https://docs.ipfs.tech/concepts/nodes/#relay) - Explicación sobre el concepto de relay en la red IPFS.
- [IPFS HTTP Gateways](https://docs.ipfs.tech/concepts/how-ipfs-works/#ipfs-http-gateways) - Explicación sobre gateway de IPFS.
- [Understanding CGNAT](https://en.wikipedia.org/wiki/Carrier-grade_NAT) - Detalles técnicos sobre las limitaciones y desafíos del CGNAT.

## Notas Adicionales

Este proyecto está en constante evolución, y los resultados documentados aquí reflejan un proceso de aprendizaje continuo.

