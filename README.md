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
