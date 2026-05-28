---
layout: default
title: Roadmap
nav_order: 10
---

# Roadmap

This page outlines planned and in-progress capabilities for the Entry SDK platform. Items are subject to change and do not represent committed delivery dates.

## Configurable liveness challenge thresholds

Allow client applications to configure the confidence threshold that must be met for a liveness session to pass. Today, Entry uses a fixed internal threshold. This feature will expose a per-client configuration so that integrators can tune the trade-off between security strictness and user pass rate to match their specific risk profile.

## Configurable liveness challenge type

Allow client applications to select which liveness challenge mode is required:

| Mode             | Description                                                                                                                |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Movement only    | User performs a head movement (e.g. turn, nod). Lower friction, suitable for lower-risk use cases.                         |
| Movement + light | User performs a movement and the screen flashes a light sequence. Higher assurance, recommended for higher-risk use cases. |

Clients will be able to set a default mode at the application level and optionally override it per session.

## Full OIDC compliance

Expose Entry as a standards-compliant OpenID Connect (OIDC) identity provider. This will allow Entry to be used as a drop-in IdP in any system that supports OIDC — including access control platforms, SSO solutions, and enterprise identity brokers — without requiring custom SDK integration. The verified identity will be returned as a signed ID token containing standard OIDC claims alongside any registered user metadata.

---

## Have feedback or a feature request?

Contact [support@synapser.com](mailto:support@synapser.com).
