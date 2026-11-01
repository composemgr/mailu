## 👋 Welcome to mailu 🚀

Simple yet full-featured mail server as a set of Docker containers

## 📋 Description

Mailu is a full mail server as a set of containers: an nginx front end that
terminates TLS, Postfix for SMTP, Dovecot for IMAP and POP3, an Unbound
resolver, Rspamd and ClamAV for spam and virus filtering, Radicale for
CalDAV, an optional webmail, and a Mailu admin UI. This stack publishes the
mail protocols on the dockerhost bind address, behind the host nginx proxy.

## 🚀 Services

- **app**: ghcr.io/mailu/nginx:latest (front end, published)
- **mailu-admin**: ghcr.io/mailu/admin:latest
- **mailu-imap**: ghcr.io/mailu/dovecot:latest
- **mailu-smtp**: ghcr.io/mailu/postfix:latest
- **mailu-antispam**: ghcr.io/mailu/rspamd:latest
- **mailu-antivirus**: clamav/clamav:latest
- **mailu-oletools**: ghcr.io/mailu/oletools:latest
- **mailu-webdav**: ghcr.io/mailu/radicale:latest
- **mailu-webmail**: ghcr.io/mailu/webmail:latest
- **mailu-fetchmail**: ghcr.io/mailu/fetchmail:latest
- **mailu-resolver**: ghcr.io/mailu/unbound:latest

### Infrastructure Components

- **mailu-redis**: Redis database
- **mailu-fts-attachments**: Apache Tika, full text search of attachments

## 📦 Installation

### Option 1: Quick Install
```bash
curl -q -LSsf "https://raw.githubusercontent.com/composemgr/mailu/main/docker-compose.yaml" -o compose.yml
```

### Option 2: Git Clone
```bash
git clone "https://github.com/composemgr/mailu" ~/.local/srv/docker/mailu
cd ~/.local/srv/docker/mailu
cp default.env .env
docker compose --env-file .env up -d
```

### Option 3: Using composemgr
```bash
composemgr install mailu
```

## 🔧 Configuration

### Environment Variables

```shell
TZ=America/New_York
APP_SECRET_KEY=replace-with-a-random-16-byte-hex-string
APP_ADMIN_USER=admin
APP_ADMIN_PASS=replace-with-a-random-password
MAILU_API_TOKEN=replace-with-a-random-api-token
MAILU_POSTMASTER=postmaster
MAILU_TLS_FLAVOR=cert
IPV4_NETWORK=10.1.1.0/24
IPV6_NETWORK=fdab:03bb:c928:beef::/64
MAILU_MESSAGE_SIZE_LIMIT=50000000
```

See `docker-compose.yaml` for complete list of configurable options.

### Environment Files

`default.env.sample` holds the full set of variables and is copied to `.env`
before starting the stack:

```bash
cp default.env .env
docker compose --env-file .env up -d
```

`app.env.sample` holds the app-specific overrides and is copied to `app.env`:

```bash
cp app.env.sample app.env
```

`composemgr up` applies `app.env` and `default.env` automatically. Raw
`docker compose` commands need `--env-file .env` to pick up the same values.

### Reverse Proxy

Three standalone compose files ship in this repo:

| File | Use |
|------|-----|
| `docker-compose.yaml` | Default, admin UI published on `172.17.0.1:64244` |
| `docker-compose.traefik.yaml` | Behind an existing external `traefik` network |
| `docker-compose.tunnel.yaml` | Behind an existing external `cloudflare` tunnel network |

```bash
docker compose -f docker-compose.traefik.yaml --env-file .env up -d
```

Only the `app` service, the Mailu nginx front end, carries the proxy labels
and joins the external network, proxying its HTTPS port `443`. The proxy
variants publish no ports at all, so the mail protocol ports are not
reachable from the host in those variants unless you map them yourself.

## 🌐 Access

- **Admin UI**: http://172.17.0.1:64244
- **Webmail**: http://172.17.0.1:64244/webmail
- **Mail protocols**, all bound to `172.17.0.1`:
  - `25` SMTP, inbound mail from other servers
  - `465` SMTPS, submission over implicit TLS
  - `587` SMTP submission, STARTTLS
  - `110` POP3, `995` POP3S
  - `143` IMAP, `993` IMAPS
  - `4190` ManageSieve, server side filtering

A mail server is only useful if the other mail servers can reach it, so the
SMTP, IMAP, POP3, and submission ports are published on their standard
numbers rather than on a high host port. They are bound to the dockerhost
address `172.17.0.1`, so they are only reachable from the host and from other
containers on that bridge. Redirect ports 25, 465, 587, 110, 995, 143, 993,
and 4190 on your router to this host, and set the MX record for
`BASE_DOMAIN_NAME` to it. Nothing else is published, the resolver,
antivirus, macro scanner, Tika, Redis, and the internal backends stay on the
project network.

## 📂 Volumes

- `./volumes/data/mailu/certs` - TLS certificates
- `./volumes/data/mailu/mail` - Mailbox storage
- `./volumes/data/mailu/mailqueue` - Outbound mail queue
- `./volumes/data/mailu/filter` - Rspamd data, ClamAV lives under
  `filter/clamav`
- `./volumes/data/mailu/dkim` - DKIM signing keys
- `./volumes/data/mailu/dav` - Radicale CalDAV storage
- `./volumes/data/mailu/webmail` - Webmail data
- `./volumes/data/mailu/fetchmail` - Fetchmail state
- `./volumes/config/mailu/nginx` - nginx override templates
- `./volumes/config/mailu/dovecot` - Dovecot override templates
- `./volumes/config/mailu/postfix` - Postfix override templates
- `./volumes/config/mailu/rspamd` - Rspamd override templates
- `./volumes/config/mailu/snappymail` - SnappyMail override templates
- `./volumes/db/sqlite/mailu` - Admin database
- `./volumes/db/redis/mailu` - Redis data

The `mailu-webdav`, `mailu-webmail`, `mailu-antivirus`, and `mailu-oletools`
networks exist to mirror the upstream Mailu layout, Radicale and the webmail
are not allowed to reach the rest of the stack directly.

## 🔐 Security

- Set `APP_SECRET_KEY` to 16 random bytes as hex, it signs internal tokens
  between containers
- Set `APP_ADMIN_PASS` before the first start, the admin account is only
  created while the database is empty
- `MAILU_API_TOKEN` authenticates machine clients against the Mailu API
- `MAILU_TLS_FLAVOR=cert` ships a self-signed certificate, switch to
  `letsencrypt` or mount your own certificate once the public hostname
  resolves, otherwise mail clients will warn
- The mail protocol ports are published on the standard numbers so the server
  can be reached from the internet, do not expose them to untrusted networks
  beyond the ports your clients need
- `MAILU_RECIPIENT_DELIMITER` defaults to `+` for plus addressing, set it
  blank to turn that off
- Keep `MAILU_DISABLE_STATISTICS=True` if you do not want usage statistics
- Back up `./volumes/db/sqlite/mailu` and `./volumes/data/mailu/mail`, the
  first is all accounts and the second is every mailbox
- Protect `./volumes/data/mailu/dkim`, those keys sign your outgoing mail
- Regularly update Docker images for security patches
- Check spam and antivirus settings, `clamav` takes a while to load its
  signature database on the first start

## 🔍 Logging

```shell
docker compose --env-file .env logs -f app
```

## 🛠️ Management

```bash
# Start services
docker compose --env-file .env up -d

# Stop services
docker compose --env-file .env down

# Update to latest images
docker compose --env-file .env pull && docker compose --env-file .env up -d

# View logs
docker compose --env-file .env logs -f

# Restart services
docker compose --env-file .env restart
```

## 📋 Requirements

- Docker Engine 20.10+
- Docker Compose V2+
- A public hostname for `BASE_HOST_NAME` that resolves to this host
- DNS MX and SPF records for `BASE_DOMAIN_NAME`
- Roughly 2GB of RAM, ClamAV alone wants about 1GB

## 🤝 Author

🤖 casjay: [Github](https://github.com/casjay) 🤖  
🦄 composemgr: [Github](https://github.com/composemgr) 🦄
