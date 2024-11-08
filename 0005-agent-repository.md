---
RIP: 5
Title: Agent Repository
Author: '@fintohaps <fintan.halpenny@gmail.com>, @lorenz <lorenz@leutgeb.xyz>'
Status: Draft
Created: 2024-10-23
License: CC0-1.0
---

RIP #5: Agent Repository
========================
This RIP introduces the concept of an agent repository, a specialized kind of
repository, into the Heartwood protocol. The aim is to define a repository that
can represent agents in the network, holding any required metadata that helps
define this agent.

**Agent**: An agent is any entity that we want to identify uniquely, interacting
with others on the Radicle network. Human users are considered agents, but we
also want to consider computers as agents, e.g. CI/CD systems, security bots,
etc.

Motivation
----------
The motivation for this RIP is to define the foundational element of the agent
repository, to enable us to further work on more refined approaches to user
identity and multi-device support – this RIP aims to set the stage for these
future RIPs.

The initial approach was exploring how to approach multi-device support, and
after some thought and discussions, we decided that it would be best to define
multiple RIPs in order to achieve our end goals.

The agent repository came from the idea that we have a generic component that
only needs minimal definition, specified here, and leaves room for extension
points for later RIPs – including KERI and multi-device support.

Table of Contents
-----------------

1. [RIP #5: Agent Repository](#rip-5-agent-repository)
2. [Motivation](#motivation)
3. [Overview](#overview)
4. [The Agent Repository](#the-agent-repository)
   - [Identity Document](#identity-document)
   - [Storage](#storage)
5. [Agent Repository References](#agent-repository-references)
   - [DID Namespace](#did-namespace)
     * [Methods](#methods)
     * [Signing and Verification](#signing-and-verification)
     * [Example: `did:key`](#example-didkey)
     * [Example: `did:keri`](#example-didkeri)
   - [Radicle Namespace](#radicle-namespace)
     * [Example: Multi-Device](#example-multi-device)
6. [Repository Identity](#repository-identity)
   - [Canonical References](#canonical-references)
   - [Agent Payload](#agent-payload)
7. [Referencing Agent Repositories](#referencing-agent-repositories)
   - [Namespaces](#namespaces)
   - [References](#references)
     * [Lookup Agent Repository](#lookup-agent-repository)
8. [Future Work](#future-work)
   - [KERI Agent Repository](#keri-agent-repository)
   - [Multi-Device](#multi-device)
9. [Appendix](#appendix)
   - [JSON Schema](#json-schema)
10. [Copyright](#copyright)
11. [References](#references)

Overview
--------
In this RIP, we will define what an agent repository is and how it fits into the
existing storage layout of the Heartwood protocol – comparing and contrasting it
to the existing project repositories.

We will then discuss the reference hierarchy expected of agent repositories, and
how they relate to DID methods, and the Heartwood protocol.

We will go on to discuss the repository identity for agent repositories, and a
new type of payload to help provide metadata for agents.

Finally, we will close out with how agent repositories are expected to be
cross-referenced from other repositories.

The Agent Repository
--------------------
In RIP-2[^0], Identity, the repository identity and its document were established.
This provided the terminology and methods for establishing and verifying a
repository in the Heartwood protocol.

In the same RIP, the `payload` for a project was established, starting the
Heartwood protocol off with the ability to collaborate on code and its social
artifacts.

<!-- TODO: clarify personas are more than one repository, i.e. identities are 1-to-1 -->
In this RIP, we establish a new kind of repository in the Heartwood protocol,
which specifically aims to represent agents within the Heartwood protocol. An
agent can be any entity that we want to identify uniquely, and which takes
actions within the protocol. The most common use case of an agent repository
will be for the concept of a user or user persona, e.g. I can identify myself as
`alice` for this project, but I may want to use establish a different persona
for some other projects. Other use cases we may want to consider are CI/CD
agents that will have permission to merge patches into projects. We leave the
potential up to your imagination, and only define the necessary foundations
here.

### Identity Document
Since this is still a repository in the Heartwood protocol, it will still share
similar attributes to the initially proposed repositories in RIP-2[^0]. That is,
it will still be identified by its Repository Identity (RID), implying that it
will also have the identity document.

### Storage
As well as having an identity document, agent repositories will still live within
the same storage layout as other repositories, as defined in RIP-3[^2].

    <storage>       # Storage root containing all local repositories
    ├── <rid>       # Some repository, e.g. a project, as a bare git repository
    │   └── refs    # All Git references under this project
    ├── <rid>
    │   └── refs
    ├── <rid>       # Some other repository, but in this case it is an agent repository
    │   └── refs    # All Git references related to the agent
    └── ...

Agent Repository References
---------------------------
We have established that an agent repository is similar to that of a project
repository. In the upcoming sections, we will discuss the reference layout
required for an agent repository and what is expected to exist within its
namespaces.

The existing layout[^3] will still apply, copied below:

    <storage>
    └─ <rid>                    # The "physical" Git repository
       └─ refs
          └─ namespaces         # All forks are stored under this namespace
             ├─ <nid>           # One peer's fork is stored here
             │  └─ refs
             ├─ <nid>           # Another peer's fork is stored here
             │  └─ refs
             └─ <nid>           # Etc.
                └─ refs

However, it is expected that instead of forks, there will be a single namespace
for the delegate that is managing the agent repository. This may change in the
future, but is up to a future RIP to discuss.

### DID Namespace
In the agent repository, we reserve a Git namespace, *not* under
`refs/namespaces`, specifically for the DID method[^4] and its required references.
This namespace is under `did`, and looks like the following tree:

    <storage>
    └─ <rid>                    # The "physical" Git repository
       └─ refs
          ├─ rad
          │   └─ id             # Canonical identity reference
          └─ did                # All DID relevant material
             └─ <method>...     # The method name for categorisation
                └─ ...          # Any references required for the method

The `<method>` defines the placeholder name for the kind of DID method that this
agent is using, and its contents are discussed in the next section.

#### Methods
<!-- TODO: Perhaps if there is a document we should define a well-known reference to -->
<!-- it? We may also want to ensure the verification of the document, i.e. it's -->
<!-- updated and signed crypotgraphically by the agent's signer -->
The agent repository is not prescriptive about the references inside the `did`
namespace. The namespace is simply reserved so that methods can effectively use
them for storing metadata and helping to resolve to a DID document, or providing
other data for cryptographic operations.

When a `method` for an agent repository is proposed it must define what
references are stored under this namespace and what the contents of those
references are, i.e. Git commits, trees, and blobs.

#### Signing and Verification
It is required that the DID method defines the way in which it signs data
payloads and verifies signatures. For example, if the method resolves to a DID
document, and that document contains verification methods, then this can be used
to verify.

This means that the signing and verification are a black box, when it comes to
the protocol using an agent repository for signing and verification. The
standard API for this signing and verification is inspired by the
`signature`[^5] crate, from the Rust ecosystem. The crate provides two traits,
`Signer`[^6] and `Verifier`[^7]. The former defines a method `try_sign`, which
attempts to sign the bytes provided, noting that it can fail. The latter defines
a method `verify` which attempts to verify the bytes and signature provided,
and only returning `Ok(())` if it succeeded. It's important to note that the
signature here is kept generic. DID methods will differ in their signature
outputs, and in fact, they may have complex signature schemes, for example,
multi-signature schemes.

We want to avoid arbitrarily executing signing and verification operations of
DID methods within clients of the protocol. This means that new DID methods for
agent repositories must be first proposed as their own RIP, so that the signing
and verification methods can be audited, and implemented as part of the
protocol.

The Heartwood protocol already makes heavy use of `ssh-agent`, so new DID
methods may easily use this approach when it comes to signing data.

#### Example: `did:key`
The simplest example of a DID method as an agent repository is `did:key`. The
reference scheme could be proposed as:

    <storage>
    └─ <rid>
       └─ refs
          └─ did
             └─ key
                └─ <suffix>

The `<suffix>` is the latter portion of `did:key:<suffix>`, and this is already
enough information for obtaining the corresponding public key. Optionally the
DID document could be pointed to by that reference, otherwise it could be an
empty Git commit.

As mentioned, the existing `ssh-agent` approach can still be used, while the
verification is based on the derived public-key.

#### Example: `did:keri`
To give a taste for a more complex DID method, an agent repository could be
instantiated as a `did:keri` repository. The reference scheme may be proposed
as:

    <storage>
    └─ <rid>
       └─ refs
          └─ did
             └─ keri
                └─ kel

The `kel` is the KERI Event Log (KEL), which can be used to check the state of
the KERI entity.

The signing and verification of data is completed via the specified approach in
the KERI spec[^8], via the KEL.

### Radicle Namespace
A second Git namespace is also reserved for any Heartwood specific extensions to
the agent repository. This namespace is reserved under the name `radicle`, and
looks like the following:

    <storage>
    └─ <rid>
       └─ refs
          ├─ rad
          │   └─ id
          ├─ did
          │  └─ <method>...
          │     └─ ...
          └─ radicle            # All Heartwood relevant material
             └─ ...             # Any references used by the Heartwood protocol

This RIP does not propose any material to go under this namespace – it merely
sets it up for further use. We will propose an example, in the next section,
which will be further developed in a future RIP.

#### Example: Multi-Device
We plan to use the `radicle` hierarchy to propose a multi-device approach.
That is, we plan to use references to associate nodes to an agent so that when a
user works on different devices, their device keys can be unified under their
agent. The current thoughts on the reference hierarchy is to have something like
the following, but may be subject to change:

    <storage>
    └─ <rid>
       └─ refs
          ├─ rad
          │   └─ id
          ├─ did
          │  └─ <method>...
          │     └─ ...
          └─ radicle            # All Heartwood relevant material
             ├─ nodes           # All nodes currently associated with the agent
             │  ├─ <nid>        # A node associated with the agent
             │  └─ <nid>        # A second node
             └─ revoked         # Revoked nodes
                └─ <nid>        # A revoked node

<!-- TODO: sketching forks below -->

    <storage>
    └─ <rid>
       └─ refs
          ├─ rad
          │   └─ id
          └─ namespaces/<nid> # Could be two
             └─ refs
                ├─ did
                │  └─ <method>...
                │     └─ ...
                └─ radicle            # All Heartwood relevant material
                   ├─ nodes           # All nodes currently associated with the agent
                   │  ├─ <nid>        # A node associated with the agent
                   │  └─ <nid>        # A second node
                   └─ revoked         # Revoked nodes
                      └─ <nid>        # A revoked node

Repository Identity
-------------------
As mentioned, the agent repository is still a Heartwood repository, which means
it has an RID and an identity document. In its current state (as of RIP-4[^9]
Canonical References), the identity document requires a `delegates` field and a
set of values. Previously, this would be expected to be filled with `did:key`
DIDs. However, with the advent of the agent repository – and attempting to
support multiple DID methods – the underlying DID method can and should be used
in this field. It is recommended that only this DID is used, but it is not
forbidden to add more than one DID into this set.

In the upcoming sections, we will discuss the other fields `canonicalRefs` and
`payload`.

### Canonical References
The use of canonical references may also be used in the agent repository. The
repository may not be defined to be used as code, but that does not prohibit the
use of textual data within the repository – essentially providing an extension
point for users of the protocol to come up with more use cases of the repository
itself.

However, any implementation of canonical references must prohibit any rules from
writing the new, reserved namespaces mentioned in [DID
Namespace][#did-namespace] and [Radicle Namespace][#radicle-namespace], i.e.
`refs/did` and `refs/radicle`.

### Agent Payload
Similarly to the `xyz.radicle.agent` payload for defining metadata about the
project, we define a `agent` payload, under `xyz.radicle.agent`, which can
hold metadata about the agent. We define this payload as follows (with the [JSON
Schema][#json-schema] provided):

    {
      "xyz.radicle.agent": {
        "alias": "alice",
        "avatar": "ede1bc5c976e3d4d3e59b14d845c890498b530e6",
        "websites": ["bsky.com/alice"]
      }
    }

The fields and their values are described as follows:

- The `alias` field is a name or nickname for the agent, represented by a
  string, which can be no longer than 32 characters. We also restrict the string
  to reject any whitespace or control characters – a regex is provided in [JSON
  Schema][#json-schema].
- The `avatar` field can either be a URI or a Git SHA, and should point to an
  image to be used for the agent in client code.
- The `websites` field is a set of URLs that agent associates themselves with.

Referencing Agent Repositories
------------------------------
Agent repositories will become an important part of the Heartwood protocol.
For one, it will be a way to unify the view of a single user across multiple
devices. This means that it will be important to be able to look up an agent
repository from other repositories. In this section we will present a
standardized way of achieving this, which repository implementations must use.
For now, this is only extending the layout of project repositories, outlined in
RIP-3[^2].

### Gossip Protocol

<!-- TODO: account for multiple DIDs associating with an RID -->
One approach to finding out which RIDs identify which agents is through the
gossip protocol. A new gossip message would be introduced that would advertise
which RID is associated with a particular DID. We can think of this as another
kind of address book.

Nodes broadcast this message when they are made aware of this pairing. In the
case of the node that created the agent repository, it can be broadcasted to the
network immediately. This message is relayed and propagated to the rest of the
network. If a node comes across a DID they are not aware of, then they would
send a request to the network to find out the RID and DID pair.

This information would be stored alongside the rest of the other network
information that is recorded, e.g. in an `sqlite` database.

A natural question would be, "What happens when nodes fake this information?"
This is possible, but can easily be verified. When the repository is being
fetched, it should only first fetch the `rad/id` reference which points to the
identity document. The identity document can then be checked to see if it
contains the DID. If it does, then the rest of the fetch can resume. Otherwise,
the network protocol may wish to penalize a node that advertises a spoofed
pairing. Note that this verification process can be done on demand, rather than
immediately.

#### Create

The first case a node will want to create an entry for an RID and DID pair is
when the node itself is creating the agent repository for the given DID. As
mentioned, they will broadcast this information as part of the gossip protocol.

<!--
The second case is when a DID is added as a delegate to the agent repository,
such that the new DID becomes associated with the agent repository.

TODO: I'm not sure this makes sense, unless the DID can also be verified in the
agent repository, so we would need to reconcile that.
-->

The final case is when a node receives a gossip message about the RID and DID.
The node may record this entry and verify that the association is correct –
through the fetch and verification process mentioned above.

#### Read

<!-- TODO: actually this confirms for me that we shouldn't allow another -->
<!-- delegate to be identified by an RID, we will end up with ambiguity, right? -->
To know which RID identifies the DID, the underlying node address book is used
to look up the RID for the given DID. Note that the DID one, and only one, RID.

The RID can then be used to replicate and resolve the agent for usage within the
protocol, e.g. signing and verification, examining its payload for metadata.

#### Updating

#### Delete

### Git References

Another approach to this problem is by further utilizing the underlying Git
storage of the Radicle storage[^2]. When a node is using an agent to perform an
action, e.g. creating a new COB, it will create an entry in its namespace
hierarchy that records the DID and the RID of the agent repository.

The name of the reference will follow the pattern `did-<method>-<suffix>`, e.g.
`did-keri-EXq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148`,
`did-key-z6MkireRatUThvd3qzfKht1S44wpm4FEWSSa4PRMTSQZ3voM`,
`did-dns-danubetech.com`, etc.

---
**Note**

The `<method>` and `<suffix>` must be valid Git reference components, which can
be checked via `git-check-ref-format`[^11]. If they are not valid path
components, then the RIP proposing them must standardize how those characters
are escaped or replaced.

---

This means that the storage in the non-agent repository could look like the
following:

    <storage>
    └─ <rid>                           # The "physical" Git repository
       └─ refs
          └─ namespaces                # All forks are stored under this namespace
             ├─ <nid>                  # One peer's fork is stored here
             │  ├─ did-key-<suffix>    # A did:key repository agent
             │  └─ refs
             │     └─ ...
             └─ <nid>                  # Another peer's fork is stored here
                ├─ did-keri-<suffix>   # A did:keri repository agent
                └─ refs
                   └─ ...

The reference will point to a Git commit that contains a Git tree, and that tree
points to a single Git blob, `rid`. The `rid` blob contains the RID in
plain-text.

### Namespaces
We will use the Git namespaces mechanism, once again, to define a hierarchy for
agent repository references to live under. The name of the namespace will
follow the pattern `did-<method>-<suffix>`, e.g.
`did-keri-EXq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148`,
`did-key-z6MkireRatUThvd3qzfKht1S44wpm4FEWSSa4PRMTSQZ3voM`,
`did-dns-danubetech.com`, etc.

---
**Note**

The `<method>` and `<suffix>` must be valid Git reference components, which can
be checked via `git-check-ref-format`[^11]. If they are not valid path
components, then the RIP proposing them must standardize how those characters
are escaped or replaced.

---

With this in mind, the layout of the repository in storage will now look like
the following:

    <storage>
    └─ <rid>                        # The "physical" Git repository
       └─ refs
          └─ namespaces             # All forks are stored under this namespace
             ├─ <nid>               # One peer's fork is stored here
             │  └─ refs
             ├─ <nid>               # Another peer's fork is stored here
             │  └─ refs
             ├─ did-keri-<suffix>   # A did:keri repository agent
             │  └─ refs
             └─ did-key-<suffix>    # A did:key repository agent
                └─ refs

### References
For any method, there must be at least a single reference underneath its `refs`
hierarchy, the `rad/id` reference. This reference must peel to a Git blob that
contains the RID of the agent repository. That is, the reference can point
directly to a Git blob or point to any Git object that *peels* to a Git blob,
where the blob's contents contain:

    rad:<suffix>

Where `<suffix>` is the unique hash generated for the identity of the agent
repository.

The rest of the `refs` hierarchy is left open for further specification in
future RIPs, e.g. the multi-device RIP will make use of this space for canonical
references of the agent.

#### Lookup Agent Repository
The lookup of an agent repository will only require the DID method. The DID
method string will be converted into its equivalent `did-<method>-<suffix>`
string, which can be used to form the reference name:

    refs/namespaces/did-<method>-<suffix>/refs/rad/id

This reference name can be used to resolve the Git blob that contains the RID,
which in turn can be used to lookup the repository in storage.

The agent repository can then be used for any required data that it holds, e.g.
looking up the alias and avatar associated with the DID method.

##### Verification

- We have two blobs:
  a. `rid`: contains the RID
  b. `signature`: contains the signature of the agent, signature of the `rid` blob
- Verification:
  a. agent::verify(`signature`)

<!-- TODO: Do we need a verification mechanism to guarantee that the DID leads to the -->
<!-- correct RID? i.e. prevent an attacker from leading them to the wrong agent -->
<!-- repository? -->

#### Adding an Agent

Future Work
-----------

### KERI Agent Repository
This RIP only goes as far as specifying the agent repository, however, it will
be important for the protocol to have its first instance of an agent repository
and integrate one into the protocol. Riding on the coat tails of this RIP, will
be a RIP that defines an implementation that is based on the KERI
specification[^2].

### Multi-Device
As well as this in [Radicle Namespace][#radicle-namespace], we mentioned the
concept of being able to manage multiple devices under the same persona. This
RIP carved out the reference space for associating keys within an agent
repository, and the details of this approach will be specified in a future RIP.

Appendix
--------

### JSON Schema

    {
      "$schema": "http://json-schema.org/draft/2020-12/schema",
      "type": "object",
      "properties": {
        "xyz.radicle.agent": {
          "type": "object",
          "properties": {
            "alias": {
              "type": "string",
              "maxLength": 32,
              "pattern": "^[^\s\p{Cc}]+$"
            },
            "avatar": {
              "type": "string",
              "oneOf": [
                {
                  "format": "uri",
                  "maxLength": 2048
                },
                {
                  "pattern": "^[0-9a-fA-F]{40}$"
                }
              ]
            },
            "websites": {
              "type": "array",
              "items": {
                "type": "string",
                "format": "uri",
                "maxLength": 2048
              }
            }
          },
          "required": ["alias"],
          "additionalProperties": false
        }
      },
      "required": ["xyz.radicle.agent"],
      "additionalProperties": false
    }

Copyright
---------
This document is licensed under the Creative Commons CC0 1.0 Universal license.

References
----------
[^0]: https://app.radicle.xyz/nodes/seed.radicle.garden/rad:z3trNYnLWS11cJWC6BbxDs5niGo82/tree/0002-identity.md
[^1]: https://app.radicle.xyz/nodes/seed.radicle.garden/rad:z3trNYnLWS11cJWC6BbxDs5niGo82/tree/0002-identity.md#repository-identity
[^2]: https://app.radicle.xyz/nodes/seed.radicle.garden/rad:z3trNYnLWS11cJWC6BbxDs5niGo82/tree/0003-storage-layout.md
[^3]: https://app.radicle.xyz/nodes/seed.radicle.garden/rad:z3trNYnLWS11cJWC6BbxDs5niGo82/tree/0003-storage-layout.md#layout
[^4]: https://www.w3.org/TR/did-core/
[^5]: https://docs.rs/signature/latest/signature/
[^6]: https://docs.rs/signature/latest/signature/trait.Signer.html
[^7]: https://docs.rs/signature/latest/signature/trait.Verifier.html
[^8]: https://trustoverip.github.io/tswg-keri-specification
<!-- TODO: update to latest version when it is merged -->
[^9]: https://app.radicle.xyz/nodes/seed.radicle.garden/rad:z3trNYnLWS11cJWC6BbxDs5niGo82/tree/df1bc0f914e4e5592d9f04a73a5c916f2e94bfcd/0004-canonical-references.md
[^10]: https://git-scm.com/docs/gitnamespaces
[^11]: https://git-scm.com/docs/git-check-ref-format
