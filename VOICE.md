# Unlayer Voice — Architecture, Engineering Contract & Roadmap

Unlayer Voice is the Bun-first telephony platform. It owns SIP signalling, registrations, call control, routing, trunks, numbers, CDRs, media coordination, billing events and programmable voice APIs.

## Boundaries

Identity owns canonical users, organizations, SIP credentials and authorization. Voice asks Identity whether a SIP identity is authentic and authorized; Voice owns the call.

Database owns durable Voice state. HA owns generic node coordination, quorum, authority and fencing. Voice must not reimplement either.

## Architecture

```
SIP endpoint -> Voice signalling -> Identity authz
                         |
                         +-> routing -> endpoint/trunk
                         |
                         +-> call/media state
                         |
                         +-> Database
                         |
                         +-> HA
```

SIP signalling and RTP/media are separate planes. RTP packets must never depend on a database or Identity round trip.

## Identity contract

Voice consumes a service contract equivalent to:

```ts
interface SipIdentityProvider {
  challenge(request: SipAuthRequest): Promise<SipChallenge>
  authenticate(request: SipAuthRequest): Promise<SipAuthenticationResult>
  authorizeRegistration(request: RegistrationAuthorizationRequest): Promise<AuthorizationResult>
  authorizeCall(request: CallAuthorizationRequest): Promise<AuthorizationResult>
}
```

Voice must never read Identity's private tables directly.

SIP credentials are distinct from web credentials. Authorization results may be cached briefly, but revocation and policy propagation must have a defined bound.

## SIP

Initial protocol:
REGISTER, INVITE, ACK, BYE, CANCEL, OPTIONS, provisional/final responses, transactions, dialogs and SDP boundaries.

Authentication target:
realm, nonce, qop, replay protection, SHA-256 and SHA-512/256, with strict malformed-input handling.

## Registration

A registration records AOR, contact, endpoint, transport, source metadata, expiry, capabilities and authorization context. Identity remains the credential authority; Voice owns operational registration state.

## Calls

Lifecycle:

```
received -> authenticated -> authorized -> routing -> dialing
-> ringing -> answered -> established -> terminating -> completed/failed
```

A call has a stable ID, tenant scope, caller, destination, route, trunk, timestamps, termination cause, media metadata and billing metadata.

## Routing

Support E.164 normalization, exact/prefix matching, extensions, users, organization dial plans, route priority/weight, trunk selection, failover, time conditions and caller/destination policy. Selection must be deterministic for identical state.

## Numbers and trunks

Numbers are operational telephony resources linked to platform ownership/authorization. Trunks contain provider, targets, transport, credentials, codecs, capacity, health and failover policy. Secrets are never returned by normal APIs or logged.

## Media

Use Bun primitives first, then narrow FFI boundaries where native media functionality is genuinely required. Media may include RTP, RTCP, SRTP, jitter, sequencing, codecs, transcoding and recording. WebRTC is a later boundary.

## HA

Voice consumes `unlayer/ha`. Do not make every SIP request leader-only. SIP signalling and media should support active-active operation where safe; authoritative configuration and selected coordination operations may require a leader/term. Fencing must prevent stale nodes from mutating authoritative state.

## Database

Persist configuration, numbers, routes, trunks, CDRs, billing events and required registration state. High-frequency ephemeral packet state stays in the media/signalling plane.

## APIs

Canonical public boundary: `api.unlayer.network/v1/voice`.

Resources: calls, registrations, numbers, extensions, routes, dial-plans, trunks, recordings, CDRs, policies and webhooks.

## Security and fraud

Protect against SIP scanning, credential stuffing, replay, registration hijacking, toll fraud, tenant breakout and resource exhaustion. Include destination restrictions, spend/concurrency limits, international/premium-rate policy, rate limiting, service identity and auditability.

## Testing

Required: parser, transaction, digest, registration, authorization, routing, trunk, call-state, media, malformed-input, tenant isolation, HA, chaos, load and soak tests.

## First milestone

Bun Voice -> SIP REGISTER -> Identity Digest authentication -> registration -> INVITE -> authorization -> local endpoint call -> BYE -> CDR.

Then trunk calls, media, HA failure, billing and production hardening.
