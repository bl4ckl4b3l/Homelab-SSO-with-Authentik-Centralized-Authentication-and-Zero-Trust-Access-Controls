# Homelab-SSO-with-Authentik-Centralized-Authentication-and-Zero-Trust-Access-Controls
This project implements Authentik (version 2025.10.3) as the central OpenID Connect (OIDC) identity provider for my Docker-based homelab.

# Goal
The goal was to replace scattered local accounts with centralized authentication, enforce multi-factor authentication (MFA/TOTP), and apply zero-trust principles through native OIDC integrations and forward authentication.

# Key outcomes:

- Single sign-on across 10+ services (Portainer, Immich, Grafana, Mealie, Jellyfin, etc.)
- Mandatory TOTP for sensitive applications
- Network segmentation and container hardening (non-root, no-new-privileges, isolated networks)
- Public exposure via Traefik reverse proxy with Cloudflare DNS and Let's Encrypt certificates

This project demonstrates practical skills in identity and access management (IAM), container security, reverse proxy configuration, and secure integration patterns—directly applicable to enterprise environments.

# Architecture & Security Design

Internet
   │
   ▼
Cloudflare DNS → Traefik (HTTPS termination + CrowdSec middleware)
   │
   ├──► Authentik (auth.zerofaction.net)
   │     ├─ Embedded proxy outpost (forward auth)
   │     └─ OIDC provider
   │
   └──► Homelab services (portainer.internal.zerofaction.net, immich.zerofaction.net, etc.)
         ├─ Native OIDC (Portainer, Grafana, Immich, etc.)
         └─ Forward auth via Traefik middleware (Cloudreve, Jellyfin, Traefik dashboard)
         
# Hardening & Defense-in-Depth Measures

- Container security: All Authentik services run non-root with no-new-privileges:true
- Network segmentation: Dedicated internal bridge network for PostgreSQL and worker; only server joins public proxy network
- Dependency minimization: Upgraded to Authentik 2025.10.3 → removed Redis entirely (PostgreSQL handles caching, tasks, sessions)
- Secret management: Strong secrets stored in .env (chmod 600), loaded via Docker Compose
- MFA enforcement: TOTP required via expression policies bound to applications
- Rate limiting: CrowdSec middleware chained on Authentik and protected services
- Exposure control: Public access only through Traefik with valid Cloudflare certificates

<img width="499" height="459" alt="image" src="https://github.com/user-attachments/assets/19231889-be7c-49c3-92bc-4c1450fdedc4" />
<img width="594" height="460" alt="image" src="https://github.com/user-attachments/assets/4327cfe4-1faf-4c36-89c7-bc292a05767f" />

# Technologies Used

Authentik 2025.10.3 (self-hosted IdP)
Traefik (reverse proxy with forward auth)
Docker Compose (orchestration)
PostgreSQL 16-alpine (sole database backend)
Cloudflare (DNS + certificate management)
CrowdSec (brute-force protection middleware)

# Learning Outcomes

- Designed and deployed a production-grade identity provider in a containerized environment
- Implemented native OIDC and proxy forward authentication patterns
- Applied container hardening best practices (non-root execution, capability restrictions)
- Migrated Authentik across major versions, removing deprecated Redis dependency
- Troubleshot complex issues: OIDC redirect mismatches, claim mapping, inter-container networking, secret propagation
- Enforced zero-trust access with centralized authentication and mandatory MFA

This project serves as a practical demonstration of modern IAM principles in a real-world homelab environment.

#Future Enhancements

- Complete forward auth rollout for all remaining services
- Implement group-based role mapping across applications
- Add SCIM provisioning for select services
- Integrate with Proxmox for host-level SSO
