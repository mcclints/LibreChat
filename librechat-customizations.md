# LibreChat Logo and Style Customization Guide

This guide provides step-by-step instructions for customizing the LibreChat logo and styling, ensuring changes persist through Docker container rebuilds.

## Prerequisites

- LibreChat fork or clone on your local machine
- Docker and Docker Compose installed
- Git configured with appropriate credentials
- Basic understanding of Docker and command line operations

---

## Overview

This customization approach:
- Builds LibreChat from source using a custom Dockerfile
- Embeds your custom logo before the frontend build
- Compiles style changes (like Tailwind classes) into the JavaScript bundles
- Ensures all customizations persist through Docker rebuilds

---

## Step 1: Prepare Your Custom Logo

1. Create a `custom-assets` folder in your LibreChat root directory:
   ```powershell
   mkdir custom-assets
   ```

2. Place your custom logo file in this folder:
   ```
   custom-assets/logo.svg
   ```
   
   **Note**: The logo should be named `logo.svg` and be in SVG format for best results.

---

## Step 2: Modify the Authentication Layout Styles

1. Open the AuthLayout component:
   ```
   client/src/components/Auth/AuthLayout.tsx
   ```

2. Locate the logo container div (around line 63):
   ```tsx
   <div className="mt-6 h-10 w-full bg-cover">
   ```

3. Modify the Tailwind classes as desired. For example, to make the logo larger:
   ```tsx
   <div className="mt-6 h-40 w-full bg-cover">
   ```
   
   **Common height classes**:
   - `h-10` = 40px (default)
   - `h-16` = 64px
   - `h-24` = 96px
   - `h-32` = 128px
   - `h-40` = 160px

---

## Step 3: Create Custom Dockerfile

1. Create a new file named `Dockerfile.logo` in your LibreChat root directory

2. Copy the following content into `Dockerfile.logo`:

```dockerfile
# v0.8.2 - Custom build with logo and styling modifications

# Base node image
FROM node:20-alpine AS node

# Install jemalloc
RUN apk add --no-cache jemalloc
RUN apk add --no-cache python3 py3-pip uv

# Set environment variable to use jemalloc
ENV LD_PRELOAD=/usr/lib/libjemalloc.so.2

# Add `uv` for extended MCP support
COPY --from=ghcr.io/astral-sh/uv:0.9.5-python3.12-alpine /usr/local/bin/uv /usr/local/bin/uvx /bin/
RUN uv --version

# Set configurable max-old-space-size with default
ARG NODE_MAX_OLD_SPACE_SIZE=6144

RUN mkdir -p /app && chown node:node /app
WORKDIR /app

USER node

COPY --chown=node:node package.json package-lock.json ./
COPY --chown=node:node api/package.json ./api/package.json
COPY --chown=node:node client/package.json ./client/package.json
COPY --chown=node:node packages/data-provider/package.json ./packages/data-provider/package.json
COPY --chown=node:node packages/data-schemas/package.json ./packages/data-schemas/package.json
COPY --chown=node:node packages/api/package.json ./packages/api/package.json

RUN \
    # Allow mounting of these files, which have no default
    touch .env ; \
    # Create directories for the volumes to inherit the correct permissions
    mkdir -p /app/client/public/images /app/logs /app/uploads ; \
    npm config set fetch-retry-maxtimeout 600000 ; \
    npm config set fetch-retries 5 ; \
    npm config set fetch-retry-mintimeout 15000 ; \
    npm ci --no-audit

COPY --chown=node:node . .

# Copy custom logo BEFORE building frontend (this replaces the default logo)
COPY --chown=node:node custom-assets/logo.svg ./client/public/assets/logo.svg

RUN \
    # React client build with configurable memory (includes AuthLayout.tsx h-40 customization)
    NODE_OPTIONS="--max-old-space-size=${NODE_MAX_OLD_SPACE_SIZE}" npm run frontend; \
    npm prune --production; \
    npm cache clean --force

# Node API setup
EXPOSE 3080
ENV HOST=0.0.0.0
CMD ["npm", "run", "backend"]
```

**Key section**: The line `COPY --chown=node:node custom-assets/logo.svg ./client/public/assets/logo.svg` copies your custom logo before the frontend build, ensuring it gets compiled into the final assets.

---

## Step 4: Update docker-compose.yml

1. Open `docker-compose.yml` in your LibreChat root directory

2. Locate the `api` service section (around lines 5-15)

3. Ensure it has the `build` section pointing to your custom Dockerfile:
   ```yaml
   api:
     build:
       context: .
       dockerfile: Dockerfile.logo
     container_name: LibreChat
     ports:
       - "${PORT}:${PORT}"
     depends_on:
       - mongodb
       - rag_api
     restart: always
   ```

4. **CRITICAL**: Remove any `image:` directive if present (e.g., `image: ghcr.io/danny-avila/librechat-dev:latest`)
   
   **Why?** When both `build` and `image` are specified, Docker Compose prioritizes the `image` directive and ignores your custom Dockerfile, causing your changes to be lost.

---

## Step 5: Build and Test Locally

1. Stop any running containers:
   ```powershell
   docker compose down
   ```

2. Build the custom image (use `--no-cache` to ensure a fresh build):
   ```powershell
   docker compose build --no-cache
   ```

3. Start the containers:
   ```powershell
   docker compose up -d
   ```

4. Monitor the logs to verify successful startup:
   ```powershell
   docker compose logs -f api
   ```

5. Test in your browser:
   - Navigate to `http://localhost:3080`
   - Verify your custom logo appears on the login page
   - Verify the logo size matches your Tailwind class changes
   - Use browser DevTools (F12) to inspect the element and confirm the classes

---

## Step 6: Commit Changes to Git

1. Stage your customization files:
   ```powershell
   git add docker-compose.yml Dockerfile.logo client/src/components/Auth/AuthLayout.tsx custom-assets/logo.svg
   ```

2. Commit with a descriptive message:
   ```powershell
   git commit -m "Add custom logo and styling modifications"
   ```

3. Push to your repository:
   ```powershell
   git push origin main
   ```

**Note**: `docker-compose.override.yml` is intentionally gitignored and should not be committed.

---

## Step 7: Deploy to Production/Testing Server

1. SSH into your server or access your deployment environment

2. Pull the latest changes:
   ```bash
   git pull origin main
   ```

3. Stop running containers:
   ```bash
   docker compose down
   ```

4. Build with your customizations:
   ```bash
   docker compose build --no-cache
   ```

5. Start the containers:
   ```bash
   docker compose up -d
   ```

6. Verify the deployment:
   ```bash
   docker compose logs -f api
   ```

---

## Troubleshooting

### Changes Don't Appear After Rebuild

**Cause**: Conflicting `image:` directive in `docker-compose.yml`

**Solution**: Remove the `image:` line from the `api` service in `docker-compose.yml`. Only the `build:` section should be present.

---

### Logo Not Displaying

**Cause**: Logo file path or name mismatch

**Solution**: 
- Verify `custom-assets/logo.svg` exists
- Check the COPY command in `Dockerfile.logo` matches your file structure
- Ensure the logo is named exactly `logo.svg`

---

### Tailwind Classes Not Applied

**Cause**: Style changes not compiled into bundles

**Solution**: 
- Verify you modified `client/src/components/Auth/AuthLayout.tsx` before building
- Use `docker compose build --no-cache` to force a fresh build
- Check that the Dockerfile copies all source files before running `npm run frontend`

---

### MIME Type Errors (text/html instead of application/javascript)

**Cause**: Using volume mount approach instead of building assets into image

**Solution**: Don't use volume mounts for `client/dist/assets`. Let the Dockerfile build process handle asset compilation.

---

## Making Additional Customizations

To add more customizations in the future:

1. **Edit source files**: Modify any `.tsx`, `.css`, or other source files as needed

2. **Update assets**: Add/replace files in `custom-assets/` if needed

3. **Rebuild**: Run the build commands:
   ```powershell
   docker compose down
   docker compose build --no-cache
   docker compose up -d
   ```

4. **Commit and deploy**: Follow Steps 6-7 to push changes to production

---

## Key Concepts

### Why This Approach Works

1. **Builds from source**: Uses your local codebase instead of prebuilt images
2. **Compiles customizations**: Style changes in `.tsx` files get compiled into JavaScript bundles
3. **Embeds assets**: Custom logo is copied before the build process
4. **Persists through rebuilds**: Everything is baked into the Docker image
5. **Git-friendly**: All customizations are in source control for easy deployment

### What NOT to Do

❌ Don't use `docker-compose.override.yml` volume mounts for `client/dist/assets`  
❌ Don't try to build from prebuilt images (`FROM ghcr.io/danny-avila/librechat:latest`)  
❌ Don't have both `build:` and `image:` in the same service  
❌ Don't copy assets after the frontend build completes  

---

## File Structure Summary

```
LibreChat/
├── custom-assets/
│   └── logo.svg                    # Your custom logo
├── client/
│   └── src/
│       └── components/
│           └── Auth/
│               └── AuthLayout.tsx  # Modified with h-40 class
├── docker-compose.yml              # Updated to use Dockerfile.logo
├── Dockerfile.logo                 # Custom build with logo injection
└── .env                           # Your environment configuration
```

---

## Additional Resources

- [LibreChat Documentation](https://www.librechat.ai/docs)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)

---

**Last Updated**: February 1, 2026  
**LibreChat Version**: v0.8.2
