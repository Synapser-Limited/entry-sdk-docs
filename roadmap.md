---
layout: default
title: Roadmap
nav_order: 10
---

# Roadmap

This page outlines planned and in-progress capabilities for the Entry SDK platform. Items are subject to change and do not represent committed delivery dates.

## ✅ Configurable liveness challenge thresholds

Client applications can now configure the confidence threshold that must be met for a liveness session to pass. The threshold can be set at three levels, resolved in this order:

1. **Per-session override** — pass `passThreshold` in the `identifyUser()` call (0–100)
2. **Per-client default** — set `LivenessPassConfidencePercentage` on the `ClientApp` record (per platform: iOS / Android / Web)
3. **Global default** — environment variable on the Amplify backend (default: **80**)

## ✅ Configurable liveness challenge type

Client applications can now select which liveness challenge mode is used:

| Mode             | Value | Description                                                                                                                |
| ---------------- | ----- | -------------------------------------------------------------------------------------------------------------------------- |
| Movement only    | `1`   | User performs a head movement (e.g. turn, nod). Lower friction, suitable for lower-risk use cases.                         |
| Movement + light | `2`   | User performs a movement and the screen flashes a light sequence. Higher assurance, recommended for higher-risk use cases. |

The challenge type resolves in this order:

1. **Per-session override** — pass `challengeType` (`1` or `2`) in the `identifyUser()` call
2. **Per-client default** — set `LivenessChallengeType` on the `ClientApp` record
3. **Global default** — `FaceMovementAndLightChallenge` (`2`)

## Full OIDC compliance

Expose Entry as a standards-compliant OpenID Connect (OIDC) identity provider. This will allow Entry to be used as a drop-in IdP in any system that supports OIDC — including access control platforms, SSO solutions, and enterprise identity brokers — without requiring custom SDK integration. The verified identity will be returned as a signed ID token containing standard OIDC claims alongside any registered user metadata.

---

## Have feedback or a feature request?

Contact [support@synapser.com](mailto:support@synapser.com).
