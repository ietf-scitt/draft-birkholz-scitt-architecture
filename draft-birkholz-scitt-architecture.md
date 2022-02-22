---
title: An Architecture for Trustworthy Supply Chain Ledger Services
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
- ins: A. Delignat
  name: Antoine Delignat
  organization: Microsoft
  email: antdl@microsoft.com
  country: UK

normative:

informative:

--- abstract

A transparent and authentic ledger service in support of a supply chain's integrity, transparency, and trust requires all peers that contribute to the ledgers operations to be trustworthy and authentic. In this document, the supply chain context is illustrated using problem statements, requirements are derived from use case definitions, and architectural constituents are specified and illustrated in usage scenarios. The resulting architecture is intended to enable multi-layer interoperability to produce and leverage believable trust assertions while maintaining a minimal adoption threshold.

--- middle

# Introduction

The need for an understandable, scalable and resilient system that provides trustworthy transparency for various kinds of existing and emerging supply chains is a global one. This memo specifies such a system called a Transparency Ledger (TL). Transparency in the context supply chains always a well-scoped quality for each instance of a TL. Transparency does not imply everybody, unconditionally. A TL always limits the entities with the authority to include claims into the TL system. On the other hand. Analogously, a TL typically limits the entities to which transparency is granted. Nevertheless, it is of great import to provide global interoperability for a multitude of instances as the composition and configuration of supply chains is ever changing and always in flux.

A TL provides visibility into claims made by supply chain entities and their sub-system. These claims Digital Supply Chain Artifacts (DSCA). More importantly, a TL vouches for specific and well-defined metadata about these claims, including "when was the DSCA recorded by the TL", "who issued the DSCA to the TL", or "what type of DSCA are stored in the TL". In fact, a DSCA can be opaque to the TL, if so desired. The metadata that must be transparent and that has to warrant trust are about how DSCAs are handled by a component of the TL system, its inherent trustworthiness characteristics, and a record of the timeliness and accuracy of its operations.

## Requirements Notation

{::boilerplate bcp14-tagged}

# Use Cases



{: #mybody}
# Digital Artifacts in Supply Chains

Context

# Privacy Considerations

Privacy Considerations

# Security Considerations

Security Considerations

# IANA Considerations

See Body {{mybody}}.

--- back

# Attic

Not ready to throw these texts into the trash bin yet.
