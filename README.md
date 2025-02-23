# Instavote Platform Configuration

This repository contains platform-level configurations for the Instavote application, including tenant management, RBAC, network policies, and deployment patterns.

## Repository Structure

```
instavote-platform-config/
├── tenants/              # Tenant-specific configurations
│   ├── dev/
│   ├── staging/
│   └── prod/
├── rbac/                 # RBAC configurations
│   ├── groups/
│   └── roles/
├── applicationsets/      # ApplicationSet definitions
└── docs/                 # Additional documentation
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
   git clone https://github.com/initcron/instavote-platform-config.git
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
Available under Apache 2.0 license. Owner of the repo is Initcron Systems Private Limited (School of Devops) . All rights reserved.

