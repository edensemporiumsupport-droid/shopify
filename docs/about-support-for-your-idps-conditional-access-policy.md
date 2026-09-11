---
title: About support for your IdP’s Conditional Access Policy
product: "GitHub Enterprise Cloud"
description: "How GitHub uses your identity provider's Conditional Access Policy (CAP) when you configure OIDC SSO for Enterprise Managed Users."
versions:
  - enterprise-cloud@latest
---

## Summary

When you configure OpenID Connect (OIDC) Single Sign-On (SSO) for Enterprise Managed Users, GitHub can evaluate and honor IP-based conditions from your identity provider's Conditional Access Policy (CAP). This page explains what GitHub enforces, current limitations, and recommended admin actions.

**Scope:** This capability is available for Enterprise Managed Users that use OIDC SSO with Microsoft Entra ID (formerly Azure AD). Some features (for example, web session protection) may be in public preview.

## Quick facts

- Supported IdP (current): Microsoft Entra ID (Azure AD).
- What GitHub enforces from the IdP: IP-based access conditions (allowed/blocked IP ranges or named locations).
- What GitHub does not enforce: device-compliance conditions (for example, Intune compliance) and client-side device posture.
- Preview features: web session protection (behavior may change while in preview).
- Non-user credentials such as deploy keys are not evaluated by CAP because they do not represent a user session.

## How CAP affects user access

- If a user’s sign-in satisfies the IdP’s IP conditions → the user gets normal access to the enterprise and resources.
- If a user’s sign-in does not satisfy IP conditions and web session protection is enabled:
  - The user can still list and filter resources they own.
  - The user cannot view item details in some areas (for example: notification details, search result details, dashboard item details, or starred repository details). This prevents access to sensitive content while allowing minimal navigation.
- Programmatic authentication (personal access tokens, SSH) is evaluated by the IdP’s CAP at sign-in; requests that fail CAP will be blocked by the IdP.

## IP allow lists vs IdP CAP

If you enable CAP-based web session protection via OIDC SSO, you can rely on your IdP’s configured named locations or IP allow lists instead of GitHub’s separate IP allow list. Confirm your IdP configuration covers the client IPs that should have access.

## Integration considerations

- Deploy keys and machine/service credentials:
  - Deploy keys (SSH keys tied to a repository) do not operate as a user and are not subject to IdP CAP rules.
  - Service principals or machine identities that authenticate outside of a user flow may not be covered; verify your IdP policies for those app types.
- Originating IP address:
  - GitHub forwards the originating client IP to the IdP for CAP evaluation. Ensure any intermediate proxies or load balancers preserve the correct client IP (for example, via X-Forwarded-For) and that your IdP policy is configured to evaluate the expected IP values.
- Personal access tokens (PATs):
  - PAT sign-ins are subject to CAP evaluation when the token is used to authenticate. If tokens are used from disallowed IPs, the IdP may deny access.

## Limitations and things to know

- Device compliance: GitHub cannot assert or enforce device-compliance conditions on the IdP’s behalf. Device posture checks (Intune, JAMF device signals, etc.) must be enforced and validated by the IdP.
- MFA enforcement: Multifactor authentication requirements are enforced by the IdP during sign-in; GitHub relies on the IdP to perform MFA.
- Public preview features: Web session protection behavior is subject to change while it remains in preview. Revisit this page after the feature reaches general availability.
- Audit trail: For denied sign-ins, consult both GitHub audit/event logs and your IdP’s sign-in logs to determine the reason and associated IP.

## Troubleshooting checklist

1. Reproduce and capture evidence:
   - Note the user, timestamp, and the originating IP shown in GitHub’s auth/audit event.
   - Confirm the same sign-in attempt in your IdP sign-in logs and inspect the CAP evaluation reason.
2. Verify IP propagation:
   - If you use proxies/load balancers, ensure they preserve and forward the client IP that your IdP expects.
3. Check policy targeting:
   - Confirm the CAP targets the GitHub enterprise app or OIDC client in Entra ID (or the app registration you use for OIDC).
4. Test with a known-good IP:
   - Temporarily allow your test IP in the IdP named locations and confirm access is restored.
5. Programmatic flow debugging:
   - For blocked programmatic requests, check whether the request’s IP is within allowed ranges, and whether the client uses the same network path as interactive users.
6. If deploy keys or service tokens fail unexpectedly:
   - Confirm they are not being treated as user sessions; if the IdP is blocking them as app sign-ins, adjust application policy allowances.

## Example: Azure conditional access considerations

- In Microsoft Entra ID, ensure you:
  - Target the Conditional Access Policy at the GitHub application (or OIDC app registration).
  - Configure “Named locations” or IP ranges to represent your allowed networks.
  - Review policy assignments so they apply to the right user groups and applications.
- For web session protection, pilot the policy with a small user group before enabling enterprise-wide.

## References

- Microsoft: Conditional Access in Microsoft Entra ID (see Microsoft docs for configuring named locations and app-targeted policies).
- GitHub Docs:
  - Configuring OIDC SSO for Enterprise Managed Users
  - About IP allow lists for your enterprise

## Admin checklist (quick)

- [ ] Configure OIDC SSO between GitHub and Microsoft Entra ID.
- [ ] Confirm GitHub app / OIDC client is targeted by the CAP.
- [ ] Add and validate Named locations / IP ranges in the IdP.
- [ ] Pilot web session protection with a small group.
- [ ] Monitor GitHub and IdP sign-in logs for blocked attempts.
