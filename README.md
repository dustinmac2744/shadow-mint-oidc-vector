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
* **Status:** Deprioritized via automated triage / Marked "Infeasible" / "Intended Behavior"
* **Summary:** The report detailed the potential for arbitrary audience minting and cross-domain token execution. Institutional response shifted the issue into automated processing queues without addressing the root reliance model or issuing remediation guidance for downstream relying parties.

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

## 6. GOOGLE VRP TRIAGE FAILURE & RESEARCH SUPPRESSION

Google’s Vulnerability Reward Program (VRP) increasingly functions as an asymmetric bureaucratic barrier designed to deny researcher credit, avoid bounty payouts, and quietly cannibalize external security research.

### Institutional Exploitation Patterns

* **Automated Rejection as First-Line Defense:** Issue Tracker `#522452897` was summarily dismissed using canned "intended behavior" rationale without technical review by identity infrastructure engineers.
* **Silent Assimilation & Credit Erasure:** Valid structural vectors submitted through official channels are walled off as non-issues, only for the underlying infrastructure and threat models to be silently updated, patched, or integrated into Google threat intelligence publications—completely stripping attribution from the researcher who uncovered the flaw.
* **Asymmetric Exploitation of Independent Researchers:** Google leverages crowdsourced security research as a zero-cost, zero-accountability auditing tool. Researchers invest technical labor to map architectural vulnerabilities, while Google uses automated triage gates to retain full control over public disclosure, credit, and remediation timelines behind closed doors.
