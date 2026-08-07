# 📊 Netdata Docker - Monitoreo Real-Time Autohospedado

[![GitHub Stars](https://img.shields.io/github/stars/genbyte/netdata-docker?style=for-the-badge&logo=github)](https://github.com/genbyte/netdata-docker)
[![Docker Pulls](https://img.shields.io/docker/pulls/netdata/netdata?style=for-the-badge&logo=docker)](https://hub.docker.com/r/netdata/netdata)
[![License](https://img.shields.io/github/license/genbyte/netdata-docker?style=for-the-badge)](LICENSE)
[![Netdata Version](https://img.shields.io/docker/v/netdata/netdata/stable?style=for-the-badge&logo=netdata)](https://hub.docker.com/r/netdata/netdata/tags)

## 📋 Descripción general

**Netdata** es una plataforma de monitoreo real-time de infraestructura diseñada para capturar miles de métricas por segundo con auto-detección total y cero configuración, reemplazando completamente pilas como **Prometheus+Grafana+Alertmanager** por un único agente inteligente.

Este repositorio proporciona una configuración Docker Compose lista para producción que despliega Netdata en segundos, con persistencia de datos, auto-detección de contenedores Docker, integración con systemd-journal, colectores eBPF y soporte opcional para Netdata Cloud.

> 🎯 **Propuesta clave**: Métricas per-segundo (no 15-60 segundos como otros), auto-detección total, cero configuración, 400+ alertas pre-configuradas, ML anomaly detection, dashboard automático interactivo, todo en 1-3% CPU y 100-300MB RAM.

## ✨ Características principales

- ⚡ **Métricas per-segundo real-time** — 1000+ métricas/seg vs 15-60 seg de otras herramientas
- 🔍 **Auto-detección total cero config** — Servicios, contenedores, apps, logs, APIs, synthetic checks
- 📊 **Dashboards automáticos interactivos** — Gráficos en vivo sin configurar Grafana
- 🚨 **400+ alertas pre-configuradas** — CPU, RAM, disco, network con thresholds inteligentes
- 🤖 **ML Anomaly Detection** — 18 modelos k-means por métrica, 99% reducción falsos positivos
- 🐳 **Per-container monitoring** — Auto-descubre todos los contenedores (CPU, RAM, network, disk I/O)
- 🌐 **eBPF collectors** — Network tracing, syscall tracing, file I/O a nivel kernel
- 📝 **Systemd-journal integration** — Logs correlacionados con métricas sin pipelines centralizadas
- 🔄 **Multi-host streaming** — Arquitectura parent/child para agregación centralizada escalable
- ☁️ **Netdata Cloud opcional** — Observabilidad centralizada SaaS (zero data egress, solo metadata)
- 📈 **Exportadores métricas** — Prometheus, JSON, OpenMetrics, StatsD
- 🛠️ **Browser-based troubleshooting** — Shell interactivo, netstat, systemd, container logs sin SSH
- 💾 **dbengine tiered storage** — 3 tiers (segundo/minuto/hora), comprime weeks en GB
- 🏗️ **Multiarch** — amd64, armv7, arm64, Raspberry Pi compatible
- 📜 **AGPL-3.0 open source** — 80K+ GitHub stars, comunidad activa

## 📋 Requisitos del sistema

- ✅ **Docker** y **Docker Compose** v2+
- 🔌 **Docker Socket** (`/var/run/docker.sock`) accesible para auto-detección de contenedores
- 💾 **200-500 MB RAM** mínimo (típicamente 100-300MB en operación)
- 💿 **1-5 GB espacio disco** (depende retention policy, default 14 días per-second)
- 🌐 **Puerto 19999** (configurable)
- 🔐 **Capacidades Linux**: `SYS_PTRACE`, `SYS_ADMIN` (para eBPF)
- 📁 **Host filesystem mounts** para monitoreo sistema: `/proc`, `/sys`, `/etc`, `/var/log`
- 💾 **Volúmenes persistentes**: `/etc/netdata`, `/var/lib/netdata`, `/var/cache/netdata`
- 🌍 **Navegador moderno** (Chrome, Firefox, Safari, Edge)
- ☁️ **Opcional**: Cuenta Netdata Cloud (para multi-host centralizado)

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

1. **Zona horaria** — Modifica `TZ=Europe/Madrid` según tu ubicación
2. **Privacidad** — `DO_NOT_TRACK=1` deshabilita telemetría anónima
3. **Netdata Cloud** — Añade `NETDATA_CLAIM_TOKEN`, `NETDATA_CLAIM_URL`, `NETDATA_CLAIM_ROOMS` para centralizar
4. **Retención de datos** — Edita `/etc/netdata/netdata.conf` sección `[db]`:
   ```ini
   [db]
   storage tiers = 3
   dbengine multihost disk space MB = 1024
   ```
5. **Frecuencia recolección** — En `[global]`: `update every = 1` (default 1 seg, subir a 2-5 para reducir CPU)
6. **Alertas personalizadas** — `docker exec netdata bash -c "cd /etc/netdata && ./edit-config health_alarm_notify.conf"`
7. **Notificaciones Slack** — Configura `SLACK_WEBHOOK_URL` en `health_alarm_notify.conf`

## 🚀 Primeros pasos

1. **Explorar dashboard principal** — Abre `http://localhost:19999`, verás System Overview (CPU, RAM, Load, Uptime) y métricas per-segundo
2. **Ver contenedores Docker monitoreados** — Sección "Containers": todos tus containers listados con CPU, RAM, network, disk I/O auto-detectados
3. **Monitoreo de procesos (reemplaza `top`)** — Sección "Procesos": ordenados por CPU/RAM, click para detalles (syscalls, conexiones network)
4. **Revisar alertas pre-configuradas** — Settings ⚙️ → Alerts: 400+ alertas listas (CPU 80%, RAM 80%, disco lleno, etc.) con integraciones Slack, PagerDuty, Teams, email, webhooks
5. **Anomaly Advisor (IA)** — Tab "Anomalies": ML correlaciona miles de métricas e identifica root cause automático sin configuración
6. **Browser-based troubleshooting** — Sección "Functions": shell interactivo, netstat, systemd journal, container logs sin SSH
7. **Logs correlacionados** — Click en cualquier container → "Logs": systemd-journal automático correlacionado con spikes de métricas
8. **Ajustar retención storage** — `docker exec -it netdata bash` → edita `/etc/netdata/netdata.conf` sección `[db]`
9. **Integrar Netdata Cloud** — Crea cuenta en `app.netdata.cloud` (free tier), obtén token + room ID, añade a docker-compose.yml
10. **Exportar métricas** — `curl -s http://localhost:19999/api/v1/allmetrics?format=prometheus`
11. **Listar todos los charts** — `curl -s http://localhost:19999/api/v1/charts | python3 -m json.tool | head -50`
12. **Reducir CPU si necesario** — `docker exec netdata bash -c "cd /etc/netdata && ./edit-config netdata.conf"` → `update every = 2`

## 💡 Casos de uso

- 🏠 **Homelabs** — Monitoreo completo sin engineering overhead; una herramienta reemplaza stack entero
- 👨‍💻 **DevOps/SREs** — Real-time troubleshooting, browser-based debugging sin SSH, root cause analysis automático
- 🌐 **Infraestructura multi-host** — Netdata Cloud centraliza múltiples agentes; parent/child streaming escalable
- ☸️ **Orquestación contenedores** — Kubernetes, Docker Swarm; per-pod/per-container métricas con auto-discovery
- 🔄 **Reemplazo Prometheus+Grafana** — Zero-config, alertas pre-built, no PromQL, ML anomaly detection, costo 90% menor
- 📱 **Edge/IoT** — Raspberry Pi compatible, lightweight, log inference sin indexing central

## 🔒 Acceso remoto seguro

> ⚠️ **IMPORTANTE**: Netdata no incluye autenticación por defecto. Para exposición remota **obligatorio** usar reverse proxy con autenticación.

### HTTPS con Caddy (producción)

```bash
# Caddyfile
monitor.tudominio.com {
    reverse_proxy localhost:19999
}
```

```bash
# Con autenticación básica
monitor.tudominio.com {
    basicauth {
        usuario hash_password_aqui
    }
    reverse_proxy localhost:19999
}
```

Acceso: `https://monitor.tudominio.com` con HTTPS automático (Let's Encrypt).

## 🛠️ Gestión y mantenimiento

| Acción | Comando |
|--------|---------|
| **Ver logs** | `docker logs -f netdata` |
| **Backup config** | `docker cp netdata:/etc/netdata ./netdata-config-backup-$(date +%Y%m%d)` |
| **Restaurar config** | `docker stop netdata && docker cp ./netdata-config-backup-YYYYMMDD/netdata netdata:/etc/ && docker start netdata` |
| **Reiniciar** | `docker compose restart netdata` |
| **Actualizar versión** | `docker compose pull && docker compose up -d` |
| **Monitorear consumo** | `docker stats netdata` (típico: 100-300MB RAM, 1-3% CPU) |
| **Tamaño database** | `docker exec netdata du -sh /var/lib/netdata` |
| **Configurar alertas Slack** | `docker exec netdata bash -c "cd /etc/netdata && ./edit-config health_alarm_notify.conf"` |

## 📝 Licencia

Este proyecto está licenciado bajo **AGPL-3.0** — misma licencia que Netdata upstream.

```
Copyright (c) 2024 Netdata Contributors
Licensed under the GNU Affero General Public License v3.0
```

## 📚 Referencias oficiales

- [GitHub Repository - netdata/netdata](https://github.com/netdata/netdata) (80K+ ⭐)
- [Netdata Docker Installation Guide](https://learn.netdata.cloud/docs/agent/packaging/docker)
- [Docker Hub - netdata/netdata](https://hub.docker.com/r/netdata/netdata)
- [Netdata Documentation](https://learn.netdata.cloud/docs)
- [Netdata Cloud (SaaS - opcional)](https://app.netdata.cloud)
- [Alerting](https://learn.netdata.cloud/docs/agent/health/alerts)
- [Health Checks](https://learn.netdata.cloud/docs/agent/health)
- [Configuration](https://learn.netdata.cloud/docs/agent/configure)
- [Performance Tuning Guide](https://learn.netdata.cloud/docs/agent/performance)

---

> 📖 **Guía completa**: [Cómo instalar Netdata en Docker - Monitoreo real-time autohospedado](https://genbyte.blogspot.com/2026/08/como-instalar-netdata-en-docker.html)