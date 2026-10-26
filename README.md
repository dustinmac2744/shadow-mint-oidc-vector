# SHADOW-MINT: Unconstrained OIDC Target Audience Minting & VRP Disclosure Analysis

## OVERVIEW

This advisory details a structural identity boundary vulnerability in cloud CLI identity token generation routines (`gcloud`), alongside a documented case study of automated VRP triage workflows suppressing architectural findings.

---

## 1. TECHNICAL PRIMITIVE

By design, local credential generation tools allow clients to pass arbitrary external parameters during OpenID Connect (OIDC) identity token requests:

    gcloud auth print-identity-token --audiences="https://sts.amazonaws.com"

### Protocol Path

1. **Request:** The agent requests a signed assertion with an unconstrained target audience (`aud`).
2. **Signing:** The root Identity Provider issues a cryptographically valid JSON Web Token (JWT) signed by official Google public keys (`https://www.googleapis.com/oauth2/v3/certs`).
3. **Implicit Trust Failure:** Downstream systems (e.g., AWS STS, Azure AD, custom relying parties) that validate only issuer (`iss`) signatures without strict audience scoping permit cross-cloud lateral identity execution.

---

## 2. VRP TRIAGE & DISCLOSURE RECORD

* **Ticket Reference:** Google Issue Tracker `#522452897`
* **Title:** Living off the Land Identity Hijack: Access and Identity Token Minting for Secret Key Extraction and Critical Cloud Infrastructure Cross-Domain Execution
* **Initial Status:** Closed via automated triage as `Won't Fix (Intended Behavior)` (Priority P4 / Severity S4)
* **Escalation & Override Status:** Reopened and Assigned to Google Identity Engineering (`wo...@google.com`) — Priority escalated to `P3`
* **Summary:** The report details arbitrary OIDC audience minting via `gcloud` permitting cross-domain token execution. While initial automated triage dismissed the primitive as intended interoperability requiring local terminal access, formal escalation forced a manual override and assignment to human security staff.

---

## 3. IMPLICIT TRUST MECHANICS & LATERAL BOUNDARY RISKS

Downstream relying parties often configure federated identity providers (IdPs) by specifying only the issuer URL (`https://accounts.google.com`). When a relying party does not enforce strict validation of the target audience field (`aud`), an attacker with valid CLI execution privileges can mint an OIDC token scoped to external audiences.

    +------------------+          +--------------------+          +--------------------+
    | Local Attacker / |  (1)     | Google OAuth /     |  (2)     | Target Downstream  |
    | CLI execution    | -------->| OIDC Identity Provider| -------->| Relying Party / STS |
    | (gcloud auth)    | Token    | (Cryptographic Sig)| Valid    | (Assumes Identity) |
    +------------------+ Request  +--------------------+ Token    +--------------------+

### Risk Surface Vectors
* **Cross-Cloud Identity Spoofing:** Passing custom STS endpoints to obtain tokens accepted by third-party cloud configurations.
* **Service Account Impersonation:** Leveraging local credential tokens to assert identity across unverified federated trust boundaries.

---

## 4. ARCHITECTURAL REMEDIATION & DETECTION

### Detection Strategy
Monitor cloud environment logging for anomalous `gcloud auth print-identity-token` execution or OIDC token requests containing unexpected audience targets (`--audiences`).

### Recommended Mitigation for Relying Parties
* **Strict Audience Validation:** Always enforce explicit audience check logic in downstream webhooks, API gateways, and IAM role trust relationships.
* **Issuer & Audience Pair Binding:** Reject any token where `aud` does not explicitly match the specific service endpoint being accessed.

---

## 5. CONCLUSION & POLICY IMPLICATIONS

The ability to mint arbitrarily scoped OIDC assertions via standard CLI tools presents an architectural challenge for federated trust boundaries. When institutional triage mechanisms automatically dismiss structural vulnerability reports as "intended behavior" without publishing security guidance, downstream consumers remain exposed to silent cross-cloud lateral movement risks.

---

## 6. GOOGLE VRP TRIAGE MECHANICS & ESCALATION ANALYSIS

The operational record of Issue Tracker `#522452897` demonstrates the friction point between automated VRP triage filters and valid architectural disclosure[span_7](start_span)[span_7](end_span)[span_8](start_span)[span_8](end_span).

### Verified Case Timeline
* **Automated Dismissal (Jun 10, 2026):** Automated triage closed the report as "Intended Behavior," citing third-party trust requirements and prior local terminal access[span_9](start_span)[span_9](end_span).
* **Formal Technical Escalation (Jun 10, 2026):** Rebuttal submitted highlighting CI/CD pipeline execution, containerized runner exposure, and unconstrained token-minting risks[span_10](start_span)[span_10](end_span).
* **Institutional Override (Jun 11, 2026):** Ticket status flipped from `Won't Fix` to `Assigned`, priority escalated from `P4` to `P3`, and routed directly to a Google engineer (`wo...@google.com`)[span_11](start_span)[span_11](end_span).

### Takeaways for Independent Security Research
* **Automated Triage Wall:** Standard VRP front-ends use canned "prior access" templates to drop architectural reports before reach of platform engineers[span_12](start_span)[span_12](end_span).
* **Forced Human Escalation:** Technical breakdowns challenging the narrow definition of terminal execution can force manual reopening and priority escalation[span_13](start_span)[span_13](end_span).
