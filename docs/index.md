# ITL.ArgoCD Documentation

Welcome to the ITlusions ArgoCD configuration documentation. This chart provides a complete ArgoCD setup with GitHub OAuth integration and local user management for automation and CI/CD workflows.

## 🚀 Quick Start

New to ArgoCD or need to get started quickly? Follow these steps:

1. **[Deploy the Chart](#deployment)** - Get ArgoCD running with your configuration
2. **[Login to ArgoCD](CLI_LOGIN.md)** - Access ArgoCD via web UI or CLI
3. **[Generate API Tokens](TOKENS.md)** - Set up automation access
4. **[Manage Users](USER_MANAGEMENT.md)** - Add additional users and service accounts

## 📚 Documentation Index

### Core Guides

| Document | Description | When to Use |
|----------|-------------|-------------|
| **[Complete Authentication Guide](AUTHENTICATION_COMPLETE.md)** | Comprehensive guide covering all authentication providers, setup, and deployment | Primary reference for all authentication-related tasks |
| **[CLI Login Guide](CLI_LOGIN.md)** | Complete CLI authentication methods | When you need to access ArgoCD from command line |
| **[Token Management](TOKENS.md)** | API token generation and usage | For automation, CI/CD pipelines, and API access |
| **[User Management](USER_MANAGEMENT.md)** | Local user configuration and management | When adding service accounts or automation users |

### Reference Guides

| Document | Description | When to Use |
|----------|-------------|-------------|
| **[GitHub Setup](GITHUB_SETUP.md)** | GitHub OAuth configuration details | For understanding or modifying GitHub integration |

### Configuration Reference

| File | Purpose | Location |
|------|---------|----------|
| `values.yaml` | Main configuration values | `Chart/values.yaml` |
| `argocd-server-config-cm.yaml` | ArgoCD server configuration | `Chart/templates/` |
| `argocd-rbac-cm.yaml` | Role-based access control | `Chart/templates/` |
| `argocd-secret.yaml` | User passwords and secrets | `Chart/templates/` |

### Deployment Guides

| Document | Description | When to Use |
|----------|-------------|-------------|
| **[Complete Authentication Guide](AUTHENTICATION_COMPLETE.md)** | Includes comprehensive deployment procedures | Primary deployment reference for authentication setup |

## 🔐 Authentication Overview

### Two Authentication Methods

#### 1. GitHub OAuth (Recommended for Humans)
- **Who**: ITlusions organization members
- **How**: SSO through GitHub with team-based roles
- **Access**: Web UI + CLI with browser authentication

#### 2. Local Users (Recommended for Automation)
- **Who**: Service accounts, CI/CD systems, automation scripts
- **How**: Username/password with auto-generated secure passwords
- **Access**: API tokens for programmatic access

### User Types & Permissions

| User Type | GitHub Teams | ArgoCD Role | CLI Access | API Access |
|-----------|--------------|-------------|------------|-------------|
| **GitHub Admins** | `ITlusions:Admins`, `ITlusions:Devops` | `admin` | ✅ Full | ✅ Full |
| **GitHub Developers** | `ITlusions:Developers` | `developer` | ✅ Limited | ✅ Limited |
| **GitHub Readonly** | `ITlusions:Readonly` | `readonly` | ✅ View-only | ✅ View-only |
| **Local Automation** | N/A | Configurable | ✅ API-only | ✅ Full |

## 🛠️ Deployment

### Prerequisites
- Kubernetes cluster with ArgoCD installed
- Helm 3.x
- kubectl configured for your cluster
- GitHub OAuth app configured (see [GitHub Setup](GITHUB_SETUP.md))

### Deploy Configuration

```bash
# Navigate to chart directory
cd d:\repos\ITL.ArgoCD\Chart

# Deploy with default values
helm upgrade --install argocd-config . -n argocd

# Or deploy with custom values
helm upgrade --install argocd-config . -n argocd -f my-values.yaml
```

### Verify Deployment

```bash
# Check if ArgoCD is running
kubectl get pods -n argocd

# Verify configuration was applied
kubectl get cm argocd-cm -n argocd -o yaml | grep -A 5 "dex.config"

# Test GitHub OAuth (replace with your URL)
curl -I https://argocd.dev.itlusions.nl/api/dex/auth
```

## 🎯 Common Use Cases

### Scenario 1: New Team Member Access
1. Add user to appropriate GitHub team in ITlusions organization
2. User visits ArgoCD UI and logs in via GitHub OAuth
3. Access granted automatically based on team membership

**Guide**: [CLI Login → GitHub SSO Login](CLI_LOGIN.md#github-sso-login-recommended-for-team-members)

### Scenario 2: CI/CD Pipeline Integration
1. Configure automation user in `values.yaml`
2. Deploy configuration to get auto-generated password
3. Generate API token for the automation user
4. Use token in CI/CD pipeline

**Guide**: [User Management → Automation Account](USER_MANAGEMENT.md#automation-account)

### Scenario 3: Service Account for Monitoring
1. Create readonly automation user
2. Generate API token with limited permissions
3. Configure monitoring tools to use the token

**Guide**: [User Management → Service Account](USER_MANAGEMENT.md#service-account)

### Scenario 4: Emergency Admin Access
1. Use local admin account (if enabled)
2. Or create emergency automation user with admin role
3. Generate temporary API token

**Guide**: [CLI Login → Admin Login](CLI_LOGIN.md#admin-login-emergency-access)

## 🔧 Configuration Examples

### Basic Local User Setup
```yaml
# values.yaml
users:
  - name: "automation"
    capabilities: "apiKey"
    role: "admin"
    # Password auto-generated (64 characters)
```

### Advanced Multi-User Setup
```yaml
# values.yaml
users:
  - name: "ci-pipeline"
    capabilities: "apiKey"
    role: "developer"
    
  - name: "monitoring"
    capabilities: "apiKey"
    role: "readonly"
    
  - name: "emergency-access"
    capabilities: "apiKey,login"
    password: "CustomSecurePassword123!"
    role: "admin"
```

### GitHub OAuth Configuration
```yaml
# values.yaml (secrets redacted)
github:
  clientId: "Ov23lig***********"
  clientSecret: "[MANAGED_EXTERNALLY]"
  organization: "ITlusions"

argocd:
  url: "https://argocd.dev.itlusions.nl"
```

## 🔒 Security Features

- **Auto-Generated Passwords**: 64-character secure passwords for local users
- **GitHub OAuth Integration**: Leverage existing GitHub organization security
- **Role-Based Access Control**: Fine-grained permissions based on roles
- **API Token Management**: Secure token generation with expiration support
- **Secrets Redaction**: Sensitive information properly redacted in documentation

## 🚨 Troubleshooting

### Common Issues

| Problem | Solution | Reference |
|---------|----------|-----------|
| Can't login via GitHub | Check team membership in ITlusions organization | [GitHub Setup](GITHUB_SETUP.md#team-membership) |
| CLI authentication fails | Verify server URL and use `--grpc-web` flag | [CLI Login](CLI_LOGIN.md#troubleshooting) |
| API token doesn't work | Check token format and account permissions | [Tokens](TOKENS.md#troubleshooting) |
| Local user can't login | Verify user exists in ConfigMap and secret | [User Management](USER_MANAGEMENT.md#user-cant-login) |

### Debug Commands
```bash
# Check ArgoCD configuration
kubectl get cm argocd-cm -n argocd -o yaml

# Check user accounts
kubectl get cm argocd-cm -n argocd -o yaml | grep "accounts\."

# Check RBAC configuration
kubectl get cm argocd-rbac-cm -n argocd -o yaml

# View ArgoCD server logs
kubectl logs deployment/argocd-server -n argocd

# Test CLI connection
argocd login argocd.dev.itlusions.nl --grpc-web --loglevel debug
```

## 📝 Getting Help

### Documentation Structure
```
docs/
├── index.md              # This overview page
├── CLI_LOGIN.md          # CLI authentication guide
├── TOKENS.md             # API token management
├── USER_MANAGEMENT.md    # Local user configuration
└── GITHUB_SETUP.md       # GitHub OAuth setup
```

### Quick Links
- **Need to login?** → [CLI Login Guide](CLI_LOGIN.md)
- **Setting up automation?** → [Token Management](TOKENS.md)
- **Adding users?** → [User Management](USER_MANAGEMENT.md)
- **GitHub not working?** → [GitHub Setup](GITHUB_SETUP.md)

### Support
- **Issues**: Check the troubleshooting sections in each guide
- **Configuration**: Refer to the template files in `Chart/templates/`
- **Examples**: Each guide contains practical examples and use cases

---

**Last Updated**: September 13, 2025  
**ArgoCD Version**: Compatible with ArgoCD 2.x  
**Chart Version**: ITL.ArgoCD v1.0