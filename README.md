# ITL.ArgoCD

ArgoCD configuration chart for ITlusions infrastructure with GitHub OAuth integration and local user management.

## Features

- **GitHub OAuth Authentication**: Integrates with GitHub for user authentication via Dex
- **Local User Management**: Support for local ArgoCD users with auto-generated passwords
- **Role-Based Access Control (RBAC)**: Fine-grained permissions based on GitHub teams and local roles
- **API Token Support**: Generate tokens for automation and CI/CD pipelines
- **Secure Configuration**: Proper OAuth app configuration with encrypted secrets

## Current Configuration

Your ArgoCD is currently configured with:
- **GitHub OAuth App**: `Ov23lig***********`
- **Organization**: `ITlusions`
- **Callback URL**: `https://argocd.dev.itlusions.nl/api/dex/callback`
- **Local Users**: Configurable automation and service accounts

## User Types

### GitHub OAuth Users
- **ITlusions team members** authenticate via GitHub
- **Role mapping** based on GitHub team membership
- **SSO login** through web UI and CLI

### Local ArgoCD Users
- **API automation accounts** for CI/CD pipelines
- **Service accounts** with specific role assignments
- **Auto-generated passwords** for security

## Quick Start

1. **Configure Users** in `values.yaml`:
   ```yaml
   users:
     - name: "automation"
       capabilities: "apiKey"
       role: "admin"
       # Password auto-generated if not specified
   ```

2. **Deploy**:
   ```bash
   helm upgrade --install argocd-config ./Chart -n argocd
   ```

3. **Get Auto-Generated Passwords**:
   ```bash
   kubectl get secret argocd-secret -n argocd -o jsonpath='{.data.automation\.password}' | base64 -d
   ```

4. **Restart ArgoCD** (if needed):
   ```bash
   kubectl rollout restart deployment/argocd-server -n argocd
   kubectl rollout restart deployment/argocd-dex-server -n argocd
   ```

## Authentication Methods

### GitHub OAuth Flow
1. User visits ArgoCD UI
2. Clicks "LOG IN VIA GITHUB"
3. Redirected to GitHub for OAuth authentication
4. After successful login, user is redirected back to ArgoCD via Dex
5. ArgoCD validates the OAuth token and maps user teams to roles
6. User gains access based on their team membership in the ITlusions organization

### Local User Login
1. Login directly with username/password
2. Access level determined by configured role
3. Ideal for automation and service accounts

## Access Control

### GitHub Team to Role Mapping

| GitHub Team | ArgoCD Role | Description |
|-------------|-------------|-------------|
| `ITlusions:Admins` | `admin` | Full administrative access |
| `ITlusions:Devops` | `admin` | Full administrative access |
| `ITlusions:Developers` | `developer` | Application deployment and sync |
| `ITlusions:Readonly` | `readonly` | Read-only access |

### Local User Configuration

Local users are defined in `values.yaml` with flexible role assignment:

```yaml
users:
  - name: "automation"
    capabilities: "apiKey"      # API-only access
    role: "admin"              # Full permissions
  - name: "ci-system"
    capabilities: "apiKey,login" # API + login access  
    role: "developer"           # Limited permissions
```

## Documentation

- **[CLI_LOGIN.md](./CLI_LOGIN.md)** - CLI authentication methods
- **[TOKENS.md](./TOKENS.md)** - API token generation and usage
- **[GITHUB_SETUP.md](./GITHUB_SETUP.md)** - GitHub OAuth configuration

## Files Structure

- `Chart/values.yaml` - Configuration values
- `Chart/templates/argocd-server-config-cm.yaml` - Main ArgoCD configuration (argocd-cm)
- `Chart/templates/argocd-rbac-cm.yaml` - RBAC policies
- `Chart/templates/argocd-secret.yaml` - Secret management (optional)
- `Chart/templates/argocd-cmd-params-cm.yaml` - Server parameters
- `GITHUB_SETUP.md` - Detailed GitHub OAuth configuration guide

## Security Notes

- GitHub OAuth client secret is currently stored in the ConfigMap
- Consider moving sensitive data to Kubernetes Secrets for better security
- Ensure proper team membership management in GitHub organization
