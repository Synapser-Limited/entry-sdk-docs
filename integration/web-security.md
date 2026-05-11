---
layout: default
title: Web Security & Compliance
nav_order: 3
---

# Web SDK — Security and Compliance Guide

## Security features

### End-to-end encryption

- AES-256-GCM encryption for all data transmission
- Dynamic key generation per session using PBKDF2
- Session-based authentication with daily key rotation
- Perfect forward secrecy with ephemeral session keys

### Biometric security

- AWS Rekognition liveness detection prevents photo/video spoofing (iBeta Level 2 certified)
- Template-based storage — no raw biometric data is ever stored
- Configurable confidence thresholds (default: 80%+ liveness, 95%+ face match)

### Additional protections

- Device fingerprinting using browser characteristics
- Comprehensive audit logging and event tracking
- Session management with timeout and validation
- Request tracing with unique session identifiers
- Domain and IP whitelisting

### Audit & compliance

- Complete audit trail
- Privacy-by-design architecture
- GDPR-ready features: user consent, data minimisation, deletion
- Configurable data retention policies

---

## Compliance considerations

### GDPR

| Requirement       | SDK support                                                        |
| ----------------- | ------------------------------------------------------------------ |
| User consent      | Consent mechanisms for biometric data collection                   |
| Data minimisation | Only essential data is collected and stored                        |
| Right to deletion | `deleteUser()` removes all user data including biometric templates |
| Privacy by design | No raw biometric data stored in browser or transmitted             |

### Healthcare (HIPAA)

- Audit logging for all authentication events
- Secure session management
- No PHI stored in the SDK itself
- Integrates with existing healthcare compliance systems

### Financial services (PCI DSS, SOC 2)

- Strong authentication for high-value transactions
- Comprehensive audit trails
- Session security and timeout management

### Age-restricted content (COPPA)

- Age verification via biometric-linked identity documents
- Automatic flagging for underage users
- Regional compliance support (16+ EU, 18+ for alcohol/gambling)

---

## Best practices

### Production deployment

- Use `EntryApiEnvironment.Live` in production
- Configure domain whitelisting with the Entry team
- Set up error monitoring (Sentry or equivalent)
- Enforce HTTPS for all pages that use the SDK
- Implement Content Security Policy (CSP) headers

### Data protection

- The SDK never stores raw biometric data in the browser
- Only encrypted templates are stored in the secure backend
- Browser data is cleared on session end
- Implement appropriate session timeouts and clear sensitive data on logout

### Audit logging

- Log all authentication attempts with error codes and timestamps
- Track success/failure rates over time
- Monitor for anomalous patterns (repeated failures, unusual devices)
- Retain logs per your compliance requirements

---

## Security testing

### Recommended test scenarios

| Area                  | Scenario                                                 |
| --------------------- | -------------------------------------------------------- |
| Liveness              | Attempt to authenticate using a photo or video recording |
| Similarity threshold  | Verify threshold enforcement rejects near-matches        |
| Device fingerprinting | Test with a different device or browser profile          |
| Session management    | Verify session expiration behaviour                      |
| Network security      | Test with network interruptions mid-flow                 |
| Data storage          | Verify no biometric data appears in browser storage      |
| Data deletion         | Call `deleteUser()` and confirm data is removed          |
| Domain whitelisting   | Attempt SDK calls from an unlisted domain                |

---

## Incident response guidance

1. **Detection** — Monitor error rates; alert on spikes in `LIVENESS_CHECK_FAILED` or unusual `FACE_MATCH_FAILED` patterns
2. **Response** — Review Sentry and CloudWatch audit logs; identify affected user IDs and sessions
3. **Containment** — Invalidate active sessions for affected users; tighten confidence thresholds temporarily if spoofing is suspected
4. **Recovery** — Reset affected sessions; update security configurations; notify affected users as required by your compliance obligations
5. **Post-incident** — Document the incident; update internal runbooks; implement preventive measures

Contact [support@synapser.com](mailto:support@synapser.com) for security-related incidents involving the Entry platform.
