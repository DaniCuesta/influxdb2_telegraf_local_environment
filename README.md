# Despliegue de InfluxDB y Telegraf con Podman/Docker

Este proyecto proporciona una guía para el despliegue de InfluxDB 2 y Telegraf utilizando contenedores de Podman o Docker. A continuación, se detallan los pasos para configurar el entorno, crear una red y ejecutar los contenedores necesarios.

## Prerrequisitos
1. Instalar Podman o Docker
2. Asegúrate de que tu entorno esté correctamente configurado para ejecutar contenedores.

## Pasos para el despliegue

### 1. Crear una red para el stack
Crea una red para que los contenedores se comuniquen entre sí:

```bash
podman network create mi-red
```

### 2. Desplegar InfluxDB 2
Ejecuta el siguiente comando para iniciar un contenedor con InfluxDB 2:

```bash
podman run --name influxdb2 -d --network=mi-red -p 8086:8086   -v "$PWD/data:/var/lib/influxdb2"   -v "$PWD/config:/etc/influxdb2"   -e DOCKER_INFLUXDB_INIT_MODE=setup   -e DOCKER_INFLUXDB_INIT_USERNAME=admin   -e DOCKER_INFLUXDB_INIT_PASSWORD=admin_hello_1234   -e DOCKER_INFLUXDB_INIT_ORG=DAR   -e DOCKER_INFLUXDB_INIT_BUCKET=hello-dar   -e DOCKER_INFLUXDB_INIT_RETENTION=30d   -e DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=1xtZbx1bAe   docker.io/influxdb:2.7.5-alpine
```

### 3. Crear el archivo de configuración `telegraf.conf`
Crea un archivo llamado `telegraf.conf` en el directorio de trabajo y agrega la siguiente configuración:

```ini
# Configuración global de Telegraf
[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "0s"
  precision = ""

# Habilita el input de métricas del sistema
# Plugin de entrada para la CPU
[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = false
  core_tags = false

[[inputs.net]]

[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]

# Configuración del output para enviar las métricas a InfluxDB 2.x
[[outputs.influxdb_v2]]
  urls = ["http://influxdb2:8086"]
  token = "1xtZbx1bAe"
  organization = "DAR"
  bucket = "hello-dar"
```

### 4. Desplegar Telegraf
Ejecuta el siguiente comando para iniciar un contenedor con Telegraf:

```bash
podman run --name telegraf -d --network=mi-red   -v "$PWD/telegraf.conf:/etc/telegraf/telegraf.conf:ro"   -u telegraf   docker.io/telegraf:1.30.1-alpine
```

### 5. Verificar el funcionamiento
Después de ejecutar ambos contenedores, verifica el funcionamiento accediendo a la interfaz web de InfluxDB en [http://localhost:8086](http://localhost:8086) e inicia sesión con las credenciales:

- **Usuario:** `admin`
- **Contraseña:** `admin_hello_1234`

Los datos recopilados por Telegraf deben estar disponibles en el bucket `hello-dar`.

### Resumen

1. Crear la red `mi-red`:
   ```bash
   podman network create mi-red
   ```
2. Desplegar InfluxDB:
   ```bash
   podman run --name influxdb2 -d --network=mi-red -p 8086:8086      -v "$PWD/data:/var/lib/influxdb2" -v "$PWD/config:/etc/influxdb2"      -e DOCKER_INFLUXDB_INIT_MODE=setup -e DOCKER_INFLUXDB_INIT_USERNAME=admin      -e DOCKER_INFLUXDB_INIT_PASSWORD=admin_hello_1234      -e DOCKER_INFLUXDB_INIT_ORG=DAR -e DOCKER_INFLUXDB_INIT_BUCKET=hello-dar      -e DOCKER_INFLUXDB_INIT_RETENTION=30d      -e DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=1xtZbx1bAe docker.io/influxdb:2.7.5-alpine
   ```
3. Crear el archivo `telegraf.conf`.
4. Desplegar Telegraf:
   ```bash
   podman run --name telegraf -d --network=mi-red  -v $PWD/telegraf.conf:/etc/telegraf/telegraf.conf:ro -u telegraf docker.io/telegraf:1.30.1-alpine
   ```
