# ArgoCD Authentication Tokens

This guide explains how to generate and use authentication tokens for ArgoCD CLI, especially useful for automation, CI/CD pipelines, and programmatic access.

## Token Types Available

### 1. Admin Account Token
- **Use case**: Full access for automation scripts
- **Permissions**: Complete administrative access
- **Expiration**: Configurable (default: no expiration)

### 2. Local User Tokens
- **Use case**: Automation with specific role permissions
- **Permissions**: Based on user's assigned role
- **Expiration**: Configurable

### 3. GitHub SSO Session Token  
- **Use case**: Personal use while logged in via browser
- **Permissions**: Based on your GitHub team membership
- **Expiration**: Session-based

## Method 1: Generate Admin API Token (Legacy)

### Step 1: Login as Admin
```bash
argocd login argocd.dev.itlusions.nl --grpc-web --username admin --password <admin-password>
```

### Step 2: Generate Token
```bash
argocd account generate-token --account admin --id <token-name>
```

## Method 2: Generate Local User Tokens (Recommended)

### Step 1: Get User Password
For users with auto-generated passwords:
```bash
kubectl get secret argocd-secret -n argocd -o jsonpath='{.data.automation\.password}' | base64 -d
```

### Step 2: Login as Local User
```bash
argocd login argocd.dev.itlusions.nl --grpc-web --username automation --password <generated-password>
```

### Step 3: Generate Token
```bash
# Generate token for current user
argocd account generate-token --id automation-token

# Or specify a different account (if you have permissions)
argocd account generate-token --account automation --id ci-cd-token
```

**Example:**
```bash
argocd account generate-token --id automation-token
```

**Output:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.[PAYLOAD_REDACTED].[SIGNATURE_REDACTED]
```

### Step 4: Use the Token

**Via Environment Variable (Recommended):**
```powershell
# PowerShell
$env:ARGOCD_AUTH_TOKEN = "your-token-here"
argocd app list --server argocd.dev.itlusions.nl --grpc-web

# Bash
export ARGOCD_AUTH_TOKEN="your-token-here"
argocd app list --server argocd.dev.itlusions.nl --grpc-web
```

**Via Command Line Flag:**
```bash
argocd app list --server argocd.dev.itlusions.nl --grpc-web --auth-token "your-token-here"
```

## Method 2: List and Manage Tokens

### List Existing Tokens
```bash
argocd account get --account admin
```

### Delete a Token
```bash
argocd account delete-token --account admin --id <token-id>
```

## Method 3: Extract Current Session Token (Advanced)

If you're logged in via SSO, you can extract the session token for automation:

```bash
# This requires manual extraction from browser developer tools
# or ArgoCD configuration files
```

## Generated Token Example

Here's an example of what a generated admin token looks like:

```
TOKEN: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.[PAYLOAD_REDACTED].[SIGNATURE_REDACTED]
```

**To use a token:**
```powershell
# Set environment variable
$env:ARGOCD_AUTH_TOKEN = "your-actual-token-here"

# Test the token
argocd app list --server argocd.dev.itlusions.nl --grpc-web
```

## Automation Examples

### PowerShell Script
```powershell
# Set token
$env:ARGOCD_AUTH_TOKEN = "your-token-here"

# Define server
$ARGOCD_SERVER = "argocd.dev.itlusions.nl"

# List applications
argocd app list --server $ARGOCD_SERVER --grpc-web

# Sync specific application
argocd app sync ilt-istio --server $ARGOCD_SERVER --grpc-web

# Get application status
argocd app get ilt-istio --server $ARGOCD_SERVER --grpc-web
```

### Bash Script
```bash
#!/bin/bash
export ARGOCD_AUTH_TOKEN="your-token-here"
export ARGOCD_SERVER="argocd.dev.itlusions.nl"

# Sync application
argocd app sync ilt-istio --server $ARGOCD_SERVER --grpc-web

# Wait for sync to complete
argocd app wait ilt-istio --server $ARGOCD_SERVER --grpc-web
```

### GitHub Actions Example
```yaml
name: Deploy to ArgoCD
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Sync ArgoCD Application
        env:
          ARGOCD_AUTH_TOKEN: ${{ secrets.ARGOCD_TOKEN }}
        run: |
          argocd app sync ilt-istio --server argocd.dev.itlusions.nl --grpc-web
```

## Security Best Practices

### Token Storage
- **Never commit tokens to version control**
- Use environment variables or secure secret management
- Rotate tokens regularly
- Use specific token IDs for easy identification

### Token Scope
- Use admin tokens only when necessary
- Consider creating service accounts with limited permissions
- Monitor token usage in ArgoCD logs

### Token Management
```bash
# List all tokens for admin account
argocd account get --account admin

# Generate token with expiration (if supported)
argocd account generate-token --account admin --id temp-token --expires-in 24h

# Delete old tokens
argocd account delete-token --account admin --id old-token
```

## Troubleshooting

### Token Not Working
1. Verify token format (should be a JWT)
2. Check token hasn't expired
3. Ensure proper server URL and flags
4. Verify account permissions

### Common Issues
```bash
# Invalid token error
time="..." level=fatal msg="rpc error: code = Unauthenticated desc = Invalid auth token"

# Solution: Regenerate token or check token value
```

### Debug Mode
```bash
argocd app list --server argocd.dev.itlusions.nl --grpc-web --loglevel debug
```

## Token Information

Your current admin token details:
- **ID**: `cli-token`
- **Account**: `admin`
- **Permissions**: Full administrative access
- **Generated**: September 13, 2025
- **Expiration**: No expiration set

**⚠️ Security Warning**: Store this token securely and never share it publicly!