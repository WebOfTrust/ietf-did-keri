---
title: "The did:keri DID Method"
abbrev: "DID-KERI"
docname: draft-pfeairheller-did-keri-latest
category: info

ipr: trust200902
area: TODO
workgroup: TODO Working Group
keyword: Internet-Draft

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
-
    name: Phil Feairheller
    organization: GLEIF
    email: Philip.Feairheller@gleif.org

normative:
  CESR:
    target: https://datatracker.ietf.org/doc/draft-ssmith-cesr/
    title: Composable Event Streaming Representation (CESR)
    author:
      ins: S. Smith
      name: Samuel M. Smith
      org: ProSapien LLC
    date: 2021

informative:
  KERI:
    target: https://arxiv.org/abs/1907.02143
    title: Key Event Receipt Infrastructure (KERI)
    author:
      ins: S. Smith
      name: Samuel M. Smith
      org: ProSapien LLC
    date: 2021



tags: IETF, KERI, CESR

--- abstract

KERI provides a means for secure and decentralised key management. This specification defines a DID method based on KERI.


--- middle

# Introduction

The Key Event Receipt Infrastructure is a system for secure self-certifying Identifiers which aims at minimum sufficiency and maximum security. It defines mechanisms for proving the Root of Trust for self-certifying Identifiers and their associated Key State. This spec defines a transform from Key State to DID Document, such that any valid Key Event Log can be processed into a DID Document.

A close analogy is did:peer, except that where the data model of did:peer is a DID Document and JSON patches on said Document, the basic data model of did:keri is an append-only log of Key Events. KERI-based Identifiers are suitable for both any-wise and n-wise purposes.

# Concepts

## Events

A Key Event is an atomic transaction over the Key State of an Identifier. While all events have some semantic meaning within KERI, only a subset will change the keys in a Key State (rotation and delegated rotation).

## Key Event Log

The Key Event Log is a type of hash-chained data structure from which the Key State of an Identifier can be derived. It can always be used to recreate the state at any point ("event-sourcing").

## Key Event Receipt Log

The Key Event Receipt Logs are built from receipts of events signed by the witnesses of those events (these are called `commitments`); these are also append-only but not hash-chained.

## Key State

Key State represents the current values for the keys, witnesses and thresholds for a given identifier, signed by a provider. The signature from the provider signifies that the provider has verified the KEL for the identifier and the result of that verification is the key state. The key state is represented in a Key State Notification Message detailed fully in [Event Serialization Key State Messages](https://github.com/decentralized-identity/keri/blob/master/kids/kid0003.md#key-state-messages). The following fields are defined for a Key State Notification Message:

*   `v`: Version String
*   `i`: Identifier Prefix
*   `s`: Sequence Number
*   `t`: Message Type
*   `d`: Event Digest (Seal or Receipt)
*   `p`: Prior Event Digest
*   `kt`: Keys Signing Threshold
*   `k`: List of Signing Keys (ordered key set)
*   `n`: Next Key Set Commitment
*   `wt`: Witnessing Threshold
*   `w`: List of Witnesses (ordered witness set)
*   `wr`: List of Witnesses to Remove (ordered witness set)
*   `wa`: List of Witnesses to Add (ordered witness set)
*   `c`: List of Configuration Traits/Modes
*   `a`: List of Anchors (seals)
*   `da`: Delegator Anchor Seal in Delegated Event (Location Seal)
*   `di`: Delegator Identifier Prefix in Key State
*   `rd`: Merkle Tree Root Digest
*   `e`: Last received Event Map in Key State
*   `ee`: Last Establishment Event Map in Key State
*   `vn`: Version Number ("major.minor")

Key state notification messages differ depending on whether the signer is using a delegated identifier. The follow examples detail the fields needed for each permutation.

            {
              "v": "KERI10JSON00011c\_",
              "i": "EaU6JR2nmwyZ-i0d8JZAoTNZH3ULvYAfSVPzhzS6b5CM",
              "s": "2",
              "t": "ksn",
              "d": "EAoTNZH3ULvaU6JR2nmwyYAfSVPzhzZ-i0d8JZS6b5CM",
              "te": "rot",
              "kt": "1",
              "k": \["DaU6JR2nmwyZ-i0d8JZAoTNZH3ULvYAfSVPzhzS6b5CM"\],
              "n": "EZ-i0d8JZAoTNZH3ULvaU6JR2nmwyYAfSVPzhzS6b5CM",
              "wt": "1",
              "w": \["DnmwyYAfSVPzhzS6b5CMZ-i0d8JZAoTNZH3ULvaU6JR2"\],
              "c": \["eo"\],
              "ee":
                {
                  "s":  "1",
                  "d":  "EAoTNZH3ULvaU6JR2nmwyYAfSVPzhzZ-i0d8JZS6b5CM",
                  "wr": \["Dd8JZAoTNZH3ULvaU6JR2nmwyYAfSVPzhzS6b5CMZ-i0"\],
                  "wa": \["DnmwyYAfSVPzhzS6b5CMZ-i0d8JZAoTNZH3ULvaU6JR2"\]
                },
              "di": "EJZAoTNZH3ULvYAfSVPzhzS6b5CMaU6JR2nmwyZ-i0d8"
            }


            {
              "v": "KERI10JSON00011c\_",
              "i": "EaU6JR2nmwyZ-i0d8JZAoTNZH3ULvYAfSVPzhzS6b5CM",
              "s": "2",
              "t": "ksn",
              "d": "EAoTNZH3ULvaU6JR2nmwyYAfSVPzhzZ-i0d8JZS6b5CM",
              "te": "rot",
              "kt": "1",
              "k": \["DaU6JR2nmwyZ-i0d8JZAoTNZH3ULvYAfSVPzhzS6b5CM"\],
              "n": "EZ-i0d8JZAoTNZH3ULvaU6JR2nmwyYAfSVPzhzS6b5CM",
              "wt": "1",
              "w": \["DnmwyYAfSVPzhzS6b5CMZ-i0d8JZAoTNZH3ULvaU6JR2"\],
              "c": \["eo"\],
              "ee":
                {
                  "s":  "1",
                  "d":  "EAoTNZH3ULvaU6JR2nmwyYAfSVPzhzZ-i0d8JZS6b5CM",
                  "wr": \["Dd8JZAoTNZH3ULvaU6JR2nmwyYAfSVPzhzS6b5CMZ-i0"\],
                  "wa": \["DnmwyYAfSVPzhzS6b5CMZ-i0d8JZAoTNZH3ULvaU6JR2"\]
                },
              "di": "EJZAoTNZH3ULvYAfSVPzhzS6b5CMaU6JR2nmwyZ-i0d8"
            }


## Resolver Metadata

did:keri defines `keyState` DID Document Metadata (see [DID Document Metadata](https://w3c-ccg.github.io/did-resolution/#output-documentmetadata) in \[\[?DID-RESOLUTION\]\]).

*   `keyState` is the verified state of the KEL for the identifier represented by this DID Doc (See [Key State]()).

        {
            "didDocument": DID\_DOCUMENT\_OBJECT,
            "didDocumentMetadata": {
                "keyState":  {
                  "v": "KERI10JSON00011c\_",
                  "i": "EaU6JR2nmwyZ-i0d8JZAoTNZH3ULvYAfSVPzhzS6b5CM",
                  "t": "ksn",
                  "kt": "1",
                  "k": \["DaU6JR2nmwyZ-i0d8JZAoTNZH3ULvYAfSVPzhzS6b5CM"\],
                  "n": "EZ-i0d8JZAoTNZH3ULvaU6JR2nmwyYAfSVPzhzS6b5CM",
                  "wt": "1",
                  "w": \["DnmwyYAfSVPzhzS6b5CMZ-i0d8JZAoTNZH3ULvaU6JR2"\],
                  "c": \["eo"\],
                  "e":
                    {
                      "s": "2",
                      "t": "rot",
                      "d": "EAoTNZH3ULvaU6JR2nmwyYAfSVPzhzZ-i0d8JZS6b5CM",
                    },
                  "ee":
                    {
                      "s": "1",
                      "d": "EAoTNZH3ULvaU6JR2nmwyYAfSVPzhzZ-i0d8JZS6b5CM",
                      "wr": \["Dd8JZAoTNZH3ULvaU6JR2nmwyYAfSVPzhzS6b5CMZ-i0"\],
                      "wa": \["DnmwyYAfSVPzhzS6b5CMZ-i0d8JZAoTNZH3ULvaU6JR2"\]
                    },
                  "di": "",
                  "a": {}
                }
            }
        }


## The DID Document

The following field MAY appear in a did:keri DID document: `verificationMethod`

Non-normative note: multiple namespaces using the same DID method could prove control or replay history for one controlling keypair that has been used across multiple ledgers. KERL logs could be stored in a secondary root of trust (i.e., a ledger), and similar or identical resolver metadata could be returned by querying the same identifier there.

The following field is not a core field in a did:keri DID document: `Services`. The did:keri method is provided for interoperability for the purpose of using KERI to establish control authority of the current public keys associated with KERI identifier behind the keri:did DID. It is anticipated that KERI tunnel methods (eg. did:indy:sov:keri) will provide these features to enable addition interop.

# The did:keri Format

The format for the did:keri method conforms to the \[\[DID-CORE\]\] specification and is simple. It consists of the `did:keri:` prefix, followed by the identifier prefix.

## Method Name

The method name that identifies this DID method SHALL be: `keri`

A DID that uses this method MUST begin with the following prefix: `did:keri:`. Per the DID specification, this string MUST be in lowercase. The remainder of the DID, after the prefix, is the method specific identifier (MSI) described below.

## Method Specific Identifier

The method specific identifier for the did:keri method is the prefix for a content self-addressing self-certifying identifier.

A self-addressing, self-certifying identifier is cryptographically bound to the inception keys used to create it. The rationale and process for the derivation of an identifier is described in detail in the [Derivation Codes](https://github.com/decentralized-identity/keri/blob/master/kids/kid0001.md#derivation-codes) section of \[\[KID0001\]\]

         did:keri:EXq5YqaL6L48pf0fu7IUhL0JRaU2\_RxFP0AL43wYn148


# Operations

The following section outlines the DID operations for the did:keri method.

## Create

Creation of a did:keri DID is accomplished by creating, signing and publishing an Inception event. If witnesses are listed in the inception event, the receipts are also required for DID creation to be complete.

Detailed steps for prefix derivation are in \[\[KID0001\]\] and witness configuration in \[\[KID0009\]\]. Inception events are covered in \[\[KID0003\]\].

## Read

Steps to resolve a \`did:keri:$PREFIX\` DID:

1.  Find the Key Event Log for the prefix \`$PREFIX\`. The method for discovering the Key Event Log is outside the scope of this specification. Possible implementations include the use of a Distributed Hash Table (DHT), anchoring a KEL in a ledger or the use of a gossip protocol involving a witness network.
2.  Process the KEL into a Key State, according to the state validation rules/semantics of KERI as defined in \[\[KID0008\]\]
3.  Create a DID Document with the DID \`did:keri:$PREF\`
4.  For each key K in the Key State, add a Verification Method to the DID Doc containing K and the appropriate type, with the ID of K being K's prefix form (alternatively, the ID could be an integer of K's index in the current signer list)
5.  Add the Key State as DID Document Metadata as defined in[Resolver Metadata]()

Establishment of control authority can be done independently of DID document contents, as long as the Key State is provided in the DID Document Metadata. See [Resolver Metadata]().

## Update

Updating a did:keri DID is accomplished by publishing establishment events to the KEL for performing operations such as key rotation and updating signature thresholds, witnesses and delegates.

A detailed description of event types, their semantics and uses can be found in \[\[KID0003\]\].

## Deactivate

Deactivation of a did:keri DID consists of rotation to 0 controlling keys, which terminates the ability to recover the identifier and indicates that the identifier has been abandoned. Identifiers which are delegated to by an abandoned Identifier are also considered abandoned (delegating Ixn events can no longer be created).

Detailed steps are specified in \[\[KID0003\]\].

# Privacy Considerations

A breakdown of the privacy considerations from \[\[RFC6973\]\] section 5 is provided below.

## Surveillance

A robust witness network along with consistent witness rotation provides protection from monitoring and association of an individual's activity inside a KERI network.

## Stored Data Compromise

For resolvers that simply discover the Key State endorsed by another party in a discovery network, caching policies of that network would guide stored data security considerations. In the event that a resolver is also the endorsing party, meaning they have their own KERI identifier and are verifying the KEL and signing the Key State themselves, leveraging the facilities provided by the KERI protocol (key rotation, witness maintenance, multi-sig) should be used to protect the identities used to sign the Key State.

See \[\[KID0005\]\] for information on KERI key rotation, \[\[KID0009\]\] for a discussion on witnesses and \[\[KID0004\]\] for KERI's support of multi-sig.

## Unsolicited Traffic

DID Documents are not required to provide endpoints and thus not subject to unsolicited traffic.

## Misattribution

This DID Method relies on KERI's duplicity detection to determine when the non-repudiable controller of a DID has been inconsistent and can no longer be trusted. This establishment of non-repudiation enables consistent attribution.

See \[\[KID0010\]\] for a detailed description of KERI’S Agreement Algorithm for Control Establishment (KAACE).

## Correlation

The root of trust for KERI identifiers is entropy and therefore offers no direct means of correlation. In addition, KERI provides two modes of communication, direct mode and indirect mode. Direct mode allows for pairwise (n-wise as well) relationships that can be used to establish private relationships.

See \[\[KID0001\]\] for a description of KID prefix generation and \[\[KID0009\]\] for a comparison between Direct and Indirect modes.

## Identification

The root of trust for KERI identifiers is entropy and therefore offers no direct means of identification. In addition, KERI provides two modes of communication, direct mode and indirect mode. Direct mode allows for pairwise (n-wise as well) relationships that can be used to establish private relationships.

See \[\[KID0001\]\] for a description of KID prefix generation and \[\[KID0009\]\] for a comparison between Direct and Indirect modes.

## Secondary Use

The Key State made available in the metadata of this DID method is generally available and can be used by any party to retrieve and verify the state of the KERL for the given identifier.

## Disclosure

No data beyond the Key State for the identifier is provided by this DID method.

## Exclusion

This DID method provides no opportunity for correlation (See [](#correlation)), identification (See [](#identification)) or disclosure (See [](#disclosure)) and therefore there is no opportunity to exclude the controller from knowing about data that others have about them.

# Security Considerations

## Key State Verification

Users of a did:keri did method resolver MUST verify the key state returned in the document metadata of the resolution result. The signature of the resolver can be used to determine if the resolver is dysfunctional and should no longer be trusted. However it should not be used to verify the key state.

The only definitive method for verifying the key state is to pass the key state to a KERI library and perform the verification of that key state.

A breakdown of the security considerations from \[\[RFC3552\]\] is provided below.

## Confidentiality Violations, Password Sniffing

All private keys used to establish control over the KERI identifier of a KERI DID method DID should be held secret. KERI's use of delegation ensures that private keys for each identifier never need to be transferred. DID controllers can delegate levels of authority of other identities to enable remote agents.

See \[\[KID0007\]\] for a discussion of delegation.

In addition, pre-rotation mechanism provides key rotation capabilities while eliminating exposure of the next public key until it is rotated into the current signing key. Enforcing key rotation for every event of a given identifier provides further protection against key exposure.

See \[\[KID0005\]\] for a description of pre-rotation and the protections it provides

## Replay Attacks

did:keri relies on the KERI protocol which is not susceptible to replay attacks. The hash linking, signatures and sequence numbers of events ensures that replayed messages do not effect the protocol.

See \[\[KID0003\]\] for a description of KERI events.

## Message Insertion, Deletion, Modification

did:keri relies on the KERI protocol which is not susceptible to message deletion or modification attacks. The hash linking, signatures and sequence numbers of events ensures that messages can not be modified or deleted without immediate detection. In addition, KERI's duplicity detection mechanisms allow easy detection of inserted messages allowing validators to determine the consistency of a DID controller.

See \[\[KID0003\]\] for a description of KERI events and \[\[KID0010\]\] for the KAACE algorithm.

## Man-In-The-Middle Attacks

The protections mentioned in [Replay Attacks]()and [Message Attacks]()render Man-In-The-Middle attacks ineffective against the KERI protocol and this DID Method.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
