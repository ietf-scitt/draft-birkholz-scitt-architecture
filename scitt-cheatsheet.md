# SCITT Cheat sheet

This is intended as a quick reference to the main concepts defined in the [SCITT architecture](https://github.com/ietf-scitt/draft-birkholz-scitt-architecture). 


## Roles

Issuer: A person or an Entity making Claims about an Artefact that is involed in a Supply Chain

Verifier: An independent Entity (a peice of software) that is Verifying  the presence of a Claim inside a Transparent Registry and thus establishing trustworthiness to the Supply Chain.

Auditor: A role that is responsible for checking the consistency and correctness of all the Claims in a Transparent Registry

Notary: The entity that verifies the identity of an issuer, submitting content to the system based on policy and issuing a receipt for valid entry into a registry

## Definition of Terms

Artefact: the physical or non-physical item that is moving along the supply chain.

Statement: any serializable information about an Artifact. To help interpretation of Statements, they must be tagged with a media type (as specified in [RFC6838]).

Claim: an identifiable and non-repudiable statement about an Artifact made by an Issuer. In SCITT, Claims are encoded as COSE signed objects; the payload of the COSE structure contains the Statement.

Envelope: The metadata added to the Statement by the Issuer to make it a Claim. It contains the identity of the Issuer and other information to help Verifiers identify the Artifact referred in the Statement. A Claim binds the Envelope to the Statement. In COSE, the Envelope consists of protected headers.

Endorsements: In SCITT terminology Endorsements can be any authenticated data that can be part of the SCITT Registry and can be verified independently using SCITT Receipts.

Attestations: In SCITT terminology, an attestation is authenticated metadata about one or more artefacts.

Transparent Registry:  The verifiable data structure that stores Claims in a transparency service. SCITT supports multiple Regitry formats to accommodate different transparency service implementations, such as historical Merkle Trees and sparse Merkle Trees.

Transparency Service: Entity that maintains and extends the Transparent Registry and endorses its state. A Transparency Service can be a complex distributed system, and SCITT requires the TS to provide many security guarantees about its Transparent Registries. TS responsibility is to define the policy that governs the execution of Transparent Registries. 

Receipt: Receipt is a special form of COSE countersignature for Claims that embeds cryptographic evidence that the Claim is recorded in the Transparent Registry. It consists of a Registry-specific inclusion proof, a signature by the Transparency Service of the state of the Registry, and additional metadata (contained in the countersignature protected headers) to assist in auditing.

Transparent Claims: A Claim that is augmented with a Receipt of its registration. 

Registration Policy: Configuration for the types of identities representing issuers that may be verified or rejected by the notary before been placed in the registry.

Appraisal Policy: Configuration for the Verification logic that evaluates various claims about an Artefact.

Software Bill Of Materials (SBOM): An SBOM is a nested inventory, a list of ingredients that make up software comopnents.