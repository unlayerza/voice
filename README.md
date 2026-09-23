# Unlayer Voice

Bun-first programmable telephony infrastructure for Unlayer.

Voice owns SIP signalling, registrations, call control, routing, numbers, trunks, media coordination, CDRs, billing events and voice APIs.

## Architecture

- **Identity** owns users, organizations, SIP credentials and authorization.
- **Voice** owns calls, SIP state, routing, trunks and media.
- **Database** owns durable Voice state.
- **HA** owns generic cluster coordination, quorum, authority and fencing.

See [VOICE.md](VOICE.md) for the engineering contract and [todo/](todo/) for the implementation roadmap.

## First milestone

SIP REGISTER -> Identity Digest authentication -> registration -> INVITE -> authorization -> local call -> BYE -> CDR.
