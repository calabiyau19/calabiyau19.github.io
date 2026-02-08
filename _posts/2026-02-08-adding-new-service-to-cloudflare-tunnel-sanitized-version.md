---
layout: post
draft: false
title: "Adding a New Service to an Existing Reverse Proxy, Caddy and Cloudflare Tunnel - Generic Version"
date: 2026-02-07
description: "This guide documents adding a self-hosted application to an existing Cloudflare Tunnel and Caddy reverse proxy infrastructure in a Proxmox-based homelab environment. The goal is to expose the application via HTTPS with Cloudflare Access authentication, enabling secure remote access."
---


## Environment Overview

**Homelab Infrastructure:**

- Proxmox hypervisors running multiple Ubuntu VMs
- **proxy-vm** (192.168.1.X): Dedicated VM running Cloudflare Tunnel and Caddy reverse proxy
- **app-vm** (192.168.1.Y): VM running your application in Docker containers on port 3004
- Domain: `example.com` managed through Cloudflare
- Existing services already tunneled: Multiple subdomains already configured

**Target Configuration:**
- New subdomain: `newapp.example.com`
- HTTPS access with Cloudflare Access authentication
- Same authentication pattern as existing services

**Note on Tailscale:** If you have Tailscale configured on the app-vm for remote access, be aware that some applications can only use one frontend URL in their configuration. This guide configures it for Cloudflare Tunnel access. The `.env` file must use the public HTTPS URL (`https://newapp.example.com`), not a Tailscale IP or local IP address.

## Step 1: Update Cloudflare Tunnel Configuration

The Cloudflare Tunnel running on proxy-vm needs to know about the new subdomain.

```sh
sudo nano /etc/cloudflared/config.yml
```

This opens the tunnel configuration file. Add the new hostname entry to the ingress section, before the final 404 catch-all line. The complete file should look like this:

```yaml
tunnel: your-tunnel-name
credentials-file: /home/username/.cloudflared/YOUR-TUNNEL-UUID.json
ingress:
  - hostname: service1.example.com
    service: http://localhost:80
  - hostname: service2.example.com
    service: http://localhost:80
  - hostname: service3.example.com
    service: http://localhost:80
  - hostname: newapp.example.com
    service: http://localhost:80
  - service: http_status:404
```

Save and exit (Ctrl+X, Y, Enter). The new entry tells the tunnel to accept traffic for `newapp.example.com` and forward it to Caddy running on localhost port 80.

Verify the configuration was saved correctly:

```sh
sudo cat /etc/cloudflared/config.yml
```

This displays the file contents. Confirm the newapp.example.com entry appears in the correct location.

## Step 2: Add DNS Record in Cloudflare

Cloudflare's DNS needs a CNAME record pointing the new subdomain to your tunnel.

Navigate to the Cloudflare Dashboard at https://dash.cloudflare.com, select your `example.com` domain, then click **DNS** in the left sidebar.

Click **Add record** and configure:
- **Type**: CNAME
- **Name**: newapp
- **Target**: `YOUR-TUNNEL-UUID.cfargotunnel.com` (same as other services)
- **Proxy status**: Proxied (orange cloud icon enabled)

Click **Save**. This creates the DNS routing that connects newapp.example.com to your Cloudflare Tunnel. DNS propagation is typically instant with Cloudflare.

## Step 3: Configure Caddy Reverse Proxy

Caddy on proxy-vm needs to route traffic from the tunnel to your application on app-vm.

```sh
sudo nano /etc/caddy/Caddyfile
```

This opens the Caddy configuration. Add a new reverse proxy block for your application. It should match the pattern of your existing services:

```
http://newapp.example.com {
    reverse_proxy 192.168.1.Y:3004
}
```

Save and exit. This configuration tells Caddy: when traffic arrives for newapp.example.com, forward it to the application running on 192.168.1.Y port 3004.

Note: The `http://` prefix is used because traffic between Cloudflare Tunnel and Caddy happens locally. Cloudflare handles the HTTPS termination before traffic reaches your network.

Verify the configuration:

```sh
sudo cat /etc/caddy/Caddyfile
```

Confirm the new block was added correctly and matches your other service configurations.

## Step 4: Restart Services on proxy-vm

Both Caddy and Cloudflare Tunnel need to reload their updated configurations.

```sh
sudo systemctl restart caddy
```

This restarts the Caddy web server with the new reverse proxy configuration.

```sh
sudo systemctl status caddy --no-pager
```

Verify Caddy is running. Look for **"Active: active (running)"** in green text. If you see red text or errors, check the Caddyfile syntax.

```sh
sudo systemctl restart cloudflared
```

This restarts the Cloudflare Tunnel daemon with the updated ingress rules.

```sh
sudo systemctl status cloudflared --no-pager
```

Verify cloudflared is running. Look for **"Active: active (running)"**. Both services must be active for the tunnel to work.

## Step 5: Add Cloudflare Access Application

Cloudflare Access provides authentication before users can reach your application, matching your other services.

Navigate to https://one.dash.cloudflare.com and click **Access** in the left sidebar (under Zero Trust), then click **Applications**.

### Create New Application

Click **Add an application** and select **Self-hosted**.

Configure the basic information:
- **Application name**: Your Application Name
- **Session Duration**: 24 hours (default)

Click **+ Add public hostname**:
- **Subdomain**: newapp
- **Domain**: example.com (select from dropdown)

Click **Next** to continue.

### Skip Optional Settings

The "Experience settings (optional)" page can be skipped - click **Next**.

The "Advanced settings (optional)" page can also be skipped, but don't click Save yet.

### Assign Access Policy

Before saving, you need to assign an access policy. You should see "0 policies assigned" - this needs to change to "1 policy assigned".

Look for an option to select an existing policy or add a policy. Select an existing policy that your other applications use (commonly named something like "Allow admins" or "Email authentication").

This policy determines who can authenticate and access the application. Using the same policy keeps all your services consistent.

Click **Save** to create the application.

Verify the application appears in your Applications list showing "1 policy assigned".

## Step 6: Configure Application for Public Access

Your application needs to know its public-facing URL for proper authentication and CORS handling.

```sh
cd ~/your-app-directory
```

This navigates to your application's installation directory on app-vm.

```sh
nano .env
```

This opens the environment configuration file. Find the line containing your frontend URL variable and change it to the public HTTPS URL:

```
FRONTEND_URL=https://newapp.example.com
```

Critical: Use `https://` and the public domain name. Do NOT use the local IP address (192.168.1.Y:3004) or a Tailscale IP address. The application validates requests against this URL for security.

Save and exit (Ctrl+X, Y, Enter).

Verify the change was saved correctly:

```sh
grep FRONTEND_URL .env
```

This searches the file and displays only the frontend URL line. It should show: `FRONTEND_URL=https://newapp.example.com`

## Step 7: Restart Application Containers

The application containers need to restart to pick up the new environment variable.

```sh
docker compose down
```

This stops and removes all application containers.

```sh
docker compose up -d
```

This recreates and starts all containers in detached mode with the updated configuration. The `-d` flag runs them in the background.

Verify all containers are running and healthy:

```sh
docker ps
```

This displays running containers. All your application containers should show status as "Up" and "healthy". If any show "unhealthy" or are missing, check the logs with `docker compose logs`.

## Step 8: Test Access

Open an incognito or private browser window and navigate to:

```
https://newapp.example.com
```

Expected authentication flow:
1. **Cloudflare Access screen**: Authentication prompt (method depends on your policy)
2. **Identity verification**: Enter credentials or complete authentication
3. **Application login page**: The application's own login screen (if it has one)
4. **Application access**: Full access to your application

If this flow works correctly, the setup is complete.

### Test Mobile Access

If your application has mobile-specific features (like camera access for barcode scanning), test on your phone:

On your mobile device:
1. Open your browser and go to `https://newapp.example.com`
2. Complete the Cloudflare Access authentication
3. Log into your application
4. Test any mobile-specific features

The application should now work over the secure HTTPS connection.

## Data Persistence

Application data is stored outside the Docker containers and persists across restarts, updates, and system reboots. Check your `docker-compose.yml` configuration for volume mounts to understand where your data is stored.

Typical volume locations:
- **Database**: Persistent storage for application data
- **Backups**: Backup files
- **Uploads**: User-uploaded content

This data remains intact when containers are stopped, removed, or updated.

## Summary

You have successfully:
- Added newapp.example.com to your Cloudflare Tunnel ingress configuration
- Created the DNS CNAME record in Cloudflare
- Configured Caddy reverse proxy to route traffic to your application
- Added Cloudflare Access authentication matching your other services
- Configured your application to use the public HTTPS URL
- Enabled secure HTTPS access from anywhere

Your application is now accessible securely from anywhere with the same authentication protection as your other self-hosted services.
