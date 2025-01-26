### Iteracion 2

https://github.com/docker/docker-bench-security

### Instalación inicial de Lynis y primera comprobación

Lynis es una herramienta de auditoría de seguridad y evaluación de cumplimiento diseñada para sistemas basados en Linux, macOS y sistemas Unix, que puedes instalar:

```bash
sudo apt update
sudo apt install lynis
```

Realizar un informe inicial:

```bash
sudo lynis audit system
```