## 👋 Welcome to mailu 🚀

Simple yet full-featured mail server as a set of Docker containers

## 📋 Description

Simple yet full-featured mail server as a set of Docker containers

## 🚀 Services

- **front**: ghcr.io/mailu/nginx:latest
- **resolver**: ghcr.io/mailu/unbound:latest
- **admin**: ghcr.io/mailu/admin:latest
- **imap**: ghcr.io/mailu/dovecot:latest
- **smtp**: ghcr.io/mailu/postfix:latest
- **oletools**: ghcr.io/mailu/oletools:latest
- **fts_attachments**: apache/tika:2.9.2.1-full
- **antispam**: ghcr.io/mailu/rspamd:latest
- **antivirus**: clamav/clamav-debian:1.2.3-45
- **webdav**: ghcr.io/mailu/radicale:latest
- **fetchmail**: ghcr.io/mailu/fetchmail:latest
- **webmail**: ghcr.io/mailu/webmail:latest

### Infrastructure Components

- **redis**: Redis database


## 📦 Installation

### Option 1: Quick Install
```bash
curl -q -LSsf "https://raw.githubusercontent.com/composemgr/mailu/main/docker-compose.yaml" -o compose.yml
```

### Option 2: Git Clone
```bash
git clone "https://github.com/composemgr/mailu" ~/.local/srv/docker/mailu
cd ~/.local/srv/docker/mailu
docker compose up -d
```

### Option 3: Using composemgr
```bash
composemgr install mailu
```

## 🔧 Configuration

### Environment Variables

```shell
TZ=America/New_York
APP_SECRET_KEY=changeme_secret_key
IPV4_NETWORK=10.1.1.0/24
APP_ADMIN_USER=admin
APP_ADMIN_PASS=changeme_admin_password
```

See `docker-compose.yaml` for complete list of configurable options.

## 🌐 Access

- **Web Interface**: http://172.17.0.1:59000

## 📂 Volumes

- `./rootfs/data/certs` - Data storage
- `./rootfs/data/nginx` - Data storage
- `./rootfs/data/data` - Data storage
- `./rootfs/data/dkim` - Data storage
- `./rootfs/data/mail` - Data storage
- `./rootfs/data/dovecot` - Data storage
- `./rootfs/data/mailqueue` - Data storage
- `./rootfs/data/postfix` - Data storage

## 🔐 Security

- Change all default passwords before deploying to production
- Use strong secrets for all authentication tokens
- Configure HTTPS using a reverse proxy (nginx, traefik, caddy)
- Regularly update Docker images for security patches
- Backup your data regularly

## 🔍 Logging

```shell
docker compose logs -f front
```

## 🛠️ Management

```bash
# Start services
docker compose up -d

# Stop services
docker compose down

# Update to latest images
docker compose pull && docker compose up -d

# View logs
docker compose logs -f

# Restart services
docker compose restart
```

## 📋 Requirements

- Docker Engine 20.10+
- Docker Compose V2+

## 🤝 Author

🤖 casjay: [Github](https://github.com/casjay) 🤖  
🦄 composemgr: [Github](https://github.com/composemgr) 🦄
