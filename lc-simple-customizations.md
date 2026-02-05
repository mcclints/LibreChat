# LibreChat Customizations - Quick Deployment Guide

This guide is for users who have received pre-customized LibreChat files and need to deploy them to their LibreChat installation.

## Prerequisites

- Existing LibreChat installation (v0.8.2 or compatible)
- Docker and Docker Compose installed
- Access to your LibreChat directory
- Pre-customized files provided to you

---

## What's Customized

These customizations include:
- **Custom logo** with larger display size (160px height)
- **Button colors**: Dark navy (`#081E3F`) for submit buttons
- **Link colors**: Golden/bronze (`#B6862C`) for all authentication page links
- **Input focus states**: Golden/bronze (`#B6862C`) border and label colors

---

## Step 1: Backup Your Current Installation

Before making any changes, create a backup:

```bash
# Navigate to your LibreChat directory
cd /path/to/LibreChat

# Create a backup of current files
mkdir -p backups/$(date +%Y%m%d)
cp -r client/src/components/Auth backups/$(date +%Y%m%d)/
cp packages/client/src/components/Button.tsx backups/$(date +%Y%m%d)/
cp docker-compose.override.yml backups/$(date +%Y%m%d)/ 2>/dev/null || true
cp Dockerfile.logo backups/$(date +%Y%m%d)/ 2>/dev/null || true
```

---

## Step 2: Copy Customized Files

Copy the provided files to their respective locations in your LibreChat directory:

### Authentication Components
```bash
# Copy all authentication component files
cp LoginForm.tsx client/src/components/Auth/
cp Login.tsx client/src/components/Auth/
cp Registration.tsx client/src/components/Auth/
cp Footer.tsx client/src/components/Auth/
cp RequestPasswordReset.tsx client/src/components/Auth/
cp ResetPassword.tsx client/src/components/Auth/
cp AuthLayout.tsx client/src/components/Auth/
```

### Button Component
```bash
# Copy the Button component
cp Button.tsx packages/client/src/components/
```

### Custom Logo
```bash
# Create custom-assets folder if it doesn't exist
mkdir -p custom-assets

# Copy the custom logo
cp logo.svg custom-assets/
```

### Docker Configuration Files
```bash
# Copy Docker configuration files
cp Dockerfile.logo .
cp docker-compose.override.yml .
```

---

## Step 3: Verify File Placement

Ensure all files are in the correct locations:

```bash
# Verify authentication components
ls -l client/src/components/Auth/LoginForm.tsx
ls -l client/src/components/Auth/Login.tsx
ls -l client/src/components/Auth/Registration.tsx
ls -l client/src/components/Auth/Footer.tsx
ls -l client/src/components/Auth/RequestPasswordReset.tsx
ls -l client/src/components/Auth/ResetPassword.tsx
ls -l client/src/components/Auth/AuthLayout.tsx

# Verify Button component
ls -l packages/client/src/components/Button.tsx

# Verify custom assets
ls -l custom-assets/logo.svg

# Verify Docker files
ls -l Dockerfile.logo
ls -l docker-compose.override.yml
```

All files should exist without errors.

---

## Step 4: Rebuild Docker Containers

Now rebuild your LibreChat containers with the customizations:

### Stop Running Containers
```bash
docker compose down
```

### Rebuild with No Cache
This ensures all customizations are compiled into the new build:

```bash
docker compose build --no-cache
```

**Note**: This may take 10-15 minutes depending on your system.

### Start the Containers
```bash
docker compose up -d
```

---

## Step 5: Verify Customizations

1. **Access LibreChat**: Navigate to `http://localhost:3080` (or your configured URL)

2. **Check the logo**:
   - Should appear on the login page
   - Should be larger than default (160px height)

3. **Check button colors**:
   - Login/Sign up buttons should be dark navy (`#081E3F`)
   - Hover state should be slightly lighter

4. **Check link colors**:
   - "Sign up", "Login", "Forgot password" links should be golden/bronze (`#B6862C`)
   - Hover state should be slightly lighter

5. **Check input focus states**:
   - Click into any input field (email, password, etc.)
   - Border should turn golden/bronze (`#B6862C`)
   - Label should turn golden/bronze when focused

---

## Troubleshooting

### Customizations Not Appearing

**Problem**: Changes don't appear after rebuild

**Solutions**:
1. Clear browser cache (Ctrl+Shift+Delete or Cmd+Shift+Delete)
2. Try incognito/private browsing mode
3. Verify files were copied correctly
4. Check Docker build logs for errors:
   ```bash
   docker compose logs api
   ```

### Build Errors

**Problem**: Docker build fails

**Solutions**:
1. Ensure you're using the correct LibreChat version (v0.8.2)
2. Check that all files are in the correct locations
3. Verify `Dockerfile.logo` and `docker-compose.override.yml` are present
4. Review build output for specific error messages

### Logo Not Displaying

**Problem**: Custom logo doesn't appear

**Solutions**:
1. Verify `custom-assets/logo.svg` exists
2. Ensure `Dockerfile.logo` is being used (check `docker-compose.override.yml`)
3. Rebuild with `--no-cache` flag
4. Check that logo file is valid SVG format

### Colors Not Applied

**Problem**: Button/link/input colors haven't changed

**Solutions**:
1. Verify component files were copied to correct locations
2. Clear browser cache completely
3. Check browser DevTools (F12) to inspect element classes
4. Ensure rebuild completed successfully without errors

---

## Rolling Back Changes

If you need to revert to the original LibreChat:

```bash
# Stop containers
docker compose down

# Remove customized files
rm docker-compose.override.yml
rm Dockerfile.logo
rm -rf custom-assets

# Restore from backup
cp -r backups/YYYYMMDD/Auth/* client/src/components/Auth/
cp backups/YYYYMMDD/Button.tsx packages/client/src/components/

# Rebuild with original configuration
docker compose build --no-cache
docker compose up -d
```

---

## File Locations Reference

Quick reference for where each file should be located:

```
LibreChat/
├── custom-assets/
│   └── logo.svg
├── client/
│   └── src/
│       └── components/
│           └── Auth/
│               ├── AuthLayout.tsx
│               ├── LoginForm.tsx
│               ├── Login.tsx
│               ├── Registration.tsx
│               ├── Footer.tsx
│               ├── RequestPasswordReset.tsx
│               └── ResetPassword.tsx
├── packages/
│   └── client/
│       └── src/
│           └── components/
│               └── Button.tsx
├── docker-compose.override.yml
└── Dockerfile.logo
```

---

## Production Deployment

For deploying to a production server:

1. **Transfer files**: Copy all customized files to your server
2. **Follow Steps 2-4** on the server
3. **Configure environment**: Ensure `.env` file has production settings
4. **Use production domain**: Update `DOMAIN_CLIENT` and `DOMAIN_SERVER` in `.env`
5. **Enable HTTPS**: Configure reverse proxy (nginx/Apache) with SSL certificates

---

## Maintenance

### Updating LibreChat

When updating LibreChat to a newer version:

1. **Backup customizations**: Save all modified files
2. **Pull updates**: `git pull` from LibreChat repository
3. **Reapply customizations**: Copy your customized files back
4. **Rebuild**: Run `docker compose build --no-cache`
5. **Test**: Verify all customizations still work

**Note**: Component structure may change in newer versions. You may need to reapply customizations manually if file structures have changed.

---

## Support

If you encounter issues not covered in this guide:

1. Check the main customization guide: `librechat-customizations.md`
2. Review LibreChat documentation: https://www.librechat.ai/docs
3. Check Docker Compose documentation: https://docs.docker.com/compose/

---

**Last Updated**: February 5, 2026  
**Compatible with**: LibreChat v0.8.2
