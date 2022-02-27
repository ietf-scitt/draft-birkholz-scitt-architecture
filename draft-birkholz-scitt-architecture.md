---
title: An Architecture for Trustworthy Digital Supply Chain Transparency Services
abbrev: SCITT Architecture
docname: draft-birkholz-scitt-architecture-latest
stand_alone: true
ipr: trust200902
area: Security
wg: TBD
kw: Internet-Draft
cat: std
pi:
  toc: yes
  sortrefs: yes
  symrefs: yes

author:
- ins: H. Birkholz
  name: Henk Birkholz
  org: Fraunhofer SIT
  abbrev: Fraunhofer SIT
  email: henk.birkholz@sit.fraunhofer.de
  street: Rheinstrasse 75
  code: '64295'
  city: Darmstadt
  country: Germany
- ins: A. Delignat-Lavaud
  name: Antoine Delignat-Lavaud
  organization: Microsoft
  email: antdl@microsoft.com
  country: UK
- ins: C. Fournet
  name: Cedric Fournet
  organization: Microsoft
  email: fournet@microsoft.com
  country: UK

normative:
  RFC8152: COSE

informative:
  I-D.draft-birkholz-scitt-receipts:

--- abstract
---
A transparent and authentic ledger service in support of a supply chain's integrity, transparency, and trust requires all peers that contribute to the ledgers operations to be trustworthy and authentic.

In this document, the supply chain context is illustrated using problem statements, requirements are derived from use case definitions, and architectural constituents are specified and illustrated in usage scenarios.

The resulting architecture is intended to enable multi-layer interoperability to produce and leverage believable trust assertions while maintaining a minimal adoption threshold.

--- middle

# Introduction

The need for an understandable, scalable and resilient system that provides trustworthy transparency for various kinds of existing and emerging supply chains is a global one.
This memo specifies an architecture for Transparency Service (TS) to systematically protect supply chains for digital artifacts.

Supply Chain Integrity, Transparency and Trust (SCITT) involves two complementary security guarantees:

1. artifacts must be authenticated by their issuers; and
2. an artifact's release must be recorded in a secure, append-only ledger, so that their provenance and release history can be independently reviewed.

Transparency in the context of supply chains is always a well-scoped quality for each instance of a TS.
Transparency does not imply being transparent to everybody, unconditionally.
Each instance may enforce its own policy for authorizing entities to register their claims on the TS. 
Nevertheless, it is of great import to provide global interoperability for all TS instances as the composition and configuration of involved supply chain entities and their system components is ever changing and always in flux.

A TS provides visibility into claims produced by supply chain entities and their sub-systems.
These claims are called Digital Supply Chain Artifacts (DSCA).
More importantly, a TS vouches for specific and well-defined metadata about these DSCAs, including "when was the DSCA recorded by the TS", "who issued the DSCA to the TS", or "what type of DSCA are stored in the TS".
Conversely, a DSCA itself can be opaque to the TS, if so desired.
It is the metadata that must always be transparent and that must warrant trust. That metadata includes distinct details and believable trustworthiness characteristics about the distinguishable system components that compose TS instances as well as their operations involving DSCA.

These trustworthiness assertions provide an essential basis for holding issuers accountable for the DSCA they release and (more generally) principals accountable for the claims they make about such DSCAs. 
Hence, issuers may register new claims about their artifacts, but they cannot delete or alter 
earlier claims, or hide their claims from third parties such as auditors. 

Crucially, trust in the TS itself can be supported by providing guarantees about their implementation, based on hardware attestation, and by holding them accountable, based on independent auditing of the correctness and consistency of their whole transparency ledger.  

The TS specified in this architecture caters two types of audiences:

1. DSCA Issuers: entities, stakeholders, and users involved in supply chain interactions that need to release DSCAs to a definable set of peers; and
2. DSCA Consumers: entities, stakeholders, and users involved in supply chain interactions that need to access, validate, and trust DSCAs.

DSCA Issuers rely on being discoverable and represented as the responsible parties for released DSCAs by the TS in a believable manner.
Analogously, DSCA Consumers rely on verifiable trustworthiness assertions associated with DSCAs and their processing in a believable manner.
If trust can be put into the operations that record DSCAs in a secure, append-only ledger via an online operation, the same trust can be put into a corresponding receipt that is the result of these online operations issued by the TS and that can be validated in offline operations.

The TS specified in this architecture can be implemented by various different types of services in various types of languages provided via various variants of API layouts.
The global interoperability enabled and guaranteed by the TS is enabled via core components (architectural constituents) that come with prescriptive requirements (that are typically hidden away from the user audience via APIs later).
The core components are based on the Concise Signing and Encryption standard specified in {{-COSE}}, which is used to sign released DSCAs and to build and maintain a merkle tree that functions as the append-only ledger for DSCAs.
The format and verification process for ledger-based transparency receipts are described in [Counter-Signing Receipts](https://ietf-scitt.github.io/draft-birkholz-scitt-receipts/draft-birkholz-scitt-receipts.html)

## Requirements Notation

{::boilerplate bcp14-tagged}

# Use Cases
> Henk has lock on this section. Allen also interested in writing here.
> The plan is to keep application details to this subsection.

- Public SBOM ledger
  > No need to separate firmware?.
  - from source code to binaries
  - can we keep track of the provenance of software artifacts?
  - can we share the cost of reviewing software across the industry?

- Confidential Computing
  - how can clients that connect to a CC service for the first time validate their attestation reports?
  - how to automatically patch a CC service? Trust, but audit later.
  - automated software updates
  - in particular for implementing SCITT services.

# Terminology

The terms defined in this section have special meaning in the context of Supply Chain Integrity, Transparency, and Trust throughout this document. When used in text, the corresponding terms are capitalized. To ensure readability, only a core set of terms is included in this section.[^1]

[^1]: This list is a bucket today and subject to churn.
{: source="Henk"}

Artifact:

: the physical or non-physical item that is moving along the supply chain.

Statement:

: any serializable information about an Artifact. To help interpretation of statement, they must be tagged with a media type (as specified in RFC6838) 

Claim:

: an identifiable and non-repudiable statement about an Artifact made by an Issuer. In SCITT, claims are encoded as COSE signed objects; the payload of the COSE structure contains the statement.

Issuer:

: the originator of named statements, which are signed into claims submitted to a Transparency Service for registration. The Issuer may be the owner or author of the Artifact, or a completely independent third party.

Envelope:

: the metadata added to the Statement by the Issuer to make it a Claim. It contains the identity of the issuer and other information to help Verifiers identify the Artifact referred in the statement. A Claim binds the Envelope to the statement. In COSE, the envelope consists of protected headers.

Feed:

: An identifier chosen by the Issuer for the Artifact. For every Issuer and Feed, the Ledger on a Transparency Service contains a sequence of claims about the same Artifact.
 In COSE, feed is one of the protected headers of the envelope.

Ledger:

: the verifiable data structure that stores Claims in a transparency service. SCITT supports multiple ledger formats to accomodate different transparency service implementations, such as historical Merkle Trees and sparse Merkle Trees.

Transparency Service:

: the entity that maintains and extends the Ledger, and endorses its state. A Transparency Service can be a complicated distributed system, and SCITT requires the TS to provide many security guarantees about its ledger. The identity of a TS is captured by a public key that must be known by Verifiers in order to validate Receipts.

Receipt:

: a Receipt is a special form of COSE countersignature for claims that embeds cryptographic evidence that the claim is recorded in the ledger. It consists of a ledger-specific inclusion proof, a signature by the transparency service of the state of the ledger, and additional metadata (contained in the countersignature protected headers) to assist in auditing.

Registration:

: the process of submitting a claim to a transparency service, storing it in the ledger and producing the Receipt returned to the submitter.

Transparent Claim:

: a Claim that is augmented with a receipt of its registration. A Transparent Claim remains a valid Claim (as the receipt is carred in the countersignature), and may be registered again in a different TS.

{: #mybody}

# Definition of Transparency
In this document, we use a definition of transparency built over abstract notions of ledgers and receipts. Existing transparency systems such as Certificate Transparency (CT) are instances of this definition.

A *claim* is an identifiable and non-repudiable statement made by an *issuer*.

- Claims can be issued and endorsed by principals (issuers).

- Signed Claims can be counter-signed by SCITT ledgers.

An artifact (e.g. a firmware binary) is transparent if it comes with valid transparent signed claims.

- The signature proves the artifact has been issued by the signer, with the associated metadata.

- The receipt proves this artifact and associated metadata has been recorded by the ledger.

Transparency does not prevent dishonest or compromised issuers, but it holds them accountable:
any artifact that may be used to target a particular user that checks for receipts must have been recorded in the tamper-proof ledger, and will be subject to scrutiny and auditing by other parties.

Anyone with access to the ledger can independenly verify the ledger consistency and review the list of claims registered by its issuers.

Reputable issuers are thus incentivized to carefully review their artifacts before signing them.


A ledger is a consistent, append-only, publicly available record of entries.

A receipt is an offline, universally-verifiable proof that an entry has been recorded in the ledger.

Receipts do not expire, but it is possible to append new entries that subsume older entries.

# High-level architecture

SCITT provides an interoperability framework to verify the transparency of arbitrary digital artifacts registered across many different ledgers. Although instances of SCITT transparency services may differ in their implementations, SCITT aims to enforce a common baseline accountability guarantee for auditors and consumers of transparency evidence.

## Principals

### Transparency services

As a decentralized system, SCITT allows anyone to operate their own instance of a transparency service, which maintains its own ledger

### Issuers and claims

Claims are non-repudiable statements made by issuers. In SCITT, many claims are made by authors, reviewers and distributors of digital artifacts, including source and binary packages, firmware, audit reports, etc.

### Verifiers
 including users, and anyone else


# SCITT Transparency Service

SCITT aims to support many different implementations of transparency ledgers, as long as they satisfy a set of requirements that aim to limit the trust that participants need to have in their operators. Unlike Certificate Transparency, we do not assume that the honestly of transparency services is enforced by external audit, as this approach does not scale to the large amount of instances necessary to protect the software supply chain at scale.

## Service Identity and Keying

> Details TBD, e.g. discovery, rekeying, revocation, delegation (e.g. to replicas); this may  require its own subsection, or left underspecified as ledger-specific details.

## Receipt Issuance

Compact, universally verifiable proof that claims are registered in the ledger.

Enabling offline verification of registration in the ledger. Signed with key that chains to service identity

## Ledger Security Requirements

The main requirements for the transparency service are

### Attestability of service identity

Enabling remote authentication of the hardware platforms and software TCB that run the transparency service.

  Hardware attestation report, binding a public key for receipt verification to the long-term transparency service identity.

  RATS? proof-of-work?

### Consistency
Everyone with access to the ledger sees the same sequence of registered entries.

### Finality
Ledger is append-only; once registered, its entries cannot be modified or deleted (although they may be superceded by nwer entries).

### Replayability

> Can it cover client authentication and DID resolution?

### Auditing

### Governance and Bootstrapping

  > Their specification is out of scope: we rely on ledger-specific, out-of-band protocols for it.

## Caching / Query service

Untrusted. Replicated. Indexed.

> This service could extract (read) receipts from the ledger.

# SCITT COSE Envelopes

**Format of signed claims / envelope**

> Some headers that may be missing:
  - feed (enabling separation of ids for issuers and artifacts)#
  - svn (enabling versioning and rollback protection)
  - cty (contents type/format descriptor)
  - timestamp or serial number

>  Envelopes MAY include additional protected and unprotected headers.

## Issuer Identity

SCITT issuers are identified using DID, which provides a flexible, decentralized identity framework.

The service MAY support the `did:web` method for bootstrapping identities from domain ownership via https certificates.

The service MUST resolve the issuer DID before registering their claims. 

The service SHOULD record a transcript of the DID resolution at the time of registration.

The service can cache and re-use DID resolution.
- Evidence capture?
- What does it mean in terms of transcript? 

> Details TBD. We could e.g. include a digest of the DID document at the time of registration in the leaf, or introduce another kind of record in the ledger. 

> The rest of this section is based on an earlier syntactic spec.

Digital supply chain artifacts are heterogeneous and originated from sources using various formats and encodings Large scale ledger services in support of supply chain authenticity and transparency require a simply and well-supported signing envelope that is easy to use and interoperates with the semantic of the ledger services. In this document, a COSE profile is defined that limits the potential use of a COSE envelope to the requirements of such a supply chain ledger, leveraging solutions and principles from the Concise Signing and Encryption (COSE) space.

### Why?

COSE is generic, every application needs to constrain it to their own use cases, in this case the SCITT use cases.

##  SCITT COSE profile


[TODO] describe semantics of `iss` and how it can be used together with `kid` to identity a verification method in a resolved DID document

A SCITT COSE message is a tagged COSE_Sign1 message.

The protected header must contain the following registered parameters:

- alg (label: `1`): Asymmetric signature algorithm as integer, for example `-35` for ECDSA with SHA-384, see [COSE Algorithms registry](https://www.iana.org/assignments/cose/cose.xhtml)
- payload type (label: `3`): Media type of payload as string, for example `application/spdx+json`
- issuer (label: `TBD`, to be registered): DID (Decentralized Identifier, see [W3C Candidate Recommendation](https://www.w3.org/TR/did-core/)) of the signer as string, for example `did:web:example.com`

Depending on the DID method the protected header must contain one or more of the following registered parameters:

- kid (label: `4`): Key identifier as a UTF-8 string encoded as a byte string, required for `did:web`
- x5c (label: `33`): X.509 certificate chain (including Root CA certificate), required for TBD


The unprotected header may contain the following parameters:

- receipts (label: `TBD`, to be registered): Array of SCITT receipts, defined in TBD

In CDDL ([RFC 8610](https://datatracker.ietf.org/doc/html/rfc8610)) notation, the envelope is defined as follows:

```
SCITT_Envelope = COSE_Sign1_Tagged

COSE_Sign1_Tagged = #6.18(COSE_Sign1)

COSE_Sign1 = [
  protected : bstr .cbor Protected_Header,
  unprotected : Unprotected_Header,
  payload : bstr,
  signature : bstr
]

Protected_Header = {
  ; mandatory
  1 => int               ; algorithm identifier
  3 => tstr              ; payload type
  258 => tstr            ; DID of issuer

  ; mandatory depending on DID method
  ? 4 => bstr            ; key identifier
  ? 33 => COSE_X509      ; X.509 certificate chain, excluding Root CA certificate

  ; optional experimental parameters
  ? -65537 => tstr       ; Feed
  ? -65538 => uint       ; SVN
}

Unprotected_Header = {
   ? 259 => SCITT_Receipt / [+ SCITT_Receipt]
}

COSE_X509 = bstr / [ 2*certs: bstr ]
```

# Protocols

## Issuing Signed Claims about an Artifact.

Issuance itself is just signing with a key currently associated with the issuer DID.

### SCITT is agnostic to claim types and formats (?)

> Are we going to specify formats for envelope payloads, aka "sets of claims" ? How are the types and formats below authenticated? We considered a "cty" protected header for that purpose.

Claim types include:
- SBOMs
- Malware scans
- Human audits
- Policies (= parameterized claims, with subject as input parameter)

Claim formats include:
- JSON-SPDX
- CBOR-SPDX
- SWID
- CoSWID
- CycloneDX

## Registering Signed Claims.

The same claim may be independently registered in multiple TS. 

To register a claim, the service performs the following steps: 

1. Client authentication.

   So far, implementation-specific, and unrelated to the issuer identity.

2. Issuer identification. The service must check that the ledger records a recent DID document for the `issuer` protected header of the envelope. This MAY require that the service resolve the issuer DID and record the resulting document. (See issuer identity above.)     

> Still missing any validation step involving prior claims, e.g., if the ledger already records any other claims from the same issuer with the same feed, check that the SVN of the new claim increments the claim of these prior claims.  

3. Envelope signature verification, as described in COSE signature, using the signature algorithm and verification key of the issuer DID document.  

4. Envelope validation. The service MUST check that the envelope has a payload and the protected headers listed above. The service MAY additionally verify the payload format and content. 

5. The service MAY apply an additional service-specific registration policy. The service SHOULD document this step, and MAY record additional evidence to enable its replayability.  

6. Commit to the ledger.

7. Sign and return receipt. 

The last two steps MAY be shared between a batch of claims recorded in the ledger. 

The service MUST ensure that the claim is committed before releasing its receipt, so that it can always back up the receipt by releasing the corresponding entry in the ledger. Conversely, the service MAY re-issue receipts for the ledger content, for instance after a transient fault during claim registration. 

## Verifying Transparent Signed Claims

Trusted input for receipt verification: the identity
and the public signature-verification key of the transparency service.

These may be included in the verifier's trusted configuration, or determined by a trusted policy.

Verification steps:

1. Verify receipt (see other draft)
2. Verify issuer signature.
3. Freshness/revocation?
4. Validate format of the envelope contents.

> Steps 2 and 3 are still TBD.

Once verified, the claims together with their authenticated issuer and transparent ledger identities can be used as input to an authorization policy.

# Federation

We explain how multiple, independent transparency services can be composed to distribute supply chains without a single transparency authority trusted by all parties.

> Mostly out of scope for the first drafts? We should make sure our architecture supports it.

Multiple SCITT instances, governed and operated by different organizations.

For example,
- a small, simple SCITT instance may keep track specifically of the software used for operating SCITT services.
- an air-gapped data center may operate its own SCITT ledger to retain full control and auditing of its software supplies.

How?
- Policy-based. Within an organization, local verifiers contact an authoritative SCITT that records the latest policies associated with classes of artifacts; these policies indicate which issuers and ledgers are trusted for verifying transparent signed claims for these artifacts.

- Other federation mechanisms?

We'd like to attach multiple receipts to the same signed claims, each receipt endorsing the issuer signature and a subset of prior receipts. This involves down-stream ledgers verifying and recording these receipts before issuing their own receipts.


# SCITT REST API

> We may omit most of the details, or put it in an appendix.

## Messages


### Register Signed Claims

#### Request

~~~
POST <Base URL>/entries
~~~

Body: SCITT COSE_Sign1 message

#### Response

One of the following:

- HTTP Status `201` - Registration was tentatively successful pending service consensus.
- HTTP Status `400` - Registration was unsuccessful.
  - Error code `AwaitingDIDResolutionTryLater`
  - Error code `InvalidInput`

[TODO] Use 5xx for AwaitingDIDResolutionTryLater

The `201` response contains the `x-ms-ccf-transaction-id`  HTTP header which can be used to retrieve the Registration Receipt with the given transaction ID. [TODO] this has to be made generic

[TODO] probably a bad idea to define a new header, or is it ok? can we register a new one? https://www.iana.org/assignments/http-fields/http-fields.xhtml

The `400` response has a `Content-Type: application/json` header and a body containing details about the error:

```json
{
  "error": {
    "code": "<error code>",
    "message": "<message>"
  }
}
```

`AwaitingDIDResolutionTryLater` means the service does not have an up-to-date DID document of the DID referenced in the Signed Claims but is performing or will perform a DID resolution after which the client may retry the request. The response may contain the HTTP header `Retry-After` to inform the client about the expected wait time.

`InvalidInput` means either the Signed Claims message is syntactically malformed, violates the signing profile (e.g. signing algorithm), or has an invalid signature relative to the currently resolved DID document.

### Retrieve Registration Receipt

#### Request

~~~
GET <Base URL>/entries/<Transaction ID>/receipt
~~~

#### Response

One of the following:

- HTTP Status `200` - Registration was successful and the receipt is returned.
- HTTP Status `400` - Transaction exists but does not correspond to a Registration Request.
  - Error code `TransactionMismatch`
- HTTP Status `404` - Transaction is pending, unknown, or invalid.
  - Error code `TransactionPendingOrUnknown`
  - Error code `TransactionInvalid`

The `200` response contains the SCITT_Receipt in the body.

The `400` and `404` responses return the error details as described earlier.

The retrieved receipt may be embedded in the corresponding COSE_Sign1 document in the unprotected header, see TBD.

[TODO] There's also the `GET <Base URL>/entries/<Transaction ID>` endpoint which returns the submitted COSE_Sign1 with the receipt already embedded. Is this useful?


# Privacy Considerations

Privacy Considerations

# Security Considerations

Security Considerations

# IANA Considerations

See Body {{mybody}}.

--- back

# Attic

Not ready to throw these texts into the trash bin yet.
