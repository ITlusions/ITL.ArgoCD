# ArgoCD GitHub Integration

This chart configures ArgoCD to authenticate users via GitHub using OAuth through Dex.

## Prerequisites

### 1. GitHub OAuth App Configuration

In your GitHub organization settings, create an OAuth App with the following settings:

#### OAuth App Settings:
- **Application name**: `ArgoCD ITlusions`
- **Homepage URL**: `https://argocd.dev.itlusions.nl`
- **Authorization callback URL**: `https://argocd.dev.itlusions.nl/api/dex/callback`

#### Current Configuration:
- **Client ID**: `Ov23lig***********` (already configured)
- **Client Secret**: `[REDACTED]` (already configured)
- **Organization**: `ITlusions`

### 2. GitHub Organization Teams

Create/ensure the following teams exist in your GitHub organization to match the RBAC configuration:

- `Admins` - Full administrative access
- `Devops` - Full administrative access  
- `Developers` - Application deployment and management
- `Readonly` - Read-only access

### 3. Team Membership

Add users to the appropriate teams in your GitHub organization. Users must be members of the `ITlusions` organization and assigned to teams for proper role mapping.

## Current Configuration

### Authentication Flow:
1. User visits ArgoCD UI
2. Clicks "LOG IN VIA GITHUB"
3. Redirected to GitHub for OAuth authentication
4. After successful login, user is redirected back to ArgoCD via Dex
5. ArgoCD validates the OAuth token and maps user teams to roles
6. User gains access based on their team membership

### Team to Role Mapping:

| GitHub Team | ArgoCD Role | Permissions |
|-------------|-------------|-------------|
| `ITlusions:Admins` | `admin` | Full application and infrastructure management |
| `ITlusions:Devops` | `admin` | Full application and infrastructure management |
| `ITlusions:Developers` | `developer` | Application deployment and sync |
| `ITlusions:Readonly` | `readonly` | View applications and logs only |

## Installation

### 1. Update values.yaml

The current configuration in `values.yaml` matches your live setup:

```yaml
github:
  clientId: "****"
  clientSecret: "****"
  organization: "ITlusions"

argocd:
  url: "https://argocd.dev.itlusions.nl"
```

### 2. Deploy the configuration

```bash
helm upgrade --install argocd-config ./Chart -n argocd
```

### 3. Restart ArgoCD components (if needed)

```bash
kubectl rollout restart deployment/argocd-server -n argocd
kubectl rollout restart deployment/argocd-dex-server -n argocd
```

## Testing

1. Navigate to `https://argocd.dev.itlusions.nl`
2. Click "LOG IN VIA GITHUB" 
3. You should be redirected to GitHub for authentication
4. After successful login, you should be redirected back to ArgoCD with appropriate permissions based on your team membership

## Troubleshooting

### Check ArgoCD server logs:
```bash
kubectl logs -f deployment/argocd-server -n argocd
```

### Check Dex logs:
```bash
kubectl logs -f deployment/argocd-dex-server -n argocd
```

### Check current configuration:
```bash
kubectl get configmap argocd-cm -n argocd -o yaml
kubectl get configmap argocd-rbac-cm -n argocd -o yaml
```

### Verify team membership:
Ensure users are:
1. Members of the `ITlusions` GitHub organization
2. Assigned to the appropriate teams
3. Teams have the correct names matching the RBAC configuration

## Security Notes

- The client secret is currently stored in the ConfigMap (visible in the current setup)
- Consider moving the client secret to a Kubernetes Secret for better security
- Ensure the GitHub OAuth app is properly configured with the correct callback URL
- Regularly review team memberships and permissions