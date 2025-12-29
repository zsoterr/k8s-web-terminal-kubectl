# k8s-web-terminal-kubectl
Kubernetes management from Web Terminal without kubectl

## What this project does
- 

## What is it and why?


## Key features
- Technologies used: `Kubernetes v1.32 and +`,
- Architecture: To be determined later.
- Important files & folders:
  - 
  - `README.md` – this file  
- License: **MIT License** – see `LICENSE` file in the parent folder

## Security note

## Usage
1. **Pre-requisites**::
- **Kubernetes cluster**:

- **Browser**:

- Used **Image**:

2. **Deploy**:

3. **First steps**:


## Note
- This is the first version (v1), the goal was to have a working "demo" version: very basic, but I think it demonstrates well what it can be used for and how.
- It runs under AWS EKS 1.32 cluster

## Planned changes in the near future:
- Ingress ALB annotations - Hardening:
  - IP allowlist at ALB level: *alb.ingress.kubernetes.io/inbound-cidrs* (e.g., office/VPN CIDRs), 
  - TLS: ACM certificate + HTTPS listener (later "mandatory" if not on a "trusted" network) under the *overlays/internet-facing*
- RBAC - Parameterization:
  - TARGET_NAMESPACE=default by default (Kustomize overlay patch)
  - If multiple namespaces are required: multiple RoleBindings, not ClusterRole (if possible)
  - Customization of the persmissions: pods/exec + appropriate verbs
- SecurityContext:
  - use stricter settings, like *runAsNonRoot: false*, *readOnlyRootFilesystem* and drop not necessary capabilities (**CAPS**)
  - configure Pod Security Standards (Restricted)
  - if image can tolerate:
    • *runAsNonRoot: true*
    • *runAsUser: 1000* + *runAsGroup: 1000*
    • *readOnlyRootFilesystem: true*
    • *capabilities.drop: ["ALL"]*
    • *seccompProfile: RuntimeDefault*
    **Note**: Due to the operation of **ttyd --writable** and shell, writable space may be required. In this case: mount *emptyDir* to **/tmp**, but rootfs can still be read-only.
    This will be implemented in repo as follows:
      • *base/* functional (but safer),
      • *overlays/hardened/* there will be a "restricted" version..



---------------
## Some pictures during operation:

Web Terminal 1:
<img width="699" height="801" alt="Web Terminal for kubectl 1" src="https://github.com/user-attachments/assets/xxx1" />

Web Terminal 1:

<img width="1063" height="801" alt="Web Terminal for kubectl 1" src="https://github.com/user-attachments/assets/xxx2" />


## Contributions:
This project is provided as-is, primarily for educational and reference purposes. Feel free to fork and adapt it to your own use case. \
However, I am not currently accepting external pull requests due to limited review time. 

**Note:** **Discussions are open** for ideas, questions, and feedback.
However, this project does not accept feature requests, bug reports, or pull requests.  
Feel free to share ideas, but there is no guaranteed response time.


