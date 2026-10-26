# Google VRP Issue #522452897: Structured Ticket Analysis & Ledger

---

## Section 1: Original Ticket Description

[Verbatim Content]

A security boundary issue exists in how the gcloud identity token generation handles the target audience parameter. When requesting an identity token, the platform allows the operator to specify any external URL in the --audiences flag.

Because Google acts as a globally trusted root provider, it issues a cryptographically signed OIDC token explicitly matching whatever external target endpoint is requested. An attacker does not need to compromise an existing enterprise service account or hold high-level administrative permissions; they can execute this from a completely independent project created on a standard user account, or run it through an existing terminal session to utilize the active environment's network path and IP address. If an external cloud environment (such as AWS or Azure) or a deployment pipeline is configured to trust Google's root authority broadly, this token can be used to authenticate directly to those third-party networks from the outside.

---

## Section 2: Google's Initial Response

[Summary & Key Points]
* **Classification:** Closed as "Won't Fix / Intended Behavior" at P4/S4 priority.
* **Interoperability Defense:** Claimed the unconstrained audience flag is built intentionally to allow integration with external providers like AWS or Azure.
* **Third-Party Deflection:** Shifted the security burden, claiming the risk stems from downstream systems accepting tokens without validation.
* **Scope Dismissal:** Argued that requiring local terminal or CLI access makes the scenario out of scope under prior-compromise rules.

[Exact Text]

> "Hi! We've decided to close this bug. The behavior you reported, where gcloud auth print-identity-token allows specifying arbitrary external URLs in the --audiences flag to mint OIDC tokens, is working as intended. This functionality is designed for interoperability, enabling external cloud environments (like AWS or Azure) to integrate with Google's OIDC root authority if they choose to trust it. The vulnerability you described primarily relies on these external systems having broad trust in Google's OIDC tokens without sufficient additional validation on their end. Furthermore, your attack scenario requires the attacker to already have initial terminal access and gcloud CLI access on the target system. Our VRP generally considers scenarios requiring a prior system compromise to be out of scope."

---

## Section 3: Reporter Rebuttal & Escalation

[Summary & Key Points]
* **Automated Infrastructure Reality:** Completely dismantled the local-terminal assumption by proving the vector works seamlessly in CI/CD pipelines, containerized runners, and automated service accounts.
* **Policy & Documentation Gap:** Challenged the lack of guardrails, showing that without documented security boundaries, the feature acts as an unconstrained token-minting engine.
* **Architectural Breach:** Stressed that providing a primitive to generate valid signed root assertions for arbitrary external URLs destroys trust boundaries and enables cross-cloud lateral movement.

[Exact Text]

> "To: Google VRP / Security Team
> Subject: Request for Escalation: Issue #522452897
> 
> I am formally requesting this report be escalated to the Cloud IAM/OIDC engineering team for a technical risk assessment. The 'Intended Behavior' assessment is incorrect, as it narrowly assumes the attack vector requires local terminal access. This ignores the reality of modern cloud environments where this command is easily triggered via CI/CD pipelines, containerized runners, or service account exploitation. Please address:
> 
> 1. **Rebuttal of Prior Access:** The assessment incorrectly assumes the threat model is limited to a compromised terminal. This vector is entirely viable within automated infrastructure where limited service account access is sufficient to trigger token minting.
> 2. **Security Policy & Documentation Failure:** You classify the ability to mint tokens for arbitrary external audiences as 'intended interoperability.' If this is an official design feature, the platform lacks any documented security boundaries, usage guardrails, or explicit policy controls. Without validation or policy limits, this effectively functions as an unconstrained, platform-level token-minting engine. Is it truly 'intended' to allow the generation of signed root assertions for any arbitrary URL without structural verification?
> 3. **Architectural Risk:** Deflecting to third-party trust misses the point. By providing the primitive to generate valid-looking assertions for any external system, the platform violates core trust-boundary expectations and facilitates lateral movement across cloud perimeters.
> 
> I request a review by the OIDC architectural team to confirm if this design genuinely aligns with the platform's security model.
> 
> Regards,
> - Dustin McKay"

---

### Section 4: Final Status Update & Current State

[Summary & Current Status Breakdown]
* **Status & Priority Shift:** Following the escalation and policy push, Google reopened the ticket, changing priority from P4 to P3 and moving the status from "Intended Behavior" to "Assigned".
* **How They Left It Hanging:** Routed to an internal engineering contact (`wo...@google.com`), but left hanging without further updates or commentary from July 11, 2026 through October 2026—transitioning from an automated dismissal straight into an open-ended engineering assignment in a stagnant holding pattern.
