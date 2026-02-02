# Domain Replacement Guide

*Complete list of all files where you need to replace `gameapp.games` with your own domain*

## 🚨 **IMPORTANT: Replace Demo Domain Before Starting**

**Before you begin this project, you MUST replace all instances of `gameapp.games` with your own domain name.**

**Example:** If your domain is `myapp.com`, replace:
- `gameapp.games` → `myapp.com`
- `app.gameapp.games` → `app.myapp.com`
- `prometheus.gameapp.games` → `prometheus.myapp.com`

## 📁 **Files That Need Domain Replacement**

### **1. Kubernetes Configuration Files**

#### **`k8s/ingress.yaml`**
```yaml
# Lines 15, 59, 122, 123, 146, 156, 185, 198
# Replace ALL instances of:
- gameapp.games  # Production domain with SSL
- host: gameapp.games  # For production (with SSL)
- prometheus.gameapp.games  # Production Prometheus with SSL
- grafana.gameapp.games     # Production Grafana with SSL
- host: prometheus.gameapp.games  # Production Prometheus access
- host: grafana.gameapp.games  # Production Grafana access
- argocd.gameapp.games  # Production ArgoCD with SSL
- host: argocd.gameapp.games  # Production ArgoCD access
```

#### **`k8s/tunnel-ingress.yaml`**
```yaml
# Line 12
# Replace:
- host: app.gameapp.games  # Tunnel subdomain (no SSL redirect)
```

### **2. Backend Configuration**

#### **`backend/server.js`**
```javascript
// Line 39
// Replace:
'https://gameapp.games:8443',  // Add your domain for production
```

#### **`k8s/configmap.yaml`**
```yaml
# Line 13
# Replace:
CORS_ORIGIN: "http://localhost:3000,http://localhost:8080,https://gameapp.games"
```

#### **`gitops-safe/base/configmap.yaml`**
```yaml
# Line 13
# Replace:
CORS_ORIGIN: "http://localhost:3000,http://localhost:8080,https://gameapp.games"
```

### **3. Nginx Configuration**

#### **`nginx/conf.d/default.conf`**
```nginx
# Line 14
# Replace:
server_name localhost gameapp.games;
```

#### **`nginx/conf.d.dev/default.conf`**
```nginx
# Line 15
# Replace:
server_name localhost gameapp.games;
```

### **4. Cloudflare Tunnel Configuration**

#### **`~/.cloudflared/config.yml`**
```yaml
# Lines 4, 5, 13, 20
# Replace:
- hostname: app.gameapp.games
- hostname: prometheus.app.gameapp.games
- hostname: grafana.app.gameapp.games
```

### **5. Frontend Configuration**

#### **`frontend/src/config.js`**
```javascript
// Lines 31, 35
// Replace:
} else if (hostname === 'gameapp.games') {
} else if (hostname.includes('gameapp.games') || hostname.includes('app.gameapp.games')) {
```

### **6. Documentation Files**

#### **`README.md`**
```markdown
# Lines 332, 349, 351
# Replace:
security@gameapp.games
support@gameapp.games
docs.gameapp.games
```

#### **`docs/monitoring-access-guide.md`**
```markdown
# Lines 34, 35, 57, 58
# Replace:
prometheus.gameapp.games
grafana.gameapp.games
```

#### **`scripts/access-monitoring.sh`**
```bash
# Lines 124, 125
# Replace:
prometheus.gameapp.games
grafana.gameapp.games
```

#### **`Makefile`**
```makefile
# Line 220
# Replace:
@echo "🌐 Access your app: https://gameapp.games"
```

## 🔧 **How to Replace Domains**

### **Method 1: Find and Replace (Recommended)**
```bash
# Replace all instances in one command
find . -type f -name "*.yaml" -o -name "*.js" -o -name "*.md" -o -name "*.conf" -o -name "*.yml" | xargs sed -i '' 's/gameapp\.games/YOUR_DOMAIN/g'

# Replace subdomain patterns
find . -type f -name "*.yaml" -o -name "*.js" -o -name "*.md" -o -name "*.conf" -o -name "*.yml" | xargs sed -i '' 's/app\.gameapp\.games/app.YOUR_DOMAIN/g'
find . -type f -name "*.yaml" -o -name "*.js" -o -name "*.md" -o -name "*.conf" -o -name "*.yml" | xargs sed -i '' 's/prometheus\.gameapp\.games/prometheus.YOUR_DOMAIN/g'
find . -type f -name "*.yaml" -o -name "*.js" -o -name "*.md" -o -name "*.conf" -o -name "*.yml" | xargs sed -i '' 's/grafana\.gameapp\.games/grafana.YOUR_DOMAIN/g'
find . -type f -name "*.yaml" -o -name "*.js" -o -name "*.md" -o -name "*.conf" -o -name "*.yml" | xargs sed -i '' 's/argocd\.gameapp\.games/argocd.YOUR_DOMAIN/g'
```

### **Method 2: Manual Replacement**
1. Open each file listed above
2. Use Ctrl+F (Cmd+F on Mac) to find `gameapp.games`
3. Replace with your domain
4. Save the file

## 📋 **Domain Replacement Checklist**

**Before starting Milestone 1, verify you've replaced domains in:**

- [ ] `k8s/ingress.yaml`
- [ ] `k8s/tunnel-ingress.yaml`
- [ ] `backend/server.js`
- [ ] `k8s/configmap.yaml`
- [ ] `gitops-safe/base/configmap.yaml`
- [ ] `nginx/conf.d/default.conf`
- [ ] `nginx/conf.d.dev/default.conf`
- [ ] `~/.cloudflared/config.yml` (after setting up Cloudflare)
- [ ] `frontend/src/config.js`
- [ ] `README.md`
- [ ] `docs/monitoring-access-guide.md`
- [ ] `scripts/access-monitoring.sh`
- [ ] `Makefile`

## ⚠️ **Common Issues If You Don't Replace Domains**

1. **Card flipping won't work** - API calls will fail
2. **Tunnel won't connect** - Wrong hostname in configuration
3. **SSL certificates won't generate** - Domain mismatch
4. **Monitoring won't be accessible** - Wrong subdomain routing
5. **GitOps won't work** - Domain configuration errors

## 🎯 **Quick Start for Beginners**

1. **Choose your domain** (e.g., `myapp.com`)
2. **Run the find and replace commands** above
3. **Verify replacements** with the checklist
4. **Start with Milestone 1** - everything will work properly

## 🔍 **Verification Commands**

After replacing domains, verify with:
```bash
# Check if any instances remain
grep -r "gameapp.games" . --exclude-dir=node_modules --exclude-dir=.git

# Should return no results
```

---

**💡 Pro Tip:** Replace domains BEFORE starting the project. It's much easier than fixing issues later!

**🚨 Warning:** If you don't replace domains, the card flipping functionality will break and you'll get stuck at Milestone 1.
