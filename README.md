# Matrix Server Stack

Self-hosted Matrix homeserver based on **Synapse + MAS + LiveKit + Element Call**, proxied through **Caddy**.

## Stack

| Service | Purpose |
|---|---|
| [Synapse](https://github.com/element-hq/synapse) | Matrix homeserver |
| [MAS](https://github.com/element-hq/matrix-authentication-service) | OIDC authentication |
| [LiveKit](https://livekit.io/) | WebRTC media server for calls |
| [lk-jwt-service](https://github.com/element-hq/lk-jwt-service) | JWT tokens for LiveKit |
| [Synapse Admin](https://github.com/Awesome-Technologies/synapse-admin) | Web admin panel |
| [Caddy](https://caddyserver.com/) | Reverse proxy + automatic TLS |
| PostgreSQL 16 | Database |

## Quick Start

### 1. Clone the repo
```bash
git clone https://github.com/YOUR_USERNAME/matrix-server.git
cd matrix-server
```

### 2. Prepare the configs

Replace all `YOUR_*` placeholders with real values:
```bash
grep -r "YOUR_" .
```

| Placeholder | What to set |
|---|---|
| `YOUR_DOMAIN` | Your domain, e.g. `example.com` |
| `YOUR_POSTGRES_PASSWORD` | Strong password for PostgreSQL |
| `YOUR_LIVEKIT_API_KEY` | Any string — LiveKit API key |
| `YOUR_LIVEKIT_API_SECRET` | Long random string — LiveKit API secret |
| `YOUR_MAS_SYNAPSE_SHARED_SECRET` | Shared secret between MAS and Synapse |
| `YOUR_SYNAPSE_OAUTH_CLIENT_SECRET` | OAuth client secret for Synapse in MAS |
| `YOUR_MACAROON_SECRET_KEY` | Secret for Synapse macaroon tokens |
| `YOUR_FORM_SECRET` | Secret for Synapse form tokens |
| `YOUR_SMTP_HOST` | SMTP server hostname or IP |
| `YOUR_SMTP_PASSWORD` | SMTP password |

Generate random secrets:
```bash
openssl rand -hex 32       # for most secrets
openssl rand -base64 48    # for macaroon_secret_key / form_secret
```

### 3. Generate MAS signing keys
```bash
# RSA key
openssl genrsa 2048

# EC keys
openssl ecparam -name prime256v1 -genkey -noout
openssl ecparam -name secp384r1 -genkey -noout
openssl ecparam -name secp256k1 -genkey -noout
```

Paste the output into `mas-config.yaml` under `secrets.keys`.

### 4. Initialize Synapse
```bash
docker compose run --rm synapse generate
```

### 5. Start
```bash
docker compose up -d
```

### 6. Verify
```bash
# Check all services are running
docker compose ps

# Check well-known endpoints
curl https://YOUR_DOMAIN/.well-known/matrix/client
curl https://YOUR_DOMAIN/.well-known/matrix/server
```

## Repository Structure
```
.
├── docker-compose.yml       # All services
├── Caddyfile                # Reverse proxy routing
├── mas-config.yaml          # MAS (OIDC) config
├── livekit.yaml             # LiveKit config
├── .gitignore
└── synapse-data/
    └── homeserver.yaml      # Synapse config
```

## Routing (Caddy)
```
YOUR_DOMAIN
├── /.well-known/matrix/     → MAS / inline JSON
├── /auth/                   → MAS (OIDC UI)
├── /account/                → MAS (account management)
├── /_matrix/                → Synapse
├── /_synapse/               → Synapse
├── /livekit/jwt/            → lk-jwt-service
├── /livekit/sfu             → LiveKit
└── /admin/                  → Synapse Admin
```

## Security Notes

- Store real configs in a separate **private** repository
- Never commit files with real passwords or keys
- Use `.gitignore` to exclude sensitive files
- Federation is disabled by default (`federation_domain_whitelist: []`)

## License

MIT
