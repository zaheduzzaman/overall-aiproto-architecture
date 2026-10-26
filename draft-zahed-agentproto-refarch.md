---
title: "Reference Architecture for Agent Communication"
abbrev: "Agent Comm Ref Architecture"
docname: draft-zahed-agentproto-refarch-latest
category: info
ipr: trust200902
# area: ART
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
  RFC8693:
  RFC9000:
  RFC9001:
  RFC9846:

informative:
  RFC9261:
  RFC9334:
  RFC9420:
  RFC9711:
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
{{I-D.agentic-ai-usecases-requirements}}. The wire formats and protocol
elements of the agent communication protocol are outside the scope of
this document, while this document describes the the functional
decomposition of the reference architecture.

MCP {{MCP}} and A2A {{A2A}} are application-layer protocols
maintained by the Linux Foundation. They use IETF protocols such as HTTP,
OAuth 2.0, and TLS. MCP addresses agent-to-tool interactions, and A2A
addresses agent-to-agent interactions. The agent communication protocol in
this architecture is intended to be usable by these protocols through
well-defined extension points, not to replace them.

# Terminology {#terminology}

Administrative Domain:
: A set of entities, such as agents, tools, and intermediaries, that are
subject to a common authorization policy and a common authority.

Agent Identity:
: Identity information associated with an AI agent, distinct from the
  identity of the user on whose behalf it acts, and used for
  authentication, authorization, and accountability.

AI Agent:
: A software system that uses AI models to complete a task on behalf of a
  user or another AI agent.

Delegation:
: The act of an AI agent requesting another AI agent to execute a task on
  its behalf.

Dialog:
: The correlated sequence of interactions between two or more participants
  (users, agents, and tools) over the course of a task. A dialog is
  identified by a dialog context identifier, has a lifecycle, and may persist
  across multiple individual message exchanges and network connections.

Dialog Context:
: Context is a transferable structured, self-contained, task-specific and authority-scoped set of information, state, decisions, constraints, and pending or delegated tasks another agent needs to continue or contribute to a task without reconstructing the previous agents full execution history. It requires identifiers and lifecycle state needed to correlate and maintain a dialog. It does not include the content exchanged with AI models, such as conversation memory, retrieved documents, or prompts.

Dialog Context Identifier:
: An identifier, carried in the dialog context, that correlates messages
  with a dialog and survives network interruption or change.

Intermediary:
: An entity that relays or processes messages between dialog participants.

Message:
: A discrete unit of communication exchanged between dialog participants.

Modality:
: A category of data exchanged in a dialog, such as text, audio, image,
  or video.

Principal:
: The user or organization whose authority is originally exercised when an
AI agent acts on its behalf.

Task:
: A unit of work submitted by a user to an AI agent, or delegated by one
  AI agent to another.

Task Context:
: The data that a participant maintains to execute a task.

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

- Encryption, channel protection, authentication, authorization, and
  identity credentials ({{security}})
- Transport ({{transport}})
- Agent Communication Protocol, including the association of identity,
  delegation, and authorization information with a dialog ({{acp}})

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

The architecture consists of the following blocks:

- Security: provides the existing mechanisms for encryption, channel
  protection, authentication, authorization, and identity credentials.
- Transport: carries messages between dialog participants.
- Agent communication protocol: maintains dialog context and continuity.
  It has three sub-blocks: Dialog Propagation, Authorization and
  Delegation, and Identity Management. The sub-blocks carry and associate
  identity, delegation, and authorization information with a dialog.

The agent communication protocol does not authenticate participants, issue
or validate credentials, or evaluate authorization policy. Those functions
are performed by the mechanisms of the Security block, described in
{{security}} and {{existingworks}}. The Authorization and Delegation and
Identity Management sub-blocks are part of the agent communication protocol
because the information they carry has to stay associated with the dialog
across intermediaries, hops, and connection changes.

The architecture assumes that AI agents and tools can be discovered.
Discovery is expected to be addressed by the DAWN WG and is outside the
scope of this document.

# Operational flow {#opflow}

This section describes how the blocks in {{fig-arch}} take part in an
interaction, from the creation of a dialog context until the dialog
context retires.

The flow involves the following participants:

Principal:
: The user or organization on whose behalf Agent A acts.

Agent A:
: The initiating and delegating agent.

Agent B:
: The delegated agent. It can be an AI agent or a tool.

Intermediary:
: Zero or more gateways, proxies, or protocol adapters between Agent A and
  Agent B. An intermediary can terminate the transport connection, can act
  as a policy enforcement point, or can only relay messages.

Authorization service:
: Stands for the authorization mechanisms of the Security block: issuing
  tokens and acting as the authorization decision point. In an interaction
  across administrative domains each domain has its own.

Following prerequisites are assumed to be fulfilled:

- AI agents and tools have been discovered. Discovery and selection are
  outside the scope of this document.
- Each agent has an agent identity with credentials bound to it.
- Each administrative domain has its own authorization policy and
  authorization decision point. Crossing a trust boundary does not transfer
  policy authority, and the architecture assumes no global authorization
  authority.
- The principal has authorized Agent A to act on its behalf. How this is
  done is outside the scope of this document.

Flows:

1. Initiation: A pricipal starts intect with an Agent A. Agent A select
   other agents if required, in this case Agent B is discoverd and
   selected.

2. Connection Establishmend: Agent A establishes a transport connection
   toward Agent B, directly or through an intermediary. Where, if used,
   an intermediary terminates the connection, the connection between Agent A
   and the intermediary and the connection between the intermediary and
   Agent B are separate and are protected independently.

3. Authentication: Agent A and Agent B authenticate each other, at the transport layer, the application layer, or both.

4. Authorization: Agent A obtains authorization for each sub-task. The
   authorization is derived from the principal's authorization and limited
   to the sub-task.

5. Dialog Context: Agent A creates a dialog context for the current
   dialog with Agent B. The dialog is assigned a dialog context
   identifier. Similarly a context is also created between the principal
   and Agent A, not described here for brevity's sake.

6. Task Execution: The AI agents exchange messages within the dialog. The
   messages can carry multiple modalities. If Agent B delegates part of the
   task to another agent, it repeats flow 1 to 5 toward that agent, possibly
   in another administrative domain.

7. Dialog Continuity: If a transport connection is interrupted, the dialog
   continues over a new transport connection. Authorization is checked again,
   because it can expire or be revoked while the dialog is suspended or interrupted.

8. Task Completion: Agent B returns the result to Agent A and eventually 
   Agent A returns the complete result to the principal and retires the 
   dialogs context.

## Architecture Overview {#architecture}

{{fig-arch}} shows the architecture. The Agent Communication
Protocol uses the Dialog Propagation, Authorization and Delegation, and Identity
Management sub-blocks. It runs over the Transport layer, which is protected
by the Security layer.

~~~
+-------------+                       +-------------+
| User /      |                       | User /      |
| Principal   |                       | Principal   |
+------+------+                       +------+------+
       |                                     |
       v                                     v
+-------------+     +- - - - - -+     +-------------+  +--------+
|  Agent (A)  |<...>: Discovery :<...>|  Agent (B)  |  |  Tool  |
+------+------+     +- - - - - -+     +------+------+  +----+---+
       |                                     |              |
       +-------------------------+-----------+--------------+
                                 |
                                 v
+----------------------------------------------------------------+
|                Agent Communication Protocol                    |
|                                                                |
| +------------------+ +------------------+ +------------------+ |
| | Dialog           | | Authorization    | | Identity         | |
| | Propagation      | | and Delegation   | | Management       | |
| |                  | |                  | |                  | |
| | - Identifier     | | - Delegation     | | - Roles          | |
| | - Lifecycle      | | - Authorization  | | - Participant    | |
| | - Propagation    | | - Decision point | |   identity       | |
| | - Recovery       | |                  | | - Credential     | |
| |                  | |                  | |   reference      | |
| +--------+---------+ +---------+--------+ +---------+--------+ |
|          |                     |                    |          |
|          +---------------------+--------------------+          |
|                        Dialog Context                          |
+----------------------------------------------------------------+
                                 |
                                 v
+----------------------------------------------------------------+
|                          Transport                             |
|  - Multiplexed Streams                                         |
|  - Per-stream Semantics                                        |
|  - Connection continuity (Connection ID, TLS resumption)       |
|  - Protocols: HTTP, QUIC, WebTransport, WebRTC, MoQT           |
+----------------------------------------------------------------+
                                 |
                                 v
+----------------------------------------------------------------+
|                       Security                                 |
|  - Channel Protection                                          |
|  - Mutual Authentication                                       |
|  - Protocol: TLS 1.3, WIT/WPT                                  |
+----------------------------------------------------------------+
~~~
{: #fig-arch title="Reference architecture"}

TLS 1.3 provides channel protection and mutual authentication for all agent communication. Security is shown as a separate layer in {{fig-arch}} for clarity. When agents communicate over QUIC, TLS 1.3 is integrated into the QUIC handshake {{RFC9001}} and does not occupy a discrete layer above or below the transport. In architectures involving intermediaries where TLS is terminated at a proxy, application-layer authentication is required to maintain identity continuity across TLS termination points, as described in {{channel-protection}}.

QUIC provides multiplexed streams with per-stream semantics suitable for the heterogeneous communication patterns of agent interactions. MoQT (Media over QUIC Transport) adds a publish/subscribe layer over QUIC for the one-to-many and many-to-many group communication that point-to-point QUIC streams cannot provide.

OAuth 2.0 provides the authorization and delegation framework, enabling agents to obtain and present access tokens scoped to specific tasks. WIMSE provides workload identity for agents through a URI embedded in X.509 certificates at the TLS layer, and through Workload Identity Tokens (WIT) and WIMSE Proof Tokens (WPT) at the application layer. The Agent Communication Protocol maintains dialog continuity across connection changes using a stable dialog context identifier. Connection-level continuity is provided by QUIC Connection ID and TLS resumption. Agents interact at the top of the stack, each acting on behalf of a user or system.

The Agent Communication Protocol carries and associates the resulting tokens and identities with the dialog. It does not issue or validate them. The Security box in {{fig-arch}} groups the reused mechanisms. The transport-layer mechanisms protect the transport connection. The application-layer mechanisms are used by the Authorization and Delegation and Identity Management sub-blocks, and they remain in effect where an intermediary terminates the transport connection. They are drawn below the Transport layer for simplicity, not because they operate there.

Attestation, as defined in the SEAT WG, leverages the work of the RATS WG, including the RATS architecture {{RFC9334}} and Evidence formats such as the Entity Attestation Token (EAT) {{RFC9711}}, and binds attestation evidence to TLS. The evidence can be conveyed during the TLS handshake or at the application layer by extending {{RFC9261}}. Attestation is not shown as a discrete layer in {{fig-arch}} because its position in the stack is solution-specific. It is outside the scope of this document.

# Agent Communication Protocol {#acp}

The Agent Communication Protocol provides the mechanisms that keep an
interaction between users, agents, and tools coherent across network
interruptions, intermediaries, and trust boundaries. It is composed of three
sub-blocks, shown in {{fig-arch}}: Dialog Propagation, Authorization and
Delegation, and Identity Management.

The Agent Communication Protocol defines how dialog context, authorization
context, and identity information are carried and associated with a dialog.
It does not define the semantics of identity, authorization, or application
tasks. Those semantics are provided by the existing building blocks
described in {{security}} and {{existingworks}}.

## Dialog propagation {#dialog}

The Dialog propagation is the controlled transfer and transformation of dialog
context across agents that potentially exists in different trust domains, and time.
Dialog Propagation maintains the dialog context and carries it between
dialog participants, including across intermediaries. It is the sub-block
that allows a dialog context to remain recognizable over multiple hops, over time,
and across transport connections.

A propagated dialog context returns result(s) that the principal can verify when required.

### Dialog Context Identifier and Correlation {#dialog-id}

Every dialog context is required to have a context identifier, as described in
{{dialogcont}}. The dialog context identifier is required to:

- correlate all messages that belong to the dialog, in every direction and at
  every hop;
- survive the loss or replacement of a transport connection, so that the
  dialog is not tied to a TCP connection, QUIC connection, or TLS
  association;
- be usable by an intermediary to correlate the requests it receives with the
  requests it forwards, without access to task context.

A dialog can involve more than two participants, such as delegated agents,
intermediaries, or members of a group. The dialog context identifier is therefore not
required to imply a client-server relationship between participants.

### Dialog Continuity {#dialogcont}

Agent interactions are often long-lived, interruption-prone, and delegated across multiple hops. Each dialog is required to have a dialog context identifier that survives network interruption or change. If a transport connection is interrupted, an authorized agent uses the dialog context identifier to re-associate with the existing dialog instead of restarting the task. An intermediary uses the dialog context identifier to correlate the requests it receives with the requests it forwards. Neither function requires access to task context.

Dialog continuity is defined above any transport protocol connection (TCP connection, QUIC connection) or TLS association. QUIC Connection ID and TLS resumption provide connection-level continuity; they preserve transport and cryptographic state, not dialog context.

## Authorization and Delegation

Authorization and Delegation determines whether a participant is permitted to
act within a dialog and on whose behalf. This sub-block does not define an
authorization mechanism. It identifies the functions and the information that
the Agent Communication Protocol needs to associate with a dialog so that
existing authorization mechanisms can be applied at the points where they
matter.

### Delegation {#delegation}

Delegation is an act between participants, not a property of a
transport connection. The architecture distinguishes three roles:

- the principal, such as a user or an organization, whose authority is
  originally exercised;
- the delegating agent, which requests another agent to execute a task;
- the delegated agent, which executes the task.

In multi-hop delegation, an agent can be the delegated agent for one hop and
the delegating agent for the next. A delegating agent can re-delegate authurity
derived from the original principal. The delegation context associates these
roles with the dialog so that each participant can determine who it is
acting for and who is acting for it. A delegation can be revoked or terminated
independently of the dialog in which it was used.

The constraints in {{intent-execution}} and {{delegation-chain}} apply: the
authority passed at each hop is limited to the delegated task and is not
broadened along the chain.

## Identity Management

Identity Management establishes who the participants of a dialog are. It
builds on the agent identity requirements in {{agent-identity}} and on the
Agent Identity Management System model in {{I-D.ietf-wimse-aims}}. The Agent
Communication Protocol does not define identity credentials or authentication
protocols. It defines how participant identities are associated with the
dialog and how that association is preserved.

### Identity Roles {#identity-roles}

The architecture keeps the following identities distinct:

- the principal identity, which is the identity of the user or organization on
  whose behalf an agent acts;
- the agent identity, which is the identity of the agent itself and is
  distinct from that of the principal;
- the identity of an intermediary that relays or processes messages in the
  dialog.

Keeping these identities distinct is required so that a participant can
distinguish the principal, the delegating agent, and the delegated agent, and so that accountability can be
assigned to the right entity.

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

AI agents dynamically generate execution plans and issue sub-requests based on their own reasoning, which may diverge from the scope of the original user authorization. This architecture requires that authorization granted to an agent be limited to the scope of the delegated task, so that an agent cannot use it to authorize actions outside that scope. This depends on delegation mechanics in which an agent acts on behalf of the original user or principal and the authority passed at each hop is narrowed to the delegated task rather than broadened. Standardizing these delegation mechanics for AI agents is an active area of work in the OAuth WG. This document will reference the relevant outcomes as this work matures.

## Delegation Chain Integrity {#delegation-chain}

When one agent delegates a sub-task to another, the receiving agent is authorized only to the extent explicitly permitted by the delegating agent. In multi-hop scenarios, the delegation chain may span several agents before reaching the resource or tool ultimately invoked. At each hop, the receiving party is required to verify that the authority presented was derived from the human or system that originally authorized the task, has not been expanded beyond what was originally granted, and has not been forged or modified in transit. A delegating agent is not permitted to grant authority beyond what the original authorization permits for the delegated task. This draft will be updated with more details as the work progresses in the OAuth WG.

## Audit and Non-Repudiation {#audit}

Agents act on behalf of users/principals or other agents across multiple hops and
administrative domains. Audit requires that each action be traceable to the
principal that authorized it and the agent that executed it. For Non-repudiation,
this traceability is required to be cryptographically verifiable, so that
no party can later deny its role. The mechanism for recording and
verifying it is outside the scope of this document.

# Transport Aspects {#transport}

Transport for agent-to-agent communication spans several interdependent concerns: dialog continuity across long-running tasks, heterogeneous delivery semantics, explicit task and stream correlation, efficient transfer of large data, signaling for priority and cancellation, structured error propagation, and negotiation of modalities and group communication topologies. The use cases and protocol requirements that motivate these transport properties are discussed in {{I-D.agentic-ai-usecases-requirements}}. The transport substrate is not required merely to deliver bytes between endpoints; it is required to preserve the correctness, efficiency, and recoverability of delegated agent execution across administrative domains and under changing network conditions.

## Delivery Semantics {#delivery-semantics}

Agent communication requires heterogeneous transport semantics rather than a single uniform model.

Agent communication patterns fall into three broad delivery semantic classes, each with distinct transport properties:

Reliable ordered delivery:
: Used for control messages, workflow state updates, authorization checkpoints, and structured results. The transport substrate is required to guarantee in-order, lossless delivery for these message classes, and is required to isolate them from other traffic classes to avoid head-of-line blocking.

Low-latency, loss-tolerant delivery:
: Used for real-time audio, video, and high-frequency sensor or telemetry streams. The transport substrate is required to minimize latency for these streams and is not required to guarantee delivery or ordering.

High-throughput reliable delivery:
: Used for bulk transfer of large data between dialog participants. The transport substrate is required to support high-throughput reliable transfer with flow-control isolation from other traffic classes.

## Message Exchange Patterns {#message-exchange}

Agent tasks are not limited to simple request-response exchange. An agent may delegate work asynchronously, receive an acknowledgement before completion, stream partial results, emit progress updates, request additional authorization, and later return a final result or cancellation status. The transport is required to take these patterns explicitly into account, including correlation of messages to the relevant task, subtask, and dialog. Multiplexing is required so that multiple delegated tasks or tool invocations can proceed concurrently without unrelated head-of-line blocking. The transport is also required to permit out-of-order completion of concurrent subtasks while preserving per-task ordering for incremental output and terminal states.

## Priority and Scheduling {#priority}

Delegated agent work varies in urgency and consequence. Some exchanges are on the critical path of a user-visible task, while others are background enrichment, speculative reasoning, or deferred synchronization. The protocol is required to signal task priority and delivery urgency so that transport and scheduling layers can differentiate service. Such signaling is required to be advisory and policy-controlled; a sender is not permitted to raise priority beyond the authority or policy of the delegated task. Priority signaling is also required for coordination messages such as authorization responses, cancellation acknowledgements, and error notifications, because delaying them can break an otherwise correct workflow.

## Multimodal Negotiation {#multimodal}

Agents may exchange text, structured objects, images, audio, video, and sensor data within a dialog. The transport protocol is required to support negotiation of modalities and formats during dialog establishment or capability discovery. The negotiated set is binding unless explicitly updated; an endpoint is not permitted to send an unsupported modality. Transport-relevant modality properties, including reliability, latency sensitivity, and expected size, are required to inform stream selection, flow-control allocation, and framing. Modality negotiation therefore constrains both content and transport behavior.

## Structured Error and Progress Signaling {#error-signaling}

The transport is required to carry more than final results. It is required to also convey incremental output, progress updates, checkpoint states, cancellation states, and structured errors usable by an orchestrating peer. Error signaling is required to distinguish transport failure from application failure, authorization failure, policy rejection, timeout, validation failure, and interoperability problems. Progress and liveness notifications are required to be representable independently of final completion so that a peer can tell whether an agent is active, blocked on external input, or waiting in a recoverable state. It is required to ensure that terminal error or cancellation signals are not silently lost if an ordinary data stream fails.

## Cancellation and Interruption {#cancellation}

Agent execution may be interrupted by user/principal action, policy enforcement, higher-priority work, loss of authorization, or dependent-subtask failure. The protocol is required to define explicit application-layer cancellation and interruption signaling, rather than rely on connection teardown at transport protocol layer. Cancellation is required to identify the affected scope, including a single invocation, delegated subtree, or dialog. Receiving agents are required to treat cancellation as idempotent and report whether execution stopped, had already completed, or could not be fully rolled back due to an irreversible action. In multi-hop deployments, interruption semantics are required to propagate predictably so that dependent subtasks do not continue after withdrawal of the parent task.

## Multiple Communication topologies {#topologies}

Some agent interactions may use one-to-many or many-to-many communication among a group of agents whose membership may change during an exchange. The transport is required to support these delivery patterns and to handle agents joining or leaving the group while the exchange is active. An agent's authorization to participate is required to be verified when it joins, and the keying material is required to be updated whenever an agent joins or leaves, so that an agent can decrypt group traffic only while it is a member. MLS {{RFC9420}} provides group key establishment with these properties.


# Applicability of Existing IETF Work {#existingworks}
## Reuse As-Is

* TLS 1.3 {{!RFC9846}} provides mutual authentication and channel protection. Agents
  are required to negotiate TLS 1.3 or higher.
* QUIC {{!RFC9000}} and QUIC-TLS {{!RFC9001}} provide multiplexed streams and 0-RTT
  session resumption.
* OAuth 2.0 Token Exchange {{RFC8693}} provides the base token exchange mechanism.
* WIMSE Workload Credentials {{I-D.ietf-wimse-workload-creds}} are reused for agent
  identity; see {{agent-identity}}.

## Profile or Extend

The following existing protocols may require profiling or extension; this list is expected to evolve as new IETF and OAuth WG specifications emerge:

- The OAuth WG is actively discussing how existing and new mechanisms apply to AI agent authorization. {{I-D.ietf-oauth-identity-chaining}} addresses cross-domain authorization, and {{I-D.ietf-oauth-transaction-tokens}} addresses intra-domain token exchange between workloads. This document will track this work and profile the relevant outcomes once the OAuth WG reaches consensus.

- MoQT (Media over QUIC Transport): acts as a unified transport substrate for distributed agent state synchronization and real-time multimodal communications. Work needs to happen to provide a common mapping of request-response and streaming patterns onto the pub/sub model of MoQT in order to enable interoperability across diverse agent ecosystems.

## New Protocol Work

The Dialog Propagation Protocol shown in {{fig-arch}} is new protocol work.
It is expected to be specified separately. The new protocol work includes
the dialog identifier, context and lifecycle, plus the rules for carrying and associating identity, delegation and authorization information. Everything else is reused.

# Security Considerations {#secconsiderations}

AI agents enlarge the attack surface: they act autonomously, delegate across administrative domains, and operate over long-lived dialogs that rely on long-standing delegated authority. This section highlights threats specific to the building blocks in this document; detailed mitigations are expected in the specifications of those building blocks.

Delegated authority is the primary risk. A rogue or compromised agent may attempt to use delegated authority beyond the task it was granted, or to broaden it as it delegates onward. As required in {{intent-execution}} and {{delegation-chain}}, authority granted to an agent is scoped to the delegated task and cannot be broadened along the chain, and the delegation chain is verifiable at each hop so an intermediary cannot forge or escalate it.

Valid credentials do not show that an agent runs on an uncompromised platform. Attestation, as discussed in {{architecture}}, allows a peer to assess the integrity of the agent's execution environment.

QUIC and TLS 1.3 0-RTT data can be replayed by an attacker (Section 8 of {{RFC9846}}). Messages that trigger non-idempotent actions are not permitted in 0-RTT data.

# Privacy Considerations {#privacy}

Dialog context identifiers and dialog context persist across intermediaries and
trust boundaries. An intermediary or other observer can use them to link
interactions that would otherwise be separate and to track agent and user/principal's
activity over time. The Agent Communication Protocol is expected to analyze
these risks and specify mitigations, such as limiting the scope and
lifetime of dialog context identifiers.

Agent identity information is sensitive, particularly in multi-domain
deployments. Persistent agent identifiers used across dialogs and domains
enable tracking and correlation of agent activity. Pseudonymous or
temporary identifiers reduce linkability while preserving the ability to
audit and enforce accountability where required.

Because agents exchange rich, potentially sensitive multimodal task context, both message content and metadata require protection. Even with content encrypted, analysis of message sizes, timing, and stream patterns can leak the nature of an agent's tasks; mitigations such as padding may be warranted where that exposure matters.

# IANA Considerations {#ianaconsideration}

This document has no IANA actions.

# Acknowledgments
{: numbered="false"}

--- back
