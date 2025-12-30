# k8s-web-terminal-kubectl
Kubernetes management from Web Terminal without kubectl.  
<br/><br/>

## ⚠️ Security warning (read before deploying)
In this first version, **no authentication is implemented**!
> This project provides a web terminal that can execute `kubectl` against your cluster API (via RBAC).
> **Do NOT expose it publicly without at least**: IP allowlisting, TLS, and authentication: this is supported among ALB annotations: e.g. *inbound-cidrs*.

**Official documentation**: https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/guide/ingress/annotations/

Treat this as a “break-glass / limited-access” tool, not a general UI.
<br/><br/>

## What this project does
- k8s-web-terminal-kubectl is a web-based terminal for Kubernetes operations (kubectl inside the cluster), exposed via AWS ALB Ingress.
Tested on Amazon EKS 1.33.
<br/><br/>

## What is it and why?
This repository deploys a **lightweight web terminal (ttyd)** into a Kubernetes cluster, so you can **run `kubectl` from your browser** without installing `kubectl` locally and you should configure `kubeconfig`.
It is **useful when you need** a quick, controlled in-cluster admin terminal (for training, demos, or emergency access), especially in AWS EKS environments where access paths can be restricted.
<br/><br/>

## Use cases
- **Training / demo environments**: Provide a temporary browser-based kubectl terminal for students or demo sessions.
- **Break-glass access**: Emergency access path when your usual workstation/jump host/management tool or VPN path is unavailable (still requires strong controls).
- **Limited support sessions**: Time-boxed troubleshooting sessions where you want to avoid distributing kubeconfigs.
<br/><br/>

## Key features
- Technologies used: Tested on EKS 1.33 (should work on 1.32+)
- Architecture: ALB → Service → ttyd Pod → K8s API
- License: **MIT License** – see `LICENSE` file in the parent folder
<br/><br/>

## Current environment settings
- **Web Terminal**: By default, the web terminal can exec only into pods in defined namespace, named *web-kubectl* 
- **RBAC**:
  Currently, the following permissions are set as default (they can be modified in the manifest file if necessary):
  * **resource**: pod
  * **verbs**: get and list
  * **namespace**: web-kubectl
<br/><br/>

## Quickstart
### Prerequisites
- **Kubernetes cluster** (tested on **EKS 1.33**):
- **AWS Load Balancer Controller** installed and working: If you also use Amazon EKS, don't forget: we use ALB Ingress annotations, so the AWS Load Balancer Controller must be running in the cluster (otherwise Ingress will not receive ALB).  
**Detailed documentation** here: https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html
- kubectl access for deploying manifests (for example, from your admin workstation)
- A browser for accessing the terminal: validated with Chrome (Version 143.0.7499.170 (Official Build), 64-bit), on Windows 10 and 11  

**Note**: we use this image with built-in kubectl: *nikstep/kubectl-ttyd:1.0*. Don't forget to always **use the correct kubectl binary**! If necessary, create your own image and use it.

### Deploy
```bash
kubectl apply -f manifests/ns.yaml
kubectl apply -f manifests/role-rolebinding.yaml
kubectl apply -f manifests/deployment.yaml
kubectl apply -f manifests/service.yaml
kubectl apply -f manifests/ingress.yaml
```

**Get the URLGet the URL** andopen the ALB hostname in your browser:
```bash
kubectl -n web-kubectl get ingress web-kubectl
```

**First steps**:  
You should see a terminal prompt in the browser.

Try a read-only command first:
```bash
kubectl get pods -n default
```
Then an exec test (if RBAC allows it):
```bash
kubectl exec -it <pod> -n web-kubectl -- /bin/sh
```
<br/><br/>

## Security model (network boundary + RBAC + audit)
### Network boundary
• Traffic enters via 𝗔𝗪𝗦 𝗔𝗟𝗕 𝗜𝗻𝗴𝗿𝗲𝘀𝘀.  
• Without restrictions, this may be reachable from the public internet.
### RBAC (least privilege)
• The ServiceAccount used by the web terminal is bound to a 𝗻𝗮𝗺𝗲𝘀𝗽𝗮𝗰𝗲-𝘀𝗰𝗼𝗽𝗲𝗱 𝗥𝗼𝗹𝗲.  
• **Scope**: the target namespace defined by the Role/RoleBinding  
**By default**, it can:  
  • get/list pods, and  
  • create on pods/exec    
### Auditability
• Every kubectl operation results in Kubernetes API calls.  
For **production-grade usage**, enable/collect:  
  • Kubernetes audit logs (cluster-side)  
  • ALB access logs / WAF logs (edge-side, if enabled)
<br/><br/>

## Hardening (recommended before real usage)
Minimum recommended controls:
### 1. 𝗜𝗣 𝗮𝗹𝗹𝗼𝘄𝗹𝗶𝘀𝘁 𝗮𝘁 𝗔𝗟𝗕 𝗹𝗲𝘃𝗲𝗹
• Use alb.ingress.kubernetes.io/inbound-cidrs to restrict access to office/VPN CIDRs.
### 2. 𝗧𝗟𝗦
• Terminate TLS on the ALB using ACM (alb.ingress.kubernetes.io/certificate-arn) and listen on 443.
### 3. 𝗔𝘂𝘁𝗵𝗲𝗻𝘁𝗶𝗰𝗮𝘁𝗶𝗼𝗻
• Put the ALB behind an authentication layer (OIDC/Cognito) or protect it behind a private network/VPN.  
• Consider AWS WAF rules as an additional layer if internet-facing.
### 4. 𝗥𝘂𝗻 𝗮𝘀 𝗻𝗼𝗻-𝗿𝗼𝗼𝘁 + 𝗱𝗿𝗼𝗽 𝗰𝗮𝗽𝗮𝗯𝗶𝗹𝗶𝘁𝗶𝗲𝘀
• Prefer *runAsNonRoot: true*, *readOnlyRootFilesystem: true*, drop Linux capabilities, *seccompProfile: RuntimeDefault*.  
• If ttyd requires writable paths, mount a dedicated *emptyDir* (e.g., to **/tmp**) while keeping the root filesystem read-only.
### 5. 𝗥𝗲𝗱𝘂𝗰𝗲 𝗯𝗹𝗮𝘀𝘁 𝗿𝗮𝗱𝗶𝘂𝘀
• Keep RBAC namespace-scoped (avoid ClusterRole unless absolutely needed).  
• Consider a dedicated “sandbox” namespace for supported operations.
<br/><br/>
<br/><br/>

# Planned changes in the near future:
- Ingress ALB annotations - Hardening:
  - IP allowlist at ALB level: *alb.ingress.kubernetes.io/inbound-cidrs* (e.g., office/VPN CIDRs), 
  - TLS: ACM certificate + HTTPS listener (later "mandatory" if not on a "trusted" network) under the *overlays/internet-facing*
- RBAC - Parameterization:
  - TARGET_NAMESPACE=default by default (Kustomize overlay patch)
  - If multiple namespaces are required: multiple RoleBindings, not ClusterRole (if possible)
  - Customization of the persmissions: pods/exec + appropriate verbs
- SecurityContext:
  - use stricter settings, like *runAsNonRoot: true*, *readOnlyRootFilesystem: true* and drop not necessary capabilities (**CAPS**) : configure Pod Security Standards (Restricted)
  - if image can tolerate:
    • *runAsNonRoot: true*
    • *runAsUser: 1000* + *runAsGroup: 1000*
    • *readOnlyRootFilesystem: true*
    • *capabilities.drop: ["ALL"]*
    • *seccompProfile: RuntimeDefault*
    **Note**: Due to the operation of **ttyd --writable** and shell, writable space may be required. In this case: mount *emptyDir* to **/tmp**, but rootfs can still be read-only.
    This will be implemented in repo as follows:
      • *base/* functional (but safer),
      • *overlays/hardened/* there will be a "restricted" version.
- Using Helm for the installation process
- Creating a version for multiple environments
- Preparing for the Kustomization version
<br/><br/>


---------------
## Some pictures during operation:

**Web Terminal** for kubectl (inside the container) - **basic commands**:
<img width="1451" height="230" alt="Web Terminal for kubectl - basic commands 1" src="https://github.com/user-attachments/assets/20df2d19-f2f6-4d15-b748-0911fbca3092" />
and  
<img width="1427" height="92" alt="Web Terminal for kubectl - basic commands 2" src="https://github.com/user-attachments/assets/533b80b2-b0fc-4044-9285-0d3ca7ae9640" />
<br/><br/>

**Web Terminal** for kubectl - start a test nginx pod and jump into it using **kubectl exec under RBAC control**:
<img width="548" height="75" alt="Web Terminal for kubectl - basic commands - start test nginx pod" src="https://github.com/user-attachments/assets/466559aa-4978-4327-b9af-5ff9e8673301" />
and  
<img width="553" height="172" alt="Web Terminal for kubectl - basic commands - start jump into pod with kubectl exec" src="https://github.com/user-attachments/assets/2beca1a9-ecbb-4e80-a627-b0fb40537e17" />

<br/><br/>

## Contributions:
This project is provided as-is, primarily for educational and reference purposes. Feel free to fork and adapt it to your own use case. \
However, I am not currently accepting external pull requests due to limited review time. 
<br/><br/>

## Note:
**Discussions are open** for ideas, questions, and feedback.  
However, this project does not accept feature requests, bug reports, or pull requests.  
Feel free to share ideas, but there is no guaranteed response time.


