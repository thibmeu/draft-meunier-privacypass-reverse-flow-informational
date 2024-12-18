---
title: "Privacy Pass Reverse Flow"
abbrev: "Privacy Pass Reverse Flow"
category: info

docname: draft-meunier-privacypass-reverse-flow-informational-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Privacy Pass"
venue:
  group: "Privacy Pass"
  type: "Working Group"
  mail: "privacy-pass@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/privacy-pass/"
  github: "thibmeu/draft-meunier-privacypass-reverse-flow-informational"
  latest: "https://thibmeu.github.io/draft-meunier-privacypass-reverse-flow-informational/draft-meunier-privacypass-reverse-flow-informational.html"

author:
 -
    fullname: Thibault Meunier
    organization: Cloudflare Inc.
    email: ot-ietf@thibault.uk

normative:

informative:


--- abstract

This document specifies an instantiation of Privacy Pass Architecture {{!RFC9576=I-D.ietf-privacypass-architecture}}
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
A Client that has already been authorised to access an Origin can maintain that access,
without losing the unlinkable property provided by Privacy Pass. In addition, it allows
an Origin to define its own issuance policy based on an initial bootstraping attestation
method. For instance, an Origin that wants to grant 30 access for users that solved a
CAPTCHA might consume a type 0x0002 public veriable token, and use it to issue 30 type
0x0001 private tokens.


# Terminology

{::boilerplate bcp14-tagged}

We reuse terminology from {{RFC9576}}.

New terminology is defined below

Flow: Direction from PrivateToken issuance to its redemption. The entity starting the flow acts as an Issuer, while the end of the flow acts as an Origin. The Client is always included, as it finalises the TokenResponse, and coordinate interactions.
Initial Flow: Issuer -> Attester -> Client -> Origin. This flow produces a PrivateToken that is used by the Origin to kickstart a Reverse Flow.
Reverse Flow: Issuer <- Attester <- Client <- Origin. This flow allows Origin to issues PrivateToken. In the reverse flow, the Origin operates one or more Issuer, and the Client MAY provide these tokens either to the Initial Attester/Issuer, or use them against the Origin
Initial Attester/Issuer: Attester/Issuer part of the Initial Flow
Origin Issuer: Issuer operated by the Origin
Origin PrivateToken: PrivateToken issued by the Origin
Reverse Origin: An entity that consumes the Origin PrivateToken. It can be the Origin, or the Initial Attester/Issuer

# Protocol overview

Along with sending their PrivateToken for authentication (as specified in RFC9576), Client
sends TokenRequest

~~~aasvg
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
    |<-- TokenRequest ---+                                                |                   |           |
    +-- TokenResponse -->|                                                |                   |           |
    |                    |--- Response+TokenResponse(Origin Issuer) ----->|                   |           |
    |                    |                                                |                   |           |
~~~

The initial flow matches the one defined by {{ RFC9576 }}. A Client gets challenged when
accessing a resource on an Origin. The Client goes to the Attester to get issue a Token.

Through configuration mechanism not defined in this document, the Client is aware the Origin
acts as a Reverse Flow issuer.

This is an extension of {{ RFC9576 }}. The Client sends Request+Token+TokenRequest(Origin Issuer).
The Origin runs the issuance protocol based, and returns Response+TokenResponse(Origin Issuer).

TokenRequest(Origin Issuer) and TokenResponse(Origin Issuer) happen through a new HTTP Header `PrivacyPass-Reverse`.
`PrivacyPass-Reverse` is a base64url encoded binary encoded HTTP message, respectively per {{!RFC4648}} and {{!RFC9292}}.

> The use of binary encoding avoids the definition of a new ad-hoc encoding of request, response, and
> associated errors for each Privacy Pass issuance protocols. This matches the architecture as defined in {{!RFC9576}}.


## Client behaviour

Along with sending PrivateToken from the Initial Issuer to the Origin, the Client sends one TokenRequest as defined in {{!RFC9578}} or draft-batched-tokens.
The Client SHOULD consider Privacy Pass Reverse Flow like the initial flow. The Client
is responsible to coordinate between the different entities.
Specifically, if the Reverse Origin is the Initial Attester/Issuer, the Client SHOULD
account for possible privacy leakage.

## Origin/Issuer/Attester deployment

In this model, the Origin, Attester, and Issuer are all operated by the same
entity, as shown in {{fig-deploy-shared}}. The Reverse Flow is the same as
the Initial Flow, except for the request/response encapsulation.
The Origin is the Reverse Origin.

~~~ aasvg
                 +---------------------------------------------.
+--------+       |  +----------+     +--------+     +--------+  |
| Client |       |  | Attester |     | Issuer |     | Origin |  |
+---+----+       |  +-----+----+     +----+---+     +---+----+  |
    |             `-------|---------------|-------------|------'
    |<----------------------- TokenChallenge (Issuer) --+
    |                     |               |             |
    |<=== Attestation ===>|               |             |
    |                     |               |             |
    +----------- TokenRequest ----------->|             |
    |<---------- TokenResponse -----------+             |
    |                                                   |
    +------- Token+TokenRequest(Origin Issuer) -------->+
    |<--------- TokenResponse(Origin Issuer) -----------|
    |                                                   |
~~~
{: #fig-deploy-shared title="Shared Deployment Model"}

Similar to the original Shared Deployment Model, the Attester,
Issuer, and Origin share the attestation, issuance, and redemption
contexts. Even if this context changes between the Initial and
Reverse Flow, attestation mechanism that can uniquely identify
a Client are not appropriate as they could lead to unlinkability violations.

## Split Origin-Attester deployment

In this model, the Attester and Issuer are operated by the same entity
that is separate from the Origin. The Origin trusts the joint Attester
and Issuer to perform attestation and issue Tokens.
Origin Tokens can then be sent by Client on new requests, as long as the
Reverse Origin trusts the Origin to perform attestation and issue Tokens.

~~~aasvg
                                                                                     +--------------------------.
+---------------+    +--------+                                       +--------+     |  +----------+ +--------+  |
| Origin Issuer |    | Origin |                                       | Client |     |  | Attester | | Issuer |  |
+---+-----------+    +---+----+                                       +---+----+     |  +-----+----+ +----+---+  |
    |                    |                                                |           `-------|-----------|-----'
    |                    +-- TokenChallenge (Issuer) -------------------->|                   |           |
    |                    |                                                |<== Attestation ==>|           |
    |                    |                                                |                   |           |
    |                    |                                                +--------- TokenRequest ------->|
    |                    |                                                |<-------- TokenResponse -------+
    |                    |<-- Request+Token+TokenRequest(Origin Issuer) --+                   |           |
    |<-- TokenRequest ---+                                                |                   |           |
    +-- TokenResponse -->|                                                |                   |           |
    |                    |--- Response+TokenResponse(Origin Issuer) ----->|                   |           |
    |                    |                                                |                   |           |
~~~
{: #fig-deploy-joint-issuer title="Joint Attester and Issuer Deployment Model"}

The Origin Issuer MUST not issue privately verifiable tokens, as this would
lead to secret material being shared between the Origin and the Reverse Origin.

A particular deployment model is when the Reverse Origin is the Attester/Issuer.
This model is described in {{fig-deploy-joint-issuer-reserve}}

~~~aasvg
                                                                                     +--------------------------.
+---------------+    +--------+                                       +--------+     |  +----------+ +--------+  |
| Origin Issuer |    | Origin |                                       | Client |     |  | Attester | | Issuer |  |
+---+-----------+    +---+----+                                       +---+----+     |  +-----+----+ +----+---+  |
    |                    |                                                |           `-------|-----------|-----'
    |                    +-- TokenChallenge (Issuer) -------------------->|                   |           |
    |                    |                                                |<== Attestation ==>|           |
    |                    |                                                |                   |           |
    |                    |                                                +--------- TokenRequest ------->|
    |                    |                                                |<-------- TokenResponse -------+
    |                    |<-- Request+Token+TokenRequest(Origin Issuer) --+                   |           |
    |<-- TokenRequest ---+                                                |                   |           |
    +-- TokenResponse -->|                                                |                   |           |
    |                    |--- Response+TokenResponse(Origin Issuer) ----->|                   |           |
    |                    |                                                +------------ Token ----------->|
    |                    |                                                |                   |           |
~~~
{: #fig-deploy-joint-issuer-reserve title="Joint Attester and Issuer Deployment Model with reverse"}

This deployment SHOULD not allow the Reverse Origin to infer the request made
to the Origin, as it would break unlinkability.


# Privacy Considerations

Privacy Pass RFC 9576 states

> In general, limiting the amount of metadata permitted helps limit the extent to which metadata can uniquely identify individual Clients. Failure to bound the number of possible metadata values can therefore lead to a reduction in Client privacy. Most token types do not admit any metadata, so this bound is implicitly enforced.

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

Let's consider the following deployment: the Origin operates two issuers A and B. The Client sends Token_A, and (TokenRequest_A, TokenRequest_B). Issuer B is associated to a croissant aficionados.

If a Client requests croissant, or sends Token_B, the origin will provide TokenResponse_B. If not, it provides TokenResponse_A.

Over time, this means the Origin is able to track croissants aficionados.

To mitigate this, we RECOMMEND:
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
