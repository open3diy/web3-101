# Control de ancho de banda de red para un servidor de red

Esta es la solucion nombrada como `#netServer-security-network-bandwidth`.

## Contexto

Este es un tutorial que forma parte de [Web3 - IPFS - Probando un Nodo Público y de Escritorio](../../IPFS/README.md)

## Proposito

Al crear un servidor público, que estará en un VPS, creo necesario controlar el ancho de banda de la red que podrá ofrecer el servicio de IPFS. No es lo más conveniente para un servicio de producción, pero al ser mí VPS de pruebas, quería tener más control sobre las capacidad que ofrezco.

El límite de velocidad de bajada y subida configurado es: `50Mbps`.

Se descarta la solución porque simplemente no funciona en docker. Leyendo en foros parece que es un problema común.

## Configuración

- En servidor VPS.
- En [Ubuntu 24.04 LTS](https://ubuntu.com/blog/tag/ubuntu-24-04-lts).

## Pre-requisitos

- [Instalación de docker](https://voidnull.es/instalacion-de-docker-en-ubuntu-24-04/).

## Problemas conocidos que debes saber antes

### No olvidar actualizar e instalar los paquetes debian

```bash
sudo apt update
sudo apt upgrade -y
```

> Hacer casos a los avisos, no ignorarlos o luego todo será peor.

## Pasos

Crear una red virtual limitada con tc

instalar tc

```bash
sudo apt install -y iproute2
```

crear la red docker con nombre `br-limited-ipfs`

```bash
docker network create \
  --driver bridge \
  --opt "com.docker.network.bridge.name"="br-limited-ipfs" \
  limited_net
```

- **salida:**

    ```plaintext
       --driver bridge \
       --opt "com.docker.network.bridge.name"="br-limited-ipfs" \
       limited_net
    17e8c2b75b3a18a1e8efdf6c9335198bbbd696115421af0e85ffc44529180b47
    ```

Aplicar límites con tc agregando nueva cola de control para la red creada `br-limited-ipfs`:

```bash
sudo tc qdisc add dev br-limited-ipfs root tbf rate 50mbit burst 32kbit latency 400ms
```

Explicación:

- `root`: define que esta configuración será aplicada en la raíz de la jerarquía de control de tráfico de la interfaz.
- `tbf`: es el algoritmo de control de tráfico "Token Bucket Filter"
- `rate 50mbit`: establece la tasa máxima de transferencia a 50 Mbps.
- `burst 32kbit`: define el tamaño del "balde" (cantidad máxima de datos que pueden enviarse instantáneamente sin esperar nuevos tokens). Según la documentación de tc, para una tasa de 10 Mbit/s, se recomienda un burst de al menos 10 kbytes; La elección de 32kbit es común porque proporciona un equilibrio entre permitir pequeñas ráfagas de datos y mantener un control preciso del ancho de banda
- `latency 400ms`: indica el tiempo máximo de espera para rellenar el "balde" con tokens. Si la tasa de datos supera los límites definidos, los datos pueden retrasarse hasta 400 ms antes de ser descartados. Un valor de 400 ms permite cierta flexibilidad para ráfagas, pero si se requiere una latencia más baja, se debería reducir este valor

**Hacerlo persistente**.

Crear script `/etc/network/tc-setup.sh`:

```bash
#!/bin/bash
tc qdisc add dev br-limited-ipfs root tbf rate 50mbit burst 32kbit latency 400ms
```

Hacerlo ejecutable

```bash
sudo chmod +x /etc/network/tc-setup.sh
```

Añádelo al arranque del sistema usando un servicio systemd, crear el archivo `/etc/systemd/system/tc-setup.service`

```plaintest
[Unit]
Description=Traffic Control Setup
After=network.target docker.service
Requires=docker.service

[Service]
Type=oneshot
ExecStart=/etc/network/tc-setup.sh
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```

Habilita el servicio

```bash
sudo systemctl enable tc-setup.service
```

Revisar si está activo:

```bash
systemctl status tc-setup.service
```

- **salida:**

    ```plaintext
    ○ tc-setup.service - Traffic Control Setup
        Loaded: loaded (/etc/systemd/system/tc-setup.service; enabled; preset: enabled)
        Active: inactive (dead)
    ```

Prueba final

Ejecutar un contenedor con la red limitada para hacer pruebas, como wget

```bash
docker run --rm -it --network limited_net alpine sh
```

Mas prubas dentro de un contenedor:

```bash
wget -O /dev/null http://speedtest.tele2.net/100MB.zip --show-progress
```
