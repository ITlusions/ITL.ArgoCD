# ArgoCD User Management

This guide explains how to manage local ArgoCD users in addition to GitHub OAuth users.

## Overview

The ArgoCD configuration supports two types of users:
1. **GitHub OAuth Users** - ITlusions organization members
2. **Local ArgoCD Users** - Service accounts and automation users

## Local User Configuration

### Adding Users in values.yaml

```yaml
users:
  - name: "automation"
    capabilities: "apiKey"
    role: "admin"
    # password: "custom-password"  # Optional - auto-generated if omitted
    
  - name: "ci-system" 
    capabilities: "apiKey,login"
    password: "MySecurePassword123!"
    role: "developer"
    
  - name: "readonly-service"
    capabilities: "login"
    role: "readonly"
```

### User Properties

#### name
- **Required**: Unique username for the ArgoCD user
- **Format**: String, alphanumeric with hyphens/underscores
- **Example**: `"automation"`, `"ci-system"`

#### capabilities
- **Required**: Defines what the user can do
- **Options**:
  - `"apiKey"` - Can generate API tokens (no UI login)
  - `"login"` - Can login to web UI and CLI
  - `"apiKey,login"` - Both capabilities
- **Use Cases**:
  - API-only: Perfect for automation scripts
  - Login-only: Human users who don't need API access
  - Both: Flexible access for power users

#### role
- **Required**: ArgoCD role defining permissions
- **Options**: `admin`, `developer`, `readonly`
- **Permissions**:
  - `admin`: Full access to all ArgoCD functions
  - `developer`: Application management, limited cluster access
  - `readonly`: View-only access

#### password
- **Optional**: Custom password for the user
- **Auto-generation**: If omitted, a secure 64-character password is generated
- **Recommendation**: Leave empty for auto-generation unless you need a specific password

## Password Management

### Auto-Generated Passwords

When you don't specify a password, the system generates a secure 64-character random password:

```yaml
users:
  - name: "automation"
    capabilities: "apiKey" 
    role: "admin"
    # Password will be auto-generated
```

### Retrieving Auto-Generated Passwords

```bash
# Get password for specific user
kubectl get secret argocd-secret -n argocd -o jsonpath='{.data.automation\.password}' | base64 -d

# PowerShell version
kubectl get secret argocd-secret -n argocd -o jsonpath='{.data.automation\.password}' | %{[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_))}

# List all user passwords
kubectl get secret argocd-secret -n argocd -o json | jq -r '.data | to_entries[] | select(.key | endswith(".password")) | "\(.key): \(.value | @base64d)"'
```

### Custom Passwords

You can specify custom passwords in `values.yaml`:

```yaml
users:
  - name: "special-user"
    capabilities: "login"
    password: "MyComplexPassword123!"
    role: "developer"
```

**Security Notes:**
- Use strong passwords (minimum 12 characters)
- Include uppercase, lowercase, numbers, and symbols
- Consider password rotation policies
- Avoid storing passwords in version control

## Deployment Process

### 1. Update Configuration
Edit `values.yaml` to add/modify users:

```yaml
users:
  - name: "new-automation"
    capabilities: "apiKey"
    role: "admin"
```

### 2. Deploy Changes
```bash
# Navigate to chart directory
cd d:\repos\ITL.ArgoCD\Chart

# Deploy the updated configuration
helm upgrade --install argocd-config . -n argocd
```

### 3. Restart ArgoCD (if needed)
```bash
# Restart server to pick up new user accounts
kubectl rollout restart deployment/argocd-server -n argocd

# Wait for restart to complete
kubectl rollout status deployment/argocd-server -n argocd
```

### 4. Verify User Creation
```bash
# Check if user accounts are created
kubectl get cm argocd-cm -n argocd -o yaml | grep -A 5 accounts

# Verify user can login (for login-capable users)
argocd login argocd.dev.itlusions.nl --grpc-web --username new-automation --password <password>
```

## Usage Examples

### Automation Account
Perfect for CI/CD pipelines:

```yaml
users:
  - name: "github-actions"
    capabilities: "apiKey"
    role: "developer"
```

**Usage:**
```bash
# Get password
PASSWORD=$(kubectl get secret argocd-secret -n argocd -o jsonpath='{.data.github-actions\.password}' | base64 -d)

# Login and generate token
argocd login argocd.dev.itlusions.nl --grpc-web --username github-actions --password "$PASSWORD"
TOKEN=$(argocd account generate-token --id ci-token)

# Use in CI/CD
export ARGOCD_AUTH_TOKEN="$TOKEN"
argocd app sync my-application --server argocd.dev.itlusions.nl --grpc-web
```

### Service Account
For monitoring or read-only access:

```yaml
users:
  - name: "monitoring"
    capabilities: "apiKey"
    role: "readonly"
```

### Development Account
For developers who need UI access:

```yaml
users:
  - name: "dev-user"
    capabilities: "apiKey,login"
    password: "DevPassword123!"
    role: "developer"
```

## Security Best Practices

### Password Security
- ✅ Use auto-generated passwords when possible
- ✅ Rotate passwords regularly
- ✅ Use strong custom passwords (16+ characters)
- ❌ Don't commit passwords to version control
- ❌ Don't share passwords between users

### Access Control
- ✅ Use least privilege principle (assign minimum required role)
- ✅ Use API-only accounts for automation
- ✅ Regular access reviews
- ❌ Don't give admin access unless necessary
- ❌ Don't use shared accounts

### Token Management
- ✅ Generate tokens with specific expiration dates
- ✅ Use descriptive token IDs
- ✅ Rotate tokens regularly
- ❌ Don't store tokens in plain text
- ❌ Don't share tokens between systems

## Troubleshooting

### User Can't Login
1. Check if user exists in ConfigMap:
   ```bash
   kubectl get cm argocd-cm -n argocd -o yaml | grep "accounts\."
   ```

2. Verify password in secret:
   ```bash
   kubectl get secret argocd-secret -n argocd -o yaml | grep "<username>\.password"
   ```

3. Check RBAC configuration:
   ```bash
   kubectl get cm argocd-rbac-cm -n argocd -o yaml
   ```

### Auto-Generated Password Not Working
1. Ensure the secret was created:
   ```bash
   kubectl describe secret argocd-secret -n argocd
   ```

2. Check password generation in template:
   ```bash
   helm template . | grep -A 10 "Additional user passwords"
   ```

### Role Permissions Not Working
1. Verify RBAC configuration includes user mapping:
   ```bash
   kubectl get cm argocd-rbac-cm -n argocd -o yaml | grep "g, <username>"
   ```

2. Check if ArgoCD server picked up new configuration:
   ```bash
   kubectl logs deployment/argocd-server -n argocd | grep -i rbac
   ```

## Migration from Admin Account

If you're currently using the admin account for automation, migrate to dedicated users:

### 1. Create Automation User
```yaml
users:
  - name: "automation"
    capabilities: "apiKey"
    role: "admin"
```

### 2. Deploy and Get Password
```bash
helm upgrade --install argocd-config . -n argocd
PASSWORD=$(kubectl get secret argocd-secret -n argocd -o jsonpath='{.data.automation\.password}' | base64 -d)
```

### 3. Generate New Token
```bash
argocd login argocd.dev.itlusions.nl --grpc-web --username automation --password "$PASSWORD"
NEW_TOKEN=$(argocd account generate-token --id migration-token)
```

### 4. Update Automation Scripts
Replace the old admin token with the new automation token in your CI/CD systems.

### 5. Disable Admin Account (Optional)
```yaml
admin:
  password: ""  # Disables local admin account
```

This approach provides better security and audit trails for your automation processes.