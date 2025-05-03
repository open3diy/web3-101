# Web3 - 101 - IPFS - Probando un Nodo Público y de Escritorio - Instalación de IPFS con docker en un nodo público

## Problemas conocidos que debes saber antes

### No olvidar actualizar e instalar los paquetes Debian

```bash
sudo apt update
sudo apt upgrade -y
```

> Hacer casos a los avisos, no ignorarlos o luego todo será peor.

### Permitir a los nodos establecer conexiones directas y autonat

Es importante permitir al nodo realizar conexiones directas cuando se ofrece el servicio relay, para eso la configuración [`EnableHolePunching`](https://github.com/ipfs/kubo/blob/master/docs/config.md#swarmenableholepunching) indicada a `True` es fundamental. No se indica como paso porque por defecto ya está habilitado, pero por favor, revisa en la configuración que no lo tengas a `False`.

Igualmente, [`Autonat`](https://github.com/ipfs/kubo/blob/master/docs/config.md#autonatservicemode) debe estar establecido a `enabled`, siendo ya la configuración por defecto, motivo por el que no se indica como paso, pero revisa por favor que no lo tengas a `disabled`.

### La configuración indicada en este tutorial provoca error

Siempre tener como referencia [la referencia configuración `IPFS`](https://github.com/ipfs/kubo/blob/master/docs/config.md) porque puede cambiar y afectar a esta explicación.

### Necesitas resetear nodo IPFS

En el host `/var/docker-ipfs/` están los directorios de datos y staging de IPFS montados para dar soporte al contenedor.

Si necesitamos reiniciar este directorio por cualquier problema, pero no quieres perder la configuración, sobre todo la del PEER_ID o simplemente quieres hacer una copia de seguridad, puedes seguir los pasos de:

Parar y borrar el contenedor:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml stop ipfs-host
docker-compose -f /etc/appserver/docker/docker-compose.yml rm -f ipfs-host
```

Archivar configurar en .tar en carpeta backup, por ejemplo, `/etc/appserver/ipfs-backup`:

```bash
mkdir -p /etc/appserver/ipfs-backup
sudo tar -cvf /etc/appserver/ipfs-backup/ipfs-backup-$(date +%Y-%m-%d).tar -C /var/docker-ipfs/data config keystore
```

Borrar carpeta datos y staging (aunque estará vacía):

```bash
sudo rm /var/docker-ipfs/data/* -rf
sudo rm /var/docker-ipfs/staging/* -rf
```

Iniciar el contenedor y ver logs:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml up -d ipfs-host --force-recreate
docker-compose -f /etc/appserver/docker/docker-compose.yml logs ipfs-host
```

> El contenedor iniciará IPFS, como si fuera la instrucción `ipfs init` para crear el resto de directorios y archivos necesarios

Parar el contenedor de nuevo:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml stop ipfs-host
```

Restaurar copia mas reciente:

```bash
sudo tar -xvf $(ls -t /etc/appserver/ipfs-backup/ipfs-backup-*.tar | head -n 1) -C /var/docker-ipfs/data/
```

> Puedes elegir también el nombre de archivo concreto si lo prefieres.

Iniciar el contenedor y ver logs:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml start ipfs-host
docker-compose -f /etc/appserver/docker/docker-compose.yml logs ipfs-host
```

Enlazar a archivo de configuración:

> Es un paso que se [explicará](#crear-configuración-de-ipfs-en-el-host).

```bash
[ ! -L /etc/appserver/ipfs/config ] && sudo ln -s /var/docker-ipfs/data/config /etc/appserver/ipfs/config
```

Dar permiso a usuarios del grupo `infrastructure`:

```bash
sudo setfacl -m g:infrastructure:rX /var/docker-ipfs
sudo setfacl -m g:infrastructure:rX /var/docker-ipfs/data
sudo setfacl -m g:infrastructure:rw /var/docker-ipfs/data/config
```

Quitar permiso de escritura a `docker-ipfs`:

```bash
sudo setfacl -m u:docker-ipfs:r-- /var/docker-ipfs/data/config
```

Publicar un CID vacío al IPNS del peer_id:

```bash
docker-compose -f /etc/appserver/docker/docker-compose.yml exec ipfs-host ipfs name publish /ipfs/QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn
```

> Esto es complicado de explicar...cuando resolvemos la URL <https://web3-101-ipfs.open3diy.org/ipns/k51qzi5uqu5di56caajjiel546q92pme0hgnh4gofey4tbwlfdr64ur7vu9s9t>, donde `k51qzi5uqu5di56caajjiel546q92pme0hgnh4gofey4tbwlfdr64ur7vu9s9t` es el peer_id, debe publicarse por lo menos un CID y si no lo hay, como el caso, se debe publicar a lo que se denomina un directorio vacío, que es siempre `QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn`.
