---
title: "KYAPay Profile"
#abbrev: ""
category: info

docname: draft-skyfire-kyapayprofile-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
#number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: TODO
keyword:
 - agent
venue:
  github: "skyfire-xyz/kyapay-ietf-draft"
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  latest: "https://skyfire-xyz.github.io/kyapay-ietf-draft/draft-skyfire-kyapayprofile.html"

author:
-
  name: Ankit Agarwal
  organization: Skyfire
  email: ankit@skyfire.xyz
-
  ins: M. Jones
  name: Michael B. Jones
  organization: Self-Issued Consulting
  email: michael_b_jones@hotmail.com
  uri: https://self-issued.info/

contributor:
    name: Dmitri Zagidulin
# see https://github.com/cabo/kramdown-rfc/wiki/Syntax2#authors-contributors

normative:
  RFC7518:
  RFC7519:
  RFC6749:
  RFC8693:
  OpenID.Core:
    author:
    - ins: N. Sakimura
      name: Nat Sakimura
    - ins: J. Bradley
      name: John Bradley
    - ins: M. Jones
      name: Michael B. Jones
    - ins: B. de Medeiros
      name: Breno de Medeiros
    - ins: C. Mortimore
      name: Chuck Mortimore
    date: December 2023
    target: https://openid.net/specs/openid-connect-core-1_0.html
    title: OpenID Connect Core 1.0 incorporating errata set 2

informative:
  RFC8725:

...

--- abstract

This document defines a profile for agent identity and payment tokens in
JSON web token (JWT) format. Authorization servers and resource servers from
different vendors can leverage this profile to consume identity and payment
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

Note that, in the future,
the payment token functionality could be split into a separate specification,
if desired by a working group adopting the specification.
It is retained here at present for ease of reviewing.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The terms `iss`, `iat`, `exp`, `jti`, `aud`, `typ` are defined in {{RFC7519}}.

The `alg` value `ES256` is a digital signature algorithm described in
{{Section 3.4 of RFC7518}}.

## Roles

Agent:
: An application, service, or specific software process, executing on behalf
  of a Principal.

Agent Identity:
: A unique identifier and a set of claims describing an agent. Grouped into the
  `aid` claim for convenience. Because an agent can be public or confidential
  (as described in {{Section 2.1 of RFC6749}}), the level of assurance for these
  claims varies dramatically. Agents also vary in terms of longevity -- they can
  have stable long-running identities (such as those of a server-side confidential
  client), or they can be transient and ephemeral, and correspond to individual
  API calls or compute workloads.

Agent Platform:
: The service provider and runtime environment hosting the Agent, such as a
  cloud compute provider or AI operator service. Assertions about the agent
  platform are grouped into the `apd` claim, and are primarily used to identify
  the Principal entity operating the platform, allowing consumers of the token to
  apply reputation-based logic or offer platform-specific services.

Principal:
: A legal entity (human or organization) on whose behalf / in whose authority
an agent or service is operating.

### Buy-Side Roles

Buyer Agent:
: An Agent performing tasks on behalf of a Buyer Principal, that has its own
  Agent Identity.

Buyer Agent Platform:
: The Agent Platform hosting the Buyer Agent. Some use cases require the Platform
  to have its own verified identity assertions, grouped into the `apd` claim.

Buyer Identity:
: The aggregate verified identity assertions of the buy-side entities, typically
  encompassing the Buyer Principal, the Buyer Agent Platform, and the Buyer Agent
  itself. This composite identity is conveyed via the KYA token, allowing the
  seller to verify the entire chain of responsibility behind a request.
  Grouped into the `bid` claim.

Buyer Principal:
: A legal entity (human or organization) behind the purchase / consumption of a
  product or service. The Principal typically interacts with the seller via a
  Buyer Agent. Many sellers are required to be able to determine the Buyer
  Identity in order to comply with KYC/AML regulations, accounting standards,
  and to maintain a direct customer relationships.

### Sell-Side Roles

Seller Agent:
: An Agent performing tasks on behalf of a Seller Principal, directly interacting
  with Buyer Agents to facilitate discovery and purchase. Typically runs on
  Internet-connected infrastructure, and discoverable via service directories.

Seller Agent Platform:
: The Agent Platform that hosts Seller Agents.

Seller Identity:
: The aggregate verified identity assertions of the sell-side entities, typically
  encompassing the Seller Principal, the Seller Agent Platform, as well as the
  Seller Agent Identity.
  These various aspects of Seller Identity allow Buyers and Buyer Agents to
  perform reputation-based logic, to verify that they are interacting with
  the authorized (and expected) counter-party, and to fulfill KYC/AML regulation
  requirements.

Seller Principal:
: A legal entity (human or organization) that owns the product, service, or
  content being sold, and serves as the ultimate beneficiary of a business
  transaction.

### Ecosystem Infrastructure Roles

Identity Token Issuer:
: A trusted neutral entity that conducts Know Your Customer (KYC) and Know Your
  Business (KYB) verifications. It is responsible for issuing cryptographically
  signed `kya` tokens that attest to the identity of the Principal, Agent, and Agent
  Platform, for both Buyers and Sellers.

Payment Token Issuer:
: A trusted entity responsible for facilitating the exchange of payments and
  credentials between the Buyer and Seller. It issues signed `pay` tokens that
  enable settlement via various schemes (Cards, Banks, Cryptocurrency), without
  exposing raw credentials or secrets.

# KYAPay Token Schemas

## Common Token Claims

The following are claims in common, used within the KYA (Know Your Agent),
PAY (Payment), and KYA-PAY (combined Know Your Agent and Payment) Tokens.

`iss`:
: REQUIRED - Url of the token's issuer. Used for discovering JWK Sets for token
  signature verification, via the `/.well-known/jwks.json` suffix mechanism.

`sub`:
: REQUIRED - Subject Identifier. Must be pairwise unique within
  a given issuer.

`aud`:
: REQUIRED - Audience (used for audience binding and replay attack mitigation),
  uniquely identifying the seller agent.
  A single string value.

`iat`:
: REQUIRED - as defined in {{Section 4.1.6 of RFC7519}}.  Identifies the time at which the JWT was issued.  This claim must have a value in the past and can be used to determine the age of the JWT.

`jti`:
: REQUIRED - Unique ID of this JWT as defined in {{Section 4.1.7 of RFC7519}}.

`exp`:
: REQUIRED - as defined in {{Section 4.1.4 of RFC7519}}.  Identifies the expiration time on or after which the JWT MUST NOT be accepted for processing.

`sdm`:
: OPTIONAL - Seller domain, associated with the audience claim, the token is intended for.

`srl`:
: OPTIONAL - Seller resource locator - the URL the agent is intended to access.

`ori`:
: OPTIONAL - URL of the token's originator.

`env`:
: OPTIONAL - Issuer environment (such as "sandbox" or "production").

`ver`:
: OPTIONAL - Version of the token schema.

`ssi`:
: OPTIONAL - Seller Service ID that this token was created for.

`btg`:
: OPTIONAL - Buyer tag, an opaque reference ID internal to the buyer.

## KYA Token

The following identity related claims are used within KYA and KYA-PAY tokens:

`bid`:
: REQUIRED (Required for buyer identity use cases) - A map of buyer identity
  claims.

`apd`:
: OPTIONAL - Agent Platform identity claims.

`aid`:
: REQUIRED - Agent identity claims.

`scope`
: OPTIONAL - String with space-separated scope values, per {{RFC8693}}

The following informative example displays a decoded KYA type token.

~~~
{
  "kid": "YjFdJgFNWj9AkUmtoXILwoeb37PsBuGWVK6_QvFLwJw", // JWK Key ID
  "alg": "ES256",
  "typ": "kya+jwt"
}.{
  "iss": "https://example.com/issuer", // Issuer URL
  "iat": 1742245254,
  "exp": 1773867654,
  "jti": "b9821893-7699-4d24-af06-803a6a16476b",
  "sub": "bb713104-c14e-460f-9b7c-f8140fa9bea4", // Buyer Agent Account ID
  "aud": "7434230d-0861-46f2-9c2c-a6ee33d07f17", // Seller Agent Account ID

  "env": "production",
  "ver": "1",
  "ssi": "bc3ff89f-069b-4383-82a9-8cfe53c55fc3", // Seller Service ID
  "btg": "4f6cbd39-215c-4516-bf33-cab22862ee60", // Buyer Tag (Internal Reference ID)

  "bid": {
    "email": "buyer@buyer.com”
  },
  "apd": {
    "id": "d3306fc0-602b-47e6-9fe2-3d55d028fbd2"
    "name": "Acme Shopping Agents", // Agent platform name
    "email": "platform@acme.com", // Email address for the agent platform
    "phone_number": "+12345677890", // Phone number for the agent platform
    "organization_name": "Acme Shopping Inc.", // Legal name of the agent platform
    "verifier": "https://www.verifier.com/", // URL of the Identity verifier
    "verification_status": "VERIFIED", // Outcome of the verifier's KYA
    "verification_id": "a23c1fe4-a4b7-442d-8bca-3c8fad5ec3a6" // Verifier's verification ID
  },
  "aid": {
    "name": "Acme Agent Extraordinaire",
    "creation_ip": "54.86.50.139", // IP Address where token was created
    "source_ips": ["54.86.50.139-54.86.50.141", "1.1.1.0/24",
      "2001:db8:abcd:0012::/64", "acme.com"]
      // IP addresses from which the buyer agent will make requests to the seller
  }
}
~~~
{: #example-decoded-kya-token align="left" title="A KYA type token"}

### `bid` - Buyer Identity Sub-claims

The Buyer Identity (`bid`) claim contains sub-claims useful for buyer use cases,
as follows.

`email`:
: REQUIRED - Buyer email.

#### OPTIONAL Human Principal Sub-claims

`given_name`:
: Given name(s) or first name(s) of buyer human principal.

`middle_name`:
: Middle name(s) of buyer human principal.

`family_name`:
: Surname(s) or last name(s) of buyer human principal.

`phone_number`:
: Phone number associated with principal.

`verifier`:
: URL of the Identity Verifier

`verification_status`:
: Verification status.  One of "VERIFIED", "UNVERIFIED".

`verification_id`:
: Verification identifier. Identifier for the verification performed, such as a GUID.

#### OPTIONAL Organizational or Business Entity Principal Sub-claims

`organization_name`:
: Name of principal entity.

`phone_number`:
: Phone number associated with principal.

`verifier`:
: URL of the Identity Verifier

`verification_status`:
: Verification status.  One of "VERIFIED", "UNVERIFIED".

`verification_id`:
: Verification identifier. Identifier for the verification performed, such as a GUID.

### Agent Platform Identity `apd` Sub-claims

The `apd` claim is optional. If present, it contains the following sub-claims.

`id`:
: REQUIRED - Agent Platform identifier.

`name`:
: REQUIRED - Agent Platform name.

`email`:
: OPTIONAL - Email associated with agent platform.

`phone_number`:
: OPTIONAL - Phone number associated with agent platform.

`organization_name`:
: OPTIONAL - Legal name associated with agent platform.

`verifier`:
: OPTIONAL - URL of the Identity Verifier

`verification_status`:
: OPTIONAL - Verification status.  One of "VERIFIED", "UNVERIFIED".

`verification_id`:
: OPTIONAL - Verification identifier. Identifier for the verification performed, such as a GUID.

### Agent Identity `aid` Sub-claims

The `aid` claim is optional. If present, it contains the following sub-claims.

`name`:
: REQUIRED - Agent name. The name should reflect the business purpose of the agent.

`creation_ip`:
: OPTIONAL - The public IP address of the system / agent that requested the token. Its value is a string containing the public IPv4 or IPv6 address from where the token request originated. It MUST be captured directly from the token request.

`source_ips`:
: OPTIONAL - Valid public IP address, or range of public IP addresses, from where the system / agent's requests to merchants / services will originate. Array of comma-separated IPv4 addresses or ranges, IPv6 addresses or ranges, or domain names resolvable to an IP address via DNS. IPv4 and IPv6 addresses can be a single IPv4 or IPv6 address or a range of IPv4 or IPv6 addresses in CIDR notation or start-and-end IP pairs.

## PAY Token

The following payment related claims are used within PAY and KYA-PAY type tokens:

`spr`:
: OPTIONAL - JSON string representing seller service price in currency units.

`sps`:
: OPTIONAL - Seller pricing scheme, which represents a way for the seller list how it charges for its service or content. One of `PAY_PER_USE`, `SUBSCRIPTION`, `PAY_PER_MB`, or `CUSTOM`.

`amount`:
: REQUIRED - JSON string representing token amount in currency units.

`cur`:
: REQUIRED - Currency unit, represented as an ISO 4217 three letter code, such as "EUR".

`value`:
: REQUIRED - JSON string representing token amount in settlement network's units.

`mnr`:
: OPTIONAL - JSON number representing maximum number of requests when `sps` is `PAY_PER_USE`.

`stp`:
: REQUIRED - Settlement type (one of `COIN` or `CARD`).

`sti`:
: REQUIRED - Meta information for payment settlement, depending on settlement.
  type.

### Agent Identity `sti` Sub-claims

The `sti` claim is optional. If present, it MAY contain the following sub-claims,
all of which are OPTIONAL.

`type`:
: REQUIRED - "type" is dependant on the "stp" value; for "COIN" - "USDC" or "x402"; for "CARD" - "VISA_VIC"

`paymentToken`:
: OPTIONAL - String containing Virtual Payment Card Number in ISO/IEC 7812 format. 12-19 characters.

`tokenExpirationMonth`:
: OPTIONAL - String containing two-digit Expiration Month Number.

`tokenExpirationYear`:
: OPTIONAL - String containing four-digit Expiration Year.

`tokenSecurityCode`:
: OPTIONAL - String containing 3 or 4 digit CVV code.


### PAY Token Example

The following informative example displays a decoded PAY type token.

~~~
{
  "kid": "FgT4q8c5IqbBCCjcho5JdeGQvuK1keMDFc9IwCm8J7Y", // JWK Key ID
  "alg": "ES256",
  "typ": "pay+jwt"
}.{
  "iss": "https://example.net/pay_token_issuer", // Issuer URL
  "iat": 1742245254,
  "exp": 1773867654,
  "jti": "b9821893-7699-4d24-af06-803a6a16476b",
  "sub": "8b810549-7443-494f-b4ad-5bc65871e32b", // Buyer Agent Account ID
  "aud": "37888095-2721-48d9-a2df-bfe4075f223a", // Seller Agent Account ID

  "env": "sandbox",
  "ver": "1",
  "ssi": "274efc47-024e-466f-b278-152d2ee73955", // Seller Service ID
  "btg": "16c135ce-a99a-453d-a7b5-4958fd91de5f", // Buyer Tag (Internal Reference ID)

  "spr": "0.01",
  "sps": "PAY_PER_USE",
  "amount": "15",
  "cur": "USD",
  "value": "15000000",
  "mnr": 1500,
  "stp": "CARD",
  "sti": {
    "type": "VISA_VIC",
    "paymentToken": "1234567890123456",
    "tokenExpirationMonth": "03",
    "tokenExpirationYear": "2030",
    "tokenSecurityCode": "123",
    "verifier": "https://verifier.example.info", // URL of payment method verifier
    "verification_status": "VERIFIED", // Outcome of the verifier's payment method verification - one of "VERIFIED", "UNVERIFIED" (OPTIONAL)
    "verification_id": "3a6e1b76-8f78-4c24-b1bd-dc78a8cc3711" // Identifier for the verification performed, such as a GUID.
  }
}

~~~
{: #example-decoded-pay-token align="left" title="A PAY type token"}

## KYA-PAY Token

The following informative example displays a decoded KYA-PAY type token.

~~~
{
  "kid": "YjFdJgFNWj9AkUmtoXILwoeb37PsBuGWVK6_QvFLwJw", // JWK Key ID
  "alg": "ES256",
  "typ": "kya-pay+jwt"
}.{
  "iss": "kya-pay.example.org", // Issuer URL
  "iat": 1742245254,
  "exp": 1773867654,
  "jti": "b9821893-7699-4d24-af06-803a6a16476b",
  "sub": "f24a431d-108c-46e6-9357-b428c528210e", // Buyer Agent Account ID
  "aud": "5e00177d-ff7f-424b-8c83-2756e15efbed", // Seller Agent Account ID

  "env": "production",
  "ver": "1",
  "ssi": "3e6d33a1-438e-482e-bba5-6aa69544727d", // Seller Service ID
  "btg": "c52e0ef2-e27d-4e95-862e-475a904ae7b2", // Buyer Tag (Internal Reference ID)

  "bid": {
    "email": "buyer@buyer.com”,
    ...
  },
  "apd": {
    "id": "4b087db2-b6e5-48b8-8737-1aa8ddf4c4fe", // Agent platform ID
    "name": "Acme Shopping Agents", // Agent platform name
    "email": "platform@acme.com", // Email address for the agent platform
    "phone_number": "+12345677890", // Phone number for the agent platform
    "organization_name": "Acme Shopping Inc.", // Legal name of the agent platform
    "verifier": "https://www.verifier.com/", // URL of the Identity verifier
    "verification_status": "VERIFIED", // Outcome of the verifier's KYA
    "verification_id": "a23c1fe4-a4b7-442d-8bca-3c8fad5ec3a6" // Verifier's verification ID
  },
  "aid": {
    "name": "Agentic Excellence Я Us",
    "creation_ip": "128.2.42.95", // IP Address where token was created
    "source_ips": ["54.86.50.139-54.86.50.141", "1.1.1.0/24",
      "2001:db8:abcd:0012::/64", "agentic-excellence.example.com"]
      // IP addresses from which the buyer agent will make requests to the seller
  },

  "spr": "0.01",
  "sps": "PAY_PER_USE",
  "amount": "15",
  "cur": "USD",
  "value": "15000000",
  "mnr": 1500,
  "stp": "CARD",
  "sti": {
    "type": "VISA_VIC",
    "paymentToken": "1234567890123456",
    "tokenExpirationMonth": "03",
    "tokenExpirationYear": "2030",
    "tokenSecurityCode": "123"
  }
}

~~~
{: #example-decoded-kya-pay-token align="left" title="A KYA-PAY type token"}

# Token Validation

## Validating KYA and PAY Tokens

### JWT Header Validation

1. `alg` - JWTs MUST be signed using allowed JWA algorithms (currently, `ES256`).
2. `kid` - The `kid` claim MUST be present, and set to a valid key id discoverable
   via the issuer's (payload `iss` claim) JWK Set.
3. `typ` - The `typ` claim MUST be one of: `kya+jwt`, `pay+jwt`, or `kya-pay+jwt`

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
6. **Validate the `aud` Claim** - ...
7. **Validate the `env` Claim** - Ensure that the Environment claim is set to
  an expected and usecase-appropriate value (such as `production`, `sandbox`, etc.)

## Validating PAY Tokens

For tokens of type `pay+jwt` or `kya-pay+jwt`, perform the steps described in
the Validating KYA and PAY Tokens section.

In addition, perform the following steps.

1. The `value` claim is greater than 0.
2. The `amount` claim is greater than 0.
3. The `cur` claim is set to a currency the seller supports (such as `USD`)
4. The `sps` claim, if present, matches the pricing scheme that you configured in
  the seller's service
5. The `spr` claim, if present, matches the price that you configured in the
  seller's service

# Security Considerations

When validating the JWTs described in this specification, implementers SHOULD
follow the best practices and guidelines laid out in {{RFC8725}}.

# IANA Considerations

// TBD Register newly defined claims

--- back
