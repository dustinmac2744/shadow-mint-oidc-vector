# SHADOW-MINT: Unconstrained OIDC Audience Minting & Disclosure Analysis

## Overview

This advisory documents a trust-boundary issue in cloud CLI identity token generation. Local tools such as `gcloud` can mint OpenID Connect (OIDC) identity tokens with an arbitrary external audience (`aud`) value. Although the JWT is cryptographically valid and signed by a trusted identity provider, downstream systems may accept it if they validate only the issuer and not the audience.

This creates a cross-cloud identity issue: a token meant for one service may be repurposed for another if the relying party does not enforce strict audience binding.

---

## Technical Primitive

Example command:
```
gcloud auth print-identity-token --audiences="[https://sts.amazonaws.com](https://sts.amazonaws.com)"
```

This requests a signed JWT from Google's identity infrastructure, with the caller-specified target audience set to an external service.

A representative token structure looks like:
```
{
  "header": {
    "alg": "RS256",
    "kid": "official_google_key_id",
    "typ": "JWT"
  },
  "payload": {
    "iss": "[https://accounts.google.com](https://accounts.google.com)",
    "aud": "[https://sts.amazonaws.com](https://sts.amazonaws.com)",
    "sub": "109876543210987654321",
    "email": "operator@domain.com",
    "azp": "32555940559.apps.googleusercontent.com",
    "iat": 1700000000,
    "exp": 1700003600
  }
}
```

The key issue is not that the token is invalid; it is validly signed. The problem is that relying parties may assume issuer trust alone is enough, even when the audience is mismatched or unconstrained.

---

## Risk Surface

This creates several risk vectors:

- Cross-cloud identity spoofing
- Service account impersonation across federated trust boundaries
- Reuse of valid tokens outside their intended service context

If a relying party checks only the issuer and signature, but not the exact audience or service binding, the token can be abused across trust domains.

---

## Disclosure Case Study

The issue was reported under Google VRP as ticket `#522452897`.

Timeline:

- Submission: June 10, 2026
- Initial closure: automated triage marked it as "Won't Fix (Intended Behavior)" at P4 / S4
- Escalation: the report was rebutted and clarified with CI/CD and automation context
- Reopened: status changed to "Assigned," and priority was escalated to P3

This demonstrates how architectural identity-boundary issues can be dismissed when triage frames them as a local terminal-access problem rather than a trust-policy problem.

---

## Remediation

Relying parties should:

- enforce strict audience validation
- reject tokens whose `aud` does not exactly match the expected service
- bind issuer and audience to the specific relying party context
- validate service scope and token expiry consistently

The secure model is not "trusted issuer = trusted identity everywhere," but "trusted issuer + exact expected audience + correct service binding."

---

## Conclusion

This advisory illustrates a structural identity boundary vulnerability: valid identity tokens can be minted for arbitrary endpoints, and the risk emerges when downstream systems trust the issuer without enforcing a correct audience contract. The issue is not only about local CLI execution; it is about whether federated trust systems enforce the correct service boundaries.

This document covers the technical primitive and the public disclosure timeline. The separate VRP triage failure note remains a standalone analysis of the triage and escalation process.
