# Navigate to the Nextcloud directory
cd ~/homelab/nextcloud

# Start all containers in detached mode
docker compose up -d

# Check that all containers are running
docker compose ps

# -------------------------
# Trusted Domains
# -------------------------

# Add local IP (for access on the local network)
docker compose exec --user www-data app php occ config:system:set trusted_domains 1 --value=192.168.x.x

# Add public domain (for access via Cloudflare Tunnel)
docker compose exec --user www-data app php occ config:system:set trusted_domains 2 --value=cloud.cottihomelab.uk

# Verify trusted domains are set correctly
docker compose exec --user www-data app php occ config:system:get trusted_domains

# -------------------------
# Permissions Fix
# -------------------------

# Fix ownership of config.php if Nextcloud shows permission errors
docker compose exec app chown www-data:www-data /var/www/html/config.php

# -------------------------
# Logs & Debugging
# -------------------------

# Follow live logs for the app container
docker compose logs -f app

# Show last 20 lines of app logs
docker compose logs app --tail 20

# -------------------------
# Maintenance
# -------------------------

# Restart all containers
docker compose restart

# Stop all containers
docker compose down

# Pull latest images and restart
docker compose pull && docker compose up -d
