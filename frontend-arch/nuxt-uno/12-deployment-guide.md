# Deployment Guide

## Overview

This document covers deployment strategies for Nuxt applications, including **SPA (Single Page Application)**, **SSR (Server-Side Rendering)**, and **Hybrid** modes. Each deployment mode has specific requirements, configurations, and platform considerations.

## Deployment Modes Comparison

| Mode | Server Required | SEO | Initial Load | Dynamic Content |
|------|-----------------|-----|--------------|-----------------|
| **SPA** | No (static hosting) | Limited | Fast | Client-side only |
| **SSR** | Yes (Node.js) | Full | Slower | Full support |
| **SSG** | No (static hosting) | Full | Fast | Build-time only |
| **Hybrid** | Yes/Optional | Full | Optimized | Route-specific |

---

## SPA Deployment (Static)

### Configuration

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Enable SPA mode
  ssr: false,
  
  // Generate static files
  nitro: {
    preset: 'static',
  },
  
  // Configure app
  app: {
    baseURL: '/', // or '/subfolder/' for subdirectory deployment
    buildAssetsDir: '/_nuxt/',
  },
  
  // Route rules for SPA
  routeRules: {
    '/**': { ssr: false },
  },
})
```

### Build Command

```bash
# Generate static SPA
pnpm generate

# Output in .output/public/
```

### GitHub Pages

```yaml
# .github/workflows/deploy-gh-pages.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v2
        with:
          version: 8
          
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
          
      - name: Install dependencies
        run: pnpm install
        
      - name: Build
        run: pnpm generate
        env:
          NUXT_APP_BASE_URL: /${{ github.event.repository.name }}/
          
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: .output/public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Netlify (SPA)

```toml
# netlify.toml
[build]
  command = "pnpm generate"
  publish = ".output/public"

[build.environment]
  NODE_VERSION = "20"

# SPA fallback for client-side routing
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### Vercel (SPA)

```json
// vercel.json
{
  "buildCommand": "pnpm generate",
  "outputDirectory": ".output/public",
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

### Cloudflare Pages (SPA)

```toml
# wrangler.toml (optional)
name = "my-app"
compatibility_date = "2024-01-01"

[site]
bucket = ".output/public"
```

Dashboard settings:
- Build command: `pnpm generate`
- Build output directory: `.output/public`
- Root directory: `/`

---

## SSR Deployment (Server)

### Configuration

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Enable SSR (default)
  ssr: true,
  
  // Server preset
  nitro: {
    preset: 'node-server', // or 'node-cluster' for multi-core
  },
})
```

### Build and Start

```bash
# Build for production
pnpm build

# Start server
node .output/server/index.mjs

# Or with environment variables
PORT=3000 HOST=0.0.0.0 node .output/server/index.mjs
```

### PM2 Process Manager

```javascript
// ecosystem.config.cjs
module.exports = {
  apps: [
    {
      name: 'nuxt-app',
      script: '.output/server/index.mjs',
      instances: 'max',
      exec_mode: 'cluster',
      env: {
        PORT: 3000,
        NODE_ENV: 'production',
      },
      env_production: {
        PORT: 3000,
        NODE_ENV: 'production',
      },
      // Logging
      log_file: './logs/combined.log',
      error_file: './logs/error.log',
      out_file: './logs/out.log',
      time: true,
      // Process management
      max_memory_restart: '500M',
      min_uptime: 5000,
      max_restarts: 10,
    },
  ],
}
```

```bash
# Install PM2 globally
npm install -g pm2

# Start application
pm2 start ecosystem.config.cjs --env production

# Save process list
pm2 save

# Setup startup script
pm2 startup
```

### Docker Deployment

```dockerfile
# Dockerfile
FROM node:20-alpine AS base
RUN npm install -g pnpm

# Dependencies
FROM base AS deps
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

# Builder
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN pnpm build

# Production
FROM base AS production
WORKDIR /app

ENV NODE_ENV=production
ENV HOST=0.0.0.0
ENV PORT=3000

# Create non-root user
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nuxtjs

COPY --from=builder --chown=nuxtjs:nodejs /app/.output ./.output

USER nuxtjs

EXPOSE 3000

CMD ["node", ".output/server/index.mjs"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - API_SECRET=${API_SECRET}
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

```bash
# Build and run
docker-compose up -d --build

# View logs
docker-compose logs -f app

# Scale horizontally
docker-compose up -d --scale app=3
```

### Nginx Reverse Proxy

```nginx
# /etc/nginx/sites-available/nuxt-app
upstream nuxt_app {
    server 127.0.0.1:3000;
    keepalive 64;
}

server {
    listen 80;
    server_name example.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;
    
    # SSL configuration
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
    
    # Static assets
    location /_nuxt/ {
        alias /var/www/nuxt-app/.output/public/_nuxt/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Public assets
    location /public/ {
        alias /var/www/nuxt-app/.output/public/;
        expires 30d;
        add_header Cache-Control "public";
    }
    
    # Proxy to Nuxt
    location / {
        proxy_pass http://nuxt_app;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
    
    # Health check endpoint
    location /health {
        access_log off;
        proxy_pass http://nuxt_app;
    }
}
```

---

## Edge Deployment

### Vercel (SSR/Edge)

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'vercel', // or 'vercel-edge' for edge functions
  },
})
```

```json
// vercel.json
{
  "buildCommand": "pnpm build",
  "outputDirectory": ".vercel/output",
  "framework": "nuxt"
}
```

### Netlify (SSR/Edge)

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'netlify', // or 'netlify-edge'
  },
})
```

```toml
# netlify.toml
[build]
  command = "pnpm build"
  publish = ".output/public"
  
[functions]
  directory = ".netlify/functions-internal"
```

### Cloudflare Workers

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  nitro: {
    preset: 'cloudflare-pages',
    // or 'cloudflare-module' for Workers
  },
})
```

```toml
# wrangler.toml
name = "my-nuxt-app"
compatibility_date = "2024-01-01"
pages_build_output_dir = ".output/public"

[vars]
MY_ENV_VAR = "value"
```

---

## Hybrid Deployment

### Route-Specific Rendering

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    // Static pages - prerendered at build time
    '/': { prerender: true },
    '/about': { prerender: true },
    '/blog/**': { prerender: true },
    
    // SSR pages - rendered on each request
    '/dashboard/**': { ssr: true },
    '/account/**': { ssr: true },
    
    // SPA pages - client-side only
    '/admin/**': { ssr: false },
    
    // ISR - regenerate after expiry
    '/products/**': {
      isr: 3600, // Regenerate every hour
    },
    
    // SWR - stale while revalidate
    '/api/posts/**': {
      swr: true,
      cache: {
        maxAge: 60,
        staleMaxAge: 600,
      },
    },
    
    // CDN caching
    '/static/**': {
      headers: {
        'Cache-Control': 'public, max-age=31536000, immutable',
      },
    },
  },
})
```

---

## Environment Configuration

### Environment Variables

```bash
# .env.production
NUXT_PUBLIC_API_URL=https://api.production.com
NUXT_PUBLIC_APP_NAME=MyApp
NUXT_PRIVATE_API_SECRET=secret-key
DATABASE_URL=postgresql://user:pass@host:5432/db
```

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    // Private (server-only)
    apiSecret: '',
    databaseUrl: '',
    
    // Public (client + server)
    public: {
      apiUrl: '',
      appName: '',
    },
  },
})
```

### Usage in Application

```typescript
// Access in composables/components
const config = useRuntimeConfig()

// Public config (available on client and server)
console.log(config.public.apiUrl)

// Private config (server-only)
// Only available in server routes and middleware
console.log(config.apiSecret)
```

### Build-Time vs Runtime

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Build-time (baked into bundle)
  appConfig: {
    version: process.env.APP_VERSION || '1.0.0',
  },
  
  // Runtime (read at startup)
  runtimeConfig: {
    apiSecret: process.env.API_SECRET,
  },
})
```

---

## CI/CD Pipeline

### GitHub Actions (Complete)

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'
  PNPM_VERSION: '8'

jobs:
  # Quality checks
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
          
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: pnpm
          
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
        
      - name: Lint
        run: pnpm lint
        
      - name: Type check
        run: pnpm typecheck
        
      - name: Test
        run: pnpm test:run

  # Build
  build:
    needs: quality
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
          
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: pnpm
          
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
        
      - name: Build
        run: pnpm build
        env:
          NUXT_PUBLIC_API_URL: ${{ vars.API_URL }}
          
      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: .output
          retention-days: 1

  # Deploy to staging
  deploy-staging:
    if: github.event_name == 'pull_request'
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: ${{ steps.deploy.outputs.url }}
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build
          path: .output
          
      - name: Deploy to staging
        id: deploy
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}

  # Deploy to production
  deploy-production:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build
          path: .output
          
      - name: Deploy to production
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

### Docker CI/CD

```yaml
# .github/workflows/docker.yml
name: Docker Build and Push

on:
  push:
    branches: [main]
    tags: ['v*']

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        
      - name: Log in to registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
          
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha
            
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

## Health Checks and Monitoring

### Health Check Endpoint

```typescript
// server/api/health.get.ts
export default defineEventHandler(async (event) => {
  const checks = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    version: useRuntimeConfig().public.version,
    checks: {} as Record<string, boolean>,
  }
  
  // Database check
  try {
    await checkDatabase()
    checks.checks.database = true
  } catch {
    checks.checks.database = false
    checks.status = 'degraded'
  }
  
  // External API check
  try {
    await checkExternalApi()
    checks.checks.externalApi = true
  } catch {
    checks.checks.externalApi = false
    checks.status = 'degraded'
  }
  
  // Set status code based on health
  if (checks.status !== 'healthy') {
    setResponseStatus(event, 503)
  }
  
  return checks
})
```

### Readiness Probe

```typescript
// server/api/ready.get.ts
export default defineEventHandler(async (event) => {
  // Check if app is ready to receive traffic
  const isReady = await checkAppReadiness()
  
  if (!isReady) {
    setResponseStatus(event, 503)
    return { ready: false }
  }
  
  return { ready: true }
})
```

### Liveness Probe

```typescript
// server/api/live.get.ts
export default defineEventHandler(() => {
  // Simple check - if this endpoint responds, the app is alive
  return { alive: true }
})
```

---

## Deployment Checklist

### Pre-deployment

- [ ] All tests passing
- [ ] Linting clean
- [ ] Type checking passes
- [ ] Environment variables configured
- [ ] Build successful locally
- [ ] Database migrations ready
- [ ] API endpoints documented

### SPA Deployment

- [ ] Configure base URL (if subdirectory)
- [ ] Setup 404 redirect for SPA routing
- [ ] Enable CDN caching
- [ ] Configure custom domain
- [ ] Enable HTTPS

### SSR Deployment

- [ ] Configure reverse proxy (Nginx)
- [ ] Setup process manager (PM2)
- [ ] Configure health checks
- [ ] Setup log aggregation
- [ ] Configure auto-scaling
- [ ] Enable SSL/TLS

### Post-deployment

- [ ] Verify all routes work
- [ ] Check error tracking
- [ ] Monitor performance metrics
- [ ] Verify SEO (SSR/SSG)
- [ ] Test authentication flow
- [ ] Verify API endpoints

---

## Troubleshooting

### Common Issues

**1. 404 on page refresh (SPA)**
```toml
# Add spa fallback
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

**2. Environment variables not loading**
```bash
# Ensure variables are prefixed correctly
NUXT_PUBLIC_* # For public config
NUXT_* # For private config
```

**3. Build fails with memory error**
```bash
# Increase Node.js memory
export NODE_OPTIONS="--max-old-space-size=4096"
pnpm build
```

**4. SSR hydration mismatch**
```vue
<!-- Wrap client-only code -->
<ClientOnly>
  <BrowserOnlyComponent />
</ClientOnly>
```

**5. API routes not working on Vercel/Netlify**
```typescript
// Ensure correct preset in nuxt.config.ts
nitro: {
  preset: 'vercel', // or 'netlify'
}
```

---

## Summary

This deployment guide covers the essential configurations and strategies for deploying Nuxt applications across different platforms and rendering modes. Choose the deployment strategy that best fits your application's requirements:

- **SPA**: Fast, simple static hosting, limited SEO
- **SSR**: Full SEO, dynamic content, requires server
- **SSG**: Best of both worlds for content sites
- **Hybrid**: Maximum flexibility, route-specific optimization

For production applications, always ensure proper monitoring, health checks, and CI/CD pipelines are in place.