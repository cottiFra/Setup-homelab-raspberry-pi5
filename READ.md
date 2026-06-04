# cottiFra — Raspberry Pi 5 HomeLab

![Platform](https://img.shields.io/badge/platform-Raspberry%20Pi%205-c51a4a)
![OS](https://img.shields.io/badge/OS-Raspberry%20Pi%20OS%2064--bit%20Lite-a22846)
![Docker](https://img.shields.io/badge/docker-ready-2496ed)
![Domain](https://img.shields.io/badge/domain-cottihomelab.uk-f6821f)

Welcome to the official documentation of my personal HomeLab. This guide tracks the entire deployment journey, from hardware selection and OS flashing all the way to hosting cloud services and custom web applications under the domain `cottihomelab.uk`.

---

## Table of Contents

- [Phase 1 — Hardware & OS Selection](#phase-1--hardware--os-selection)
- [Phase 2 — Flashing & Initial Configuration](#phase-2--flashing--initial-configuration-headless)
- [Phase 3 — Installing Docker & Docker Compose](#phase-3--installing-docker--docker-compose)
- [Phase 4 — Setting Up Cloudflare Tunnels](#phase-4--setting-up-cloudflare-tunnels-zero-trust)
- [Phase 5 — Deploying Web Services](#phase-5--deploying-web-services)
- [Directory Structure](#directory-structure)
- [Prerequisites](#prerequisites)
- [Troubleshooting](#troubleshooting)

---

## Phase 1 — Hardware & OS Selection

### The Hardware

| Component | Details |
|---|---|
| SBC | Raspberry Pi 5 (8GB RAM) |
| Storage | MicroSD / NVMe SSD / SSD via USB 3.1 / USB Drive |
| Power Supply | Official Raspberry Pi 5 27W USB-C (required to prevent voltage drops under heavy Docker workloads) |

### The Operating System: Why Raspberry Pi OS 64-bit Lite?

**Raspberry Pi OS 64-bit (Lite/Headless)** was chosen for the following reasons:

1. **64-bit Architecture** — Essential to fully utilize the 8GB of RAM and ensure native compatibility with modern Docker images (ARM64).
2. **Lite version (no GUI)** — A server does not need a desktop environment. Removing the graphical interface frees hundreds of megabytes of RAM and CPU cycles, dedicated entirely to running containers.
3. **Debian stability** — Built on top of Debian Linux, guaranteeing rock-solid stability and long-term security updates.

---

## Phase 2 — Flashing & Initial Configuration (Headless)

The entire setup was performed in headless mode, without ever connecting the Raspberry Pi to a monitor or keyboard, managing everything remotely via SSH.

### 1. Flashing with Raspberry Pi Imager

Using the official **Raspberry Pi Imager** on the main PC, the following options were configured in the advanced OS customization panel before flashing:

- SSH server enabled (password authentication).
- Main user account (`cotti`) created with a secure password.
- Home network credentials pre-configured (Wi-Fi / Ethernet).

### 2. First Boot & System Update

Once the Raspberry Pi booted and registered on the local network, the IP address was retrieved either directly on the machine or from another device on the same network:

```bash
# On the Raspberry Pi
hostname -I

# From another device on the network
ping raspberrypi.local
```

With the IP confirmed, the first remote connection was established:

```bash
ssh cotti@192.168.1.x
```

Once connected, the system was fully updated:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Phase 3 — Installing Docker & Docker Compose

To run all services in an isolated, lightweight, and reproducible environment, Docker was installed along with the Docker Compose plugin.

### 1. Automated Installation Script

The cleanest way to install Docker on Raspberry Pi OS is via the official convenience script:

```bash
# Download the official Docker setup script
curl -fsSL https://get.docker.com -o get-docker.sh

# Execute the script to install Docker
sudo sh get-docker.sh
```

### 2. Managing Docker as a Non-Root User

To avoid typing `sudo` before every Docker command, the user `cotti` was added to the `docker` group:

```bash
sudo usermod -aG docker $USER
```

> **Note:** You must log out and log back in (or restart the SSH session) for this change to take effect.

### 3. Verify Installation

```bash
docker --version
docker compose version
```

---

## Phase 4 — Setting Up Cloudflare Tunnels (Zero Trust)

Instead of opening ports on the home router and exposing the home IP address to the public internet, a **Cloudflare Tunnel** was deployed to securely route external traffic to local services.

### 1. Homelab Directory Structure

A centralized directory was created to keep all configuration files organized:

```bash
mkdir -p ~/homelab
```

### 2. Launching the Cloudflare Connector

The tunnel daemon (`cloudflared`) is deployed as a detached container that automatically restarts on reboot:

```bash
docker run -d \
  --name cloudflare-tunnel \
  --restart unless-stopped \
  cloudflare/cloudflared:latest tunnel \
  --no-autoupdate run --token <YOUR_CLOUDFLARE_TOKEN>
```

> **Note:** The token is generated from the Cloudflare Zero Trust dashboard under **Networks > Tunnels**. Never commit it to a public repository.

---

## Phase 5 — Deploying Web Services

Every service is managed via its own dedicated `docker-compose.yml` file inside the `~/homelab/` directory.

### Nextcloud — Private Cloud Hub

| Property | Value |
|---|---|
| Internal Port | `8080` |
| Public Domain | `https://cloud.cottihomelab.uk` |

```bash
cd ~/homelab/nextcloud
docker compose up -d

# Allow the Cloudflare domain in Nextcloud's trusted domains
docker compose exec --user www-data app php occ config:system:set trusted_domains 2 --value=cloud.cottihomelab.uk
```

<details>
<summary>docker-compose.yml</summary>

```yaml
services:
  db:
    image: mariadb:10.11
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
      MYSQL_PASSWORD: nextcloudpassword
    volumes:
      - db_data:/var/lib/mysql

  app:
    image: nextcloud:latest
    restart: unless-stopped
    ports:
      - "8080:80"
    depends_on:
      - db
    environment:
      MYSQL_HOST: db
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
      MYSQL_PASSWORD: nextcloudpassword
    volumes:
      - nextcloud_data:/var/www/html

volumes:
  db_data:
  nextcloud_data:
```

</details>

---

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

<details>
<summary>docker-compose.yml</summary>

```yaml
services:
  web:
    image: php:8.2-apache
    restart: unless-stopped
    ports:
      - "8081:80"
    volumes:
      - ./html:/var/www/html

  db:
    image: mariadb:10.11
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: unibank
      MYSQL_USER: unibank
      MYSQL_PASSWORD: unibankpassword
    volumes:
      - unibank_db:/var/lib/mysql

volumes:
  unibank_db:
```

</details>

---

## Directory Structure

```
~/homelab/
├── nextcloud/
│   └── docker-compose.yml
└── unibank/
    ├── docker-compose.yml
    └── html/
        └── (PHP/HTML/CSS source files)
```

---

## Prerequisites

Before starting, make sure you have the following:

- A **Raspberry Pi 5** (4GB or 8GB RAM recommended)
- A storage medium: MicroSD (Class 10 / A2) or SSD via USB 3.1
- The **Official 27W USB-C Power Supply** (third-party supplies may cause instability)
- **Raspberry Pi Imager** installed on your main PC ([download here](https://www.raspberrypi.com/software/))
- A **Cloudflare account** with a registered domain and Zero Trust enabled
- Basic familiarity with SSH and the Linux command line

---

## Troubleshooting

### Docker: permission denied after `usermod`
The group change requires a full logout. If relogging does not work, try:
```bash
newgrp docker
```

### Nextcloud: "Access through untrusted domain" error
Run the trusted domains fix command again and verify the value was saved:
```bash
docker compose exec --user www-data app php occ config:system:get trusted_domains
```

### Cloudflare Tunnel: connector not connecting
Check the container logs for authentication errors:
```bash
docker logs cloudflare-tunnel
```
Ensure the token is valid and has not expired in the Zero Trust dashboard.

### SSH: cannot reach the Pi after reboot
Verify the Pi is on the network and confirm the IP has not changed (consider setting a static IP or DHCP reservation on your router).

---

*Documentation by [cottiFra](https://github.com/cottiFra) — Raspberry Pi 5 HomeLab*
