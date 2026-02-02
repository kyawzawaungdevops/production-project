# Cloudflare Tunnel Setup Guide

## Overview
This guide provides step-by-step instructions for setting up Cloudflare Tunnels using both the terminal (cloudflared CLI) and the Cloudflare Zero Trust dashboard.

## Prerequisites
- Cloudflare account with Zero Trust enabled
- Domain added to Cloudflare
- `cloudflared` CLI installed on your machine

## Method 1: Terminal Setup (Recommended for Development)

### Step 1: Install cloudflared CLI

#### macOS (using Homebrew)
```bash
brew install cloudflare/cloudflare/cloudflared
```

#### Manual Installation
```bash
# Download latest version
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-darwin-amd64 -o cloudflared

# Make executable
chmod +x cloudflared

# Move to PATH
sudo mv cloudflared /usr/local/bin/
```

### Step 2: Authenticate with Cloudflare
```bash
# Login to Cloudflare
cloudflared tunnel login

# This will open your browser to authenticate
# Select your domain (e.g., gameapp.games)
# Download the certificate file to ~/.cloudflared/
```

### Step 3: Create a New Tunnel
```bash
# Create tunnel
cloudflared tunnel create gameapp-tunnel

# This will output:
# Tunnel gameapp-tunnel created with ID: [TUNNEL_ID]
# Save the ID for later use
```

### Step 4: Configure Tunnel Ingress Rules
Create `~/.cloudflared/config.yml`:

```yaml
tunnel: [TUNNEL_ID]  # Replace with your actual tunnel ID
credentials-file: ~/.cloudflared/[TUNNEL_ID].json

ingress:
  # Main application
  - hostname: app.gameapp.games
    service: http://172.20.10.3:8080
    originRequest:
      originServerName: gameapp.local
      noTLSVerify: true
      disableChunkedEncoding: true

  # Main domain
  - hostname: gameapp.games
    service: http://172.20.10.3:8080
    originRequest:
      originServerName: gameapp.games
      noTLSVerify: true
      disableChunkedEncoding: true

  # Monitoring services
  - hostname: grafana.gameapp.games
    service: http://172.20.10.3:8080
    originRequest:
      originServerName: grafana.gameapp.local
      noTLSVerify: true
      disableChunkedEncoding: true

  - hostname: prometheus.gameapp.games
    service: http://172.20.10.3:8080
    originRequest:
      originServerName: prometheus.gameapp.local
      noTLSVerify: true
      disableChunkedEncoding: true

  # ArgoCD
  - hostname: argocd.gameapp.games
    service: http://172.20.10.3:8080
    originRequest:
      originServerName: argocd.gameapp.local
      noTLSVerify: true
      disableChunkedEncoding: true

  # Catch-all for unmatched hostnames
  - service: http_status:404
```

### Step 5: Run the Tunnel
```bash
# Start tunnel in foreground
cloudflared tunnel run gameapp-tunnel

# Or run in background
cloudflared tunnel run gameapp-tunnel &
```

### Step 6: Route Traffic to Tunnel
```bash
# Route traffic for your domain to the tunnel
cloudflared tunnel route dns gameapp-tunnel gameapp.games
cloudflared tunnel route dns gameapp-tunnel app.gameapp.games
cloudflared tunnel route dns gameapp-tunnel grafana.gameapp.games
cloudflared tunnel route dns gameapp-tunnel prometheus.gameapp.games
cloudflared tunnel route dns gameapp-tunnel argocd.gameapp.games
```

## Method 2: Dashboard Setup (Recommended for Production)

### Step 1: Access Cloudflare Zero Trust Dashboard
1. Go to [https://one.dash.cloudflare.com/](https://one.dash.cloudflare.com/)
2. Select your account/zone
3. Navigate to **Networks** → **Tunnels**

### Step 2: Create New Tunnel
1. Click **"Create a tunnel"**
2. Enter tunnel name: `gameapp-tunnel`
3. Click **"Save tunnel"**

### Step 3: Configure Tunnel
1. **Download cloudflared** (if not already installed)
2. **Run the provided command** to authenticate:
   ```bash
   cloudflared service install [TOKEN_FROM_DASHBOARD]
   ```

### Step 4: Add Public Hostnames
1. Click **"Configure"** on your tunnel
2. Click **"Add a public hostname"**
3. Add each hostname:

#### Main Application
- **Subdomain:** `app`
- **Domain:** `gameapp.games`
- **Service Type:** `HTTP`
- **Service URL:** `172.20.10.3:8080`

#### Main Domain
- **Subdomain:** `@` (root domain)
- **Domain:** `gameapp.games`
- **Service Type:** `HTTP`
- **Service URL:** `172.20.10.3:8080`

#### Grafana
- **Subdomain:** `grafana`
- **Domain:** `gameapp.games`
- **Service Type:** `HTTP`
- **Service URL:** `172.20.10.3:8080`

#### Prometheus
- **Subdomain:** `prometheus`
- **Domain:** `gameapp.games`
- **Service Type:** `HTTP`
- **Service URL:** `172.20.10.3:8080`

#### ArgoCD
- **Subdomain:** `argocd`
- **Domain:** `gameapp.games`
- **Service Type:** `HTTP`
- **Service URL:** `172.20.10.3:8080`

### Step 5: Save Configuration
1. Click **"Save"** for each hostname
2. Verify all hostnames are listed
3. Check tunnel status shows "Healthy"

## Method 3: Hybrid Approach (Best of Both Worlds)

### Step 1: Create Tunnel via Dashboard
1. Create tunnel in Zero Trust dashboard
2. Download and install cloudflared service

### Step 2: Use Local Config for Development
1. Create local `~/.cloudflared/config.yml`
2. Use local config for development/testing
3. Dashboard config for production

### Step 3: Switch Between Configs
```bash
# Use local config
cloudflared tunnel run --config ~/.cloudflared/config.yml gameapp-tunnel

# Use dashboard config (default)
cloudflared tunnel run gameapp-tunnel
```

## Verification and Testing

### Check Tunnel Status
```bash
# Check tunnel info
cloudflared tunnel info gameapp-tunnel

# List all tunnels
cloudflared tunnel list

# Check tunnel routes
cloudflared tunnel route dns list
```

### Test Connectivity
```bash
# Test local connectivity
curl -H "Host: gameapp.games" http://172.20.10.3:8080/

# Test through tunnel
curl -I https://gameapp.games
curl -I https://app.gameapp.games
curl -I https://grafana.gameapp.games
curl -I https://prometheus.gameapp.games
curl -I https://argocd.gameapp.games
```

## Troubleshooting

### Common Issues

#### Tunnel Status "DOWN"
1. Check if cloudflared is running
2. Verify credentials file exists
3. Check network connectivity
4. Verify tunnel ID matches

#### DNS Resolution Issues
1. Verify CNAME records point to tunnel
2. Check tunnel routes are configured
3. Wait for DNS propagation (up to 24 hours)

#### Connection Timeouts
1. Verify service IP/port is correct
2. Check firewall settings
3. Verify Kubernetes ingress is working
4. Check tunnel ingress rules

### Debug Commands
```bash
# Run with debug logging
cloudflared tunnel run --loglevel debug gameapp-tunnel

# Check tunnel logs
cloudflared tunnel info gameapp-tunnel

# Test specific hostname
cloudflared tunnel --loglevel debug run --url http://172.20.10.3:8080
```

## Security Considerations

### Best Practices
1. **Use HTTPS** - Always use HTTPS for production
2. **Restrict Access** - Use Cloudflare Access for authentication
3. **Monitor Logs** - Regularly check tunnel logs
4. **Update Regularly** - Keep cloudflared updated
5. **Secure Credentials** - Protect certificate files

### Access Control
1. **IP Restrictions** - Limit access to specific IP ranges
2. **Authentication** - Use Cloudflare Access policies
3. **Audit Logs** - Monitor access patterns

## Maintenance

### Regular Tasks
1. **Check Tunnel Status** - Daily monitoring
2. **Update cloudflared** - Monthly updates
3. **Review Logs** - Weekly log analysis
4. **Backup Configs** - Configuration backup

### Backup Configuration
```bash
# Backup tunnel config
cp ~/.cloudflared/config.yml ~/.cloudflared/config.yml.backup

# Backup credentials
cp ~/.cloudflared/*.json ~/.cloudflared/backup/
```

## Next Steps

After setting up your tunnel:

1. **Deploy Kubernetes Resources** - Follow home-lab.md deployment steps
2. **Configure Ingress** - Set up proper ingress rules
3. **Test All Services** - Verify all hostnames work
4. **Monitor Performance** - Set up monitoring and alerting
5. **Document Changes** - Keep this guide updated

---

## Quick Reference Commands

```bash
# Essential commands
cloudflared tunnel login                    # Authenticate
cloudflared tunnel create [NAME]            # Create tunnel
cloudflared tunnel list                     # List tunnels
cloudflared tunnel info [NAME]              # Tunnel info
cloudflared tunnel run [NAME]               # Run tunnel
cloudflared tunnel route dns [NAME] [HOST]  # Route DNS
cloudflared tunnel delete [NAME]            # Delete tunnel
```

---

*Last updated: $(date)*
*For issues, check the troubleshooting section or refer to [Cloudflare documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)*
