# Netdata Monitoring Role

Instala y configura Netdata para monitoreo en tiempo real del homeserver.

## ¿Qué hace?

### Instalación
- Instala Netdata vía kickstart script (recomendado) o package manager
- Configura storage tiered (per-second, per-minute, per-hour)
- Habilita Machine Learning para anomaly detection
- Configura alertas de health

### Monitoreo
- Sistema: CPU, RAM, Disk, Network
- Containers: Docker, LXC, systemd services
- Procesos individuales
- Hardware sensors (temperatura, fans, etc.)
- Network QoS
- Logs (systemd-journal)

## Uso

```bash
# Instalación básica
cd ~/dev/homeserver/ansible
ansible-playbook -i inventory playbooks/04_netdata.yml

# Con variables personalizadas
ansible-playbook -i inventory playbooks/04_netdata.yml \
  -e "netdata_dbengine_tier0_retention_days=14" \
  -e "netdata_enable_cloud=true"
```

## Configuración

### Variables principales (`defaults/main.yml`)

```yaml
# Instalación
netdata_install_method: kickstart  # o 'package'
netdata_disable_telemetry: true

# Web server
netdata_web_port: 19999
netdata_web_bind_to: "*"  # o "localhost" para solo local

# Retención de datos
netdata_dbengine_tier0_retention_days: 7    # 1-second
netdata_dbengine_tier1_retention_days: 90   # 1-minute  
netdata_dbengine_tier2_retention_days: 365  # 1-hour

# Espacio en disco
netdata_dbengine_tier0_disk_space: "1GiB"
netdata_dbengine_tier1_disk_space: "1GiB"
netdata_dbengine_tier2_disk_space: "1GiB"

# Features
netdata_enable_ml: true
netdata_enable_health: true

# Cloud (opcional)
netdata_enable_cloud: false
```

### Personalización por host

Crear `host_vars/homeserver.yml`:

```yaml
netdata_dbengine_tier0_retention_days: 14  # 2 semanas per-second
netdata_dbengine_tier0_disk_space: "2GiB"
netdata_email_notifications: true
netdata_email_recipient: "sos@example.com"
```

## Acceso

Después de la instalación:

```bash
# Local (desde el server)
http://localhost:19999

# Remoto (desde tu laptop)
http://homeserver:19999
# o
http://192.168.1.15:19999
```

### Acceso seguro vía SSH tunnel

```bash
# Desde tu laptop
ssh -L 19999:localhost:19999 homeserver

# Luego abre en browser
http://localhost:19999
```

## Integración con Traefik (opcional)

Si quieres exponer Netdata vía Traefik con HTTPS:

1. Editar `defaults/main.yml`:
```yaml
netdata_expose_via_traefik: true
netdata_traefik_domain: "monitor.yourdomain.com"
```

2. El role agregará los labels necesarios al config de Netdata

## Estructura de archivos generados

```
/etc/netdata/
├── netdata.conf           # Config principal
├── health_alarm_notify.conf  # Alertas
├── .opt-out-from-anonymous-statistics  # Telemetry disabled
└── ...

/var/log/netdata/
├── access.log
├── error.log
└── debug.log

/var/cache/netdata/
└── dbengine/              # Metrics storage
    ├── tier0/             # Per-second
    ├── tier1/             # Per-minute
    └── tier2/             # Per-hour
```

## Comandos útiles

```bash
# Ver status del servicio
systemctl status netdata

# Restart Netdata
sudo systemctl restart netdata

# Ver logs en tiempo real
sudo journalctl -u netdata -f

# Ver configuración actual
sudo netdatacli reload-health

# Test de alertas
sudo /usr/libexec/netdata/plugins.d/alarm-notify.sh test

# Editar config manualmente
sudo nano /etc/netdata/netdata.conf
sudo systemctl restart netdata
```

## Troubleshooting

### Netdata no inicia
```bash
# Ver logs
sudo journalctl -u netdata -n 50

# Verificar config
sudo netdata -W unittest

# Permisos
sudo chown -R netdata:netdata /var/cache/netdata
```

### Dashboard no carga
```bash
# Verificar puerto
sudo netstat -tlnp | grep 19999

# Verificar firewall
sudo ufw status
sudo ufw allow 19999/tcp
```

### Machine Learning no funciona
```bash
# Verificar en dashboard si ML está enabled
# Settings → Machine Learning

# Si no está, editar config
sudo nano /etc/netdata/netdata.conf
# [ml]
#   enabled = yes

sudo systemctl restart netdata
```

## Desinstalación

```bash
ssh homeserver
sudo /usr/libexec/netdata/netdata-uninstaller.sh --yes
```

## Referencias

- [Netdata Documentation](https://learn.netdata.cloud/)
- [Configuration Guide](https://learn.netdata.cloud/docs/configuring/configuration)
- [Health Monitoring](https://learn.netdata.cloud/docs/alerting/health-configuration-reference)
- Investigación completa: `~/A-Mann/Media/AI/Automatización Updates Homeserver.md`
