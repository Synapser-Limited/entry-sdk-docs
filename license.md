---
layout: default
title: License
nav_order: 11
---

# Proprietary Software License

**Entry Web SDK**  
Copyright © 2025 Synapser Limited. All rights reserved.

---

## BACKGROUND AND PURPOSE

The Entry Web SDK (the "Software") is a production-ready TypeScript/React library developed and owned by Synapser Limited ("Synapser", "we", "us" or "our"). The Software provides secure biometric authentication capabilities using AWS services (Amplify, Cognito, and Rekognition). It enables client applications to integrate facial recognition and active liveness detection for user identification and registration.

**Key SDK Capabilities:**

- Biometric user registration and identification workflows
- Active liveness detection to prevent spoofing (photo, video, mask, or other presentation attacks)
- Integration with AWS Rekognition for configurable confidence thresholds (default: 80% liveness, 95% match)
- End-to-end AES-256-GCM encryption for data in transit
- No raw biometric data storage – only mathematical face templates are processed and matched

**Distribution Model:**

`#TODO: ASK DR ABOUT THIS PARAGRAPH. I THINK THE NPM PACKAGE SHOULD BE PUBLIC.`
The Software is distributed exclusively via GitHub Packages under the scoped package name `@synapser-limited/entry-web-sdk` to authorized customers, partners, or organizations that have executed a separate commercial agreement or subscription order with Synapser. Access requires a valid GitHub Personal Access Token with `read:packages` scope and is restricted to entities approved by Synapser.

- **Package Registry:** <https://github.com/Synapser-Limited/entry-web-sdk/pkgs/npm/entry-web-sdk>
- **Installation Documentation:** <https://synapser-limited.github.io/entry-web-sdk-docs/>

---

## IMPORTANT NOTICE

This Software and associated documentation files (collectively, the "Software") are the proprietary and confidential property of Synapser Limited. Unauthorized use, reproduction, or distribution is strictly prohibited and may result in civil and criminal liability.

## GRANT OF LICENSE

Subject to your compliance with this License Agreement and any separate commercial agreement or subscription order executed between you (the "Licensee") and Synapser (the "Order"), Synapser grants you a limited, non-exclusive, non-transferable, non-sublicensable license during the term of the Order to:

(a) install and use the Software solely in object code form as an integrated component within your internal or customer-facing applications (the "Licensee Application");

(b) make a reasonable number of copies of the Software solely for backup, archival, and testing purposes; and

(c) use the Software for development, testing, and production deployment of the Licensee Application, provided such use is consistent with the documentation and the permitted scope set out in the Order.

This license is granted only to the specific entity named in the Order and only for the number of installations, users, or other metrics expressly authorized therein. Any use beyond the scope of the Order requires a separate written agreement.

## RESTRICTIONS

You may NOT, without Synapser's prior written authorization:

- Copy (except as expressly permitted above), modify, adapt, translate, create derivative works of, reverse engineer, decompile, disassemble, or otherwise attempt to discover the source code of the Software;
- Distribute, sublicense, lease, rent, loan, sell, or otherwise transfer or make available the Software (or any portion thereof) to any third party;
- Remove, obscure, or alter any proprietary notices, labels, trademarks, or copyright statements on or in the Software;
- Use the Software in any manner that violates applicable export control laws, sanctions regimes, or biometric data processing regulations (including POPIA in South Africa, GDPR in the EU/UK, and applicable US state biometric privacy laws);
- Use the Software for any unlawful purpose, to infringe third-party rights, or in any application where failure of the Software could reasonably be expected to result in death, personal injury, or severe property or environmental damage ("High-Risk Use");
- Circumvent, disable, or otherwise interfere with security-related features of the Software, including liveness detection thresholds or encryption mechanisms.

## OWNERSHIP

Synapser retains all right, title, and interest in and to the Software, including all intellectual property rights. This license does not constitute a sale of the Software or any portion thereof. All rights not expressly granted are reserved.

## SUPPORT AND MAINTENANCE

Support and maintenance (including bug fixes, updates, and new versions) are provided only under a separate support agreement or as expressly stated in the Order. Synapser has no obligation to provide support for any modified or unauthorized use of the Software.

## WARRANTY DISCLAIMER

THE SOFTWARE IS PROVIDED "AS IS" AND "AS AVAILABLE", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, TITLE, NON-INFRINGEMENT, OR THAT THE SOFTWARE WILL BE ERROR-FREE, SECURE, OR UNINTERRUPTED. SYNAPSER DOES NOT WARRANT THAT THE SOFTWARE WILL MEET LICENSEE'S REQUIREMENTS OR THAT ANY BIOMETRIC MATCHING OR LIVENESS DETECTION WILL BE 100% ACCURATE OR ERROR-FREE. LICENSEE ASSUMES ALL RISK ARISING FROM USE OF THE SOFTWARE, INCLUDING RISK OF INACCURATE IDENTITY VERIFICATION.

## LIMITATION OF LIABILITY

TO THE MAXIMUM EXTENT PERMITTED BY LAW, IN NO EVENT SHALL SYNAPSER OR ITS AFFILIATES, LICENSORS, OR SUPPLIERS BE LIABLE FOR ANY INDIRECT, INCIDENTAL, SPECIAL, CONSEQUENTIAL, PUNITIVE, OR EXEMPLARY DAMAGES (INCLUDING LOSS OF PROFITS, DATA, GOODWILL, BUSINESS INTERRUPTION, OR SECURITY BREACHES), WHETHER ARISING IN CONTRACT, TORT (INCLUDING NEGLIGENCE), STRICT LIABILITY, OR OTHERWISE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGES. SYNAPSER'S TOTAL AGGREGATE LIABILITY ARISING OUT OF OR RELATED TO THIS LICENSE SHALL NOT EXCEED THE FEES PAID BY LICENSEE TO SYNAPSER UNDER THE APPLICABLE ORDER IN THE TWELVE (12) MONTH PERIOD IMMEDIATELY PRECEDING THE EVENT GIVING RISE TO THE CLAIM.

## TERM AND TERMINATION

This license commences on the earlier of (i) your first installation or use of the Software or (ii) the effective date of the Order and continues for the term specified in the Order (or until terminated earlier). Synapser may terminate this license immediately upon written notice if Licensee breaches any material provision and fails to cure within thirty (30) days (or immediately in case of breach of restrictions or High-Risk Use). Upon termination or expiry, Licensee must cease all use of the Software, delete or destroy all copies, and certify such destruction upon request.

## GOVERNING LAW

This License Agreement shall be governed by and construed in accordance with the laws of the Republic of South Africa, without regard to its conflict of laws principles. The parties submit to the non-exclusive jurisdiction of the High Court of South Africa (Western Cape Division, Cape Town) for any dispute arising out of or in connection with this Agreement. Notwithstanding the foregoing, Synapser may seek injunctive or equitable relief in any court of competent jurisdiction.

## CONTACT INFORMATION

Synapser Limited  
132 Haygarth Road, Kloof, 3610  
Email: <support@synapser.com>  
Website: <https://www.synapser.com/entry-mfa>

---

**For licensing inquiries, please contact:** <support@synapser.com>

---

## ACCEPTANCE

By installing, copying, accessing, or using the Software, you acknowledge that you have read, understood, and agree to be bound by this License Agreement. If you do not agree, do not install or use the Software.

---

**Version:** 1.0 – February 2026
