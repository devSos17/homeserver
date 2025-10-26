# Auto-Updates Role

Configura actualizaciones automáticas para el homeserver (helios).

## ¿Qué hace?

### Sistema (Debian)
- Instala y configura `unattended-upgrades`
- Actualiza lista de paquetes diariamente
- Instala actualizaciones de seguridad automáticamente
- Limpia paquetes viejos semanalmente

### Docker Containers
- Despliega Watchtower container
- Actualiza containers diariamente a las 4 AM
- Limpia imágenes Docker viejas automáticamente
- Mantiene la misma configuración de containers

## Uso

```bash
# Aplicar configuración de auto-updates
cd ~/dev/homeserver/ansible
ansible-playbook -i inventory playbooks/03_auto-updates.yml

# Verificar estado
ansible homeserver -i inventory -m shell -a "systemctl status unattended-upgrades" --become
ansible homeserver -i inventory -m shell -a "docker ps --filter name=watchtower"
```

## Configuración

### Variables (opcional)

Por defecto usa valores sensibles. Si quieres customizar, crea `group_vars/homeserver.yml`:

```yaml
watchtower_schedule: "0 0 4 * * *"  # Daily at 4 AM (cron format)
watchtower_cleanup: "true"          # Remove old images
apt_update_interval: 1              # Daily
apt_autoclean_interval: 7           # Weekly
```

### Excluir containers de auto-update

Si quieres que Watchtower ignore ciertos containers, agrégales el label:

```yaml
# En tu docker-compose.yml
services:
  mi-servicio:
    labels:
      - "com.centurylinklabs.watchtower.enable=false"
```

## Verificación

```bash
# Ver logs de unattended-upgrades
ssh homeserver sudo tail -f /var/log/unattended-upgrades/unattended-upgrades.log

# Ver logs de Watchtower
ssh homeserver docker logs -f watchtower

# Verificar configuración APT
ssh homeserver sudo apt-config dump APT::Periodic

# Forzar actualización manual (testing)
ssh homeserver sudo unattended-upgrade --dry-run --debug
```

## Troubleshooting

### Unattended-upgrades no corre
```bash
# Verificar timer systemd
systemctl list-timers | grep apt

# Ejecutar manualmente
sudo unattended-upgrade --debug
```

### Watchtower no actualiza containers
```bash
# Ver logs
docker logs watchtower

# Verificar que tiene acceso al socket
docker inspect watchtower | grep docker.sock

# Ejecutar actualización manual
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower --run-once --debug
```

## Referencias

- [Debian Unattended Upgrades](https://wiki.debian.org/UnattendedUpgrades)
- [Watchtower Docs](https://containrrr.dev/watchtower/)
- Documento de investigación: `~/A-Mann/Media/AI/Automatización Updates Homeserver.md`
