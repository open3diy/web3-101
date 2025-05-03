# Web3 - 101 - IPFS - Probando un Nodo Público y de Escritorio - Instalación de IPFS con docker en un nodo público

## Pasos

### Crear servicio en `docker-compose` - Explicación

- `restart unless-stopped`: Docker reinicia el contenedor después de un reinicio del sistema o fallo, pero no lo hará si lo detuviste manualmente con comandos como docker stop.
- `user 1012:1014"`: Por seguridad iniciar el contenedor para un usuario concreto creado para este contenedor.
- `container_name ipfs-host`: Asigna el nombre ipfs-host al contenedor para identificarlo.
- `-v ...:/export`: Monta una carpeta local (ipfs_staging) en el contenedor para almacenar archivos que quieras importar/exportar a IPFS.
- `-v ...:/data/ipfs`: Monta una carpeta local para almacenar los datos persistentes de IPFS (como el almacenamiento de bloques).
- En `ports` de `4001:4001` y `4001:4001/udp`: Expone el puerto 4001 (TCP y UDP) para conexiones entre nodos en la red IPFS (intercambio de bloques y descubrimiento de peers) para todas las interfaces disponibles para salir al exterior.
  - Indicamos los puertos, pero es cierto que si hubiéramos iniciamos con la opción `--network host` nos habríamos ahorrado todo esto, pero por seguridad, he preferido aplicar esta solución.
- En `ports` `127.0.0.1:5001:5001`: Expone el puerto 5001 para la API de control de IPFS, accesible únicamente desde el host (loopback 127.0.0.1).

Aclaración:

- En `ports` podría poner `127.0.0.1:8080:8080`para el gateway HTTP de IPFS, accesible únicamente desde el host (loopback 127.0.0.1), pero irá publicado en el proxy inverso y no es necesario.

**¿Qué es el gateway de IPFS?**

El gateway HTTP de IPFS permite acceder a los contenidos almacenados en la red IPFS mediante un navegador web o peticiones HTTP.

Ver el artículo [ipfs gateway](https://docs.ipfs.tech/concepts/ipfs-gateway/#gateway-request-lifecycle).
> A continuación, lo incluiremos en el proxy inverso traefik.

**¿Qué es el API expuesto el puerto 5001?**

El [API de kubo](https://docs.ipfs.tech/reference/kubo/rpc/) es una interfaz que permite interactuar y controlar un nodo IPFS programáticamente.