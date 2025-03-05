# Iteración #1

## Deshabilitar el acceso `root`

```bash
sudo passwd -l root
```

## Deshabilitar el acceso SSH para root

Editar archivo de configuración:

```bash
sudo nano /etc/ssh/sshd_config
```

Busca la línea si existe `PermitRootLogin yes` y cambiar a `PermitRootLogin no` o incluirlo al final.

Reinicia el servicio SSH:

```bash
sudo systemctl restart ssh.service
sudo systemctl daemon-reload
```

## Operar sin conexión SSH

En momentos iniciales se operará por comodidad por SSH e incluso usando el puerto 22, pero una vez se haya acabado, para configuraciones menores, será desactivado y se usará la configuración directa desde el propio servidor o si es un VPS, usando el [KVM](https://es.wikipedia.org/wiki/Switch_KVM) disponible.

Cuando sea necesario, iniciar sesión presencial o KVM y ejecutar:

```bash
sudo systemctl disable ssh
```

## Instalar firewall ufw y configuración inicial

Instalar:

```bash
sudo apt update
sudo apt install ufw
```

Verificar:

```bash
sudo ufw status
```

Permitir acceso SSH:

```bash
sudo ufw allow ssh
```

Activar ufw:

```bash
sudo ufw enable
```

## Política de Privilegios Limitados (Limited Privileges Policy)

- Crear un usuario para cada proceso o servicio:
    Asignar cuentas específicas para cada proceso o servicio en lugar de usar cuentas genéricas como `root`.
    En Docker, evitar el usuario root especificando un usuario con `--user` o en el Dockerfile usando `USER`.
- Configurar permisos mínimos necesarios:
    Restringir el acceso a directorios, archivos y recursos al mínimo requerido para cada usuario o servicio.
- Evitar el uso de `sudo` en scripts automatizados:
    Usar cuentas configuradas con los privilegios específicos en lugar de depender de comandos que requieren `sudo`.
- Bloquear accesos no necesarios:
    Deshabilitar el acceso al `shell` para cuentas de servicio o procesos que no lo requieran.
- Revisar configuraciones predeterminadas:
    Modificar configuraciones por defecto que usen `root` o permisos elevados.
- Habilitar auditorías y registros:
    Supervisar cuentas y procesos para detectar accesos indebidos o intentos de escalación de privilegios.
- Aislar procesos y usuarios en entornos restringidos:
    Usar herramientas como chroot, contenedores, o entornos sandbox para limitar el impacto de un proceso comprometido.
