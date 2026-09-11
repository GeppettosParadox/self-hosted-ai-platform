# Architecture Notes

This document gives a slightly deeper view of the platform without exposing the live environment.

## Design goals

The architecture has been shaped around a few practical goals:

1. Keep model inference local.
2. Separate higher-risk browser/research capabilities from core model services.
3. Make service boundaries obvious enough to debug when something fails.
4. Avoid giving agent services unnecessary host or Docker privileges.
5. Keep the platform modular so individual services can be replaced without rebuilding everything around them.

## Service roles

### Open WebUI
Primary user-facing interface for interacting with local models and agent-enabled workflows.

### Ollama
Local model runtime responsible for inference on NVIDIA GPUs.

### SearXNG
Search aggregation layer used by research workflows. Kept separate from browser automation so search and page interaction do not share the same trust boundary.

### Routing / gateway layer
Coordinates requests between UI, model, search, and agent services. This layer helps keep service-specific behavior from leaking into every component.

### Research agent
Performs research-oriented tool calls and public-web retrieval. Its outbound network access is constrained through a dedicated egress path.

### Browser tooling
Playwright/MCP-oriented services provide browser automation capability. These are isolated more aggressively because arbitrary page content can influence agent behavior.

### Outbound egress proxy
Controls public-network access for services that should not have unrestricted connectivity. Private and metadata address ranges are blocked at this boundary.

### Caddy
Provides reverse-proxy and HTTPS responsibilities for the user-facing edge.

## Network model

The live deployment uses separate Docker networks for different service groups. The public case study intentionally does not publish network names, CIDRs, hostnames, or addresses.

Conceptually, the platform separates:

- user-facing services
- model/inference services
- internal search/cache services
- browser automation services
- controlled internet egress

The browser/research path is deliberately not treated as equivalent to ordinary internal service traffic.

## Trust boundaries

The most important trust boundary is between **model reasoning** and **external content**.

Search results, websites, documents, and browser pages are considered untrusted input. Tool-enabled services are therefore given narrower permissions than the rest of the platform where practical.

A second boundary exists between containers and the Docker/host control plane. Services that do not need Docker control do not receive the Docker socket.

## Failure philosophy

I prefer a blocked request or a visibly failed tool call over silently allowing a service to bypass an isolation boundary. That makes the system occasionally less convenient, but much easier to reason about and safer to iterate on.
