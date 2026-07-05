# k8s-birdstreetexotics

GitOps repo for the **birdstreetexotics.com** tenant, reconciled by **Argo CD
running inside the tenant's vcluster** on the ultra-piggy platform. Self-contained
and isolated — this tenant only ever touches this repo and its own vcluster.

## Layout
- `bootstrap/root.yaml` — app-of-apps root Application (applied once at bootstrap).
  Argo then watches `apps/` and self-reconciles.
- `apps/` — one Argo `Application` per component.
- `manifests/` — the manifests each Application points at.

## Current contents
- `manifests/landing/` — static "coming soon" landing page (nginx serving inline
  HTML from a ConfigMap). Public/ungated. Served at https://birdstreetexotics.com
  and https://www.birdstreetexotics.com.

## Bootstrap (once, after the host vcluster is up)
```fish
# 1. Fetch the tenant kubeconfig from the host (see ultra-piggy k3s.nix helper), e.g.:
#      vc-kubeconfig birdstreetexotics   # -> ~/.kube/vc-birdstreetexotics.yaml
set -x KUBECONFIG ~/.kube/vc-birdstreetexotics.yaml

# 2. Install Argo CD in the vcluster.
kubectl create namespace argocd
kubectl -n argocd apply -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 3. Add the read-only GitHub deploy key as Argo's repo credential (private key file
#    generated at bootstrap; public key added as a repo Deploy Key on GitHub).
kubectl -n argocd create secret generic repo-k8s-birdstreetexotics \
  --from-literal=type=git \
  --from-literal=url=git@github.com:bigkraig/k8s-birdstreetexotics.git \
  --from-file=sshPrivateKey=<path-to-deploy-key>
kubectl -n argocd label secret repo-k8s-birdstreetexotics argocd.argoproj.io/secret-type=repository

# 4. Apply the app-of-apps root; Argo takes over from here.
kubectl -n argocd apply -f bootstrap/root.yaml
```

## DNS
- Public: `birdstreetexotics.com` + `www` → WAN IP (Cloudflare proxied).
- LAN-only: `k8s.birdstreetexotics.com` → `192.168.1.51` (vcluster API; never public).
