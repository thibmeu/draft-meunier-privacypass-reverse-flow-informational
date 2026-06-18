---
title: "Privacy Pass Reverse Flow HTTP Transport"
abbrev: "Privacy Pass Reverse Flow HTTP Transport"
category: info

docname: draft-meunier-privacypass-reverse-flow-http-latest
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
  latest: "https://thibmeu.github.io/draft-meunier-privacypass-reverse-flow-informational/draft-meunier-privacypass-reverse-flow-http.html"

author:
 -
    fullname: Thibault Meunier
    organization: Cloudflare Inc.
    email: ot-ietf@thibault.uk

normative:
  BASE64: RFC4648
  REVERSE-FLOW: I-D.draft-meunier-privacypass-reverse-flow

informative:
  RFC9110:
  RFC9577:


--- abstract

This document specifies an instantiation of Privacy Pass Reverse Flow {{REVERSE-FLOW}}
where HTTP is used as a transport mechanism.

It describes a novel HTTP header field that Clients and Origins can use to carry
reverse flow data.

--- middle

# Introduction

This document specifies an instantiation of Privacy Pass Reverse Flow {{REVERSE-FLOW}}
where HTTP is used as a transport mechanism.

{{REVERSE-FLOW}} specifies an architecture in which a client can both present a token
and initiate a new credential issuance flow.

As described in {{Section 4 of REVERSE-FLOW}}, this requires in-band encoding of information
used by the issuance protocol.

This document introduces a new HTTP header field as defined in {{RFC9110}}. This allows Clients
to convey a CredentialRequest, and Origins to transmit a CredentialResponse.

# Terminology

{::boilerplate bcp14-tagged}

# `PrivacyPass-Reverse` Header Field

A Client or an Origin following {{REVERSE-FLOW}} MAY include a
`PrivacyPass-Reverse` header field to communicate Privacy Pass protocol
data. This header field contains a base64url (per {{BASE64}}) encoded CredentialRequest
or CredentialResponse.

~~~aasvg
+---------------+        +--------+                                     +--------+
| Origin Issuer |        | Origin |                                     | Client |
+--+------------+        +---+----+                                     +---+----+
   |                         |                                              |
   |                         |<-- PrivacyPass-Reverse: CredentialRequest ---+
   |<-- CredentialRequest ---+                                              |
   +--- CredentialResponse ->|                                              |
   |                         +-- PrivacyPass-Reverse: CredentialResponse -->|
   |                         |                                              |
~~~
{: #fig-reverse-flow-architecture title="Privacy Pass with a Reverse Flow through PrivacyPass-Reverse header field"}

## Example

Below is an example request that uses {{RFC9577}} to pass the request Token, as well as `PrivacyPass-Reverse` for its reverse flow.

~~~
GET /foo HTTP/1.1
Host: example.com
Authorization: PrivateToken token="abc..."
PrivacyPass-Reverse: "def..."

HTTP/1.1 200 OK
PrivacyPass-Reverse: "001..."

[BODY]
~~~

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO

# Changelog
{:numbered="false"}

v00

- Initial draft
