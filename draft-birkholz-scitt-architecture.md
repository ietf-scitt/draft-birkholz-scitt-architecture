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
  RFC9162: CT
  RFC6838:

informative:
  I-D.draft-birkholz-scitt-receipts:

--- abstract

Supply Chain transparency requires organizations to accurately identify and collect data from all links in the supply chain and to communicate this information both internally and externally at the level of detail required or desired. Supply Chain
transparency is of paramount importance in todays digital world, to address security challenges and risks posed to supply chain.

In this document, the supply chain context is illustrated using problem statements, requirements are derived from use case definitions, and architectural constituents are specified and illustrated in usage scenarios.

The resulting architecture is intended to enable multi-layer interoperability to produce and leverage believable trust assertions while maintaining a minimal adoption threshold.

--- middle

# Introduction

This document describes a scalable and flexible decentralized architecture to enhance auditability and accountability in various existing and emerging supply chains.
It achieves this goal by enforcing the following complementary security guarantees:

1. statements made by issuers about supply chain artifacts must be identifiable, authentic, and non-repudiable;
2. such statements must be registered on a secure append-only ledger so that their provenance and history can be independently and consistently audited;
3. issuers can efficiently prove to any other party the registration of their claims; verifying this proof ensures that the issuer is consistent and non-equivocal when making claims.

The first guarantee is achieved by requiring issuers to sign their statements and associated metadata using a distributed public key infrastructure. The second guarantee is achieved by storing the signed statement on an immutable, append-only transparency ledger. The last guarantee is achieved by implementing the ledger using a verifiable data strcture (such as a Merkle Tree), and by the requiring the transparency service that operates the ledger to endorse its state at the time of registration.

The guarantees and techniques used in this document generalize those of Certificate Transparency ({{-CT}}), which can be re-interpreted as an instance of this architecture for the supply chain of X.509 certificates. However, the range of use cases and applications in this document is much broader, which requires much more flexibility in how transparency services (TS) implement and operate their ledgers. Each service may enforce its own policy for authorizing entities to register their claims on the TS. Some TS may also enforce access control policies to limit who can audit the full ledger, or keep some information on the ledger encrypted. Nevertheless, it is critical to provide global interoperability for all TS instances as the composition and configuration of involved supply chain entities and their system components is ever changing and always in flux.

A TS provides visibility into claims issued by supply chain entities and their sub-systems. These claims are called Digital Supply Chain Artifacts (DSCA).
A TS vouches for specific and well-defined metadata about these DSCAs. Some metadata is selected (and signed) by the issuer, indicating, e.g., "who issued the DSCA" or "what type of DSCA is described" or "what is the DSCA version"; whereas additional metadata is selected (and countersigned) by the TS, indicating, e.g., "when was the DSCA registered in the ledger". The DSCA contents can be opaque to the TS, if so desired: it is the metadata that must always be transparent in order to  warrant trust.

Transparent claims provide a common basis for holding issuers accountable for the DSCA they release and (more generally) principals accountable for auxiliary claims they make about DSCAs. Hence, issuers may register new claims about their artifacts, but they cannot delete or alter earlier claims, or hide their claims from third parties such as auditors.

Trust in the TS itself is supported both by protecting their implementation (using replication and system attestation) and by enabling independent audits of the correctness and consistency of its ledger, thereby holding the organization that operates it accountable. Unlike CT, where independent auditors are responsible for enforcing the consistency of multiple independent instances of the same global ledger, we require each TS to guarantee the consistency of its own ledger (for instance, through the use of a consensus algorithm between replicas of the ledger), but assume no consistency between different transparency services.

The TS specified in this architecture caters two types of audiences:

1. DSCA Issuers: entities, stakeholders, and users involved in supply chain interactions that need to release DSCAs to a definable set of peers; and
2. DSCA Consumers: entities, stakeholders, and users involved in supply chain interactions that need to access, validate, and trust DSCAs.

DSCA Issuers rely on being discoverable and represented as the responsible parties for released DSCAs by the TS in a believable manner.
Analogously, DSCA Consumers rely on verifiable trustworthiness assertions associated with DSCAs and their processing in a believable manner.
If trust can be put into the operations that record DSCAs in a secure, append-only ledger via an online operation, the same trust can be put into a corresponding receipt that is the result of these online operations issued by the TS and that can be validated in offline operations.

The TS specified in this architecture can be implemented by various different types of services in various types of languages provided via various variants of API layouts.
The global interoperability enabled and guaranteed by the TS is enabled via core components (architectural constituents) that come with prescriptive requirements (that are typically hidden away from the user audience via APIs later). The core components are based on the Concise Signing and Encryption standard specified in {{-COSE}}, which is used to sign released DSCAs and to build and maintain a merkle tree that functions as the append-only ledger for DSCAs.
The format and verification process for ledger-based transparency receipts are described in [Counter-Signing Receipts](https://ietf-scitt.github.io/draft-birkholz-scitt-receipts/draft-birkholz-scitt-receipts.html)

## Requirements Notation

{::boilerplate bcp14-tagged}

# Use Cases

This section presents representative and solution-agnostic use cases to illustrate the scope of SCITT and the processing of Digital Supply Chain Artifacts.

## Software Bill of Materials (SBOM)

As the ever increasing complexity of large software projects requires more modularity and abstractions to manage, keeping track of their full Trusted Computing Base (TCB) is becoming increasingly difficult. Each component may have its own set of dependencies and libraries. Some of these dependencies are binaies, which means their TCB depends not only on their source, but also on the build environment (compilers and toolchains). Many source and binary packages are distributed through various channels and repositories that may not be trustworthy.

Software Bills of Materials (SBOM) help the authors, packagers, distributors, auditors and users of software understand its provenance and who may have the ability to introduce a vulnerability that can affect the supply chain downstream. However, the usefulness of SBOM in protecting end users is limited if supply chain actors cannot be held accountable for their contents. For instance, consider a package repository for an open source operating system distribution. The operator of this repository may decide to provide a malicious version of a package only to users who live in a specific country. They can write two equivocal SBOMs for the honest and backdoored versions of the package, so that nobody outside the affected country can discover the malicious version, but victims are not aware they are being targetted.

## Confidential Computing

Confidential Computing can leverage hardware-protected trusted execution environments (TEEs) to operate cloud services that protect the confidentiality of data that they process. It relies on remote attestation, which allows the service to prove to remote users what is hash of its code, as measured and signed by the hardware.

For instance, consider a speech recognition service that implements machine learning inference using a deep neural network model. The operator of the service wants to prove to its users that the service preserves the user's privacy, that is, the submitted recordings can only be used to detect voice commands but no other purpose (such as storing the recordings or detecting mentions of brand names for advertisement purposes).
When the user connects to the TEE implementing the service, the TEE presents attestation evidence that includes a hardware certificate and a software measurement for their task; the user verifies this evidence before sending its recording.

But how can users verify the software measurement for their task? And how can operators update their service, e.g., to mitigate security vulnerabilities or improve accuracy,
without first convincing all users to update the measurements they trust?

A supply chain that maintains a transparent record of the successive software releases for machine-learning models and runtimes, recording both their software measurements
and their provenance (source code, build reports, audit reports,...) can provide users with the information they need to authorize these tasks, while holding the service operator accountable for the software they release for them.

## Cold Chains for Seafood

Once seafood is caught, its quality is determined -- amongst other criteria -- via the integrity of a cold chain that ensures a regulatory perspective freshness mandating a continuous storing temperature between 1{{{°}}}C and 0{{{°}}}C (or -18{{{°}}}C and lower for frozen seafood). The temperature is recorded by cooling units adhering to certain compliance standards automatically. Batches of seafood can be split or aggregated before arriving in a shelf so that each unit can potentially have a potentially unique cold chain record whose transparency impacts the accuracy of the shelf-life associated with it. Especially in early links of the supply chain, Internet connection or sophisticated IT equipment are typically not available and sometimes temperature measurements are recorded manually and digital records are created in hindsight.

## Maintenance Telemetry from Truck Fleets

Measurements from Electronic Control Units (ECUs) that are components of fleets of trucks are recorded while they are on the road in support of supply chain logistics. Continuously recorded measurements are aggregated in batches by each truck and handed off to a telemetry store via mobile Internet or whenever the truck is in close proximity to a WiFi gate. Telemetry records stored are the fuel for machine learning approaches or prediction models based on AI methods. Telemetry records are considered confidential and are only exposed to other supply chain entities that are able to make their compliance to certain security standards transparent (or can present other contractual trust relationships).

# Terminology

The terms defined in this section have special meaning in the context of Supply Chain Integrity, Transparency, and Trust throughout this document. When used in text, the corresponding terms are capitalized. To ensure readability, only a core set of terms is included in this section.[^1]

[^1]: This list is a bucket today and subject to churn.
{: source="Henk"}

Artifact:

: the physical or non-physical item that is moving along the supply chain.

Statement:

: any serializable information about an Artifact. To help interpretation of statements, they must be tagged with a media type (as specified in {{RFC6838}})

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

: the process of submitting a claim to a transparency service, applying its registration policy, storing it in the ledger and producing the Receipt returned to the submitter.

Transparent Claim:

: a Claim that is augmented with a receipt of its registration. A Transparent Claim remains a valid Claim (as the receipt is carred in the countersignature), and may be registered again in a different TS.

Verifier:

: the entity that consumes Transparent Claims, verifying their proofs and inspecting their statements, either before using their Artifacts, or later to audit their supply chain.

{: #mybody}

# Definition of Transparency

In this document, we use a definition of transparency built over abstract notions of ledgers and receipts. Existing transparency systems such as Certificate Transparency are instances of this definition.

A *claim* is an identifiable and non-repudiable statement made by an *issuer*. The issuer selects additional metadata and attaches a proof of endorsement (in most cases, a signature) using the identity key of the issuer that binds the statement and its metadata. Claims can be made *transparent* by attaching a proof of registration by a TS, in the form of a receipt that countersigns the claim and witnesses its inclusion in the ledger of a TS. By extension, we may say an artifact (e.g. a firmware binary) is *transparent* if it comes with one or more transparent claims from its author or owner, though the contect should make it clear what type of claim is expected for a given artifact.

Transparency does not prevent dishonest or compromised issuers, but it holds them accountable: any artifact that may be used to target a particular user that checks for receipts must have been recorded in the tamper-proof ledger, and will be subject to scrutiny and auditing by other parties.

Transparency is implemented by a ledger that provides a consistent, append-only, publicly available record of entries. Implementations of TS may protect their ledger using a combination of trusted hardware, replication and consensus protocols, and cryptographic evidence. A receipt is an offline, universally-verifiable proof that an entry is recorded in the ledger. Receipts do not expire, but it is possible to append new entries that subsume older entries.

Anyone with access to the ledger can independenly verify its consistency and review the complete list of claims registered by each issuer. However, the ledgers of separate transparency services are generally disjoint, though it is possible to take a claim from one ledger and register it again on another (if its policy allows it), so the authorization of the issuer and of the ledger by the verifier of the receipt are generally independent.

Reputable issuers are thus incentivized to carefully review their statements before signing them into claims. Similarly, reputable TS are incentivized to secure their ledger, as any inconsistency can easily be pinpointed by any auditor with read access to the ledger. Some ledger formats may also support consistency auditing through receipts, that is, given two valid receipts the TS may be asked to produce a cryptographic proof that they are consistent. Failure to produce this proof can indicate that the TS operator misbehaved.

# Architecture Overview

The SCITT architecture consists of a very loose federation of transparency services, and a set of common formats and protocols for issuing, registering and auditing claims.
In order to accomodate as many TS implementations as possible, this document only specifies the format of claims (which must be used by all issuers) and a very thin wrapper format for receipts, which specifies the TS identity and the ledger algorithm. Most of the details of the receipt's contents are specific to the ledger algorithm. The [Counter-Signing Receipts](https://ietf-scitt.github.io/draft-birkholz-scitt-receipts/draft-birkholz-scitt-receipts.html) document defines two initial ledger algorithms (for historical and sparse Merkle Trees), but other ledger formats (such as blockchains, or hybrid historical and indexed Merkle Trees) may be proposed later.

In this section, we describe at a high level the three main roles in SCITT: issuers and the claim issuance process, transparency ledgers and the claim registration process, and verifiers and the receipt validation process.

## Claim Issuance

### Issuer Identity

Before an issuer is able to produce claims, it must first create its [decentralized identifier](https://www.w3.org/TR/did-core) (also known as a DID).
A DID can be *resolved* into a *key manifest* (a list of public keys indexed by a *key identifier*) using many different methods.

Issuers MAY chose the DID method they prefer, but with no guarantee that all TS will be able to register their claim. To facilitate interoperability, all transparency service implementations SHOULD suport the `did:web` method from [https://w3c-ccg.github.io/did-method-web/]. For instance, if the issuer publishes its manifest at `https://sample.issuer/user/alice/did.json`, the DID of the issuer is `did:web:sample.issuer:user:alice`.

Issuers SHOULD use a consistent decentralized identifiers for all their artifacts, to simplify authorization by verifiers and auditing. They MAY update their DID manifest, for instance to refresh their signing keys or algorithms, but they should not remove or change any prior keys unless they intend to revoke all claims issued with those keys. This DID appears in the `issuer` header of the claim's envelope, while the version of the key from the manifest used to sign the claim is written in the `kid` header.

### Naming artifacts

Many issuers issue claims about different artifacts under the same DID, so it is important for everyone to be able to immediately recognize by looking at the envelope of a claim what artifact it is referring to. This information is stored in the `feed` header of the envelope. Issuers MAY use different signing keys (identified by `kid` in the resolved key manifest) for different artifacts, or sign all claims under the same key.

### Claim metadata

Besides `issuer`, `feed` and `kid`, the only other mandatory metadata in the claim is the type of the payload, indicated in the `cty` envelope header.
However, this set of mandatory metadata is not sufficient to express many important registration policies. For example, a ledger may only allow a claim to be registered if it was signed recently. While the issuer is free to add any information in the payload of the claim, the TS (and most of its auditor) can only be expected to interpret information in the envelope.

Such metadata, meant to be interpreted by the TS during registration policy evaluation, should be added to the `reg_info` header. While the header MUST be present in all claims, its contents consist of a map of named attributes. Some attributes (such as the issuer's timestamp) are standardized with a defined type, to help uniformize their semantics across TS. Others are completely customizable and may have arbitrary types. In any case, all attributes are optional so the map MAY be empty.

## Transparency Service (TS)

The role of transparency service can be decomposed into several major functions. The most important is maintaining a ledger, the verifiable data structure that records claims, and enforcing a registration policy. It also maintains a service key, which is used to endorse the state of the ledger in receipts. All TS MUST expose standard endpoints for registration of claims and receipt issuance, which is described in [Section 9.1]. Each TS also defines its registration policy, which MUST apply to all entries in the ledger.

The combination of ledger, identity, registration policy evaluation, and registration endpoint constitute the trusted part of the transparency service. Each of these components SHOULD be carefully protected against both external attacks and internal misbehavior by some or all of the operators of the TS. For instance, the code for policy evaluation, ledger extension and endorsement may be protected by running in a TEE; the ledger may be replicated and a consensus algorithm such as PBFT be used to protect against malicious or vulnerable replicas; threshold signatures may be use to protect the service key, etc.

Beyond the trusted components, transparency services may operate additional endpoints for auditing, for instance to query for the history of claims made by a given issuer and feed. Implementations of TS SHOULD avoid using the service identity and extending the ledger in auditing endpoints; as much as practical, the ledger SHOULD contain enough evidence to re-construct verifiable proofs that the results returned by the auditing endpoint are consistent with a given state of the ledger.

### Service Identity and Keying

We assume that all TS have a public service identity, which must be known by verifiers when validating a receipt, and may have a corresponding private service key. Implementers of verifier libraries should not assume a service identity is necessarily a signing public keys, as it is technically possible to use other cryptographic techniques to endorse the ledger state in receipts.

#### Attestability of service identity

Enabling remote authentication of the hardware platforms and software TCB that run the transparency service.

  Hardware attestation report, binding a public key for receipt verification to the long-term transparency service identity.

  RATS? proof-of-work?

### Registration Policy

Each transparency service is parameterized by a Registration Policy
that takes as input the protected headers (but not the statement) of the claim to be registered
and the ledger (possibly including prior transparent claims from the same `issuer` and `feed`).

The `reg_info` header provides an extensible mechanism that lets the issuer selects which tagged inputs to include,
adnd thus parameterize the registration policy to be applied to each claim.

Future revisions of this note will specify a default policy that MUST be enforced for every tag present in `reg_info` and supported by the TS. For example, a `release_note` may be uninterpreted, whereas a `sequence number` may be accepted only if its value is either 1 if this is the first claim for this `feed` or n+1 if the last registered claim for this feed had `sequence number` n.

The transparency service MUST document its registration policy and the `reg_info` tags it supports, both to minimize the risk of rejecting claims presented by issuers, and to advertise the properties implied by receipt verification.

The transparency service MAY apply additional policies. As an example, a TS for a sofware provider may
additionally authenticate and authorize the parties that submit claims for registration.

### Ledger Requirements

There are many different candidate verifiable datat structures that may be used to implement the ledger, such as historical Merkle Trees, sparse Merkle Trees, full blockchains, and many other variants. SCITT requires the ledger to support concise receipts (i.e. whose size grows at most logarithmically in the number of entries in the ledger). This does not necessarily rule out blockchains as a ledger, but may necessitate advanced receipt schemes that use arguments of knowledge and other verifiable computing techniques.

Since the details of how to verify a receipt are specific to the data strcture, we do not specify any particular ledger format in this document. Instead, we propose two initial formats for ledgers in [draft-birkholw-scitt-receipts] using historical and sparse Merkle Trees. Beyond the format of receipts, we require generic properties that should be satisfied by the components in the TS that have the ability to write to the ledger.

#### Finality

The ledger is append-only: once a claim is registered, it cannot be modified, deleted, or moved. In particular, once a receipt is returned for a given claim, the claim and any preceding entry in the ledger become immutable, and the receipt provides universally-verifiable evidence of this property.

#### Consistency

There is no fork in the ledger: everyone with access to its contents sees the same sequence of entries, and can check its consistency with any receipts they have collected.
TS implementations SHOULD provide a mechanism to verify that the state of the ledger encoded in an old receipt is consistent with the current ledger state.

#### Replayability and Auditing

Everyone with access to the ledger can check the correctness of its contents. In particular,

- the TS defines and enforces deterministic registration policies that can be re-evaluated based solely on the contents of the ledger at the time of registraton, and must then yield the same result.

- The ordering of entries, their cryptographic contents, and the ledger governance may be non-deterministic, but they must be verifiable.

- The TS SHOULD store evidence about the resolution of distributed identifiers into manifests.

- The TS MAY additionally support verifiability of client authentication and access control.

#### Governance and Bootstrapping

The TS must support governance, with well-defined procedures for allocating resources to operate the ledger (e.g., for provisioning trusted hardware and registering their attestation materials in the ledger) and for updating its code (e.g., relying on transparent claims about code updates, secured on the ledger itself, or on some auxiliary TS ).

Governance procedures, their auditing, and their transparency are implementation specific. The TS SHOULD document them.

- Governance may be based in a consortium of members that are jointly responsible for the TS, or automated based on the contents of an auxiliary governance TS.

- Governance typically involves additional records in the ledger to enable its auditing. Hence, the ledger may contain both transparent claims and governance entries.

- Issuers, verifiers, and third-party auditors may review the TS governance before trusting the service, or on a regular basis.

## Verifying Transparent Claims

For a given Artifact, Verifiers take as trusted inputs
its Issuer identity and DID document,
its Feed, and
its TS receipt-verification key.

When presented with a transparent claim for the Artifact,
they verify its Issuer identity, signature, and receipt.
They may additionally apply a validation policy based on the protected headers
and the statement itself, which may include security-critical Artifact-specific details.

Some verifiers may systematically resolve the issuer DID to fetch
their latest DID document. This strictly enforces the revocation of compromised keys:
once the issuer has updated its document to mark a key as revoked,
all claims signed with this key will be rejected.
Others may delegate DID resolution to a trusted third party and/or cache its results.

Some verifiers may decide to skip the DID-based signature verification,
relying on the TS registration policy and the scrutiny of other verifiers.
Although this weakens their guarantees against key revocation, or against a corrupt TS,
they can still keep the receipt and blame the issuer or the TS at a later point.

# Claim Issuance, Registration, and Verification

> Now merging detailed syntax and protocols.

**Format of signed claims / envelope**

> Some headers that may be missing:
  - feed (enabling separation of ids for issuers and artifacts)#
  - svn (enabling versioning and rollback protection)
  - cty (contents type/format descriptor)
  - timestamp or serial number

>  Envelopes MAY include additional protected and unprotected headers.

## Issuer Identity

> To be split between architecture subsections

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

~~~~CDDL
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
~~~~

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

> Steps 2 and 3 are still TBD; the client should verify the issuer signature against the issuers' DID document at the time of registration.

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

> We may summarize in the architecture, and put the rest in an appendix.

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
