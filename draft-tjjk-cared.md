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
title: "Client Authentication Recommendations for Encrypted DNS"
abbrev: "CARED"
category: info

docname: draft-tjjk-BUTWHICHWG???-cared-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: AREA - TODO
workgroup: WG Working Group - TODO
keyword:
 - encrypted DNS
 - client authentication
 - mTLS
venue:
  group: WG - TODO
  type: Working Group - TODO
  mail: WG@example.com - TODO
  arch: https://example.com/WG -- TODO
  github: USER/REPO - TODO
  latest: https://example.com/LATEST -- TODO

author:
 -
    fullname: Tommy Jensen
    organization: Microsoft
    email: tojens@microsoft.com
 -
    fullname: Jess Krynitsky
    organization: Microsoft
    email: jess.krynitsky@microsoft.com
 -
    fullname: Jeff ???
    organization: Amazon
    email: TBD
 -
    fullname: Matt ???
    organization: Amazon
    email: TBD

normative:

informative:


--- abstract

For privacy reasons, encrypted DNS clients need to be anonymous to their encrypted DNS
servers to prevent third parties from correlating client DNS queries with other data for
surveillance or data mining purposes. However, there are cases where the client and server
has a pre-existing relationship and each peer wants to prove its identity to the other.
For example, an encrypted DNS server may only wish to accept resolutions from encrypted
DNS clients that are managed by the same enterprise. This requires mutual authentication.

This document defines when using client authentication with encrypted DNS is appropriate,
the benefits and limitations of doing so, and the recommended authentication mechanism(s)
when communicating with TLS-based encrypted DNS protocols.

--- middle

# Introduction

Generally, it is bad practice for encrypted DNS clients to provide any kind of identifying
information to an encrypted DNS server. For example, Section 8.2 of RFC 8484 {todo: ref}
discourages use of HTTP cookies with DoH. This ensures that clients provide a minimal amount
of identifiable or correlatable information to servers.

However, there are times when a client needs to identify itself to a server. One example
would be when an encrypted DNS server only accepts connections from pre-approved clients,
such as an encrypted DNS service provided by an enterprise for its work from home employees
to access when they are not connecting to the Internet through a network managed by that
enterprise. Encrypted DNS clients trying to connect to such a server would likely run into
refused connections if they did not provide authentication that proves they have that
enterprise's permission to query this server.

Because client authentication is not generally good practice for common DNS use cases, it
is important to define the situations in which interoperable encrypted DNS clients can
use client authentication without regressing the privacy value provided by encrypted DNS
in the first place. Even then, it is important to recognize what value client authentication
provides to encrypted DNS clients versus encrypted DNS servers in the context of both
connection management and DNS resolution utility. This document will go through these topics
as well as make best practice recommendations for authentication.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Benefits of client authentication with encrypted DNS

Strong identification of encrypted DNS clients by the encrypted DNS server allows the DNS
server to apply client-specific resolution policy in a securely verifiable way. Today, a
common practice is to establish client-specific resolution policies that differentiate
clients based on their observed IP address. This is not only an insecure method that cannot
account for clients routing requests through proxies, but it can only be used when expected
IP addresses are known in advance. This is not practical for enterprises with remote
employees without introducing a dependency on tunneling DNS traffic to a managed gateway or
proxy whose IP address is known. This in turn forces enterprises to choose between running
proxy or gateway infrastructure per client (or at least per-client IP address mappings) or
losing client resolution (cannot differentiate between clients connecting to the same proxy
or gateway).

Strong identification of encrypted DNS clients by the encrypted DNS server also brings
identification up to the application layer, which encapsulates this identity management
away from network topology changes (whenever IP address ranges need to change, name 
resolutions have to as well without some form of client authentication).

# Drawbacks of client authentication with encrypted DNS

While there are benefits in limited circumstances for using client authentication with
encrypted DNS, it has drawbacks that make it inappropriate in the general case. The
biggest reason client authentication is generally bad practice with encrypted DNS is
the obvious identification of clients and the association of their queries across 
connections. If a public DNS server (one which servers responses to anonymous clients over
the Internet) were to challenge clients for authentication, it would be forced de-anonymization
of the client's domain name history. This is unacceptable practice.

A less egregious drawback to client authentication with encrypted DNS is the required 
management of client identities and the complexity of matrixing domain name allow- and block-lists
against client identities for policy enforcement. This will be significantly more challenging for
encrypted DNS server operators to deploy and require industry-wide adoption. This drawback will reduce
over time as support for client authentication in encrypted DNS stacks is added.

# When to use client authentication with encrypted DNS

Client authentication with encrypted DNS SHOULD NOT be used outside the situations described
in this section to avoid regressing the privacy model of encrypted DNS. It
also needs to be acceptable to both peers.

Only encrypted DNS protocols that use TLS or dTLS (such as DoH {?RFC8484},
DoT {?RFC7858}, and DoQ {?RFC9250}) are in scope for this document.

## When servers should require client authentication

Encrypted DNS servers MUST NOT challenge clients for authentication if they do not need to
restrict connections to a set of clients they have a pre-existing relationship with as defined
in {{restrict-clients}}, regardless of whether or not the requirements in {{per-client}} apply.
Encrypted DNS servers that meet the requirements in {{restrict-clients}} MAY
challenge clients to authenticate to avoid achieving the same goal of identifying clients
through other, less secure means (such as IP address or data in the DNS query payload).

### Restricting connections to allowed clients {restrict-clients}

Some encrypted DNS servers provide DNS services to a specific set of clients and refuse
service to all other clients. One example of this may be an encrypted DNS server owned by an
enterprise that only allows connections from devices managed by that same enterprise.

Encrypted DNS servers SHOULD NOT challenge clients to authenticate when the server knows
it does not have a securely verifiable identity (such as when it is expecting clients to connect
opportunistically {todo: ref 7858 opportunistic mode}). Such servers do not need to
identify allowed clients, since security-minded clients need to know to whom they are
authenticating.

Encrypted DNS servers SHOULD NOT challenge clients to authenticate when... TBD anything else?

### Resolving names differently per client {per-client}

Some encrypted DNS servers need to change their resolution behavior based on the identity
of the client that issued the query because some clients have permission to resolve names
that other clients do not.

Encrypted DNS servers SHOULD NOT attempt to authenticate clients when the
difference in resolution behavior between them is opted into by the client rather than
required by the authority operating the encrypted DNS server. For example, some public
DNS services offer multiple servers that each have different resolution behavior. One might
resolve almost every name, whereas another may try to refuse to resolve names associated with
social media sites, or advertisers, or other categories customers find useful. This can be met
by defining these endpoints as separate servers on different dedicated IP addresses or using
different domain names in their encryped DNS configuration (such as DoH template), or both.

Encrypted DNS servers SHOULD NOT attempt to authenticate clients when... TBD anything else?

## When clients should attempt to authenticate

Encrypted DNS clients MUST NOT offer authentication to any encrypted DNS server unless it was
specifically configured to expect that server to require authentication, independent of the
mechanism by which the client was configured with or discovered the encrypted DNS server.

Encrypted DNS clients MUST NOT offer authentication when the server connection was established
via DDR {{?RFC9462}}, even if the IP address of the original DNS server was specifically
configured for the encrypted DNS client as one that might require authentication. This is
because in that circumstance there is not a pre-existing relationship with the encrypted DNS
server (or else DDR bootstrapping into encrypted DNS would not have been necessary).

TBD: what to do with EDSR destinations

Otherwise, an encrypted DNS client MAY choose to present authentication to a server that
requests it, but is not required to just because it was challenged to do so if the client
would rather not use the server altogether. The presented client identity SHOULD be unique
per server identity to avoid becoming a correlatable identity between colluding encrypted
DNS servers.

# Recommendations

The following requirements were considered when formulating the recommended authentication
mechanism for encrypted DNS clients:
- SHOULD be per-connection, not per-query (avoid unnecessary payload overheads)
- SHOULD use existing open standards (avoid vendor lock-in and specialized cryptography)
- SHOULD be reusable across multiple encrypted DNS protocols (avoid protocol preference)
- SHOULD NOT require human user interaction to complete authentication

This document concludes that the current best mechanism for encrypted DNS client authentication
is mTLS {?RFC8705} for the following reasons:
- mTLS identifies and authenticates clients, not users, per-connection
- mTLS is an exiting standard and is often already configured for TLS clients
- x.509 certificates used for TLS client authentication allow the server to identify the client's organization via PKI heiracrchy
- mTLS is reusable across multiple encrypted DNS protocols
- mTLS allows session resumption {?RFC5077}
- mTLS does not require user interaction or apaplication layer input for authentication

Encrypted DNS clients and servers that support offering or requesting client authentication
MUST support at least the use of mTLS in addition to whatever other mechanism they wish to support.

Encrypted DNS clients and servers SHOULD prefer long-lived connections when using client
authentication to minimize the cost over time of doing repeated TLS handshakes and identity
verification.

## Why these requirements were chosen

### Per-connection scope

Any data added to each DNS message will have greater bandwidth and processing costs than presenting
authentication once per connection. This is especially true in this case because the drawback
of having long-running encrypted DNS connections is the decreased privacy through increased
volume of directly correlatable queries. However, this privacy threat does not apply to this
situation because the client's queries can already be correlated by the identity it presents.
Therefore, per-connection bandwidth and data processing overhead is expected to be much lower
than per-query because there is no incentive for clients and servers to not have long-running
encrypted DNS connections.

This is not expected to create excessive cost for server operators because supporting encrypted
DNS without client authentication already requires per-connection state management.

### Reuse open standards

Reusing open standards ensures wide interoperability between vendors that choose to implement
client authentication in their encrypted DNS stacks.

### Reusable across protocols

If a client authentication method for encrypted DNS were defined or recommended that would only
be usable by some TLS-encrypted DNS protocols, it would encourage the development of a second
or even third solution later.

### Do not require human interaction

Humans using devices that use encrypted DNS, when given any kind of prompt or login relating to
establishing encrypted DNS connectivity, are unlikely to understand what is happening and why. This
will inevitably lead to click-through behavior. Because the scope of scenarios where client authentication
for encrypted DNS is limited to pre-existing relationships between the client and server, there should
be no need for at-run-time intervention by a human user.

## Why alternatives are not recommended

### Web tokens

OAuth or JSON web tokens alone require HTTP to validate, so would not be a solution for encrypted DNS protocols other than DoH.
Web access tokens can be used as certificate-bound access tokens in combination with mTLS if they are needed to prove
identity with another authorization server, as described in {?RFC8705}.

### HTTP authentication

HTTP authentication as defined in {?RFC7235} provides a basic authentication scheme for the HTTP protocol. Unless it is used with TLS,
i.e. over HTTPS, the credentials are encoded but not encrypted which is insecure. As TLS is already used by the encrypted DNS
protocols in this document's scope, it is simpler to handle client authentication and authorization at the TLS layer. 
Additionally, mTLS is more broadly adopted than HTTP authentication. HTTP authentication would only be a viable option for DoH, 
and not extensible to other encrypted DNS solutions.

### FIDO

Web Authentication (WebAuthN) and the FIDO2 Client to Authenticator Protocol (CTAP) use CBOR Object Signing and Encryption (COSE),
described in {?RFC8812}. FIDO and WebAuthN are passkey solutions designed to replace passwords for user authentication for online services,
and they are not typically used for general client authentication. Passkeys are unique for each online service and require user input
for registration, and would require DNS servers to support the WebAuthN protocol. Additionally, each sign-in requires user input
for local verification, using biometric, local PIN, or a FIDO security key.

### Designing a novel solution

Designing a novel solution is never recommended when there is an existing standard that meets the requirements.
Doing so would make the encrypted DNS solution more difficult and time-consuming to adopt, and most likely would
introduce vendor lock-in.

# Deployment Considerations

## Avoiding connectivity deadlocks

Deployers should carefully consider how they will handle certificate rollover and revocation. If an
encrypted DNS server only allows connections from clients with valid certificates, and the client is configured
to only use the encrypted DNS server, then there will be a deadlock when the certificate expires or is revoked
such that the client device will not have the connectivity needed to renew or replace its certificate. 

Encrypted DNS servers that challenge clients for authentication SHOULD have a separate resolution policy for
clients that do not have valid credentials that allows them to resolve the subset of names needed to connect to
the infrastructure needed to acquire certificates. Alternatively, encrypted DNS clients that are configured to use encrypted DNS
servers that will require authentication MAY consider configuring knowledge of certificate issuing infrastructure in advance
so that the DNS deadlock can be avoided without introducing less secure DNS servers to their configuration (i.e. 
hard coding IP addresses and host names for certificate checking).

# Security Considerations

This document describes when and how encrypted DNS clients can authenticate themselves to
an encrypted DNS server. It does not introduce any new security considerations not already
covered by TLS {TOOD: ref} and mTLS {TODO: ref}. This document does not define recommendations
for when and how to use encrypted DNS client authentication for encrypted DNS protocols that
are not TLS-based.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.