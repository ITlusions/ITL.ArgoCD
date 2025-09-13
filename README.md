# ITL.ArgoCD

ArgoCD configuration chart for ITlusions infrastructure with GitHub OAuth integration and local user management.

[![Helm](https://img.shields.io/badge/Helm-v3-blue)](https://helm.sh/)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-v2.x-green)](https://argo-cd.readthedocs.io/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## 📚 Documentation

**[📖 Complete Documentation](docs/index.md)** - Start here for comprehensive guides and examples

### Quick Links
- **[🔐 CLI Login Guide](docs/CLI_LOGIN.md)** - Authentication methods for CLI and web UI
- **[🎫 Token Management](docs/TOKENS.md)** - API token generation for automation
- **[👥 User Management](docs/USER_MANAGEMENT.md)** - Local user configuration and management
- **[🔗 GitHub Setup](docs/GITHUB_SETUP.md)** - GitHub OAuth configuration details

## 🚀 Features

- **🔒 GitHub OAuth Authentication**: SSO integration via Dex for ITlusions organization members
- **👥 Local User Management**: Automated service accounts with secure password generation
- **🛡️ Role-Based Access Control**: Fine-grained permissions based on GitHub teams and local roles
- **🔑 API Token Support**: Secure token generation for automation and CI/CD pipelines
- **⚡ Auto-Configuration**: Template-based setup with sensible defaults
- **🎨 UI Customization**: Branded interface with ITlusions styling

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

## ⚡ Quick Start

### Prerequisites
- Kubernetes cluster with ArgoCD installed
- Helm 3.x installed
- kubectl configured for your cluster
- GitHub OAuth app configured (see [GitHub Setup](docs/GITHUB_SETUP.md))

### 1. Configure Secrets
Update `Chart/values.yaml` with your GitHub OAuth credentials:

```yaml
github:
  clientId: "your-github-oauth-client-id"
  clientSecret: "your-github-oauth-client-secret"
  organization: "ITlusions"
```

### 2. Deploy Configuration
```bash
# Navigate to chart directory
cd Chart

# Deploy with Helm
helm upgrade --install argocd-config . -n argocd

# Verify deployment
kubectl get pods -n argocd
```

### 3. Access ArgoCD
- **Web UI**: https://argocd.dev.itlusions.nl
- **CLI Login**: `argocd login argocd.dev.itlusions.nl --grpc-web --sso`

### 4. Get Automation Passwords (if using local users)
```bash
kubectl get secret argocd-secret -n argocd -o jsonpath='{.data.automation\.password}' | base64 -d
```

## 🔐 Authentication Overview

### GitHub OAuth Users (Primary)
- **Who**: ITlusions organization members
- **Access**: SSO through GitHub with team-based roles
- **Login**: Web UI + CLI with browser authentication

### Local ArgoCD Users (Automation)
- **Who**: Service accounts, CI/CD systems, automation scripts
- **Access**: API tokens with auto-generated secure passwords
- **Login**: API-only access for programmatic operations

## 🎯 User Roles & Permissions

| User Type | GitHub Team | ArgoCD Role | Web UI | CLI | API | Permissions |
|-----------|-------------|-------------|--------|-----|-----|-------------|
| **GitHub Admin** | `ITlusions:Admins`, `ITlusions:Devops` | `admin` | ✅ | ✅ | ✅ | Full access |
| **GitHub Developer** | `ITlusions:Developers` | `developer` | ✅ | ✅ | ✅ | App management |
| **GitHub Readonly** | `ITlusions:Readonly` | `readonly` | ✅ | ✅ | ✅ | View only |
| **Local Automation** | N/A | `admin` | ❌ | ❌ | ✅ | API automation |

## 📁 Repository Structure

```
ITL.ArgoCD/
├── 📄 README.md                      # This file
├── 📁 docs/                          # Documentation
│   ├── 📄 index.md                  # Documentation hub
│   ├── 📄 CLI_LOGIN.md              # CLI authentication
│   ├── 📄 TOKENS.md                 # API token management
│   ├── 📄 USER_MANAGEMENT.md        # User configuration
│   └── 📄 GITHUB_SETUP.md           # GitHub OAuth setup
└── 📁 Chart/                        # Helm chart
    ├── 📄 Chart.yaml                # Chart metadata  
    ├── 📄 values.yaml               # Configuration values
    ├── 📁 templates/                # Helm templates
    │   ├── 📄 argocd-server-config-cm.yaml  # Main ArgoCD config
    │   ├── 📄 argocd-rbac-cm.yaml           # RBAC policies
    │   ├── 📄 argocd-secret.yaml            # User passwords
    │   ├── 📄 argocd-cmd-params-cm.yaml     # Server parameters
    │   ├── 📄 certificate.yaml              # TLS certificate
    │   └── 📁 traefik/                      # Traefik ingress
    │       ├── 📄 ingress.yaml              # Main ingress
    │       └── 📄 redirect-nl.yaml          # Redirect rules
    └── � appsets/                          # Application sets
        └── 📄 itlminio.disabled             # Disabled MinIO app
```

## ⚙️ Configuration Examples

### Basic Configuration
```yaml
# Chart/values.yaml
argocd:
  url: "https://argocd.dev.itlusions.nl"

github:
  clientId: "your-client-id"
  clientSecret: "your-client-secret"  
  organization: "ITlusions"

users:
  - name: "automation"
    capabilities: "apiKey"
    role: "admin"
    # Password auto-generated (64 characters)
```

### Advanced Configuration
```yaml
# Chart/values.yaml with multiple users
users:
  - name: "ci-pipeline"
    capabilities: "apiKey"
    role: "developer"
    
  - name: "monitoring"
    capabilities: "apiKey"
    role: "readonly"
    
  - name: "emergency-access"
    capabilities: "apiKey,login"
    password: "CustomPassword123!"
    role: "admin"

ui:
  bannerContent: "Managed by ITLusions"
  bannerPermanent: "false"
  bannerPosition: "top"
  bannerUrl: "https://www.itlusions.com"
```

## 🛠️ Common Operations

### Deploy Configuration Changes
```bash
# Apply configuration updates
helm upgrade argocd-config Chart/ -n argocd

# Restart ArgoCD components (if needed)
kubectl rollout restart deployment/argocd-server -n argocd
kubectl rollout restart deployment/argocd-dex-server -n argocd
```

### Get User Passwords
```bash
# Get automation user password
kubectl get secret argocd-secret -n argocd -o jsonpath='{.data.automation\.password}' | base64 -d

# List all user passwords
kubectl get secret argocd-secret -n argocd -o json | jq -r '.data | to_entries[] | select(.key | endswith(".password")) | "\(.key): \(.value | @base64d)"'
```

### Generate API Tokens
```bash
# Login as automation user
argocd login argocd.dev.itlusions.nl --grpc-web --username automation --password <password>

# Generate API token
argocd account generate-token --id ci-cd-token

# Use token for automation
export ARGOCD_AUTH_TOKEN="your-token-here"
argocd app sync my-app --server argocd.dev.itlusions.nl --grpc-web
```

## 🔍 Troubleshooting

### Check Configuration
```bash
# Verify ArgoCD configuration
kubectl get cm argocd-cm -n argocd -o yaml

# Check RBAC settings
kubectl get cm argocd-rbac-cm -n argocd -o yaml

# View ArgoCD logs
kubectl logs deployment/argocd-server -n argocd
```

### Common Issues
- **GitHub login fails**: Check team membership in ITlusions organization
- **CLI authentication fails**: Use `--grpc-web` flag and verify server URL
- **Local user can't login**: Verify user exists in ConfigMap and secret
- **Token doesn't work**: Check token format and account permissions

See [Documentation](docs/index.md) for detailed troubleshooting guides.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Make your changes
4. Test the configuration: `helm template Chart/ --debug`
5. Commit your changes: `git commit -am 'Add new feature'`
6. Push to the branch: `git push origin feature/your-feature`
7. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- **Documentation**: [docs/index.md](docs/index.md)
- **Issues**: Create an issue in this repository
- **ITlusions Internal**: Contact the DevOps team

---

**Maintained by**: ITlusions DevOps Team  
**Last Updated**: September 13, 2025  
**ArgoCD Version**: Compatible with v2.x  
**Helm Version**: Requires v3.x+
