# LibreChat Logo + Tailwind Customization Summary (Windows 11/Docker Desktop)

## Goal
- Replace default LibreChat login logo with custom SVG
- Resize logo via Tailwind classes in `AuthLayout.tsx` (e.g., `h-10` → `h-16`)
- Build Docker image with changes for GitHub fork/prod deployment

## Core Problem Journey

### Initial Attempts (Failed)
```
1. Edit AuthLayout.tsx → npm run frontend → docker compose up -d
   ❌ Tailwind classes reset (Docker rebuilds official source)

2. docker-compose.override.yml volume mount: ./custom-assets:/app/client/dist/assets
   ❌ MIME errors (text/html) - missing JS/CSS files in custom-assets

3. npm run frontend → copy dist/assets → custom-assets → docker up
   ❌ Cyclic copy errors + incomplete sync → 404s on react-interactions.js
```

### Working Native Test (Step 3 Success)
```
docker compose down
npm ci && npm run frontend && npm run backend
✅ localhost:3080 shows h-16 class but default logo
```

### Docker Build Failures
```
docker compose build → uses official LibreChat Dockerfile
❌ Overwrites local client/dist/assets + AuthLayout.tsx changes
❌ Returns to stock h-10 + default logo.svg
```

## Key Insights
1. **Native mode** (`npm run backend`) works - proves your TSX edit compiles
2. **Docker `build`** ignores local source - runs official `npm run frontend` inside container  
3. **Volume mounts** require **complete** dist/assets copy (200+ files) - fragile
4. **Tailwind purge** may skip unused classes - needs `tailwind.config.js` touch

## Current State
```
✅ AuthLayout.tsx edited (h-16 works native)
✅ custom-assets/logo.svg ready 
❌ Docker build reverts to stock (official Dockerfile problem)
❌ Volume mount causes MIME/404 errors (incomplete assets)
```

## Recommended Path Forward (Windsurf IDE)
**Option 1: Custom Dockerfile (Permanent)**
```dockerfile
# Dockerfile.logo (repo root)
FROM ghcr.io/danny-avila/librechat:latest
COPY client/src/components/Auth/AuthLayout.tsx client/src/components/Auth/AuthLayout.tsx
COPY custom-assets/logo.svg client/public/assets/logo.svg
RUN cd client && npm ci && npm run frontend
```

Update `docker-compose.yml`:
```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile.logo
```

**Option 2: Pre-build Assets (Simpler)**
```
docker compose down
npm ci && npm run frontend
copy custom-assets\logo.svg client\dist\assets\logo.svg
docker compose up -d --build  # Uses your local dist
```

**Option 3: Native Dev → Docker Prod**
```
# Dev: npm run frontend:dev + npm run backend (live reload)
# Prod: Commit TSX + logo.svg → git push → docker compose --build on server
```

## For Windsurf IDE
```
1. Clone your LibreChat fork
2. Edit client/src/components/Auth/AuthLayout.tsx (h-16)
3. Add custom-assets/logo.svg 
4. Create Dockerfile.logo (Option 1 above)
5. docker compose build --no-cache && docker compose up -d
6. Test: F12 Elements shows h-16 + Network tab logo.svg 200OK
7. git commit/push fork
```

**Root cause**: Docker's official build process bypasses your local changes. Custom Dockerfile injects them before rebuild.

**Status**: Native working → Docker integration needed (Dockerfile approach recommended).