# KubernetesGitSync
**Securely sync your Kubernetes objects to Git.**

## Overview
KubernetesGitSync monitors Kubernetes objects for changes and ensures their YAML representations are securely committed to a Git repository. For sensitive objects (e.g., annotated or secrets), the tool encrypts them using **[SOPS](https://github.com/mozilla/sops)** before committing.

## Key Features
- **Change Detection:** Tracks updates to Kubernetes objects based on annotations or labels.
- **Git Integration:** Writes updated YAML configurations to a specified Git repository, with automated commits and pushes.
- **Secure Handling:** Encrypts sensitive objects before committing, ensuring security and compliance.

## Use Case
Imagine Cluster A running **cert-manager**, which generates a wildcard TLS secret. You need to propagate this TLS secret to other clusters without recreating it. KubernetesGitSync facilitates this by syncing the secret across clusters using GitOps principles, maintaining consistency and security.

## Annotations Example
To enable and configure KubernetesGitSync for specific Kubernetes objects, use the following annotations:

```yaml
kubernetes-git-sync/enabled: "true"
kubernetes-git-sync/git-filepath: test.yml
kubernetes-git-sync/git-secret: default/git-secret
kubernetes-git-sync/git-url: git@github.com:xonvanetta/kubernetes-git-sync.git
kubernetes-git-sync/sops-age-recipients: age1y4hfh6duxd6xv32k7rx0nf2majvp2yywlwsemzl28z6y59pju56swcsctg
```

- `kubernetes-git-sync/enabled`: Enables syncing for the object.
- `kubernetes-git-sync/git-filepath`: Specifies the file path in the Git repository where the object will be written.
- `kubernetes-git-sync/git-secret`: Refers to the Kubernetes secret containing Git credentials.
- `kubernetes-git-sync/git-url`: The Git repository URL.
- `kubernetes-git-sync/sops-age-recipients`: Specifies SOPS recipients for encrypting sensitive objects.

## Secret Example
To configure Git authentication, create a Kubernetes secret with the required credentials:

```yaml
apiVersion: v1
data:
  password: ""
  private-key: LS0t.... # Replace with your private key (base64 encoded)
kind: Secret
metadata:
  name: git-secret
  namespace: default
type: Opaque
```

## Challenges
While GitOps offers many benefits, it also presents unique challenges, particularly around resource ownership:
- **Conflict Management:** Determining ownership of resources can be tricky. For example, if a cluster generates a secret, it should not later have that secret overwritten by GitOps.
- **Testing Needed:** Scenarios with overlapping ownership should be thoroughly tested to understand the impact and avoid conflicts.

## Roadmap
Here are some planned enhancements:
1. **Add Metrics:** Introduce metrics for monitoring sync operations and object changes.
2. **Universal Object Listener:** Investigate whether KubernetesGitSync can listen to all Kubernetes objects and serialize them using Kubernetes' built-in marshal functionality, instead of relying on a custom implementation for API resources.
3. **Additional Authentication Methods:** Expand support for other authentication mechanisms.