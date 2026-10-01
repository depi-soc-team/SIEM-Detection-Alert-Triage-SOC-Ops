# ADR-001: Remote Access to the Lab via Tailscale

- **Status:** Accepted
- **Date:** 2026-10-02
- **Deciders:** Lead & Architect, Infra & Onboarding

## Context

Team members work from home and need remote access to the SOC lab, mainly Kibana, and for the Infra role the whole lab network.

Constraints:

- The whole lab runs as VMs on **one physical host** connected to a home internet line.
- The line is behind **CGNAT**: there is no public IP, so the host cannot accept inbound connections from the internet.
- The **FortiGate VM** runs on a **permanent trial license**, which supports **low encryption only**. Its SSL-VPN cannot be used with acceptable security.
- The team has 5 members and no budget for paid services.
- Access must be encrypted, per-user and revocable.

## Options Considered

### 1. FortiGate SSL-VPN / IPsec VPN

- ➕ Uses the lab's own firewall, which is realistic SOC practice, and its logs would feed the SIEM.
- ➖ The trial license only allows low encryption, so SSL-VPN is not usable with modern clients or acceptable ciphers.
- ➖ Clients still need an inbound connection to the FortiGate, which CGNAT blocks.

### 2. Port forwarding on the home router

- ➖ Impossible behind CGNAT because there is no public IP to forward from.
- ➖ Even with a public IP, it would expose Kibana directly to the internet.

### 3. Cloudflare Tunnel

- ➕ Outbound-only connection, so it works behind CGNAT. Free tier available.
- ➕ Good fit for publishing a single web app (Kibana) with Cloudflare Access policies.
- ➖ Requires a domain managed in Cloudflare.
- ➖ Publishes Kibana on a public hostname; protection depends entirely on Access policies being correct.
- ➖ Network-level access to the rest of the lab (SSH, FortiGate, agents) needs the WARP client and more Zero Trust configuration.

### 4. Tailscale (chosen)

- ➕ WireGuard-based, end-to-end encrypted mesh, with no open ports. NAT traversal works behind CGNAT.
- ➕ Per-user identity (sign in with GitHub), with simple invite and revoke from the admin console.
- ➕ **Node sharing** gives least-privilege access to the SIEM VM only, because shared machines do not advertise subnet routes to the tailnets they are shared into. A **subnet router** gives full-lab access to tailnet users in the roles that need it.
- ➕ Free plan, quick to set up, Windows client for members.
- ➖ Depends on a third-party coordination service.
- ➖ The free plan limits the number of tailnet users.

## Decision

Use **Tailscale** for all remote access to the lab:

- **Default:** node-share the SIEM VM with each member, giving Kibana access only.
- **Full-lab access:** invite as a tailnet user, routed through an Ubuntu **subnet router** that advertises the lab subnet. This is only for roles that need it, such as Infra.
- Real addresses, tailnet name and invite links are shared privately by the Lead and are never committed.

Member and admin procedures: [docs/REMOTE-ACCESS.md](../../docs/REMOTE-ACCESS.md).

## Consequences

**Positive**

- Encrypted remote access with no inbound ports and no exposure of Kibana to the internet.
- Least privilege by default: most members can reach only Kibana.
- Access is per-user and can be revoked instantly by removing the user or the share.

**Negative / accepted risks**

- **Single point of failure:** the lab host must stay powered on and online for anyone to work remotely.
- **Free plan limit of 6 users:** this fits the team of 5 with little headroom. Adding members or guests (e.g. the instructor) needs care.
- **Kibana is served over HTTP inside the tunnel:** traffic is encrypted by WireGuard end to end, but Kibana itself does not use TLS. This is accepted as a lab limitation and would not be acceptable in production.
- Shared machines do not advertise subnet routes to the tailnets they are shared into. This is why node sharing gives SIEM-only access, and it also means node-shared users must reach Kibana by its Tailscale address, not its lab IP. This is documented in the troubleshooting guide.
- The FortiGate VM remains in the lab for syslog and traffic logging, but not as the remote-access gateway.
