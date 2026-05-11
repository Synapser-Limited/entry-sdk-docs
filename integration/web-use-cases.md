---
layout: default
title: Web Use Cases
nav_order: 5
parent: Integration Guides
---

# Web SDK — Use Cases and Implementation Patterns

Common scenarios and code patterns for integrating the Entry Web SDK.

## Primary use cases

- **Secure user onboarding** — Replace password-based registration with biometric enrollment
- **Passwordless authentication** — Frictionless login via face liveness
- **Identity verification** — Step-up verification for high-security transactions

---

## Industry scenarios

### 1. Account recovery

**Problem:** Users forget passwords or lose access to MFA devices.

**Pattern:** Biometric-based recovery lets users regain account access via facial verification without email or SMS reset links, reducing phishing risk.

**Integration points:**
- Template-based biometric storage for secure identity verification
- Configurable confidence thresholds for high-security recovery flows

### 2. Multi-device authentication

**Problem:** Users switching between devices (desktop → mobile) need seamless re-authentication.

**Pattern:** Session-based auth with device fingerprinting allows cross-device session transfer via biometric re-authentication.

**Integration points:**
- Session keys with daily rotation
- Device fingerprinting for cross-device security validation

### 3. High-compliance environments (Healthcare / Finance)

**Problem:** Regulated sectors require strong KYC for sensitive data access.

**Pattern:** Healthcare portals and financial services verify identity against stored biometric templates before granting access to records or approving transactions.

**Integration points:**
- iBeta Level 2 certified liveness detection
- Comprehensive audit logging
- Template-based verification (no raw biometric data stored)

### 4. E-commerce fraud prevention

**Problem:** High-value transactions from new devices are high-risk.

**Pattern:** Prompt biometric verification for purchases above a threshold or from unrecognised devices.

**Integration points:**
- Device anomaly detection via fingerprinting
- Configurable similarity thresholds per transaction value
- Integration with existing fraud detection systems

### 5. Remote workforce access

**Problem:** Enterprise tools need secure passwordless access from varied locations.

**Pattern:** Facial verification gates VPN or collaboration tool access, with IP whitelisting and key rotation.

**Integration points:**
- Domain and IP whitelisting
- Dynamic key generation with daily rotation

### 6. Age verification for restricted content

**Problem:** COPPA, gambling, or alcohol regulations require age verification.

**Pattern:** Cross-reference biometric template with stored identity document data to verify user age.

**Integration points:**
- Integration with `photoIdentityDocumentType` / `photoIdentityDocumentNumber` fields in `EntryUser`
- Liveness detection to prevent document spoofing
- Automatic flagging for users below the required age threshold
- Audit trail generation for regulatory compliance

---

## Implementation patterns

### High-security transaction verification

```typescript
async function verifyHighValueTransaction(transactionAmount: number) {
  if (transactionAmount <= THRESHOLD_AMOUNT) {
    return { verified: false, reason: 'verification_required' };
  }

  try {
    const container = document.getElementById('verification-overlay');
    const user = await entrySDK.identifyUser(false, container);

    if (user.isRegistrationComplete) {
      return {
        verified: true,
        userId: user.entryUserId,
        auditTrail: generateAuditLog(user, 'transaction_verification'),
      };
    }

    return { verified: false, reason: 'incomplete_registration' };
  } catch (error) {
    return handleVerificationError(error);
  }
}
```

### Cross-device session synchronisation

```typescript
async function synchroniseDeviceSession(deviceId: string) {
  try {
    const container = document.getElementById('sync-overlay');
    const user = await entrySDK.identifyUser(false, container);

    if (user.deviceId !== deviceId) {
      return await handleDeviceMismatch(user, deviceId);
    }

    return {
      sessionId: generateSessionId(),
      user,
      deviceVerified: true,
    };
  } catch (error) {
    return await fallbackAuthentication(error);
  }
}
```

### Age-gated content access

```typescript
const MIN_AGE = 18;

async function accessAgeRestrictedContent(container: HTMLElement) {
  const user = await entrySDK.identifyUser(true, container);

  const dob = new Date(user.dateOfBirth);
  const age = Math.floor((Date.now() - dob.getTime()) / (1000 * 60 * 60 * 24 * 365.25));

  if (age < MIN_AGE) {
    throw new Error('Age requirement not met');
  }

  return user;
}
```
