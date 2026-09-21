# 📊 Netdata Docker - Monitoreo Real-Time Autohospedado

[![GitHub Stars](https://img.shields.io/github/stars/genbyte/netdata-docker?style=for-the-badge&logo=github)](https://github.com/genbyte/netdata-docker)
[![Docker Pulls](https://img.shields.io/docker/pulls/netdata/netdata?style=for-the-badge&logo=docker)](https://hub.docker.com/r/netdata/netdata)
[![License](https://img.shields.io/github/license/genbyte/netdata-docker?style=for-the-badge)](LICENSE)

## 📋 Descripción general

**Netdata** es una plataforma de monitoreo real-time de infraestructura diseñada para capturar miles de métricas por segundo con auto-detección total y cero configuración, reemplazando completamente pilas como **Prometheus+Grafana+Alertmanager** por un único agente inteligente. Es la opción definitiva para homelabs que quieren visibilidad total sin engineering overhead.

Este repositorio proporciona una configuración Docker Compose lista para producción con persistencia, auto-detección de contenedores, capacidades eBPF y soporte opcional para Netdata Cloud.

## ✨ Características principales

- 🚀 **Métricas per-segundo** (1000+ métricas/seg) vs 15-60 seg de otras herramientas
- 🔍 **Auto-detección total cero config** - Servicios, contenedores, apps, logs, APIs
- 📊 **400+ alertas pre-configuradas** - CPU, RAM, disco, network con thresholds inteligentes
- 🤖 **ML Anomaly Detection** - 18 modelos k-means por métrica, 99% reducción falsos positivos
- 📈 **Dashboards automáticos interactivos** - Sin Grafana, scroll, zoom, tiempo real
- 🐳 **Per-container monitoring** - Auto-descubre todos los contenedores (CPU, RAM, network, disk I/O)
- 🔧 **Browser-based troubleshooting** - Shell, netstat, systemd journal, container logs sin SSH
- 📝 **Systemd-journal integration** - Logs correlacionados con métricas sin pipelines centralizadas
- ⚡ **eBPF collectors** - Network tracing, syscall, file I/O a nivel kernel
- 🌐 **Multi-host streaming** - Arquitectura parent/child escalable
- ☁️ **Netdata Cloud opcional** - Observabilidad centralizada SaaS (metadata only, zero data egress)
- 💾 **dbengine tiered storage** - 3 tiers (segundo/minuto/hora), comprime semanas en GB
- 🏷️ **Multiarch** - amd64, armv7, arm64, Raspberry Pi
- 📤 **Exporters** - Prometheus, JSON, OpenMetrics, StatsD
- ⚖️ **AGPL-3.0 open source** - 80K+ GitHub stars

## 📋 Requisitos del sistema

- **Docker** y **Docker Compose** instalados
- **Docker Socket** (`/var/run/docker.sock`) accesible para auto-detección de contenedores
- **200-500 MB RAM** mínimo (típicamente 100-300MB en operación)
- **1-5 GB espacio disco** (depende retention policy, default 14 días per-second)
- **Puerto 19999** disponible (configurable)
- **Capacidades Linux**: `SYS_PTRACE`, `SYS_ADMIN` (para eBPF)
- **Host filesystem mounts** para monitoreo sistema: `/proc`, `/sys`, `/etc`, `/var/log`
- **Volúmenes persistentes**: `/etc/netdata`, `/var/lib/netdata`, `/var/cache/netdata`
- **Navegador moderno** (Chrome, Firefox, Safari, Edge)
- **Opcional**: Cuenta Netdata Cloud (para multi-host centralizado)

## 🐳 Instalación

### Opción 1: Docker Compose básico (recomendado)

```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  netdata:
    image: netdata/netdata:stable
    container_name: netdata
    hostname: docker-host
    restart: unless-stopped
    pid: host
    network_mode: host
    cap_add:
      - SYS_PTRACE
      - SYS_ADMIN
    security_opt:
      - apparmor:unconfined
    volumes:
      # Persistencia config + data
      - netdataconfig:/etc/netdata
      - netdatalib:/var/lib/netdata
      - netdatacache:/var/cache/netdata
      # Host filesystem (read-only) para monitoreo
      - /:/host/root:ro,rslave
      - /etc/passwd:/host/etc/passwd:ro
      - /etc/group:/host/etc/group:ro
      - /etc/localtime:/etc/localtime:ro
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /etc/os-release:/host/etc/os-release:ro
      - /var/log:/host/var/log:ro
      # Docker socket para container auto-discovery
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - TZ=Europe/Madrid
      # Deshabilita tracking anónimo
      - DO_NOT_TRACK=1

volumes:
  netdataconfig:
  netdatalib:
  netdatacache:
EOF

docker compose up -d
```

### Opción 2: Con Netdata Cloud (centralizado multi-host)

```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  netdata:
    image: netdata/netdata:stable
    container_name: netdata
    hostname: docker-host
    restart: unless-stopped
    pid: host
    network_mode: host
    cap_add:
      - SYS_PTRACE
      - SYS_ADMIN
    security_opt:
      - apparmor:unconfined
    volumes:
      - netdataconfig:/etc/netdata
      - netdatalib:/var/lib/netdata
      - netdatacache:/var/cache/netdata
      - /:/host/root:ro,rslave
      - /etc/passwd:/host/etc/passwd:ro
      - /etc/group:/host/etc/group:ro
      - /etc/localtime:/etc/localtime:ro
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /etc/os-release:/host/etc/os-release:ro
      - /var/log:/host/var/log:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - TZ=Europe/Madrid
      - DO_NOT_TRACK=1
      # Netdata Cloud (opcional)
      - NETDATA_CLAIM_TOKEN=tu_token_aqui
      - NETDATA_CLAIM_URL=https://app.netdata.cloud
      - NETDATA_CLAIM_ROOMS=tu_room_id

volumes:
  netdataconfig:
  netdatalib:
  netdatacache:
EOF

docker compose up -d
```

### Acceso inicial

```
http://localhost:19999
```

## ⚙️ Configuración

1. **Zona horaria**: Modifica `TZ=Europe/Madrid` por tu zona (ej. `America/Mexico_City`)
2. **Privacidad**: `DO_NOT_TRACK=1` deshabilita telemetría anónima
3. **Netdata Cloud**: Añade `NETDATA_CLAIM_TOKEN`, `NETDATA_CLAIM_URL`, `NETDATA_CLAIM_ROOMS` para centralizar
4. **Retención de datos**: Edita `/etc/netdata/netdata.conf` sección `[db]` para ajustar `dbengine multihost disk space MB`
5. **Frecuencia recolección**: En `[global]` cambia `update every = 1` (default 1 seg) a 2-5 para reducir CPU
6. **Alertas personalizadas**: `docker exec netdata bash -c "cd /etc/netdata && ./edit-config health_alarm_notify.conf"`
7. **Notificaciones Slack**: Configura `SLACK_WEBHOOK_URL` en `health_alarm_notify.conf`

## 🚀 Primeros pasos

1. **Explorar dashboard principal**  
   Abre `http://localhost:19999` → Verás System Overview (CPU, RAM, Load, Uptime) + métricas per-segundo

2. **Ver contenedores Docker monitoreados**  
   Dashboard → Sección "Containers" → Todos tus containers listados con CPU, RAM, network, disk I/O auto-detectados

3. **Monitoreo de procesos (reemplaza `top`)**  
   Dashboard → Sección "Processes" → Procesos ordenados por CPU/RAM → Click para detalles (syscalls, conexiones)

4. **Revisar alertas pre-configuradas**  
   Settings (engranaje) → Alerts → 400+ alertas listas (CPU 80%, RAM 80%, Disk full, etc.) con integraciones Slack, PagerDuty, Teams, email, webhooks

5. **Anomaly Advisor (IA detección)**  
   Dashboard → Tab "Anomalies" → ML correlaciona miles de métricas e identifica root cause automático sin config

6. **Browser-based troubleshooting (reemplaza SSH)**  
   Dashboard → "Functions" (herramientas) → Shell interactivo, netstat, systemd journal, container logs con histórico

7. **Ver logs de contenedores correlacionados**  
   Dashboard → Container específico → "Logs" → Systemd-journal automático correlacionado con spikes de métricas

8. **Ajustar retención de almacenamiento**  
   `docker exec -it netdata bash` → Edit `/etc/netdata/netdata.conf` → Sección `[db]` → `storage tiers = 3`

9. **Integrar con Netdata Cloud (multi-host)**  
   Crea cuenta en `app.netdata.cloud` (free tier) → Obtén claim token + room ID → Añade a docker-compose → `docker compose up -d`

10. **Exportar métricas (Prometheus, JSON)**  
    ```bash
    curl -s http://localhost:19999/api/v1/data
    curl -s http://localhost:19999/api/v1/allmetrics?format=prometheus
    ```

11. **Listar todos los charts/métricas**  
    ```bash
    curl -s http://localhost:19999/api/v1/charts | python3 -m json.tool | head -50
    ```

12. **Reducir consumo CPU si necesario**  
    ```bash
    docker exec -it netdata bash
    cd /etc/netdata && ./edit-config netdata.conf
    # En [global]: update every = 2  # Default 1, cambiar a 2-5
    ```

## 💡 Casos de uso

- 🏠 **Homelabs**: Monitoreo completo sin engineering overhead - una herramienta reemplaza stack entero
- 👨‍💻 **DevOps/SREs**: Real-time troubleshooting, browser-based debugging sin SSH, root cause analysis automático
- 🌐 **Infraestructura multi-host**: Netdata Cloud centraliza múltiples agentes, parent/child streaming escalable
- ☸️ **Orquestación contenedores**: Kubernetes, Docker Swarm - per-pod/per-container métricas con auto-discovery
- 🔄 **Reemplazo Prometheus+Grafana**: Zero-config, alertas pre-built, no PromQL, ML anomaly detection, costo 90% menor
- 📱 **Edge/IoT**: Raspberry Pi compatible, lightweight, log inference sin indexing central

## 🔒 Acceso remoto seguro

**⚠️ IMPORTANTE**: Netdata no incluye autenticación por defecto. Para exposición remota usa reverse proxy con auth:

### Con Caddy (HTTPS automático)

```bash
# Caddyfile
monitor.tudominio.com {
    reverse_proxy localhost:19999
}
```

### Con autenticación básica (Caddy)

```bash
# Caddyfile
monitor.tudominio.com {
    basicauth {
        usuario hash_del_password
    }
    reverse_proxy localhost:19999
}
```

Genera hash: `caddy hash-password --plaintext 'tu_password'`

## 🛠️ Gestión y mantenimiento

### Ver logs en tiempo real
```bash
docker logs -f netdata
```

### Backup de configuración
```bash
docker cp netdata:/etc/netdata ./netdata-config-backup-$(date +%Y%m%d)
```

### Restaurar configuración
```bash
docker stop netdata
docker cp ./netdata-config-backup-YYYYMMDD/netdata netdata:/etc/
docker start netdata
```

### Reiniciar contenedor
```bash
docker compose restart netdata
```

### Actualizar a versión más reciente
```bash
docker compose pull
docker compose up -d
```

### Monitorear consumo de Netdata
```bash
docker stats netdata
# Típicamente: 100-300MB RAM, 1-3% CPU (muy ligero)
```

### Ver tamaño de base de datos métricas
```bash
docker exec netdata du -sh /var/lib/netdata
```

### Configurar alertas Slack (ejemplo)
```bash
docker exec -it netdata bash
cd /etc/netdata && ./edit-config health_alarm_notify.conf
# Añadir: SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK
```

## 📝 Licencia

Este proyecto está bajo licencia **MIT**. Netdata es **AGPL-3.0** (ver [licencia oficial](https://github.com/netdata/netdata/blob/master/LICENSE)).

---

> 📖 **Guía completa**: [Cómo instalar Netdata en Docker - Monitoreo real-time autohospedado](https://genbyte.blogspot.com/2026/08/como-instalar-netdata-en-docker.html)  
> 🎥 **Canal YouTube**: [Genbyte](https://youtube.com/@genbyte) | 📧 **Newsletter**: [Suscríbete](https://genbyte.blogspot.com) | ☕ **Apoya**: [Ko-fi](https://ko-fi.com/genbyte)