# Outline Self-Hosted Wiki with Dex OIDC Authentication

This repository provides a ready-to-use setup for running [Outline](https://github.com/outline/outline), a modern team knowledge base, with [Dex](https://github.com/dexidp/dex) as an OpenID Connect (OIDC) provider for authentication. The stack is orchestrated using Docker Compose and includes PostgreSQL and Redis as dependencies.

## Features

- **Outline**: Self-hosted wiki and knowledge base.
- **Dex**: OIDC provider for authentication (with example static user).
- **PostgreSQL**: Database for Outline.
- **Redis**: Caching for Outline.
- **SMTP**: Email configuration support for notifications.

## Prerequisites

- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/)

## Getting Started

### 1. Clone the Repository

First, clone this repository to your local machine:


```bash
git clone https://github.com/your-org/outline-dex-docker.git
cd outline-dex-docker
```

### 2. Configure Environment Variables

Copy the example environment file and edit the values:

```bash
cp example.env .env
```

Update the following in `.env`:

- **URL**: Public URL where Outline will be accessed (e.g., `http://YOUR_HOST:3001`).
- **DEX_URL**: Dex base URL (e.g., `http://YOUR_HOST:5556`).
- **POSTGRES_PASSWORD**: Set a strong password for the `outline` DB user.
- **SECRET_KEY** and **UTILS_SECRET**: Set long random strings.
- **CLIENT_SECRET**: Must match Dex static client secret in `config/dex/config.yaml`.
- Optional SMTP settings if you need email notifications.

Quick way to generate secrets (macOS/Linux):

```bash
openssl rand -base64 32
```

### 3. Align Dex Configuration

Open `config/dex/config.yaml` and ensure these values align with your `.env`:

- `issuer`: Must equal `DEX_URL` in `.env` (e.g., `http://YOUR_HOST:5556`).
- `staticClients[0].redirectURIs[0]`: Must equal `URL` + `/auth/oidc.callback`
  - Example: if `URL` is `http://YOUR_HOST:3001`, set `http://YOUR_HOST:3001/auth/oidc.callback`.
- `staticClients[0].secret`: Must equal `CLIENT_SECRET` in `.env`.

If you change any of these, save the file before continuing.

Default static user (for quick start):

- Email: `admin@example.com`
- Password: `password`

You can replace the `staticPasswords` entry with your own bcrypt hash later.

### 4. Start the Stack

Run the containers:

```bash
docker compose up -d
```

This starts services: `outline` (web), `dex` (OIDC), `postgres`, and `redis`.

### 5. Access Outline

- Open your browser to the value of `URL` (e.g., `http://YOUR_HOST:3001`).
- Click "Sign in with OIDC".
- Log in with the Dex user above (`admin@example.com` / `password`).

### 6. Logs and Troubleshooting

- Check logs for a service:

```bash
docker compose logs -f outline
docker compose logs -f dex
```

- Common issues:
  - Make sure `URL`, `DEX_URL`, and Dex `issuer` all point to reachable hostnames/IPs from your browser.
  - Ensure `CLIENT_SECRET` in `.env` equals Dex `staticClients[0].secret`.
  - If ports are in use, change the published ports in `docker-compose.yaml` (e.g., `3001:3000`, `5556:5556`).

### 7. Stopping and Removing

```bash
docker compose down
```

To remove volumes as well (erases DB and uploads):

```bash
docker compose down -v
```

### 8. Customization Notes

- File storage is local by default (`/var/lib/outline/data`). Mount or switch to S3-compatible storage for production.
- Update SMTP variables if you need email features.
- Consider persisting Dex state and moving from in-memory storage for production setups.


### 9. Contributing

We welcome contributors!  
If you would like to help develop, fix, or add features to this project, please open an issue or submit a pull request.  
Feel free to discuss or suggest improvements via GitHub Issues.
