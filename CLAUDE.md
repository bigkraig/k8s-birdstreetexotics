# k8s-birdstreetexotics — tenant GitOps

Argo CD runs inside the birdstreetexotics vcluster and reconciles this repo
(`apps/*` = Argo Applications, `manifests/*` = the workloads). Push to `main` →
Argo syncs. This tenant is isolated: never reference another tenant (bigkraig /
warbler) from here.

> **Shell: the user runs `fish`.** All commands must be fish syntax — command
> substitution is `(cmd)` not `$(cmd)`, inline env is `env VAR=val cmd`, no heredocs.

## Platform facts (inherited from ultra-piggy)
- Single-node k3s; each tenant is a **vcluster** (Argo inside it). vcluster
  single-namespace-syncs every object into the host ns `birdstreetexotics`.
- Workloads run under **Kata** microVMs (`runtimeClassName: kata`) — set a memory
  `limits` or the guest OOM-kills.
- **TLS:** Traefik's default cert is `*.warbler.haus`; this domain is neither the
  default nor the bigkraig catchall, so its Ingress MUST name its own secret
  (`birdstreetexotics-wildcard-tls`, issued by ROOT cert-manager into host ns
  `birdstreetexotics`). Tenant Ingresses use `router.tls: "true"` + explicit
  `tls.secretName`.
- Argo `selfHeal` reverts manual `kubectl` edits — change `replicas:`/config in git.

## Landing page
`manifests/landing/` — nginx (`nginx-unprivileged`, listens :8080) serving inline
HTML from the `landing-html` ConfigMap. Public/ungated. 2 replicas + RollingUpdate
for zero-downtime. To edit the page, change the ConfigMap `index.html` and push;
Argo redeploys (ConfigMap change → pods roll on the next sync).
