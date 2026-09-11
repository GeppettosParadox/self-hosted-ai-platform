# Security Notes

This repository documents the security model at a high level. It is not a complete deployment guide and intentionally excludes live configuration details.

## Threats considered

The platform assumes that model inputs, search results, and browser content may be untrusted. That means the design has to account for problems such as:

- prompt injection attempting to influence tool use
- SSRF-style requests targeting private services
- browser automation reaching internal resources
- compromised or misbehaving agent services
- unnecessary container privileges increasing blast radius

## Controls used

### Network restrictions

Research/browser traffic is routed through controlled egress paths where private and metadata address ranges can be denied.

### Service isolation

Browser automation, research, search, model inference, and user-facing services are separated into distinct service/network roles rather than sharing unrestricted access.

### Privilege reduction

Where practical, higher-risk services run without root privileges, without Docker socket access, and with read-only root filesystems.

### SSRF validation

The hardening baseline includes checks against:

- loopback targets
- RFC1918 private IPv4 ranges
- Docker bridge/private container ranges
- link-local and cloud metadata addresses
- IPv6 loopback

### Explicit internal exceptions

Internal dependencies that genuinely need direct access are handled as narrow exceptions rather than by giving services broad network access.

## What this does not claim

This is a homelab / learning environment, not a formally audited production security platform. The controls here reduce risk and improve isolation, but they do not make agentic browsing or tool use risk-free.

The main objective is to design the system so that a bad prompt, hostile webpage, or compromised service has fewer places it can reach and fewer privileges it can abuse.
