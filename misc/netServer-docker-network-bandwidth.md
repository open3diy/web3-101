
# Control de ancho de banda de red docker en el servidor de red 

Esta es la solucion nombrada como `#netServer-docker-network-bandwidth`.

## Contexto

Este es un tutorial de [Web3 - IPFS](../README.md) aplicable a cualquier solución, pero se explica como ejemplo para el tutorial [Web3 - IPFS - 101 - Probando un Nodo Público y de Escritorio - Instalación en docker](../web3-101-ipfs-testing-public-and-desktop-node/web3-ipfs-101-publicNode-docker-install.md)

## Proposito

Par controlar el ancho de banda de la red que se ofrecerá en un servicio docker, se crea esta solución.

Como solo dispongo de un VPS de pruebas, que tendrán mas servicios y para controlar el trafico, limito la velocidad, que desde luego no lo haría en un servicio de producción.

El límite de velocidad de bajada y subida configurado es: `50Mbps`.

## Solución

Usaremos el script de inicio y parada de un contenedor indicado en [instalación y configuración de docker](../../misc/docker-install-configuration.md).

En el script de up y down, usaremos el comando [nsenter](https://man7.org/linux/man-pages/man1/nsenter.1.html) para el contenedor afectado.

Nsenter, es una herramienta de Linux que permite entrar en los espacios de nombres (namespaces) de un proceso en ejecución. Se usa comúnmente para inspeccionar o modificar entornos de contenedores y otros procesos aislados.

Con este comando nsenter accedemos al espacio de red del contenedor para ejecutar [tc](https://man7.org/linux/man-pages/man8/tc.8.html), que añade una disciplina de cola (qdisc) con el algoritmo tbf (Token Bucket Filter) para limitar la velocidad de transmisión.


## Configuración

- En servidor VPS.
- En [Ubuntu 24.04 LTS](https://ubuntu.com/blog/tag/ubuntu-24-04-lts).

## Problemas conocidos que debes saber antes

### No olvidar actualizar e instalar los paquetes debian

```bash
sudo apt update
sudo apt upgrade -y
```

> Hacer casos a los avisos, no ignorarlos o luego todo será peor.

## Pre-requisitos

- [Configuración inicial del servidor de red](../../misc/initial-netServer-configuration.md).
- [Instalación y configuración de docker](../../misc/docker-install-configuration.md).

## Pasos

Tomando como ejemplo principal el contenedor con nombre `ipfs-host`, el cual limitaremos a `50Mbs`.

Agregar en `/etc/appserver/docker/post-up.sh`:

```bash
# Indicar aquí script de extensión al iniciar (se aplicará al iniciar el sistema también)
if [ "$CONTAINER_NAME" = "ipfs-host" ]; then
    IPFS_HOST_ID=$(docker inspect --format '{{.State.Pid}}' "$CONTAINER_NAME")

    if [ -z "$IPFS_HOST_ID" ]; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') - ERROR: No se pudo obtener el PID del contenedor $CONTAINER_NAME" >> /var/log/docker/docker-events.log
        exit 1
    fi

    # Aplicar tráfico controlado y capturar errores
    ERROR_MSG=$(sudo nsenter -t "$IPFS_HOST_ID" -n tc qdisc add dev eth0 root tbf rate 50mbit burst 32kbit latency 400ms 2>&1)

    if [ $? -eq 0 ]; then
        # Consultar la configuración aplicada de tc
        TC_STATUS=$(sudo nsenter -t "$IPFS_HOST_ID" -n tc qdisc show dev eth0 2>&1)
        echo "$(date '+%Y-%m-%d %H:%M:%S') - Contenedor $CONTAINER_NAME aplicado tc correctamente - Configuración actual: $TC_STATUS" >> /var/log/docker/docker-events.log
    else
        echo "$(date '+%Y-%m-%d %H:%M:%S') - ERROR: Fallo al aplicar tc en el contenedor $CONTAINER_NAME - Detalles: $ERROR_MSG" >> /var/log/docker/docker-events.log
    fi
fi
```

Explicación:
- `IPFS_HOST_ID`; obtenemos el PID del contenedor nombrado como `ipfs-host`.
- `nsenter -t $IPFS_HOST_ID`: permite entrar en el espacio de nombres de un proceso.
- `tc`: (Traffic Control) es la herramienta para configurar la gestión de tráfico en Linux.
- `qdisc add dev eth0 root`: agrega una nueva disciplina de cola (qdisc) en eth0 del contenedor en la raíz de la jerarquía de colas.
- `tbf`: (Token Bucket Filter), un algoritmo que limita la velocidad de transmisión.
- `rate 50mbit`: máximo de 50 Mbps permitidos.
- `burst 32kbit`: se pueden enviar hasta 32 Kbit de datos instantáneamente antes de que el control de tráfico empiece a aplicar límite.
- `latency 400ms`: máximo tiempo de espera antes de desechar paquetes es de 400ms.

## Referencias

- [nsenter](https://man7.org/linux/man-pages/man1/nsenter.1.html).
- [tc - trafic control / qdisc](https://man7.org/linux/man-pages/man8/tc.8.html).
- [tbf - Token Bucket Filter) ](https://man7.org/linux/man-pages/man8/tc-tbf.8.html).
> tbf (Token Bucket Filter) es una disciplina de cola (qdisc) dentro de tc, diseñada para controlar la velocidad de salida de paquetes.
- `chatgpt.com`.
