# HomeLab Setup — Command Reference

> Copy and paste these commands sequentially on your Raspberry Pi via SSH.

---

```bash
# -------------------------
# PHASE 2 — Initial Setup
# -------------------------

# Verify network IP
hostname -I

# Update the system
sudo apt update && sudo apt upgrade -y

# -------------------------
# PHASE 3 — Docker
# -------------------------

# Download and run the official Docker install script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to the docker group (re-login required after this)
sudo usermod -aG docker $USER

# Apply group change without re-logging
newgrp docker

# Verify installation
docker --version
docker compose version

# -------------------------
# PHASE 4 — Cloudflare Tunnel
# -------------------------

# Create homelab directory
mkdir -p ~/homelab

# Start the Cloudflare tunnel connector
docker run -d \
  --name cloudflare-tunnel \
  --restart unless-stopped \
  cloudflare/cloudflared:latest tunnel \
  --no-autoupdate run --token <YOUR_CLOUDFLARE_TOKEN>

# -------------------------
# PHASE 5 — Nextcloud
# -------------------------

cd ~/homelab/nextcloud
docker compose up -d

# Fix trusted domains
docker compose exec --user www-data app php occ config:system:set trusted_domains 2 --value=cloud.cottihomelab.uk

# Verify trusted domains
docker compose exec --user www-data app php occ config:system:get trusted_domains

# -------------------------
# PHASE 5 — Unibank
# -------------------------

mkdir -p ~/homelab/unibank/html
cd ~/homelab/unibank
docker compose up -d
```
