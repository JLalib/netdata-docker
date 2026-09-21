# 📊 Netdata Docker - Monitoreo Real-Time Autohospedado

[![GitHub Stars](https://img.shields.io/github/stars/netdata/netdata?style=flat-square&logo=github)](https://github.com/netdata/netdata)
[![Docker Pulls](https://img.shields.io/docker/pulls/netdata/netdata?style=flat-square&logo=docker)](https://hub.docker.com/r/netdata/netdata)
[![License](https://img.shields.io/github/license/netdata/netdata?style=flat-square)](https://github.com/netdata/netdata/blob/master/LICENSE)
[![Netdata Cloud](https://img.shields.io/badge/Netdata-Cloud-00D4AA?style=flat-square&logo=netdata)](https://app.netdata.cloud)

## 📋 Descripción general

**Netdata** es una plataforma de monitoreo de infraestructura en tiempo real diseñada para capturar miles de métricas por segundo con auto-detección total y **cero configuración**. Reemplaza completamente pilas complejas como Prometheus + Grafana + Alertmanager por un único agente inteligente, ideal para homelabs que buscan visibilidad total sin overhead de ingeniería.

> **Propuesta clave**: Métricas per-segundo (no 15-60 segundos como otras herramientas), auto-detección total de servicios/contenedores/aplicaciones, 400+ alertas pre-configuradas, detección de anomalías ML (18 modelos k-means por métrica), dashboard automático interactivo y troubleshooting basado en navegador que reemplaza SSH.

## ✨ Características principales

- 🚀 **Métricas per-segundo real-time** — 1000+ métricas/seg vs 15-60 seg de otras herramientas
- 🔍 **Auto-detección total cero config** — Servicios, contenedores, apps, logs, APIs, synthetic checks
- 📊 **Dashboards automáticos interactivos** — Gráficos con scroll, zoom, sin necesidad de Grafana
- 🚨 **400+ alertas pre-configuradas** — CPU, RAM, disco, network con thresholds inteligentes
- 🤖 **ML Anomaly Detection** — 18 modelos k-means por métrica, 99% reducción falsos positivos
- 🐳 **Monitoreo per-contenedor automático** — CPU, RAM, network, disk I/O, auto-descubrimiento en 1 segundo
- 🔧 **Browser-based troubleshooting** — Shell interactivo, netstat, systemd journal, container logs sin SSH
- 📝 **Systemd-journal integration** — Logs correlacionados con métricas sin pipelines centralizadas
- ⚡ **eBPF collectors** — Network tracing, syscall tracing, file I/O a nivel kernel
- 🌐 **Multi-host streaming** — Arquitectura parent/child para agregación centralizada escalable
- ☁️ **Netdata Cloud (opcional)** — Observabilidad centralizada SaaS, zero data egress, solo metadata
- 💾 **dbengine tiered storage** — 3 tiers (segundo/minuto/hora), comprime semanas en GB
- 🏷️ **Multiarch** — amd64, armv7, arm64, Raspberry Pi compatible
- 📈 **Exporters** — Prometheus, JSON, StatsD, OpenMetrics
- ⚖️ **Ultra-ligero** — 1-3% CPU, 100-300MB RAM, AGPL-3.0 open source (80K+ ⭐ GitHub)

## 📋 Requisitos del sistema

- 🐳 **Docker** instalado y funcionando
- 🔌 **Docker Socket** accesible (`/var/run/docker.sock`) para auto-descubrimiento de contenedores
- 💾 **200-500 MB RAM** mínimo (típicamente 100-300MB en operación)
- 💿 **1-5 GB espacio en disco** (depende política retención, default 14 días per-second)
- 🌐 **Puerto 19999** disponible (configurable)
- 🔐 **Capacidades Linux**: `SYS_PTRACE`, `SYS_ADMIN` (para eBPF)
- 📁 **Host filesystem mounts** para monitoreo sistema: `/proc`, `/sys`, `/etc`, `/var/log`
- 💽 **Volúmenes persistentes**: `/etc/netdata`, `/var/lib/netdata`, `/var/cache/netdata`
- 🌍 **Navegador moderno** (Chrome, Firefox, Safari, Edge)
- ☁️ **Opcional**: Cuenta Netdata Cloud (para multi-host centralizado)

## 🐳 Instalación

### Opción 1: Docker Compose Básico (Recomendado)

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

### Opción 2: Con Netdata Cloud (Centralizado Multi-Host)

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

1. **Zona horaria** — Ajusta `TZ=Europe/Madrid` a tu zona horaria (ej: `America/Mexico_City`, `UTC`)
2. **Privacidad** — `DO_NOT_TRACK=1` deshabilita telemetría anónima (recomendado)
3. **Netdata Cloud** — Para multi-host: añade `NETDATA_CLAIM_TOKEN`, `NETDATA_CLAIM_URL`, `NETDATA_CLAIM_ROOMS`
4. **Retención de datos** — Edita `/etc/netdata/netdata.conf` sección `[db]`:
   ```ini
   storage tiers = 3
   dbengine multihost disk space MB = 1024
   ```
5. **Frecuencia recolección** — En `[global]`: `update every = 1` (default 1 seg, subir a 2-5 para reducir CPU)
6. **Alertas personalizadas** — Edita `health_alarm_notify.conf` para Slack, PagerDuty, Teams, email, webhooks
7. **eBPF** — Requiere kernel 4.10+ y capabilities `SYS_ADMIN`, `SYS_PTRACE` (ya incluidas en compose)
8. **Seguridad** — `network_mode: host` y `pid: host` necesarios para métricas completas del host

## 🚀 Primeros pasos

1. **Explorar dashboard principal** — Abre `http://localhost:19999`, verás System Overview (CPU, RAM, Load, Uptime) y métricas actualizando cada segundo
2. **Ver contenedores Docker monitoreados** — Sección "Containers": todos tus containers listados con CPU, RAM, network, disk I/O por contenedor
3. **Monitoreo de procesos (reemplaza `top`)** — Sección "Procesos": ordenados por CPU/RAM, click para detalles (syscalls, conexiones network), histórico completo
4. **Revisar alertas pre-configuradas** — Settings ⚙️ → Alerts: 400+ alertas listas (CPU 80%, RAM 80%, disco lleno, etc.) con integraciones Slack, PagerDuty, Teams, email, webhooks
5. **Anomaly Advisor (IA)** — Tab "Anomalies": ML correlaciona miles de métricas e identifica root cause automático sin configuración
6. **Browser-based troubleshooting** — Sección "Functions": shell interactivo, netstat, systemd journal, container logs sin SSH, con histórico y ML
7. **Logs correlacionados** — Click en cualquier container → "Logs": systemd-journal automático, click en spike métrico → logs de ese momento exacto
8. **Ajustar retención storage** — `docker exec -it netdata bash` → edita `/etc/netdata/netdata.conf` sección `[db]`
9. **Integrar Netdata Cloud (multi-host)** — Crea cuenta en `app.netdata.cloud` (free tier), obtén claim token + room ID, añade a compose, `docker compose up -d`
10. **Exportar métricas (Prometheus, etc.)** — `curl -s http://localhost:19999/api/v1/allmetrics?format=prometheus`
11. **Listar todos charts/métricas** — `curl -s http://localhost:19999/api/v1/charts | python3 -m json.tool | head -50`
12. **Reducir CPU si necesario** — `docker exec -it netdata bash` → `cd /etc/netdata && ./edit-config netdata.conf` → `update every = 2`

## 💡 Casos de uso

- 🏠 **Homelabs** — Monitoreo completo sin engineering overhead; una herramienta reemplaza stack entero
- 👨‍💻 **DevOps/SREs** — Real-time troubleshooting, browser-based debugging sin SSH, root cause analysis automático
- 🌐 **Infraestructura multi-host** — Netdata Cloud centraliza múltiples agentes; parent/child streaming escalable
- ☸️ **Orquestación contenedores** — Kubernetes, Docker Swarm; métricas per-pod/per-container con auto-discovery
- 🔄 **Reemplazo Prometheus+Grafana** — Zero-config, alertas pre-built, sin PromQL, ML anomaly detection, costo 90% menor
- 📱 **Edge/IoT** — Raspberry Pi compatible, lightweight, log inference sin indexing central

## 🔒 Acceso remoto seguro

> ⚠️ **IMPORTANTE**: Netdata no incluye autenticación por defecto. Para exposición remota **obligatorio** usar reverse proxy con autenticación.

### HTTPS con Caddy (Producción)

```bash
# Caddyfile
monitor.tudominio.com {
    reverse_proxy localhost:19999
}
```

```bash
# Con autenticación básica (recomendado)
monitor.tudominio.com {
    basicauth {
        usuario $2a$14$hash_generado_con_htpasswd
    }
    reverse_proxy localhost:19999
}
```

Acceso: `https://monitor.tudominio.com` con HTTPS automático (Let's Encrypt)

## 🛠️ Gestión y mantenimiento

| Acción | Comando |
|--------|---------|
| **Ver logs** | `docker logs -f netdata` |
| **Backup config** | `docker cp netdata:/etc/netdata ./netdata-config-backup-$(date +%Y%m%d)` |
| **Restore config** | `docker stop netdata && docker cp ./netdata-config-backup-YYYYMMDD/netdata netdata:/etc/ && docker start netdata` |
| **Reiniciar** | `docker compose restart netdata` |
| **Actualizar versión** | `docker compose pull && docker compose up -d` |
| **Monitorear consumo Netdata** | `docker stats netdata` (típico: 100-300MB RAM, 1-3% CPU) |
| **Ver tamaño database** | `docker exec netdata du -sh /var/lib/netdata` |
| **Configurar alertas Slack** | `docker exec -it netdata bash -c "cd /etc/netdata && ./edit-config health_alarm_notify.conf"` → añadir `SLACK_WEBHOOK_URL` |

## 📝 Licencia

**AGPL-3.0** — Netdata es open source bajo licencia GNU Affero General Public License v3.0. Ver [LICENSE](https://github.com/netdata/netdata/blob/master/LICENSE) en el repositorio oficial.

---

> 📖 **Guía completa**: [Cómo instalar Netdata en Docker - Monitoreo real-time autohospedado](https://genbyte.blogspot.com/2026/08/como-instalar-netdata-en-docker.html)
>
> 🎥 **Canal YouTube**: [Genbyte](https://youtube.com/@genbyte) | 📧 **Newsletter**: [Suscríbete](https://genbyte.blogspot.com) | ☕ **Apoya**: [Ko-fi](https://ko-fi.com/genbyte)