# ArgoCD CLI Login Guide

This guide explains how to login to ArgoCD CLI using different authentication methods.

## Prerequisites

- ArgoCD CLI must be installed
- Access to the ArgoCD server at `argocd.dev.itlusions.nl`

## Authentication Methods

### GitHub SSO Login (Recommended for Team Members)

For ITlusions organization members:

```bash
argocd login argocd.dev.itlusions.nl --grpc-web --sso
```

**Requirements:**
- Must be a member of the `ITlusions` GitHub organization
- Must be assigned to configured teams (Admins, Devops, Developers, etc.)

**What happens:**
1. ArgoCD CLI opens your default browser
2. You're redirected to GitHub for authentication
3. After successful GitHub login, you're redirected back to ArgoCD
4. CLI automatically receives the authentication token
5. You're logged in with your GitHub identity

### Local User Login

For configured local ArgoCD users:

```bash
# Login with automation account (API-only)
argocd login argocd.dev.itlusions.nl --grpc-web --username automation --password <generated-password>

# Login with other local users (if configured with login capability)
argocd login argocd.dev.itlusions.nl --grpc-web --username <username> --password <password>
```

### Admin Login (Emergency Access)

For local admin account (if enabled):

```bash
argocd login argocd.dev.itlusions.nl --grpc-web --username admin --password <admin-password>
```

### Manual Browser Flow

If automatic browser opening doesn't work:

```bash
argocd login argocd.dev.itlusions.nl --grpc-web --sso --skip-browser
```

## Verify Your Login

After successful login, verify your access:

```bash
# Check your user info
argocd account get-user-info

# List applications to test permissions
argocd app list

# Check current context
argocd context
```

## Getting Auto-Generated Passwords

For local users with auto-generated passwords, retrieve them using kubectl:

```bash
# Get password for automation user
kubectl get secret argocd-secret -n argocd -o jsonpath='{.data.automation\.password}' | base64 -d

# Get password for any user
kubectl get secret argocd-secret -n argocd -o jsonpath='{.data.<username>\.password}' | base64 -d
```

## Example Successful Logins

### GitHub SSO Login
```bash
$ argocd login argocd.dev.itlusions.nl --grpc-web --sso
Performing authorization_code flow login: https://argocd.dev.itlusions.nl/api/dex/auth?...
Opening system default browser for authentication
Authentication successful
'n.weistra@itlusions.nl' logged in successfully
Context 'argocd.dev.itlusions.nl' updated

$ argocd account get-user-info
Logged In: true
Username: n.weistra@itlusions.nl
Issuer: https://argocd.dev.itlusions.nl/api/dex
Groups: ITlusions:Admins,ITlusions:Cluster Admin,ITlusions:Cluster Ops,ITlusions:Devops,n.weistra@itlusions.nl
```

### Local User Login
```bash
$ argocd login argocd.dev.itlusions.nl --grpc-web --username automation --password <generated-password>
'automation' logged in successfully
Context 'argocd.dev.itlusions.nl' updated

$ argocd account get-user-info
Logged In: true
Username: automation
```

## User Types and Access Levels

### GitHub OAuth Users
Access level depends on GitHub team membership:

| GitHub Team | ArgoCD Role | CLI Access |
|-------------|-------------|------------|
| `ITlusions:Admins` | `admin` | Full access to all CLI commands |
| `ITlusions:Devops` | `admin` | Full access to all CLI commands |
| `ITlusions:Developers` | `developer` | Application management, limited cluster access |
| `ITlusions:Readonly` | `readonly` | Read-only access |

### Local ArgoCD Users
Configured in `values.yaml`:

| User | Capabilities | Role | Purpose |
|------|-------------|------|---------|
| `automation` | `apiKey` | `admin` | API-only automation account |
| Custom users | `login`, `apiKey,login` | Configurable | Application-specific access |

## Common Commands After Login

```bash
# List all applications
argocd app list

# Get application details
argocd app get <app-name>

# Sync an application
argocd app sync <app-name>

# View application logs
argocd app logs <app-name>

# List clusters
argocd cluster list

# List repositories
argocd repo list
```

## Token Management

### For Automation/CI-CD

For automated scripts, you can use the authentication token from your browser session. The token is stored locally after SSO login.

### Check Token Location

ArgoCD CLI stores authentication info in:
- **Windows**: `%USERPROFILE%\.argocd\config`
- **Linux/Mac**: `~/.argocd/config`

## Troubleshooting

### Login Issues

1. **Browser doesn't open**: Use `--skip-browser` flag
2. **Permission denied**: Check your GitHub team membership
3. **Token expired**: Re-run the login command

### Check Configuration

```bash
# Verify ArgoCD server config
kubectl get configmap argocd-cm -n argocd -o yaml

# Check RBAC configuration
kubectl get configmap argocd-rbac-cm -n argocd -o yaml
```

### Debug Mode

Add `--loglevel debug` to any command for verbose output:

```bash
argocd login argocd.dev.itlusions.nl --grpc-web --sso --loglevel debug
```

## Security Notes

- Authentication tokens are stored locally and have expiration times
- Always logout when using shared computers: `argocd logout argocd.dev.itlusions.nl`
- Your access is tied to your GitHub team membership - changes in GitHub reflect immediately
- The CLI respects the same RBAC policies as the web UI