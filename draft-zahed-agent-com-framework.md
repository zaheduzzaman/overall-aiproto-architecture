---
title: "Reference Architecture for Agent Communication"
abbrev: "Agent Comm Architecture"
docname: draft-zahed-agent-com-framework-latest
category: info
ipr: trust200902
# area: xxxxx
# workgroup: individual submission
submissiontype: IETF
keyword: Internet-Draft
stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: Z. Sarker
    name: Zaheduzzaman Sarker
    organization: Nokia
    email: zaheduzzaman.sarker@nokia.com
  -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    email: k.tirumaleswar_reddy@nokia.com
  -
    name: Kehan Yao
    organization: China Mobile
    email: yaokehan@chinamobile.com
  -
    name: Dapeng Liu
    organization: Alibaba Cloud
    email: max.ldp@alibaba-inc.com
  -
    name: Suresh Krishnan
    organization: Cisco
    email: suresh.krishnan@gmail.com

normative:
  RFC8446:
  RFC8693:
  RFC9000:
  RFC9001:

informative:
  RFC9261:
  I-D.hardt-aauth-protocol:
  I-D.agentic-ai-usecases-requirements:
  I-D.ietf-wimse-aims:
  I-D.ietf-oauth-identity-chaining:
  I-D.ietf-oauth-transaction-tokens:
  I-D.ietf-wimse-workload-creds:
  A2A:
    title: "Agent2Agent Protocol Specification"
    author:
      - org: Google
    target: https://a2a-protocol.org/v0.2.5/specification/
    date: 2025
  MCP:
    title: "Model Context Protocol"
    author:
      - org: Anthropic
    target: https://modelcontextprotocol.io/specification
    date: 2024

--- abstract

This document describes a reference architecture for AI agent
communication, covering user-to-agent, agent-to-agent, and agent-to-tool
interactions. It defines the terms used in the architecture and identifies
the functional blocks involved: an agent communication protocol that
maintains dialog context and continuity, and the existing protocol building
blocks it reuses, including identity, authentication, authorization,
encryption, and transport. It also describes the relationships between
these blocks.

--- middle

# Introduction {#intro}

AI agents that communicate, collaborate, and delegate tasks
across the Internet introduce a new class of networked entity with
requirements that existing protocols were not designed to address.
User-to-agent, agent-to-agent, and agent-to-tool interactions create dialogs
that can be long-lived, span multiple intermediaries and trust boundaries,
and involve multiple modalities such as text, audio, and video. Messages need
to be associated with their dialog, and a dialog needs to survive transport
connection interruptions. Dialogs also depend on verifiable agent identity, delegated authorization across
agent chains, and confidentiality and integrity of the exchanged data.

This document describes a reference architecture for AI agent
communication. It defines the terms used in the architecture and identifies
the functional blocks involved: an agent communication protocol that
maintains dialog context and continuity, and the existing protocol building
blocks it reuses for identity, authentication, authorization, encryption,
and transport. It describes the relationships between these blocks. The use
cases and requirements that drive this architecture are described in
{{I-D.agentic-ai-usecases-requirements}}. The mechanisms of the agent
communication protocol are outside the scope of this document.

MCP {{MCP}} and A2A {{A2A}} are application-layer protocols
maintained by the Linux Foundation. They use IETF protocols such as HTTP,
OAuth 2.0, and TLS. MCP addresses agent-to-tool interactions, and A2A
addresses agent-to-agent interactions. The agent communication protocol in
this architecture is intended to be usable by these protocols through
well-defined extension points, not to replace them.

# Terminology {#terminology}

Agent Identity:
: Identity information associated with an AI agent, distinct from the
  identity of the user on whose behalf it acts, and used for
  authentication, authorization, and accountability.

AI Agent:
: A software system that uses AI models to complete a task on behalf of a
  user or another AI agent.

Capability:
: A task that an AI agent can perform.

Delegation:
: The act of an AI agent requesting another AI agent to execute a task on
  its behalf.

Dialog:
: TBD.

Dialog Context:
: TBD.

Dialog Identifier:
: TBD.  

Intermediary:
: An entity that relays or processes messages between dialog participants.

Message:
: A discrete unit of communication exchanged between dialog participants.

Modality:
: A category of data exchanged in a dialog, such as text, audio, image,
  or video.

Orchestrator Agent:
: An AI agent that decomposes a task into sub-tasks and delegates them to
  other AI agents.

Task:
: A unit of work submitted by a user to an AI agent, or delegated by one
  AI agent to another.

Tool:
: A service invoked by an AI agent to retrieve data or perform operations.
  A tool is not necessarily an AI agent.

Trust Boundary:
: A boundary between administrative domains.

User:
: A human who interacts with an AI agent.

# Architecture Scope {#scope}

This architecture addresses the requirements in
{{I-D.agentic-ai-usecases-requirements}}. It covers the following areas,
each described in a separate section:

- Identity, authentication, authorization, and encryption ({{security}})
- Transport ({{transport}})
- Dialog continuity ({{sessioncont}})

The following are outside the scope of this document:

- Implementation details of AI agents, including AI models, backend AI
  infrastructure, reasoning algorithms, and tool-specific business logic.

- Agent behavior, decision-making, and planning.

- Prevention of AI model misbehavior, such as hallucination.

- User interfaces and the rendering of agent outputs on end-user devices.

- Discovery of AI agents and tools.

# Design Principles {#philosophy}

The architecture is based on the following principles:

Reuse of existing protocols:
: The architecture reuses existing IETF protocols for identity,
  authentication, authorization, encryption, and transport. Where an
  existing protocol cannot meet a requirement, the gap is raised with the
  responsible working group.

Secure communication:
: All communication is authenticated, authorized, and protected for
  confidentiality and integrity.

Dialog-oriented:
: Dialogs are long-lived and stateful. Dialog context outlives any
  individual transport connection and is preserved across connection
  changes.

Incremental deployment:
: A minimal deployment, such as two AI agents in one domain exchanging
  text, interoperates with a full deployment that spans multiple domains,
  intermediaries, and modalities. A deployment implements only the
  features it uses.

# Architecture Composition {#composition}

The architecture consists of the following functional blocks:

- Security: provides identity, authentication, authorization, and
  encryption.
- Transport: carries messages between dialog participants.
- Agent communication protocol: maintains dialog context and continuity.

The architecture assumes that AI agents and tools can be discovered.
Discovery is expected to be addressed by the DAWN WG and is outside the
scope of this document.

## Operational Flow {#opflow}

The use case for orchestrator and agent collaboration is
described in Section 4.2 of {{I-D.agentic-ai-usecases-requirements}}. A
complete interaction in this use case proceeds as follows:

1. Task Initiation: A user submits a task to an Orchestrator Agent.

2. Agent Selection: The Orchestrator Agent discovers AI agents with the
   capabilities required for the sub-tasks.

3. Authentication: The Orchestrator Agent and each selected AI agent
   authenticate each other.

4. Authorization: The Orchestrator Agent obtains authorization for each
   sub-task. The authorization is derived from the user's authorization
   and limited to the sub-task.

5. Dialog Establishment: The Orchestrator Agent establishes a dialog with
   each selected AI agent over a protected transport connection.

6. Task Execution: The AI agents exchange messages within the dialog. The
   messages can carry multiple modalities.

7. Dialog Continuity: If a transport connection is interrupted, the dialog
   continues over a new transport connection.

8. Task Completion: The Orchestrator Agent returns the result to the user
   and terminates the dialogs.

## Architecture Overview {#framework}

{{fig-arch}} shows the architecture. The Agent Communication
Protocol uses the Dialog, Authorization and Delegation, and Identity
Management components. It runs over the Transport layer, which is protected
by the Security layer.

~~~
+--------------------+                      +-------------------+
|      User / App    |                      |    User / App     |
+---------+----------+                      +----------+--------+
          |                                            |
          v                                            v
+-----------------+     +-----------------+      +---------------+
|     Agent (A)   |<--->|    Discovery    |<---> |   Agent (B)   |
+---------+-------+     +-----------------+      +-------+-------+
          |                                              |
          +----------------------+-----------------------+
                                 |
                                 v
+---------------------------------------------------------------+
|                Agent Communication Protocol                   |
|                                                               |
| +-----------------+  +-----------------+  +-----------------+ |
| | Dialog          |  | Authorization   |  | Identity        | |
| |                 |  | and Delegation  |  | Management      | |
| | - Context       |  |                 |  |                 | |
| | - Identifier    |  | - OAuth 2.0     |  | - WIT, WPT      | |
| +-----------------+  +-----------------+  +-----------------+ |
+---------------------------------------------------------------+
                                 |
                                 v
+---------------------------------------------------------------+
|                          Transport                            |
|  - Multiplexed Streams                                        |
|  - Per-stream Semantics                                       |
|  - Connection continuity (Connection ID, TLS resumption)      |
|  - Protocol : QUIC, MoQT                                      |
+---------------------------------------------------------------+
                                 |
                                 v
+---------------------------------------------------------------+
|                       Security                                |
|  - Channel Protection                                         |
|  - Mutual Authentication                                      |
|  - Protocol: TLS 1.3                                          |
+---------------------------------------------------------------+
~~~
{: #fig-arch title="Reference architecture"}

TLS 1.3 provides channel protection and mutual authentication for all agent communication. Security is shown as a separate layer in {{fig-arch}} for clarity. When agents communicate over QUIC, TLS 1.3 is integrated into the QUIC handshake {{RFC9001}} and does not occupy a discrete layer above or below the transport. In architectures involving intermediaries where TLS is terminated at a proxy, application-layer authentication is required to maintain identity continuity across TLS termination points, as described in {{channel-protection}}.

QUIC provides multiplexed streams with per-stream semantics suitable for the heterogeneous communication patterns of agent interactions. MoQT (Media over QUIC Transport) adds a publish/subscribe layer over QUIC for the one-to-many and many-to-many group communication that point-to-point QUIC streams cannot provide.

OAuth 2.0 provides the authorization and delegation framework, enabling agents to obtain and present access tokens scoped to specific tasks. WIMSE provides workload identity for agents through a URI embedded in X.509 certificates at the TLS layer, and through Workload Identity Tokens (WIT) and WIMSE Proof Tokens (WPT) at the application layer. The Agent Communication Protocol maintains dialog continuity across connection changes using a stable dialog identifier. Connection-level continuity is provided by QUIC Connection ID and TLS resumption. Agents interact at the top of the stack, each acting on behalf of a user or system.

Attestation, as defined in the SEAT WG, binds attestation evidence to agent communications. The evidence can be conveyed during the TLS handshake or at the application layer by extending {{RFC9261}}. Attestation is not shown as a discrete layer in {{fig-arch}} because its position in the stack is solution-specific. It is outside the scope of this document.

# Discovery Aspects {#discovery}

TODO: This text will be revised to point to the relevant
specifications in the DAWN WG.

Discovery in open environments requires resolving an agent identifier to
current network locations and advertised capabilities across administrative
domains. Existing mechanisms do not support capability-aware, federated
resolution without pre-established trust relationships.
A2A introduces the Agent Card, a JSON document available at a well-known URI
(/.well-known/agent.json) that advertises agent capabilities. While useful,
this mechanism assumes prior knowledge of the agent's domain.
A scalable discovery system requires:

- Globally unique agent identifiers ( e.g DNS-rooted)
- Signed Agent Capability Documents
- Federation protocols with provenance tagging and hop limits

Concrete mechanisms for agent resolution and capability-based discovery are
the subject of ongoing, early-stage work in the IETF and are not yet settled.

# Security Aspects {#security}

Security for agent communication spans four interdependent concerns: verifiable agent identity, channel protection, binding of authorized intent to agent execution, and delegation chain integrity. The use cases and protocol requirements that motivate this architecture are discussed in {{I-D.agentic-ai-usecases-requirements}}. Some of the mechanisms for satisfying these requirements using existing IETF standards are described in {{I-D.ietf-wimse-aims}}.

## Agent Identity {#agent-identity}

Agent identity encompasses two concerns: a stable unique identifier and cryptographic credentials bound to that identifier.

Every agent is required to be assigned a unique, stable identifier that remains consistent across network reconnections and dialog resumptions. Credentials are required to be bound to the agent's identity. The credential lifecycle, including provisioning, rotation, and revocation, is required to be supported without manual intervention. {{I-D.ietf-wimse-aims}} defines the term Agent Identity Management System (AIMS) as a conceptual model describing the set of functions required to establish, maintain, and evaluate the identity and permissions of an agent workload.

These identity requirements are satisfied by using WIMSE Workload Credentials {{I-D.ietf-wimse-workload-creds}}: a Workload Identity Token (WIT) at the application layer and a Workload Identity Certificate (WIC) at the transport layer. Both bind a public key to the agent's workload identity.

## Channel Protection {#channel-protection}

All agent communication is required to be encrypted and mutually authenticated. This architecture does not define a fallback to cleartext or unauthenticated operation.
Authentication may operate at the transport layer, the application layer, or both. Transport-layer authentication works well when TLS connections are not terminated by intermediaries. In architectures involving proxies, application-layer authentication is required to maintain identity continuity across TLS termination points. The channel protection mechanism is required to be negotiated during transport connection establishment, with both endpoints authenticating before any message is exchanged.

## Intent-Execution Separation {#intent-execution}

AI agents dynamically generate execution plans and issue sub-requests based on their own reasoning, which may diverge from the scope of the original user authorization. This architecture requires that authorization granted to an agent be limited to the scope of the delegated task, so that an agent cannot use it to authorize actions outside that scope. This depends on delegation mechanics in which an agent acts on behalf of the original user and the authority passed at each hop is narrowed to the delegated task rather than broadened. Standardizing these delegation mechanics for AI agents is an active area of work in the OAuth WG. This document will reference the relevant outcomes as this work matures.

## Delegation Chain Integrity {#delegation-chain}

When one agent delegates a sub-task to another, the receiving agent is authorized only to the extent explicitly permitted by the delegating agent. In multi-hop scenarios, the delegation chain may span several agents before reaching the resource or tool ultimately invoked. At each hop, the receiving party is required to verify that the authority presented was derived from the human or system that originally authorized the task, has not been expanded beyond what was originally granted, and has not been forged or modified in transit. A delegating agent is not permitted to grant authority beyond what the original authorization permits for the delegated task. This draft will be updated with more details as the work progresses in the OAuth WG.

## Audit and Non-Repudiation {#audit}

Agents act on behalf of users or other agents across multiple hops and
administrative domains. Audit requires that each action be traceable to the
principal that authorized it and the agent that executed it. For Non-repudiation,
this traceability is required to be cryptographically verifiable, so that
no party can later deny its role. The mechanism for recording and
verifying it is outside the scope of this document.

# Transport protocols Aspects {#transport}

Transport for agent-to-agent communication spans several interdependent concerns: dialog continuity across long-running tasks, heterogeneous delivery semantics, explicit task and stream correlation, efficient movement of large context and data objects, signaling for priority and cancellation, structured error propagation, and negotiation of modalities and group communication topologies. The use cases and protocol requirements that motivate these transport properties are discussed in {{I-D.agentic-ai-usecases-requirements}}. The transport substrate is not required merely to deliver bytes between endpoints; it is required to preserve the correctness, efficiency, and recoverability of delegated agent execution across administrative domains and under changing network conditions.

## Delivery Semantics {#delivery-semantics}

Agent communication requires heterogeneous transport semantics rather than a single uniform model.

Agent communication patterns fall into three broad delivery semantic classes, each with distinct transport properties:

Reliable ordered delivery: : Used for control messages, workflow state updates, authorization checkpoints, and structured results. The transport substrate is required to guarantee in-order, lossless delivery for these message classes, and is required to isolate them from other traffic classes to avoid head-of-line blocking.

Low-latency, loss-tolerant delivery: Used for real-time audio, video, and high-frequency sensor or telemetry streams. The transport substrate is required to minimize latency for these streams and is not required to guarantee delivery or ordering.

High-throughput reliable delivery: Used for large context payloads and model inputs and outputs. The transport substrate is required to support high-throughput reliable transfer with flow-control isolation from other traffic classes. Where in-band transfer is impractical, a secure, integrity-protected out-of-band transfer mechanism is required to be supported.

## Message Exchange Patterns {#message-exchange}

Agent tasks are not limited to simple request-response exchange. An agent may delegate work asynchronously, receive an acknowledgement before completion, stream partial results, emit progress updates, request additional authorization, and later return a final result or cancellation status. The transport is required to take these patterns explicitly into account, including correlation of messages to the relevant task, subtask, and dialog. Multiplexing is required so that multiple delegated tasks or tool invocations can proceed concurrently without unrelated head-of-line blocking. The transport is also required to permit out-of-order completion of concurrent subtasks while preserving per-task ordering for incremental output and terminal states.

## Priority and Scheduling {#priority}

Delegated agent work varies in urgency and consequence. Some exchanges are on the critical path of a user-visible task, while others are background enrichment, speculative reasoning, or deferred synchronization. The protocol is required to signal task priority and delivery urgency so that transport and scheduling layers can differentiate service. Such signaling is required to be advisory and policy-controlled; a sender is not permitted to raise priority beyond the authority or policy of the delegated task. Priority signaling is also required for coordination messages such as authorization responses, cancellation acknowledgements, and error notifications, because delaying them can break an otherwise correct workflow.

## Multimodal Negotiation {#multimodal}

Agents may exchange text, structured objects, images, audio, video, and sensor data within a dialog. The transport protocol is required to support negotiation of modalities and formats during dialog establishment or capability discovery. The negotiated set is binding unless explicitly updated; an endpoint is not permitted to send an unsupported modality. Transport-relevant modality properties, including reliability, latency sensitivity, and expected size, are required to inform stream selection, flow-control allocation, and framing. Modality negotiation therefore constrains both content and transport behavior.

## Structured Error and Progress Signaling {#error-signaling}

The transport is required to carry more than final results. It is required to also convey incremental output, progress updates, checkpoint states, cancellation states, and structured errors usable by an orchestrating peer. Error signaling is required to distinguish transport failure from application failure, authorization failure, policy rejection, timeout, validation failure, and interoperability problems. Progress and liveness notifications are required to be representable independently of final completion so that a peer can tell whether an agent is active, blocked on external input, or waiting in a recoverable state. It is required to ensure that terminal error or cancellation signals are not silently lost if an ordinary data stream fails.

## Cancellation and Interruption {#cancellation}

Agent execution may be interrupted by user action, policy enforcement, higher-priority work, loss of authorization, or dependent-subtask failure. The protocol is required to define explicit application-layer cancellation and interruption signaling, rather than rely on connection teardown. Cancellation is required to identify the affected scope, including a single invocation, delegated subtree, or dialog. Receiving agents are required to treat cancellation as idempotent and report whether execution stopped, had already completed, or could not be fully rolled back due to an irreversible action. In multi-hop deployments, interruption semantics are required to propagate predictably so that dependent subtasks do not continue after withdrawal of the parent task.

## Multiple Communication topologies {#topologies}

Some agent interactions may use one-to-many or many-to-many communication among a group of agents whose membership may change during an exchange. The transport is required to support these delivery patterns and to handle agents joining or leaving the group while the exchange is active. An agent's authorization to participate is required to be verified when it joins, and the keying material is required to be updated whenever an agent joins or leaves, so that an agent can decrypt group traffic only while it is a member.

# Dialog Continuity {#sessioncont}

Agent interactions are often long-lived, interruption-prone, and delegated across multiple hops. Each dialog is required to have a stable dialog identifier that survives reconnection and resumption. The transport mapping is required to let an authorized peer re-attach to an interrupted dialog. Dialog continuity is thus defined above any TCP connection, QUIC connection, or TLS association. QUIC migration and TLS resumption help, but they preserve transport or cryptographic state, not the dialog state that agents require.

Connection-level continuity, which covers surviving path changes and re-establishing dropped connections, is provided by QUIC Connection ID and TLS resumption.

# Applicability of Existing IETF Work {#existingworks}
## Reuse As-Is

* TLS 1.3 {{!RFC8446}} provides mutual authentication and channel protection. Agents
  are required to negotiate TLS 1.3 or higher.
* QUIC {{!RFC9000}} and QUIC-TLS {{!RFC9001}} provide multiplexed streams and 0-RTT
  session resumption.
* OAuth 2.0 Token Exchange {{RFC8693}} provides the base token exchange mechanism.
* WIMSE Workload Credentials {{I-D.ietf-wimse-workload-creds}} are reused for agent
  identity; see {{agent-identity}}.

## Profile or Extend

The following existing protocols may require profiling or extension; this list is expected to evolve as new IETF and OAuth WG specifications emerge:

- The OAuth WG is actively discussing how existing and new mechanisms apply to AI agent authorization. {{I-D.ietf-oauth-identity-chaining}} addresses cross-domain authorization, and {{I-D.ietf-oauth-transaction-tokens}} addresses intra-domain token exchange between workloads. New proposals such as {{I-D.hardt-aauth-protocol}} are also under discussion. This document will track this work and profile the relevant outcomes once the OAuth WG reaches consensus.

- MoQT (Media over QUIC Transport): acts as a unified transport substrate for distributed agent state synchronization and real-time multimodal communications. Work needs to happen to provide a common mapping of request-response and streaming patterns onto the pub/sub model of MoQT in order to enable interoperability across diverse agent ecosystems.

- More TBD..

## New Protocol Work

The Agent Communication Protocol shown in {{fig-arch}} is new protocol work.
It is expected to be specified separately.

# Security Considerations {#secconsiderations}

AI agents enlarge the attack surface: they act autonomously, delegate across administrative domains, and operate over long-lived dialogs that rely on long-standing delegated authority. This section highlights threats specific to the building blocks in this document; detailed mitigations are expected in the specifications of those building blocks.

Delegated authority is the primary risk. A rogue or compromised agent may attempt to use delegated authority beyond the task it was granted, or to broaden it as it delegates onward. As required in {{intent-execution}} and {{delegation-chain}}, authority granted to an agent is scoped to the delegated task and cannot be broadened along the chain, and the delegation chain is verifiable at each hop so an intermediary cannot forge or escalate it.

Because agents exchange rich, potentially sensitive multimodal context, both message content and metadata require protection. Even with content encrypted, analysis of message sizes, timing, and stream patterns can leak the nature of an agent's tasks; mitigations such as padding may be warranted where that exposure matters.

# IANA Considerations {#ianaconsideration}

This document has no IANA actions.

# Acknowledgments
{: numbered="false"}

The authors thank Kehan Yao for the discussion and comments.

--- back
