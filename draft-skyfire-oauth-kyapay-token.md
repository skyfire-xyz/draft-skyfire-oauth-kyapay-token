---
title: "KYAPay Token"
#abbrev: ""
category: std

docname: draft-skyfire-oauth-kyapay-token-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
#number:
date:
consensus: true
v: 3
area: Security
workgroup: Web Authorization Protocol
keyword:
 - agent
 - identity
 - agentic
 - payment
 - commerce
venue:
  github: "skyfire-xyz/draft-skyfire-oauth-kyapay-token"
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  latest: "https://skyfire-xyz.github.io/draft-skyfire-oauth-kyapay-token/draft-skyfire-oauth-kyapay-token.html"

author:
-
  name: Ankit Agarwal
  organization: Skyfire Systems Inc.
  email: ankit_agarwal@yahoo.com
  uri: https://skyfire.xyz
-
  ins: M. Jones
  name: Michael B. Jones
  organization: Self-Issued Consulting
  email: michael_b_jones@hotmail.com
  uri: https://self-issued.info/

contributor:
-
  name: Andrew Stitt
-
  name: Dmitri Zagidulin

# see https://github.com/cabo/kramdown-rfc/wiki/Syntax2#authors-contributors

normative:
  RFC6454:
  RFC7515:
  RFC7517:
  RFC7518:
  RFC7519:
  RFC6749:
  RFC7800:
  RFC8615:
  RFC8693:
  RFC8725:

informative:
  RFC2046:
  RFC6838:
  RFC8446:
  RFC9421:
  IANA.JWT.Claims:
    author:
    - org: IANA
    target: https://www.iana.org/assignments/jwt
    title: JSON Web Token Claims
    date: false
  IANA.MediaTypes:
    author:
    - org: IANA
    target: https://www.iana.org/assignments/media-types
    title: Media Types
    date: false
  IANA.WellKnownURIs:
    author:
    - org: IANA
    target: https://www.iana.org/assignments/well-known-uris
    title: Well-Known URIs
    date: false
  I-D.skyfire-oauth-using-kyapay-tokens:
  I-D.skyfire-oauth-kyapay-token-exchange:
  I-D.skyfire-oauth-amr-values:
  I-D.skyfire-oauth-id-verification:
  I-D.skyfire-oauth-aml-methods:

...

--- abstract

This document defines a token format for agent identity and payment tokens in
JSON Web Token (JWT) format. Authorization servers and resource servers from
different vendors can leverage this token format to consume identity and payment
tokens in an interoperable manner.

--- middle

# Introduction

As software agents evolve from pre-orchestrated workflow automations to truly
autonomous or semi-autonomous assistants, they require the ability to identify
themselves -- and more importantly, identify their human principals -- to external
systems. Agents acting on behalf of users to discover services, create accounts,
or execute actions currently face significant operational barriers.

The KYAPay token addresses these challenges by providing a standard envelope to
carry verified identity and payment information. By utilizing "kya" (Agent
Identity) and "pay" (Payment) tokens, agents can identify their human principals
to services, sites, bot managers, customer identity and access management (CIAM)
systems, and fraud detectors. This enables agents to bypass common blocking
mechanisms and access services that were previously restricted to manual human
interaction.

KYAPay does not aim to define agentic identity in its entirety. Rather, it specifies
a standard and extensible JSON Web Token (JWT) {{RFC7519}} that can be used to securely share human
principal and agent identity information with websites and APIs. KYAPay tokens
provide a strong signal of human presence behind agentic requests that are
otherwise indistinguishable from programmatic and potentially malicious bot requests.

Note that, in the future,
the payment token functionality could be split into a separate specification,
if desired by a working group adopting the specification.
It is retained here at present for ease of reviewing.

## Use Cases for the KYAPay Token

Enabling agents to access websites and APIs on behalf of
the human principals they represent is a design goal of KYAPay tokens.
Today’s internet is designed primarily for humans, meaning that automated systems
are often classified as malicious and blocked by web security infrastructure.
However, the rise of AI agents has introduced a new paradigm where
programmatic clients legitimately access websites and APIs
on behalf of human principals.
Because these agents can be hard to distinguish from traditional bots,
they are often inadvertently blocked,
creating a need for the web security ecosystem to distinguish between
legitimate agentic traffic and truly malicious activity.
KYAPay tokens are designed to address this challenge by enabling agents to convey
verified identity and payment credentials.
These tokens can provide web security systems and merchants with
a strong signal that the requests are authorized by a human,
allowing them to safely permit legitimate programmatic transactions
while aggressively blocking undesired traffic.

Enabling agents to create accounts and/or log in to accounts
on behalf of their human principals is a related design goal.
To achieve this, systems can utilize a token exchange workflow {{RFC8693}}.
In this process, a Security Token Service (STS), Identity Provider (IdP),
or OAuth Authorization Server verifies incoming KYA tokens
and extracts claims associated with the human principal, such as email addresses.
The authorization server then performs a token exchange,
swapping the KYA token for a standard OAuth Access Token,
which the agent subsequently uses to interact with the target service.
Crucially, this architecture allows the service to know
that the agent is acting on behalf of the user,
making it possible to differentiate between
direct, human-present sessions and human-initiated, agentic sessions
for authorization, auditing, and security purposes.

Enabling agents to have ubiquity of access across the Internet just like their
human principals is a related design goal.
Automation typically scales as it achieves higher reliability and lower
cost-to-entry. Unlike the structured logic required by cron jobs or
low-code / no-code platforms, agentic automation leverages LLMs to execute
tasks via natural language, effectively removing the software-skill barrier.
As model reasoning improves and infrastructure scales, these agents become
increasingly dependable and affordable for the human principal.
To maximize utility, agents require ubiquitous Internet access, a feat made
possible by KYAPay Token Issuers. By providing a client-side verification
framework analogous to the server-side role of Certificate Authorities (CAs),
KYAPay builds a standardized network of acceptance across the web security
ecosystem. This allows for the seamless attestation of both the agent’s and
the human principal’s identity, ensuring secure, cross-domain task execution
without the friction of fragmented authentication silos.

Enabling the ecosystem of web security vendors to engage in finer-grained and
deliberate bad-actor mitigation is a related design goal.
KYA tokens provide a layered, verified, and extensible identity stack
specifically engineered for autonomous agents. This framework
allows the web security ecosystem to distinguish among individual agent
instances, the platforms they run on, and the human principals behind them.
By establishing this level of granular visibility, security systems can
transition from broad defensive measures to specific mitigation; rather than
being forced to block an entire platform, administrators can now isolate
and neutralize a single malicious human user or a malfunctioning software
instance without disrupting legitimate traffic.

Note that the protocols using these tokens to achieve these goals
are not defined by this specification.
The interoperable use of them for these purposes will require further specification.

Early production deployments of KYAPay tokens are described at https://kyapay.org.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The claims `iss`, `iat`, `exp`, `aud`, and `jti` are defined by {{RFC7519}}.
The header parameters `alg`, `kid`, and `typ` are defined by {{RFC7515}}.

The `alg` value `ES256` is a digital signature algorithm defined in
{{Section 3.4 of RFC7518}}.

## Roles

{:vspace}
Agent:
: An application, service, or specific software process, executing on behalf
  of a Principal.

{:vspace}
Agent Identity:
: A unique identifier and a set of claims describing an agent. Grouped into the
  `aid` claim for convenience. Because an agent can be public or confidential
  (as described in {{Section 2.1 of RFC6749}}), the level of assurance for these
  claims varies dramatically. Agents also vary in terms of longevity -- they can
  have stable long-running identities (such as those of a server-side confidential
  client), or they can be transient and ephemeral, and correspond to individual
  API calls or compute workloads.

{:vspace}
Agent Platform:
: The service provider and runtime environment hosting the Agent, such as a
  cloud compute provider or AI operator service. Assertions about the agent
  platform are grouped into the `apd` claim, and are primarily used to identify
  the Principal entity operating the platform, allowing consumers of the token to
  apply reputation-based logic or offer platform-specific services.

{:vspace}
Principal:
: A legal entity (human or organization) on whose behalf / in whose authority
an agent or service is operating.

### Initiator Roles

{:vspace}
Initiator Agent:
: An Agent performing tasks on behalf of an Initiator Principal, that has its own
  Agent Identity, grouped into the `aid` claim.

{:vspace}
Initiator Agent Platform:
: The Agent Platform hosting the Initiator Agent. Some use cases require the Platform
  to have its own verified identity assertions, grouped into the `apd` claim.

{:vspace}
Initiator Principal:
: A legal entity (human or organization) behind the purchase / consumption of a
  product or service.
  In buyer/seller transactions, the Initiator is the buyer.
  The Principal typically interacts with the target via an
  Initiator Agent. Many targets are required to be able to determine the Initiator
  Identity in order to comply with KYC/AML regulations, accounting standards,
  and to maintain a direct customer relationships. The initiator principal's
  identity is grouped into the `hid` claim.

{:vspace}
Initiator Identity:
: The aggregate verified identity assertions of the initiator entities, typically
  encompassing the Initiator Principal, the Initiator Agent Platform, and the Initiator Agent
  itself. This composite identity is conveyed via the KYA token, allowing the
  target to verify the entire chain of responsibility behind a request.
  The initiator identity utilizes the `hid`, `apd`, and `aid` claims.

### Target Roles

{:vspace}
Target Agent:
: An Agent performing tasks on behalf of a Target Principal, directly interacting
  with Initiator Agents to facilitate discovery and purchase. Typically runs on
  Internet-connected infrastructure, and discoverable via service directories.
  Target agent identity claims are also grouped into the `aid` claim
  if KYA tokens are generated for the targets.

{:vspace}
Target Agent Platform:
: The Agent Platform that hosts Target Agents. Some use cases require the Platform
  to have its own verified identity assertions, grouped into the `apd` claim.

{:vspace}
Target Principal:
: A human principal (individual or organization) that that owns the product,
  service, API, website, or content being consumed or sold, and serves as the
  ultimate beneficiary of a transaction.
  In buyer/seller transactions, the Target is the seller.
  The target principal's identity is grouped into the `hid` claim.

{:vspace}
Target Identity:
: The aggregate verified identity assertions of the target entities, typically
  encompassing the Target Principal, the Target Agent Platform, as well as the
  Target Agent Identity.
  These various aspects of Target Identity allow Initiators and Initiator Agents to
  perform reputation-based logic, to verify that they are interacting with
  the authorized (and expected) counter-party, and to fulfill KYC/AML regulation
  requirements.
  The target identity utilizes the `hid`, `apd`, and `aid` claims.

### Ecosystem Infrastructure Roles

{:vspace}
Identity Token Issuer:
: A trusted neutral entity that conducts Know Your Customer (KYC) and Know Your
  Business (KYB) (for organizations) verifications. It is responsible for issuing
  cryptographically signed `kya` tokens that attest to the identity of the
  Principal, Agent, and Agent Platform, for both Initiators and Targets.

{:vspace}
Payment Token Issuer:
: A trusted entity responsible for facilitating the exchange of payments and
  credentials between the Initiator and Target. It issues signed `pay` tokens that
  enable settlement via various schemes (Cards, Banks, Cryptocurrency), without
  exposing raw credentials or secrets.

# KYAPay Token Schemas

## Common Token Claims {#common-claims}

The following are claims in common, used within the KYA (Know Your Agent),
PAY (Payment), and KYA-PAY (combined Know Your Agent and Payment) Tokens.

{:vspace}
`iss`:
: REQUIRED - URL of the token's issuer. The value MUST be an origin as defined
  in {{Section 4 of RFC6454}}: a scheme, a host, and an optional port, with no
  path, query or fragment component. Used for locating the JWK Set for token
  signature verification, as described in {{key-discovery}}.

{:vspace}
`sub`:
: REQUIRED - Subject Identifier. Must be pairwise unique within
  a given issuer.

{:vspace}
`aud`:
: REQUIRED - Audience (used for audience binding and replay attack mitigation),
  uniquely identifying the target agent.
  A single string value.

{:vspace}
`iat`:
: REQUIRED - as defined in {{Section 4.1.6 of RFC7519}}.  Identifies the time
  at which the JWT was issued.  This claim must have a value in the past and can
  be used to determine the age of the JWT.

{:vspace}
`jti`:
: REQUIRED - Unique ID of this JWT as defined in {{Section 4.1.7 of RFC7519}}.

{:vspace}
`exp`:
: REQUIRED - as defined in {{Section 4.1.4 of RFC7519}}.  Identifies the expiration
  time on or after which the JWT MUST NOT be accepted for processing.

{:vspace}
`tdm`:
: OPTIONAL - Target domain, associated with the audience claim, the token is intended for.

{:vspace}
`ori`:
: OPTIONAL - URL of the token's originator.

{:vspace}
`env`:
: OPTIONAL - Issuer environment (such as "production" or "sandbox").  Additional values
  may be defined and used.

{:vspace}
`tsi`:
: OPTIONAL - Target Service ID that this token was created for.

{:vspace}
`itg`:
: OPTIONAL - Initiator tag - an opaque reference ID internal to the initiator.

{:vspace}
`cnf`:
: OPTIONAL - Confirmation claim, as defined in {{RFC7800}}, binding the token to
  a key held by the agent identified in the `aid` claim.
  When present, it MUST contain the `jwk` member: the agent's public key
  represented as a JWK {{RFC7517}}. The JWK MUST contain only public key
  material; a `cnf` containing private key material MUST be rejected.
  Other confirmation members MAY additionally be present, but a verifier MUST NOT
  be required to obtain the key by any means other than reading `cnf.jwk`.
  A token carrying `cnf` is sender-constrained rather than bearer: the presenter
  is required to demonstrate possession of the confirmation key. Verifier
  processing is specified in {{I-D.skyfire-oauth-using-kyapay-tokens}}.

Additional claims MAY be defined and used in these tokens.
The recipient MUST ignore any unrecognized claims.


## Issuer Key Discovery {#key-discovery}

The JWK Set used to verify a token signature is located at the well-known URI
{{RFC8615}} formed from the `iss` claim with the URI suffix `jwks.json`.
Because `iss` is an origin, this is the well-known URI construction described
in {{Section 3 of RFC8615}} applied to that origin. For example, an `iss` value
of `https://issuer.example.com` yields
`https://issuer.example.com/.well-known/jwks.json`.

The `jwks.json` URI suffix is registered in {{well-known-registration}}.

JWK Sets SHOULD be cached and refreshed according to standard JWK Set practice.
Retrieval per request is neither required nor expected; because KYAPay tokens
are self-contained, signature verification is intended to be performed locally.

A future revision of this specification may additionally define an issuer
metadata document, giving an issuer somewhere to advertise capabilities beyond
the location of its keys, such as the identity verification methods it supports.
Such a document would supplement the mechanism described here rather than
replace it.

## KYA Token {#kya-token}

The following identity related claims are used within KYA and KYA-PAY tokens:

{:vspace}
`hid`:
: REQUIRED (Required for human identity use cases) - A map of human identity
  claims (individual or organization).

{:vspace}
`apd`:
: OPTIONAL - Agent Platform identity claims.

{:vspace}
`aid`:
: REQUIRED - Agent identity claims.

{:vspace}
`scope`
: OPTIONAL - String with space-separated scope values, per {{RFC8693}}

The following informative example displays a decoded KYA type token.

~~~
{
  "kid": "YjFdJgFNWj9AkUmtoXILwoeb37PsBuGWVK6_QvFLwJw", // JWK Key ID
  "alg": "ES256",
  "typ": "kya+jwt"
}.{
  "iss": "https://issuer.example.com", // Issuer URL
  "iat": 1742245254,
  "exp": 1742245554,
  "jti": "b9821893-7699-4d24-af06-803a6a16476b",
  "sub": "bb713104-c14e-460f-9b7c-f8140fa9bea4", // Initiator Agent Account ID
  "aud": "7434230d-0861-46f2-9c2c-a6ee33d07f17", // Target Agent Account ID

  "env": "production",
  "tsi": "bc3ff89f-069b-4383-82a9-8cfe53c55fc3", // Target Service ID
  "itg": "4f6cbd39-215c-4516-bf33-cab22862ee60", // Initiator Tag (Internal Reference ID)

  "hid": {
    "email": "initiator@initiator.com"
  },
  "apd": {
    "id": "d3306fc0-602b-47e6-9fe2-3d55d028fbd2",
    "name": "Acme Shopping Agents", // Agent platform name
    "email": "platform@acme.com", // Email address for the agent platform
    "phone_number": "+12345677890", // Phone number for the agent platform
    "organization_name": "Acme Shopping Inc.", // Legal name of the agent platform
    "verifier": "https://www.verifier.com/", // URL of the Identity verifier
    "verified": true, // Outcome of the verifier's KYA verification
    "verification_id": "a23c1fe4-a4b7-442d-8bca-3c8fad5ec3a6" // Verifier's verification ID
  },
  "aid": {
    "name": "Acme Agent Extraordinaire",
    "creation_ip": "54.86.50.139", // IP Address where token was created
    "source_ips": ["54.86.50.139-54.86.50.141", "1.1.1.0/24",
      "2001:db8:abcd:0012::/64", "acme.com"]
      // IP addresses from which the initiator agent will make requests to the target
  }
}
~~~
{: #example-decoded-kya-token align="left" title="A KYA type token"}

### `hid` - Human Identity Sub-Claims

The Human Identity (`hid`) claim contains sub-claims identifying the human
principal (individual or organization) as follows.

{:vspace}
`email`:
: REQUIRED - Email address associated with the human individual or organization

{:vspace}
`given_name`:
: OPTIONAL - Given name(s) or first name(s) of the human principal if they
  are an individual.

{:vspace}
`middle_name`:
: OPTIONAL - Middle name(s) of the human principal if they are an individual.

{:vspace}
`family_name`:
: OPTIONAL - Surname(s) or last name(s) of the human principal if they are an
  individual.

{:vspace}
`phone_number`:
: OPTIONAL - Phone number associated with the human individual or organization.

{:vspace}
`organization_name`:
: OPTIONAL - Name of the organization.

{:vspace}
`verifier`:
: OPTIONAL - URL of the Identity Verifier

{:vspace}
`verified`:
: OPTIONAL - Boolean Verification status.  True if verified, otherwise false.

{:vspace}
`verification_id`:
: OPTIONAL - Verification identifier. Identifier for the verification performed,
  such as a GUID.

Additional sub-claims MAY be defined and used.
The recipient MUST ignore any unrecognized sub-claims.

### Agent Platform Identity `apd` Sub-Claims

The `apd` claim is optional. If present, it contains the following sub-claims.

{:vspace}
`id`:
: REQUIRED - Agent Platform identifier.

{:vspace}
`name`:
: REQUIRED - Agent Platform name.

{:vspace}
`email`:
: OPTIONAL - Email associated with agent platform.

{:vspace}
`phone_number`:
: OPTIONAL - Phone number associated with agent platform.

{:vspace}
`organization_name`:
: OPTIONAL - Legal name associated with agent platform.

{:vspace}
`verifier`:
: OPTIONAL - URL of the Identity Verifier

{:vspace}
`verified`:
: OPTIONAL - Boolean Verification status.  True if verified, otherwise false.

{:vspace}
`verification_id`:
: OPTIONAL - Verification identifier. Identifier for the verification performed, such as a GUID.

Additional sub-claims MAY be defined and used.
The recipient MUST ignore any unrecognized sub-claims.

### Agent Identity `aid` Sub-Claims

The `aid` claim is REQUIRED. It contains the following sub-claims.

{:vspace}
`name`:
: REQUIRED - Agent name. The name should reflect the business purpose of the agent.

{:vspace}
`creation_ip`:
: REQUIRED - The public IP address of the system / agent that requested the token.
  Its value is a string containing the public IPv4 or IPv6 address from where the
  token request originated. It MUST be captured directly from the token request.

{:vspace}
`source_ips`:
: OPTIONAL - Valid public IP address, or range of public IP addresses, from where
  the system / agent's requests to merchants / services will originate. Array of
  comma-separated IPv4 addresses or ranges, IPv6 addresses or ranges, or domain
  names resolvable to an IP address via DNS. IPv4 and IPv6 addresses can be a
  single IPv4 or IPv6 address or a range of IPv4 or IPv6 addresses in CIDR notation
  or start-and-end IP pairs.

Additional sub-claims MAY be defined and used.
The recipient MUST ignore any unrecognized sub-claims.

## PAY Token {#pay-token}

The following payment related claims are used within PAY and KYA-PAY type tokens:

{:vspace}
`tpr`:
: OPTIONAL - JSON string representing target service price in currency units.

{:vspace}
`tps`:
: OPTIONAL - Target pricing scheme, which represents a way for the target list
  how it charges for its service or content. One of `pay_per_use`,
  `subscription`, `pay_per_mb`, or `custom`.  Additional values may be defined
  and used.

{:vspace}
`amt`:
: REQUIRED - JSON string representing token amount in currency units.

{:vspace}
`cur`:
: REQUIRED - Currency unit, represented as an ISO 4217 three letter code, such as "EUR".

{:vspace}
`val`:
: REQUIRED - JSON string representing token amount in settlement network's units.

{:vspace}
`mnr`:
: OPTIONAL - JSON number representing maximum number of requests when `tps` is `pay_per_use`.

{:vspace}
`stp`:
: REQUIRED - Settlement type (one of `coin` or `card`).  Additional values may be defined and used.

{:vspace}
`sti`:
: REQUIRED - Meta information for payment settlement, depending on settlement.
  type.

### Settlement Information `sti` Sub-Claims

The `sti` claim is REQUIRED. It contains the following sub-claims,
with the requirement levels marked below.

The `sti` claim carries the settlement instrument, and its contents depend on
the value of `stp`.

When the `stp` is value `card`, the settlement instrument is an agentic payment credential
issued by a payment network under an agentic-commerce programme -- for example
Visa Intelligent Commerce (`visa_vic`) or Mastercard Agent Pay, using Secure Card
on File (`mastercard_scof`). Such a credential is provisioned for a single
transaction and is scoped to one initiator, one target, and one amount. It is not
a Primary Account Number (PAN), and it cannot be used to derive the underlying
PAN or the network's agentic token.

{:vspace}
`type`:
: REQUIRED - "type" is dependent on the "stp" value; for "coin" - "usdc";
  for "card" - "visa_vic" or "mastercard_scof".  Additional values may be defined and used.

{:vspace}
`payment_token`:
: REQUIRED when the `stp` value is `card`; otherwise OPTIONAL - String containing the
  network-issued agentic payment credential, formatted per ISO/IEC 7812,
  12-19 characters. This value MUST be a payment-network-issued agentic or
  tokenized credential. It MUST NOT be a Primary Account Number (PAN).

{:vspace}
`token_expiration_month`:
: REQUIRED when `payment_token` is present - String containing two-digit
  Expiration Month Number.

{:vspace}
`token_expiration_year`:
: REQUIRED when `payment_token` is present - String containing four-digit
  Expiration Year.

{:vspace}
`token_security_code`:
: REQUIRED when `payment_token` is present - String containing the single-use
  token cryptogram accompanying the credential in `payment_token` -- a Dynamic
  Token Verification Value (DTVV) or Token Authentication Verification Value
  (TAVV), depending on the network. 3 or 4 digits. This value is generated per
  transaction and is short-lived; its validity period is determined and enforced
  by the payment network. It MUST NOT be a static Card Verification Value
  (CVV, CVC2 or CVV2).

{:vspace}
`verifier`:
: OPTIONAL - URL of the Payment Verifier

{:vspace}
`verified`:
: OPTIONAL - Boolean Verification status.  True if verified, otherwise false.

{:vspace}
`verification_id`:
: OPTIONAL - Verification identifier. Identifier for the verification performed, such as a GUID.

Additional sub-claims MAY be defined and used.
The recipient MUST ignore any unrecognized sub-claims.

### Scope of Card Settlement

This specification supports card settlement only through payment-network
agentic-commerce programmes that issue transaction-scoped, PCI-exempt
credentials.

A PAY or KYA-PAY token MUST NOT carry a Primary Account Number, or the static
verification value associated with one.

Settlement instruments that fall within PCI DSS scope require mechanisms this
specification does not define -- including confidentiality protection of the
token contents and a defined cardholder-data-environment boundary. Support for
such instruments is left to future work, and implementers requiring it should
not attempt to carry them in the claims defined here.

### PAY Token Example {#pay}

The following informative example displays a decoded PAY type token.

~~~
{
  "kid": "FgT4q8c5IqbBCCjcho5JdeGQvuK1keMDFc9IwCm8J7Y", // JWK Key ID
  "alg": "ES256",
  "typ": "pay+jwt"
}.{
  "iss": "https://pay-issuer.example.net", // Issuer URL
  "iat": 1742245254,
  "exp": 1742245554,
  "jti": "b9821893-7699-4d24-af06-803a6a16476b",
  "sub": "8b810549-7443-494f-b4ad-5bc65871e32b", // Initiator Agent Account ID
  "aud": "37888095-2721-48d9-a2df-bfe4075f223a", // Target Agent Account ID

  "env": "sandbox",
  "tsi": "274efc47-024e-466f-b278-152d2ee73955", // Target Service ID
  "itg": "16c135ce-a99a-453d-a7b5-4958fd91de5f", // Initiator Tag (Internal Reference ID)

  "tpr": "0.01",
  "tps": "pay_per_use",
  "amt": "15",
  "cur": "USD",
  "val": "15000000",
  "mnr": 1600,
  "stp": "card",
  "sti": {
    "type": "visa_vic",
    "payment_token": "1234567890123456",
    "token_expiration_month": "03",
    "token_expiration_year": "2030",
    "token_security_code": "123",
    "verifier": "https://verifier.example.info", // URL of payment method verifier
    "verified": true, // Outcome of the verifier's payment method verification
    "verification_id": "3a6e1b76-8f78-4c24-b1bd-dc78a8cc3711" // Identifier for the verification performed, such as a GUID.
  }
}

~~~
{: #example-decoded-pay-token align="left" title="A PAY type token"}

## KYA-PAY Token {#kya-pay-token}

The following informative example displays a decoded KYA-PAY type token.

~~~
{
  "kid": "YjFdJgFNWj9AkUmtoXILwoeb37PsBuGWVK6_QvFLwJw", // JWK Key ID
  "alg": "ES256",
  "typ": "kya-pay+jwt"
}.{
  "iss": "https://kya-pay.example.org", // Issuer URL
  "iat": 1742245254,
  "exp": 1742245554,
  "jti": "b9821893-7699-4d24-af06-803a6a16476b",
  "sub": "f24a431d-108c-46e6-9357-b428c528210e", // Initiator Agent Account ID
  "aud": "5e00177d-ff7f-424b-8c83-2756e15efbed", // Target Agent Account ID

  "env": "production",
  "tsi": "3e6d33a1-438e-482e-bba5-6aa69544727d", // Target Service ID
  "itg": "c52e0ef2-e27d-4e95-862e-475a904ae7b2", // Initiator Tag (Internal Reference ID)

  "hid": {
    "email": "maryjane@initiator.example.com",
    "given_name": "Mary",
    "middle_name": "Jane",
    "family_name": "Doe",
    "phone_number": "+1-425-555-1212",
    "verified": false
  },
  "apd": {
    "id": "4b087db2-b6e5-48b8-8737-1aa8ddf4c4fe", // Agent platform ID
    "name": "Acme Shopping Agents", // Agent platform name
    "email": "platform@acme.com", // Email address for the agent platform
    "phone_number": "+12345677890", // Phone number for the agent platform
    "organization_name": "Acme Shopping Inc.", // Legal name of the agent platform
    "verifier": "https://www.verifier.com/", // URL of the Identity verifier
    "verified": true, // Outcome of the verifier's KYA verification
    "verification_id": "a23c1fe4-a4b7-442d-8bca-3c8fad5ec3a6" // Verifier's verification ID
  },
  "aid": {
    "name": "Agentic Excellence Я Us",
    "creation_ip": "128.2.42.95", // IP Address where token was created
    "source_ips": ["54.86.50.139-54.86.50.141", "1.1.1.0/24",
      "2001:db8:abcd:0012::/64", "agentic-excellence.example.com"]
      // IP addresses from which the initiator agent will make requests to the target
  },

  "tpr": "0.01",
  "tps": "pay_per_use",
  "amt": "15",
  "cur": "USD",
  "val": "15000000",
  "mnr": 1600,
  "stp": "card",
  "sti": {
    "type": "visa_vic",
    "payment_token": "1234567890123456",
    "token_expiration_month": "03",
    "token_expiration_year": "2030",
    "token_security_code": "123"
  }
}

~~~
{: #example-decoded-kya-pay-token align="left" title="A KYA-PAY type token"}

# Token Validation

## Validating KYA and PAY Tokens

### JWT Header Validation

1. `alg` - The `alg` header parameter MUST be present and, to enable
   interoperability, it is RECOMMENDED that its value be `ES256` {{RFC7518}}.
   Verifiers MUST reject any token whose `alg` value is not supported by both
   parties, including `none` {{RFC7515}}.

   Verifiers MUST use the verification algorithm in the token.
   The key is obtained from the
   issuer's JWK Set (item 2); a token is verified with that algorithm and that
   key, or rejected. In particular, a verifier MUST NOT accept a token that
   would require interpreting an Elliptic Curve public key as a symmetric key.

   Where the key retrieved from the issuer's JWK Set carries an `alg` or `use`
   parameter, the verifier MUST confirm it is consistent with
   the parameters of the issued token, and reject the token otherwise.

2. `kid` - The `kid` claim MUST be present, and set to a valid Key ID present in
   the issuer's JWK Set, located as described in {{key-discovery}}.
3. `typ` - The `typ` header parameter value MUST be one of: `kya+jwt`, `pay+jwt`, or `kya-pay+jwt`.

### JWT Payload Validation

1. **Verify JWT Signature** - Valid JWTs MUST be signed with a valid key belonging
  To the token's issuer (`iss` claim)
2. **Validate `iss` Claim** - Ensure that the token is signed by the expected
  valid issuer.
3. **Validate the `exp` Claim** - The verifier MUST validate that the token has
  not expired, within the verifier's clock drift tolerance.
4. **Validate the `iat` Claim** - The verifier MUST validate that the token was
  issued in the past, within the verifier's clock drift tolerance.
5. **Validate the `jti` Claim** - Ensure that the `jti` claim is present, and is
  a UUID.
6. **Validate the `aud` Claim** - Ensure that the `aud` identifies the recipient as the intended audience.
7. **Validate the `env` Claim** - Ensure that the Environment claim is set to
  an expected and use case appropriate value (such as `production` or `sandbox`)

## Validating PAY Tokens

For tokens of type `pay+jwt` or `kya-pay+jwt`, perform the steps described in
the Validating KYA and PAY Tokens section.

In addition, perform the following steps.

1. The `val` claim is greater than 0.
2. The `amt` claim is greater than 0.
3. The `cur` claim is set to a currency the target supports (such as `USD`)
4. The `tps` claim, if present, matches the pricing scheme that you configured in
  the target's service
5. The `tpr` claim, if present, matches the price that you configured in the
  target's service

# Security Considerations

## General JWT Practices

When validating the JWTs described in this specification, implementers SHOULD
follow the best practices and guidelines described in {{RFC8725}}.

## Confidentiality of Token Contents

The tokens defined in this specification are signed using JWS Compact
Serialization. Signing provides integrity protection, not confidentiality. The
claims in a token are encoded, not encrypted, and are readable by any party that
obtains the token.

Tokens defined in this specification carry identity claims about a human
principal and, in the case of PAY and KYA-PAY tokens, payment settlement
information. Accordingly:

* Tokens MUST be transmitted over a channel providing confidentiality, integrity
  and server authentication, such as TLS {{RFC8446}}.
* Tokens MUST NOT be written to logs, analytics pipelines, or other persistent
  stores that are not required to process the transaction.
* Tokens MUST NOT be forwarded to parties other than those necessary to complete
  the interaction for which they were issued.
* Issuers SHOULD include only those claims necessary for the interaction for
  which the token is issued. Confidentiality risk is reduced most reliably by not
  carrying a claim at all.

This specification does not define encryption of the token itself. Tokens are
intended to be read by multiple independent parties along a single request path
-- bot management and fraud systems at the edge, identity providers, and the
target service -- none of which is known to the issuer at the time the token is
minted, and between which no key-distribution relationship is assumed.
Encrypting the token to a single recipient would both require pre-established
key exchange with that recipient and withhold the token's contents from the
intermediaries that are its intended consumers. Confidentiality is therefore
provided at the transport layer and by minimising token contents, not at the
token layer.

The settlement information carried in a PAY or KYA-PAY token is additionally
protected by its own construction rather than by encryption: the credential is
transaction-scoped and its accompanying cryptogram is single-use, so a captured
token yields no reusable payment instrument.

## Settlement Credential Handling

The `sti` claim of a PAY or KYA-PAY token carries a settlement instrument. When
`stp` is `card`, that instrument is a network-issued agentic payment credential
provisioned for a single transaction, accompanied by a single-use dynamic
cryptogram. Such credentials are issued outside the scope of PCI DSS and cannot
be used to derive the underlying account number.

Implementations MUST NOT place a Primary Account Number, or a static Card
Verification Value, in the `sti` claim. Doing so would place the token, and
every system that processes it, within PCI DSS scope, for which this
specification provides no confidentiality mechanism.

## Token Lifetime

The `exp` claim of a token carrying a settlement instrument SHOULD NOT extend
beyond the validity period of that instrument. Agentic payment credentials and
their associated token cryptograms are single-use and short-lived; their
validity period is determined and enforced by the payment network. Issuing a
long-lived token around a short-lived credential provides no benefit and widens
the window in which a captured token can be replayed.

More generally, issuers SHOULD set `exp` to the shortest value compatible with
the intended interaction.

## Bearer Semantics and Proof of Possession

Tokens defined in this specification are bearer tokens unless they carry a
confirmation method: any party in possession of such a token can present it. For
bearer tokens, the protections above -- transport confidentiality, restricted
forwarding, short lifetimes, and audience restriction via the `aud` claim -- are
the primary defences against capture and replay.

A token MAY carry a `cnf` claim {{RFC7800}} binding it to a public key held by
the agent. A token carrying `cnf` is sender-constrained rather than bearer:
possession of the token alone is insufficient, and a verifier requires the
presenter to demonstrate possession of the confirmation key on each request.

Verifier behaviour for tokens carrying `cnf` -- including the requirement to
verify an HTTP Message Signature {{RFC9421}} over the confirmation key, and to
reject requests where that signature is absent or invalid -- is specified in
{{I-D.skyfire-oauth-using-kyapay-tokens}}.

Issuers SHOULD include `cnf` where the agent holds a suitable key and the target
is expected to enforce it. Issuers MUST NOT rely on `cnf` as a substitute for
the bearer-token protections above, since a token carrying `cnf` may still be
presented to a verifier that does not enforce it.

# Privacy Considerations

KYAPay tokens are designed to convey the information that
an agent is acting on behalf of a principal - a person or organization.
To do this, they will necessarily contain information about that principal
that can be verified and utilized by participants in the system.
Participants should therefore only share these tokens with other legitimate
participants and not make their contents public or disclose them to
unknown or untrustworthy parties.

Consent of the principal represented to participate in the interactions is vital.
If I authorize an agent to shop for a widget at given price,
it's legitimate for the agent to carry enough information about me
to the merchant to be able to do this for me.
Whereas, if an agent claims to be shopping for me but does not have my authorization
to do so, my privacy and possibly also my financial integrity are being violated.

The principle of minimal disclosure should be employed.
Only the information needed to facilitate the intended interactions
should be placed in the tokens and conveyed to participants.

# IANA Considerations

## JSON Web Token Claims Registration

This specification registers the following Claims in
the IANA "JSON Web Token Claims" registry {{IANA.JWT.Claims}}
established by {{RFC7519}}.

### "tdm" Claim

* Claim Name: tdm
* Claim Description: Target domain the token is intended for
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{common-claims}} of this specification

### "tsi" Claim

* Claim Name: tsi
* Claim Description: Target Service ID that this token was created for
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{common-claims}} of this specification

### "ori" Claim

* Claim Name: ori
* Claim Description: URL of the token's originator
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{common-claims}} of this specification

### "env" Claim

* Claim Name: env
* Claim Description: Issuer environment (such as "production" or "sandbox")
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{common-claims}} of this specification

### "itg" Claim

* Claim Name: itg
* Claim Description: Initiator tag, an opaque reference ID internal to the initiator
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{common-claims}} of this specification

### "hid" Claim

* Claim Name: hid
* Claim Description: JSON structure containing human identity claims
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{kya-token}} of this specification

### "apd" Claim

* Claim Name: apd
* Claim Description: JSON structure containing agent platform identity claims
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{kya-token}} of this specification

### "aid" Claim

* Claim Name: aid
* Claim Description: JSON structure containing agent identity claims
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{kya-token}} of this specification

### "tpr" Claim

* Claim Name: tpr
* Claim Description: JSON string representing target service price in currency units
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{pay-token}} of this specification

### "tps" Claim

* Claim Name: tps
* Claim Description: Target pricing scheme, which represents a way for the target list how it charges for its service or content
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{pay-token}} of this specification

### "amt" Claim

* Claim Name: amt
* Claim Description: JSON string representing token amount in currency units
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{pay-token}} of this specification

### "cur" Claim

* Claim Name: cur
* Claim Description: Currency unit, represented as an ISO 4217 three letter code, such as "EUR"
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{pay-token}} of this specification

### "val" Claim

* Claim Name: val
* Claim Description: JSON string representing token amount in settlement network's units
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{pay-token}} of this specification

### "mnr" Claim

* Claim Name: mnr
* Claim Description: JSON number representing maximum number of requests
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{pay-token}} of this specification

### "stp" Claim

* Claim Name: stp
* Claim Description: Settlement type
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{pay-token}} of this specification

### "sti" Claim

* Claim Name: sti
* Claim Description: Meta information for payment settlement, depending on settlement
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Reference: {{pay-token}} of this specification


## Media Types Registration

This section registers the following media types {{RFC2046}}
in the IANA "Media Types" registry {{IANA.MediaTypes}}
in the manner described in {{RFC6838}}.

### application/kya+jwt {#kya-jwt-media-type}

* Type name: `application`
* Subtype name: `kya+jwt`
* Required parameters: n/a
* Optional parameters: n/a
* Encoding considerations: Uses JWS Compact Serialization as defined in {{RFC7515}}
* Security considerations: See Security Considerations in in {{RFC7519}}
* Interoperability considerations: n/a
* Published specification: {{kya-token}} of this specification
* Applications that use this media type: Applications using Know Your Agent tokens
* Additional information:
  - Magic number(s): n/a
  - File extension(s): n/a
  - Macintosh file type code(s): n/a
* Person & email address to contact for further information: TBD
* Intended usage: COMMON
* Restrictions on usage: none
* Author: Michael B. Jones - michael_b_jones@hotmail.com
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com

### application/pay+jwt {#pay-jwt-media-type}

* Type name: `application`
* Subtype name: `pay+jwt`
* Required parameters: n/a
* Optional parameters: n/a
* Encoding considerations: Uses JWS Compact Serialization as defined in {{RFC7515}}
* Security considerations: See Security Considerations in in {{RFC7519}}
* Interoperability considerations: n/a
* Published specification: {{pay-token}} of this specification
* Applications that use this media type: Applications using Pay tokens
* Additional information:
  - Magic number(s): n/a
  - File extension(s): n/a
  - Macintosh file type code(s): n/a
* Person & email address to contact for further information: TBD
* Intended usage: COMMON
* Restrictions on usage: none
* Author: Michael B. Jones - michael_b_jones@hotmail.com
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com

### application/kya-pay+jwt {#kya-pay-jwt-media-type}

* Type name: `application`
* Subtype name: `kya-pay+jwt`
* Required parameters: n/a
* Optional parameters: n/a
* Encoding considerations: Uses JWS Compact Serialization as defined in {{RFC7515}}
* Security considerations: See Security Considerations in in {{RFC7519}}
* Interoperability considerations: n/a
* Published specification: {{kya-pay-token}} of this specification
* Applications that use this media type: Applications using KYA-Pay tokens
* Additional information:
  - Magic number(s): n/a
  - File extension(s): n/a
  - Macintosh file type code(s): n/a
* Person & email address to contact for further information: TBD
* Intended usage: COMMON
* Restrictions on usage: none
* Author: Michael B. Jones - michael_b_jones@hotmail.com
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com

## Well-Known URI Registration {#well-known-registration}

This specification registers the following URI suffix in the IANA
"Well-Known URIs" registry {{IANA.WellKnownURIs}} established by {{RFC8615}}.

* URI Suffix: jwks.json
* Change Controller: Michael B. Jones - michael_b_jones@hotmail.com
* Specification Document: {{key-discovery}} of this specification
* Status: permanent
* Related Information: The resource is a JWK Set {{RFC7517}} containing the
  keys with which the origin signs KYAPay tokens.


--- back

# Related Specifications

The following specifications are related to and designed to be used with this specification:

* {{I-D.skyfire-oauth-using-kyapay-tokens}} describes how security intermediaries -- bot managers, fraud managers, account-takeover (ATO) protection systems, and customer identity and access management (CIAM) systems -- consume KYAPay tokens to answer a question that traditional bot detection cannot: "Did a verified human authorize this agent?".
* {{I-D.skyfire-oauth-kyapay-token-exchange}} describes how KYAPay tokens can be exchanged for OAuth access tokens to dynamically grant agents access to resources they need to accomplish their mission.
* {{I-D.skyfire-oauth-amr-values}} defines additional "amr" (Authentication Method Reference) values to represent additional authentication methods in use today.
* {{I-D.skyfire-oauth-id-verification}} defines the "ivm" (Identity Verification Methods) claim and values for declaring how the person's identity was verified.
* {{I-D.skyfire-oauth-aml-methods}} defines the "aml" (Anti-Money Laundering Methods) claim and values for declaring what AML/CFT methods were employed.

# Document History
{: numbered="false"}

[[ to be removed by the RFC Editor before publication as an RFC ]]

-02

* Corrected the `sti` sub-claim names in the PAY and KYA-PAY examples to match
  the normative names in the Settlement Information section (`payment_token`,
  `token_expiration_month`, `token_expiration_year`, `token_security_code`).
  The examples had retained the pre-rename camelCase forms.
* Clarified that the card settlement instrument is a network-issued agentic
  payment credential and that `token_security_code` carries a single-use token
  cryptogram, not a static CVV.
* Made the `aid` and `sti` claims REQUIRED, and marked the `sti` sub-claim
  requirement levels individually.
* Added the `cnf` (confirmation) claim, binding a token to a key held by the
  agent, and described the resulting sender-constrained semantics.
* Added a Scope of Card Settlement section prohibiting Primary Account Numbers
  and static card verification values.
* Expanded Security Considerations to cover confidentiality of token contents,
  settlement credential handling, token lifetime, and bearer versus
  sender-constrained semantics.
* Tightened JWT header validation: an `alg` value of `none` and algorithm
  substitution are rejected.
* Corrected a missing comma in the KYA token example, and shortened the example
  token lifetimes, which had exceeded a year.
* Added {:vspace} syntax to definition list entries.
* Required the `iss` claim to be an origin, with no path, query or fragment
  component, and corrected the example `iss` values accordingly; one had lacked
  a scheme.
* Added an Issuer Key Discovery section specifying that the issuer's JWK Set is
  located at the `jwks.json` well-known URI formed from `iss`, and registered
  that URI suffix with IANA.

-01

* Added appendix referencing related specifications.

-00

* Renamed draft-skyfire-kyapayprofile to draft-skyfire-oauth-kyapay-token.

draft-skyfire-kyapayprofile-02

* Changed terms buyer and seller to initiator and target to generalize applicability.
  These claim names were changed: "sdm" to "tdm", "ssi" to "tsi", "btg" to "itg",
  "spr" to "tpr", and "sps" to "tps".
* Updated Settlement Information sti Sub-Claims.
* Changed specification type from Informative to Proposed Standard.

draft-skyfire-kyapayprofile-01

* Removed "srl" (Seller Resource Locator) claim.

draft-skyfire-kyapayprofile-00

* Initial Internet Draft.
