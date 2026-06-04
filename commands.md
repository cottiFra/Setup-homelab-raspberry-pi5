# Homelab Setup — Raspberry Pi 5

Documentation tracking by cottiFra. Built on a Raspberry Pi 5 infrastructure.

---

## Phase 3: Installing Docker & Docker Compose

To run all services in an isolated, lightweight, and easily reproducible environment, Docker was installed along with the Docker Compose plugin.

### Automated Installation Script

The cleanest way to install Docker on Raspberry Pi OS is by using the official convenience script provided by Docker:

```bash
# Download the official Docker setup script
curl -fsSL https://get.docker.com -o get-docker.sh

# Execute the script to install Docker
sudo sh get-docker.sh
```

### Managing Docker as a Non-Root User

To avoid typing `sudo` before every Docker command, the main user (`cotti`) was added to the `docker` group:

```bash
sudo usermod -aG docker $USER
```

> **Note:** You must log out and log back in (or restart your SSH session) for this change to take effect.

---

## Phase 4: Setting Up Cloudflare Tunnels (Zero Trust)

Instead of opening ports on the home router and exposing the home IP address to the public internet, a Cloudflare Tunnel was deployed to securely route external traffic to local services.

### Homelab Directory Structure

A centralized directory was created to keep all configuration files neatly organized:

```bash
mkdir -p ~/homelab
```

### Launching the Cloudflare Connector

The tunnel daemon (`cloudflared`) is deployed as a detached container that automatically restarts if the Raspberry Pi reboots:

```bash
docker run -d \
  --name cloudflare-tunnel \
  --restart unless-stopped \
  cloudflare/cloudflared:latest tunnel \
  --no-autoupdate run --token 
```

---

## Phase 5: Deploying Web Services

Every service is managed via its own dedicated `docker-compose.yml` file.

### Nextcloud — Private Cloud Hub

| Property | Value |
|---|---|
| Internal Port | `8080` |
| Public Domain | `https://cloud.cottihomelab.uk` |

```bash
cd ~/homelab/nextcloud
docker compose up -d

# Fix trusted domains to allow the Cloudflare URL
docker compose exec --user www-data app php occ config:system:set trusted_domains 2 --value=cloud.cottihomelab.uk
```

### Unibank — Custom PHP/HTML/CSS Website

| Property | Value |
|---|---|
| Internal Port | `8081` |
| Public Domain | `https://unibank.cottihomelab.uk` |

```bash
mkdir -p ~/homelab/unibank/html
cd ~/homelab/unibank
docker compose up -d
```
