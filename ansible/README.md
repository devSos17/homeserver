# Homeserver Ansible

Configuración de Ansible para el homeserver (helios).

## Requisitos

- Ansible instalado
- Llave de vault en `~/dev/.keys/ansible-vault.key` (cifrada con AGE en chezmoi)
- SSH configurado para acceder a `homeserver`

## Estructura

```
ansible/
├── ansible.cfg           # Configuración de Ansible
├── inventory             # Hosts
├── group_vars/
│   └── all/
│       └── vault.yml     # Secrets cifrados (Pushover, MariaDB, etc.)
└── playbooks/
    ├── roles/
    │   ├── auto-updates/ # Watchtower + unattended-upgrades
    │   ├── docker/       # Instalación de Docker
    │   └── uptime-kuma/  # Monitoreo
    ├── 01_main.yml       # Deploy principal
    ├── 02_borgbkp.yml    # Backups
    └── 03_auto-updates.yml
```

## Ansible Vault

### Setup inicial (solo una vez)

```bash
# Generar llave de vault
openssl rand -base64 32 > ~/dev/.keys/ansible-vault.key
chmod 600 ~/dev/.keys/ansible-vault.key

# Agregar a chezmoi (cifrado con AGE)
chezmoi add --encrypt ~/dev/.keys/ansible-vault.key
```

### Comandos comunes

```bash
# Ver contenido del vault
ansible-vault view group_vars/all/vault.yml

# Editar secrets (abre en $EDITOR)
ansible-vault edit group_vars/all/vault.yml

# Cifrar un archivo nuevo
ansible-vault encrypt archivo.yml

# Descifrar un archivo
ansible-vault decrypt archivo.yml

# Re-cifrar con nueva llave
ansible-vault rekey group_vars/all/vault.yml
```

### Variables en el vault

| Variable | Descripción |
|----------|-------------|
| `vault_ntfy_url` | URL de ntfy (https://ntfy.sh) |
| `vault_ntfy_topic` | Topic de ntfy para notificaciones |
| `vault_mariadb_nextcloud_password` | Password de MariaDB para Nextcloud |

## Playbooks

### Deploy principal

```bash
cd ~/dev/homeserver/ansible
ansible-playbook -i inventory playbooks/01_main.yml
```

### Auto-updates (Watchtower + unattended-upgrades)

```bash
ansible-playbook -i inventory playbooks/03_auto-updates.yml
```

### Solo verificar (dry-run)

```bash
ansible-playbook -i inventory playbooks/01_main.yml --check
```

## Contenedores

Los containers críticos tienen `com.centurylinklabs.watchtower.enable=false` para evitar updates automáticos:

- **nextcloud** - Requiere upgrade manual (30→31→32)
- **mariadb** - Cambios de versión pueden romper DB

Los demás se actualizan automáticamente via Watchtower.

## Troubleshooting

### Error de vault

```bash
# Verificar que la llave existe
cat ~/dev/.keys/ansible-vault.key

# Si no existe, restaurar desde chezmoi
chezmoi apply ~/dev/.keys/ansible-vault.key
```

### Ver logs de Watchtower

```bash
ssh homeserver "docker logs watchtower --tail 50"
```

### Verificar unattended-upgrades

```bash
ssh homeserver "sudo unattended-upgrade --dry-run --debug"
```
