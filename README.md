# Instavote Platform Configuration

> **Important**: Before using this configuration, replace the following placeholders throughout the repository:
> - `GITHUB_ORG`: Your GitHub organization name
> - `YOUR_ORGANIZATION`: Your company or organization name
> - Update email addresses and contact information in the Support section
> - Update any environment-specific URLs or endpoints

This repository contains platform-level configurations for the Instavote application, including tenant management, RBAC, network policies, and deployment patterns.

## Repository Structure with Sample Code

```
instavote-platform-config/
├── tenants/
│   ├── dev/
│   │   ├── project.yaml                # ArgoCD project definition
│   │   ├── resource-quota.yaml         # Resource limits
│   │   └── network-policies.yaml       # Network policies
│   ├── staging/
│   │   └── [similar structure as dev]
│   └── prod/
│       └── [similar structure as dev]
├── rbac/
│   ├── groups/
│   │   ├── platform-admins.yaml
│   │   ├── dev-team.yaml
│   │   └── ops-team.yaml
│   └── roles/
│       ├── tenant-admin.yaml
│       └── tenant-viewer.yaml
└── applicationsets/
    └── instavote-apps.yaml           # Application deployment patterns
```

### Sample Configurations

#### 1. ArgoCD Project (tenants/dev/project.yaml)
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: instavote-dev
  namespace: argocd
spec:
  description: Development environment for Instavote application
  sourceRepos:
    - 'https://github.com/GITHUB_ORG/instavote-gitops'  # Replace GITHUB_ORG with your organization
  destinations:
    - namespace: instavote-dev
      server: https://kubernetes.default.svc
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
    - group: 'apps'
      kind: Deployment
    - group: 'argoproj.io'
      kind: Rollout
  roles:
    - name: dev-admin
      description: Developer admin access
      policies:
        - p, proj:instavote-dev:dev-admin, applications, *, instavote-dev/*, allow
```

#### 2. Resource Quota (tenants/dev/resource-quota.yaml)
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: instavote-dev
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "20"
```

#### 3. Network Policy (tenants/dev/network-policies.yaml)
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: instavote-dev
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-vote-to-redis
  namespace: instavote-dev
spec:
  podSelector:
    matchLabels:
      app: redis
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: vote
    ports:
    - protocol: TCP
      port: 6379
```

#### 4. RBAC Role (rbac/roles/tenant-admin.yaml)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: tenant-admin
rules:
  - apiGroups: ["argoproj.io"]
    resources: ["applications", "applicationsets"]
    verbs: ["get", "list", "watch", "update", "patch"]
  - apiGroups: [""]
    resources: ["namespaces", "pods", "services"]
    verbs: ["get", "list", "watch"]
```

#### 5. ApplicationSet (applicationsets/instavote-apps.yaml)
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: instavote
  namespace: argocd
spec:
  generators:
    - matrix:
        generators:
          - git:
              repoURL: https://github.com/GITHUB_ORG/instavote-gitops  # Replace GITHUB_ORG with your organization
              revision: HEAD
              directories:
                - path: "charts/*"
          - list:
              elements:
                - environment: dev
                  namespace: instavote-dev
                - environment: staging
                  namespace: instavote-staging
                - environment: prod
                  namespace: instavote-prod
  template:
    metadata:
      name: '{{path.basename}}-{{environment}}'
    spec:
      project: instavote-{{environment}}
      source:
        repoURL: https://github.com/initcron/instavote-gitops
        targetRevision: HEAD
        path: '{{path}}'
        helm:
          valueFiles:
            - env/{{environment}}.yaml
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{namespace}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
```

## Components

### Tenant Configuration
Each tenant (environment) has its own configuration set:
- `project.yaml`: ArgoCD project definition
- `namespace.yaml`: Namespace configuration
- `resource-quota.yaml`: Resource limits and quotas
- `network-policies.yaml`: Network isolation rules

### RBAC Configuration
Role-based access control configurations:
- Platform admin roles
- Tenant-specific roles
- Team access bindings

### ApplicationSets
Deployment patterns for applications across environments:
- Application deployment configurations
- Environment-specific overrides
- Tenant isolation rules

## Getting Started

### Prerequisites
- Kubernetes cluster with network policy support
- ArgoCD v2.8 or later
- kubectl configured with admin access

### Initial Setup

1. **Clone the Repository**
   ```bash
   git clone https://github.com/GITHUB_ORG/instavote-platform-config.git  # Replace GITHUB_ORG with your organization
   cd instavote-platform-config
   ```

2. **Apply Base Configurations**
   ```bash
   # Apply RBAC configurations
   kubectl apply -f rbac/roles/
   kubectl apply -f rbac/groups/

   # Create and configure tenants
   kubectl apply -f tenants/*/namespace.yaml
   kubectl apply -f tenants/*/project.yaml
   kubectl apply -f tenants/*/resource-quota.yaml
   kubectl apply -f tenants/*/network-policies.yaml

   # Apply ApplicationSets
   kubectl apply -f applicationsets/
   ```

## Tenant Onboarding

### Adding a New Tenant
1. Create tenant directory structure:
   ```bash
   mkdir -p tenants/new-tenant/{rbac,network-policies}
   ```

2. Create required configurations:
   - Copy and modify namespace.yaml
   - Configure resource quotas
   - Set up network policies
   - Create ArgoCD project

3. Apply configurations:
   ```bash
   kubectl apply -f tenants/new-tenant/
   ```

## Security Controls

### Network Policies
- Default deny all ingress traffic
- Explicit allow rules for required communication
- Environment isolation
- Service-to-service communication rules

### Resource Quotas
Each environment has specific resource limits:
- CPU and memory limits
- Pod count restrictions
- Storage quotas

### Access Control
- Role-based access control (RBAC)
- Tenant isolation
- Least privilege principle
- Service account management

## Best Practices

### Repository Management
- Use branch protection rules
- Require pull request reviews
- Implement CI/CD validation
- Regular security scanning

### Configuration Updates
- Use pull requests for changes
- Document changes thoroughly
- Test in lower environments first
- Follow change management process

### Network Security
- Implement zero-trust networking
- Regular policy reviews
- Monitor policy violations
- Keep policies up to date

## Monitoring and Maintenance

### Health Checks
- Regular validation of configurations
- Resource quota monitoring
- Network policy effectiveness
- Access control audit

### Troubleshooting
Common issues and solutions:
- Network connectivity issues
- Resource quota exceeded
- Access denied errors
- ApplicationSet synchronization problems

## Contributing

### Guidelines
1. Create feature branch
2. Make changes
3. Test thoroughly
4. Submit pull request
5. Update documentation

### Review Process
- Code review requirements
- Testing requirements
- Documentation updates
- Security considerations

### Documentation
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Network Policy Guide](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [RBAC Documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

## License
Available under Apache 2.0 license. Owner of the repo is Initcron Systems Private Limited (School of Devops).



