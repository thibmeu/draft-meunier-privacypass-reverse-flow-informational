---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "Privacy Pass Reverse Flow"
abbrev: "Privacy Pass Reverse Flow"
category: info

docname: draft-meunier-privacypass-feedback-loop-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: AREA
workgroup: WG Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: Thibault Meunier
    organization: Cloudflare Inc.
    email: ot-ietf@thibault.uk

normative:

informative:


--- abstract

This document specifies an instantiation of Privacy Pass Architecture {{RFC9576}}
that allows for a reverse flow from the Origin to the Client/Attester/Issuer.
It describes the conceptual model of Privacy Pass reverse flow and its protocols,
its security and privacy goals, practical deployment models, and recommendations
for each deployment model, to help ensure that the desired security and privacy
goals are fulfilled.

--- middle

# Introduction

This document specifies an instantiation of Privacy Pass Architecture {{RFC9576}}
that allows for a reverse flow from the Origin to the Client/Attester/Issuer.
In other words, it specifies a way for the Origin to act as a joint Attester/Issuer.




# Terminology

{::boilerplate bcp14-tagged}

We reuse terminology from RFC9576: https://datatracker.ietf.org/doc/html/rfc9576#name-terminology

New terminology is defined below

Flow: Direction from PrivateToken issuance to its redemption. The entity starting the flow acts as an Issuer, while the end of the flow acts as an Origin. The Client is always included, as it finalises the TokenResponse, and coordinate interactions.
Initial Flow: Issuer -> Attester -> Client -> Origin. This flow produces a PrivateToken that is used by the Origin to kickstart a Reverse Flow.
Reverse Flow: Issuer <- Attester <- Client <- Origin. This flow allows Origin to issues PrivateToken. In the reverse flow, the Origin operates one or more Issuer, and the Client MAY provide these tokens either to the Initial Attester/Issuer, or use them against the Origin 
Initial Attester/Issuer: Attester/Issuer part of the Initial Flow
Origin Issuer: Issuer operated by the Origin
Origin PrivateToken: PrivateToken issued by the Origin
Reverse Origin: An entity that consumes the Origin PrivateToken. It can be the Origin, or the Initial Attester/Issuer

# Deployment

Along with sending their PrivateToken for authentication (as specified in RFC9576), Client
sends TokenRequest

+---------------+    +--------+                                       +--------+         +----------+ +--------+
| Origin Issuer |    | Origin |                                       | Client |         | Attester | | Issuer |
+---+-----------+    +---+----+                                       +---+----+         +----+-----+ +---+----+
    |                    |                                                |                   |           |
    |                    |<----- Request ---------------------------------+                   |           |
    |                    +-- TokenChallenge (Issuer) -------------------->|                   |           |
    |                    |                                                |<== Attestation ==>|           |
    |                    |                                                |                   |           |
    |                    |                                                +--------- TokenRequest ------->|
    |                    |                                                |<-------- TokenResponse -------+
    |                    |<-- Request+Token+TokenRequest(Origin Issuer) --+                   |           |
    |<-- TokenRequest ---|                                                |                   |           |
    |-- TokenResponse -->|                                                |                   |           |
    |                    |--- Response+TokenResponse(Origin Issuer) ----->+                   |           |
    |                    |                                                |                   |           |

TODO:
1. How are TokenRequest provided?
2. How are TokenResponse provided?
3. What are the error codes
< binary HTTP sounds like an overkill, but fits.

## Client behaviour

Along with sending PrivateToken from the Initial Issuer to the Origin, the Client sends one or more TokenRequest as defined in RFC9578 or draft-batched-tokens.

## Origin/Issuer/Attester deployment

VOPRF recommended.
Origin is the Reverse Origin.

TODO: Add diagram

## Split Origin-Attester deployment

Uses BlindRSA, or SHOULD NOT VOPRF with a shared secret.
Origin is the Initial Attester/Issuer.

TODO: add diagram

## Arbitrary batched tokens

Allows for more than one token to be provided.

# Privacy Considerations

Privacy Pass RFC 9576 states

    In general, limiting the amount of metadata permitted helps limit the extent to which metadata can uniquely identify individual Clients. Failure to bound the number of possible metadata values can therefore lead to a reduction in Client privacy. Most token types do not admit any metadata, so this bound is implicitly enforced.

In privacy pass with a reverse flow, users are provided with new PrivateTokens depending on their request. They can spend these tokens to continue making further requests.

While the token are still unlinkable, the token_key_id associated to them represent metadata. It leaks some information about the Client. The following subsections discuss the issues that influence the anonymity set, and possible mitigation/safeguards to protect against this underlying problem.

## Issuer face values

When setting up a reverse flow deployment, an Origin MAY setup up multiple Issuers, and assign them some metadata to them. The amount of possible metadata grows as 2^(origin_issuers).

We RECOMMEND that:
    1. Origin defines their anonimity sets, and deploy no more than log2(#anonimity_sets). This bounds the possible anonimity sets by design.
    2. Client to only send 1 PrivateToken per request. This is inline with RFC9577 and RFC (Web Authentication) which only allows one challenge response to be provided as part of Authorization HTTP header.
    3. Issuers metadata to be publicly disclosed via an origin endpoint, and externally monitored

## Token for specific Clients

In Privacy Pass with a reverse flow, the Origin MAY operate multiple Issuers, with arbitrary metadata associated to them. A malicious Origin MAY uses this opportunity to associate certain token values to a specific set of Clients.

Let's consider the following deployment: the Origin operates two issuers A and B. The Client sends Token_A, and [TokenRequest_A, TokenRequest_B]. Issuer B is associated to a croissant aficionados.

If a Client requests croissant, or sends Token_B, the origin will provide TokenResponse_B. If not, it provides TokenResponse_A.

Over time, this means the Origin is able to track croissants aficionados.

To mitigate this, we recommend:
    1. The initial PrivateToken to be provided by an Issuer not in control of the Origin. The joint Origin/Attester/Issuer model SHOULD NOT be used.
    2. Clients to reset their state regularly with the initial Issuer.

## Sending multiple tokens

While that's not part of Privacy Pass with a reverse flow, some deployment might consider allowing Clients to send multiple PrivateToken, similar to how normal Privacy Pass deployment could allow two distinct PrivateToken to be sent.

In Privacy Pass with a reverse flow deployment, there are as many bits as Issuers; each token is one bit. We RECOMMEND to have a maximum of 6 Origin operated Issuers, bounding Client information to 2^6 = 64. Accounting for the initial Issuer, this means a total of log2(64)+1=7 issuers.

Origin should have sufficient traffic to not single-out particular Client based on timings of requests.

## Swap endpoint and its privacy implication

With multiple Issuers, a Client MAY end up with a bunch of tokens, for various Issuers. Origins MIGHT propose a swap endpoint at which a Client can use to exchange one or more Origin tokens against one or more Origin token.

The Origin SHOULD ensure the traffic is important enough on this endpoint to not reduce the anonymity sets.



# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
