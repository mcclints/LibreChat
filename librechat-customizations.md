# LibreChat Logo and Style Customization Guide

This guide provides step-by-step instructions for customizing the LibreChat logo and styling, ensuring changes persist through Docker container rebuilds.

## Prerequisites

- LibreChat fork or clone on your local machine
- **LibreChat Version**: v0.8.2 (these instructions are tested on v0.8.2)
- Docker and Docker Compose installed
- Git configured with appropriate credentials
- Basic understanding of Docker and command line operations

### Version Compatibility

**Tested on**: LibreChat v0.8.2

**For fresh installs from the main LibreChat repository**:
- These instructions work out-of-the-box on v0.8.2
- File paths and component structures match the official repository
- All Tailwind classes and patterns are standard

**For other versions**:
- ⚠️ **Older versions (< v0.8.0)**: Component structure and file paths may differ
- ⚠️ **Newer versions (> v0.8.2)**: Verify file locations and class names before proceeding
- Always check that the authentication component files exist at the documented paths

**To verify your version**:
```bash
# Check package.json version
cat package.json | grep version

# Or check git commit
git log --oneline -1
```

---

## Overview

This customization approach:
- Builds LibreChat from source using a custom Dockerfile
- Embeds your custom logo before the frontend build
- Compiles style changes (like Tailwind classes) into the JavaScript bundles
- Ensures all customizations persist through Docker rebuilds
- Works on fresh clones/forks of the official LibreChat repository

### Important Notes for Fresh Installs

1. **No pre-existing customizations required** - These instructions work on a vanilla LibreChat install
2. **Fork vs Clone** - If deploying to production, fork the repository to your own GitHub account first
3. **Environment files** - You'll need to create `.env` file with your configuration (see LibreChat docs)
4. **First-time setup** - Allow extra time for initial Docker build (~10-15 minutes depending on your system)
5. **Backup recommendation** - If customizing an existing install, commit your current state first

---

## Step 1: Verify File Structure (Fresh Installs)

**For fresh LibreChat installs**, verify that all required files exist before proceeding:

```bash
# Verify authentication component files
ls client/src/components/Auth/LoginForm.tsx
ls client/src/components/Auth/Login.tsx
ls client/src/components/Auth/Registration.tsx
ls client/src/components/Auth/Footer.tsx
ls client/src/components/Auth/RequestPasswordReset.tsx
ls client/src/components/Auth/ResetPassword.tsx
ls client/src/components/Auth/AuthLayout.tsx

# Verify Button component
ls packages/client/src/components/Button.tsx

# Verify Docker files
ls docker-compose.yml
```

If all files exist, you're ready to proceed. If any are missing, verify your LibreChat version.

---

## Step 2: Prepare Your Custom Logo

1. Create a `custom-assets` folder in your LibreChat root directory:
   ```powershell
   mkdir custom-assets
   ```

2. Place your custom logo file in this folder:
   ```
   custom-assets/logo.svg
   ```
   
   **Note**: The logo should be named `logo.svg` and be in SVG format for best results.

3. Place your custom favicon file in the same folder:
   ```
   custom-assets/fiu-favcon-270x270.gif
   ```
   
   **Note**: The favicon can be in GIF, PNG, or ICO format. The filename will be referenced in the Dockerfile.

---

## Step 3: Customize UI Colors and Styles

### 3.1: Modify Logo Size

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

### 3.2: Customize Button Colors

1. Open the Button component:
   ```
   packages/client/src/components/Button.tsx
   ```

2. Locate the `submit` variant in the `buttonVariants` cva (around line 20):
   ```tsx
   submit: 'bg-surface-submit text-white hover:bg-surface-submit-hover',
   ```

3. Replace with custom hex colors:
   ```tsx
   submit: 'bg-[#081E3F] text-white hover:bg-[#0a2449]',
   ```
   
   **Example colors**:
   - `#081E3F` = Dark navy blue
   - `#0a2449` = Slightly lighter navy for hover state

### 3.3: Customize Link Colors

Update link colors in all authentication components to maintain consistent branding.

**Files to modify**:
- `client/src/components/Auth/LoginForm.tsx`
- `client/src/components/Auth/Login.tsx`
- `client/src/components/Auth/Registration.tsx`
- `client/src/components/Auth/Footer.tsx`
- `client/src/components/Auth/RequestPasswordReset.tsx`

**Find and replace** all link color classes:

**Original green classes**:
```tsx
className="... text-green-600 hover:text-green-700 dark:text-green-500 dark:hover:text-green-400 ..."
```

**Replace with custom color** (example: golden/bronze `#B6862C`):
```tsx
className="... text-[#B6862C] hover:text-[#c99635] dark:text-[#B6862C] dark:hover:text-[#c99635] ..."
```

**Example from LoginForm.tsx** (Forgot password link):
```tsx
<a
  href="/forgot-password"
  className="inline-flex p-1 text-sm font-medium text-[#B6862C] underline decoration-transparent transition-all duration-200 hover:text-[#c99635] hover:decoration-[#c99635] focus:text-[#c99635] focus:decoration-[#c99635] dark:text-[#B6862C] dark:hover:text-[#c99635] dark:hover:decoration-[#c99635] dark:focus:text-[#c99635] dark:focus:decoration-[#c99635]"
>
  {localize('com_auth_password_forgot')}
</a>
```

### 3.4: Customize Input Focus States

Update input field focus border and label colors for consistent branding.

**Files to modify**:
- `client/src/components/Auth/LoginForm.tsx`
- `client/src/components/Auth/Registration.tsx`
- `client/src/components/Auth/ResetPassword.tsx`
- `client/src/components/Auth/RequestPasswordReset.tsx`

**For input fields**, find:
```tsx
className="... focus:border-green-500 ..."
```

**Replace with**:
```tsx
className="... focus:border-[#B6862C] ..."
```

**For input labels** (peer-focus), find:
```tsx
className="... peer-focus:text-green-600 dark:peer-focus:text-green-500 ..."
```

**Replace with**:
```tsx
className="... peer-focus:text-[#B6862C] dark:peer-focus:text-[#B6862C] ..."
```

**Example from LoginForm.tsx** (Email input):
```tsx
<input
  type="email"
  id="email"
  className="webkit-dark-styles transition-color peer w-full rounded-2xl border border-border-light bg-surface-primary px-3.5 pb-2.5 pt-3 text-text-primary duration-200 focus:border-[#B6862C] focus:outline-none"
  placeholder=" "
/>
<label
  htmlFor="email"
  className="absolute start-3 top-1.5 z-10 origin-[0] -translate-y-4 scale-75 transform bg-surface-primary px-2 text-sm text-text-secondary-alt duration-200 peer-placeholder-shown:top-1/2 peer-placeholder-shown:-translate-y-1/2 peer-placeholder-shown:scale-100 peer-focus:top-1.5 peer-focus:-translate-y-4 peer-focus:scale-75 peer-focus:px-2 peer-focus:text-[#B6862C] dark:peer-focus:text-[#B6862C] rtl:peer-focus:left-auto rtl:peer-focus:translate-x-1/4"
>
  {localize('com_auth_email_address')}
</label>
```

---

## Step 4: Create Custom Dockerfile

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

# Copy custom favicon (replace LibreChat favicons with FIU favicon)
COPY --chown=node:node custom-assets/fiu-favcon-270x270.gif ./client/public/assets/favicon.gif

# Update browser tab title and favicon references in index.html
RUN sed -i 's/<title>LibreChat<\/title>/<title>PantherAI<\/title>/' ./client/index.html && \
    sed -i 's|<link rel="icon" type="image/png" sizes="32x32" href="assets/favicon-32x32.png" />|<link rel="icon" type="image/gif" href="assets/favicon.gif" />|' ./client/index.html && \
    sed -i 's|<link rel="icon" type="image/png" sizes="16x16" href="assets/favicon-16x16.png" />||' ./client/index.html

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

**Key sections**: 
- `COPY --chown=node:node custom-assets/logo.svg ./client/public/assets/logo.svg` copies your custom logo before the frontend build
- `COPY --chown=node:node custom-assets/fiu-favcon-270x270.gif ./client/public/assets/favicon.gif` copies your custom favicon
- The `sed` commands modify `index.html` to:
  - Change the browser tab title from "LibreChat" to "PantherAI" (initial HTML title)
  - Replace PNG favicon references with your custom GIF favicon
  - These changes happen before the frontend build, ensuring they persist across rebuilds

**Important**: The HTML title is overridden by the `APP_TITLE` environment variable. You must also update your `.env` file:
```bash
APP_TITLE=PantherAI
```
This ensures the title remains "PantherAI" after the React app loads. Without this, the title will revert to "LibreChat" after page load.

---

## Step 5: Create docker-compose.override.yml

**Important**: LibreChat's `docker-compose.yml` file explicitly states "Do not edit this file directly. Use a 'docker-compose.override.yaml' file if you can." This approach ensures your customizations won't be overwritten when pulling updates from the main repository.

1. Create or update `docker-compose.override.yml` in your LibreChat root directory

2. Add the following configuration:
   ```yaml
   # This override file configures LibreChat to:
   # 1. Build from a custom Dockerfile (Dockerfile.logo) that embeds the custom logo
   # 2. Mount the librechat.yaml configuration file

   services:
     api:
       build:
         context: .
         dockerfile: Dockerfile.logo
       volumes:
         - type: bind
           source: ./librechat.yaml
           target: /app/librechat.yaml
   ```

3. **Why use override file?**
   - ✅ Follows LibreChat's official best practices
   - ✅ Your customizations won't be lost when updating LibreChat
   - ✅ Cleaner separation between base config and customizations
   - ✅ The override file is already in `.gitignore` by default

4. **How it works**: Docker Compose automatically merges `docker-compose.yml` and `docker-compose.override.yml`. Your override settings take precedence over the base configuration.

---

## Step 6: Build and Test Locally

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

## Step 7: Commit Changes to Git

1. Stage your customization files:
   ```powershell
   git add docker-compose.override.yml Dockerfile.logo custom-assets/logo.svg packages/client/src/components/Button.tsx client/src/components/Auth/LoginForm.tsx client/src/components/Auth/Login.tsx client/src/components/Auth/Registration.tsx client/src/components/Auth/Footer.tsx client/src/components/Auth/RequestPasswordReset.tsx client/src/components/Auth/ResetPassword.tsx
   ```

2. Commit with a descriptive message:
   ```powershell
   git commit -m "Add custom logo and styling modifications"
   ```

3. Push to your repository:
   ```powershell
   git push origin main
   ```

**Note**: By default, `docker-compose.override.yml` may be gitignored. However, for this customization approach, you should commit it so your custom build configuration is preserved in your repository.

---

## Step 8: Deploy to Production/Testing Server

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

❌ Don't edit `docker-compose.yml` directly - use `docker-compose.override.yml` instead  
❌ Don't copy assets after the frontend build completes  
❌ Don't forget to commit `docker-compose.override.yml` to your repository  

---

## File Structure Summary

```
LibreChat/
├── custom-assets/
│   ├── logo.svg                           # Your custom logo
│   └── fiu-favcon-270x270.gif             # Your custom favicon
├── client/
│   └── src/
│       └── components/
│           └── Auth/
│               ├── LoginForm.tsx          # Updated: input focus & link colors
│               ├── Login.tsx              # Updated: link colors
│               ├── Registration.tsx       # Updated: input focus & link colors
│               ├── Footer.tsx             # Updated: link colors
│               ├── RequestPasswordReset.tsx  # Updated: input focus & link colors
│               └── ResetPassword.tsx      # Updated: input focus colors
├── packages/
│   └── client/
│       └── src/
│           └── components/
│               └── Button.tsx             # Updated: submit button color
├── docker-compose.yml                     # Base configuration (DO NOT EDIT)
├── docker-compose.override.yml            # Custom build configuration
├── Dockerfile.logo                        # Custom build with logo injection
└── .env                                   # Your environment configuration
```

---

## Step 9: Configure SAML Authentication (Optional)

LibreChat supports SAML 2.0 authentication for enterprise single sign-on (SSO) integration.

### Prerequisites

- Access to your organization's SAML Identity Provider (IdP)
- SAML metadata from your IdP (Entry Point, Issuer, Certificate)
- Admin access to configure SAML in your IdP

### SAML Configuration Steps

#### 1. Obtain SAML Metadata from Your Identity Provider

You'll need the following information from your SAML IdP:

- **Entry Point URL**: The SSO URL where LibreChat will redirect users for authentication
- **Issuer**: The unique identifier for your IdP (Entity ID)
- **Certificate**: The X.509 certificate used to verify SAML responses

**Common Identity Providers**:
- **Azure AD**: Azure Portal → Enterprise Applications → Your App → Single sign-on
- **Okta**: Admin Console → Applications → Your App → Sign On → View Setup Instructions
- **Google Workspace**: Admin Console → Apps → SAML apps → Your App

#### 2. Configure LibreChat in Your Identity Provider

In your IdP, create a new SAML application with these settings:

**Service Provider (SP) Configuration**:
- **Entity ID / Audience**: `http://your-domain.com` (or your `DOMAIN_SERVER` value)
- **ACS URL / Callback URL**: `http://your-domain.com/oauth/saml/callback`
- **Name ID Format**: Email Address (recommended)

**Attribute Mappings** (configure these in your IdP):
- `email` → User's email address
- `username` → User's username (optional)
- `givenName` → User's first name (optional)
- `familyName` → User's last name (optional)
- `name` → User's full name (optional)
- `picture` → User's profile picture URL (optional)

#### 3. Update LibreChat Environment Variables

Edit your `.env` file and configure the SAML settings:

```bash
# SAML Authentication
# Note: If OpenID is enabled, SAML authentication will be automatically disabled.
SAML_ENTRY_POINT=https://your-idp.com/saml/sso
SAML_ISSUER=http://your-domain.com
SAML_CERT="-----BEGIN CERTIFICATE-----\nMIIC...your certificate...\n-----END CERTIFICATE-----"
SAML_CALLBACK_URL=/oauth/saml/callback
SAML_SESSION_SECRET=your-random-secret-string-here

# Attribute mappings (optional - adjust based on your IdP's attribute names)
SAML_EMAIL_CLAIM=email
SAML_USERNAME_CLAIM=username
SAML_GIVEN_NAME_CLAIM=givenName
SAML_FAMILY_NAME_CLAIM=familyName
SAML_PICTURE_CLAIM=picture
SAML_NAME_CLAIM=name

# Login button settings (optional)
SAML_BUTTON_LABEL="Sign in with SSO"
SAML_IMAGE_URL=/path/to/your/sso-logo.png

# Whether the SAML Response should be signed (optional)
# - If "true", the entire SAML Response will be signed.
# - If "false" or unset, only the SAML Assertion will be signed (default behavior).
# SAML_USE_AUTHN_RESPONSE_SIGNED=false
```

**Important Notes**:
- `SAML_SESSION_SECRET`: Generate a strong random string (e.g., using `openssl rand -base64 32`)
- `SAML_CERT`: Must include the full certificate with `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----` headers
- Certificate newlines should be escaped as `\n` in the .env file
- `DOMAIN_SERVER` must match your IdP's configured Entity ID

#### 4. Enable SAML in librechat.yaml

Edit your `librechat.yaml` file to enable SAML as a social login option:

```yaml
registration:
  socialLogins: ['saml']
  # Or include with other providers:
  # socialLogins: ['github', 'google', 'saml']
```

#### 5. Restart LibreChat

After configuring SAML, restart your containers:

```bash
docker compose restart api
```

Or for a full restart:

```bash
docker compose down
docker compose up -d
```

#### 6. Test SAML Authentication

1. Navigate to your LibreChat login page
2. You should see a "Sign in with SSO" button (or your custom label)
3. Click the button to be redirected to your IdP
4. Authenticate with your IdP credentials
5. You should be redirected back to LibreChat and logged in

### Troubleshooting SAML

#### "SAML authentication failed" Error

**Possible causes**:
- Certificate mismatch or incorrect format
- Entry Point URL is incorrect
- Callback URL not configured correctly in IdP

**Solutions**:
- Verify certificate includes proper headers and escaped newlines
- Check IdP logs for detailed error messages
- Ensure `DOMAIN_SERVER` matches your IdP's Entity ID configuration

#### Users Can't Log In After SAML Setup

**Possible causes**:
- Attribute mappings don't match IdP's attribute names
- Email claim not being sent by IdP

**Solutions**:
- Check IdP's attribute mapping configuration
- Verify `SAML_EMAIL_CLAIM` matches the attribute name your IdP sends
- Review LibreChat logs: `docker compose logs api | grep -i saml`

#### SAML Button Not Appearing

**Possible causes**:
- SAML not enabled in `librechat.yaml`
- OpenID is enabled (SAML is disabled when OpenID is active)
- Missing required SAML environment variables

**Solutions**:
- Verify `socialLogins` includes `'saml'` in `librechat.yaml`
- Disable OpenID if you want to use SAML
- Check that `SAML_ENTRY_POINT`, `SAML_ISSUER`, and `SAML_CERT` are all set

### Security Best Practices

1. **Always use HTTPS in production** - SAML should never be used over HTTP in production environments
2. **Rotate session secrets regularly** - Update `SAML_SESSION_SECRET` periodically
3. **Verify certificate validity** - Ensure your IdP's certificate hasn't expired
4. **Use signed responses** - Set `SAML_USE_AUTHN_RESPONSE_SIGNED=true` for enhanced security
5. **Restrict access by domain** - Use `allowedDomains` in `librechat.yaml` to limit registration to specific email domains

### Example: Azure AD SAML Configuration

**In Azure AD**:
1. Azure Portal → Enterprise Applications → New Application → Create your own application
2. Select "Integrate any other application you don't find in the gallery (Non-gallery)"
3. Go to Single sign-on → SAML
4. Set:
   - **Identifier (Entity ID)**: `https://your-domain.com`
   - **Reply URL (ACS URL)**: `https://your-domain.com/oauth/saml/callback`
5. Download the Certificate (Base64)
6. Copy the Login URL (this is your `SAML_ENTRY_POINT`)

**In LibreChat .env**:
```bash
SAML_ENTRY_POINT=https://login.microsoftonline.com/your-tenant-id/saml2
SAML_ISSUER=https://your-domain.com
SAML_CERT="-----BEGIN CERTIFICATE-----\nMIIC8DCCAdigAwIBAgIQFE...\n-----END CERTIFICATE-----"
SAML_EMAIL_CLAIM=http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress
SAML_NAME_CLAIM=http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name
```

---

## Additional Resources

- [LibreChat Documentation](https://www.librechat.ai/docs)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)

---

**Last Updated**: February 26, 2026  
**LibreChat Version**: v0.8.2

---

## Changelog

### February 26, 2026
- Added comprehensive SAML authentication configuration section
- Updated browser tab title example to "PantherAI"
- Added APP_TITLE environment variable documentation
- Clarified favicon and title customization requirements

### February 25, 2026
- Added favicon customization instructions
- Added browser tab title customization
- Documented APP_TITLE environment variable requirement

### February 5, 2026
- Initial documentation created
- Logo customization
- UI color customizations (buttons, links, input focus states)
- Docker Compose override approach
