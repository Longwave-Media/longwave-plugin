# Security policy

## Reporting a vulnerability

Email **support@longwave.media** with `SECURITY` in the subject. If you would rather not use email,
open a [private security advisory](https://github.com/Longwave-Media/longwave-plugin/security/advisories/new)
on this repository.

Please include what you did, what you expected, and what happened — a reproduction is worth more than
a severity guess. We aim to acknowledge within **2 business days** and to give you a fix timeline
within **5**.

Please do not open a public issue for anything exploitable, and please do not test against another
creator's account. Use your own.

## Scope

**In scope**

- The MCP endpoint at `https://www.longwave.media/api/mcp` — authentication, scope enforcement,
  step-up, tool argument handling, and any way to reach data belonging to another Longwave customer.
- The OAuth 2.1 authorization server — `/api/oauth/authorize`, `/api/oauth/token`,
  `/api/oauth/register`, and the `.well-known` discovery documents.
- This package — a manifest, an MCP config, a skill file and a logo. There is no executable code in
  it, and **there are no credentials in it, by design.**

**Out of scope**

- The hosted dashboard and marketing site (report those to the same address; they are handled under
  the same policy, but they are not this repository).
- Rate limiting or denial of service by volume. We rate-limit at 1000 requests/hour per credential;
  please do not test the ceiling against production.
- Anything requiring a compromised creator account, or physical access to a device.
- Findings that need a deprecated browser or an unpatchable dependency with no demonstrated impact.

## What we consider a serious finding

A way for one Longwave customer's credential to reach another customer's videos, channels or
credentials. Cross-tenant reads and writes are the highest-severity class here — we have shipped a
public incident report for one before, and we would rather do that again than argue about it.

## Safe harbour

We will not pursue legal action against anyone who reports a vulnerability in good faith, stays
within the scope above, avoids privacy violations and service disruption, and gives us reasonable
time to fix it before disclosing publicly. We will credit you in the fix commit and in the incident
report if you would like.

## Design notes for reviewers

- **No secret is ever embedded in this package.** Agent Plugins v1.0.0 §7.2.1 forbids credentials in
  MCP `headers`, and OAuth discovery is how the spec expects authorization to work instead.
- **Google credentials never leave Longwave's servers.** An agent receives an opaque, scoped Longwave
  token and can only ask Longwave to act — it never holds anything Google-issued.
- **The default grant is read-only plus job creation.** Publishing to a live channel and changing a
  published thumbnail are separate permissions the creator grants deliberately, because both change
  something public.
- **The tool catalogue is not scope-filtered**, deliberately: a client cannot attempt a tool it cannot
  see, so filtering it made the step-up unreachable. Enforcement is the scope check on the call.
