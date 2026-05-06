# Delegated Authorization Execution Layer

**Working draft v0.1 — Components 5 (Delegated Capabilities) and 6 (Delegated Access)**

*Companion document to the Delegated Authorization Reference Architecture v1.1 and the Privacy-Preserving Profile v1.0.*

*This is an exploratory draft. The execution layer is bleeding-edge work and the patterns here should be pressure-tested before any wire-protocol commitments are made.*

---

## Table of Contents

1. [Introduction](#1-introduction)  
2. [Framing: Integration Patterns, Not New Credentials](#2-framing-integration-patterns-not-new-credentials)  
3. [Architectural Roles in the Execution Layer](#3-architectural-roles-in-the-execution-layer)  
4. [Component 5: Delegated Capabilities — The AuthZen Pattern](#4-component-5-delegated-capabilities--the-authzen-pattern)  
5. [Component 6: Delegated Access — The Credential Manager Pattern](#5-component-6-delegated-access--the-credential-manager-pattern)  
6. [Three AuthZen Deployment Models](#6-three-authzen-deployment-models)  
7. [The Two Execution Flows](#7-the-two-execution-flows)  
8. [Pressure-Test Scenarios](#8-pressure-test-scenarios)  
9. [Open Questions](#9-open-questions)  
10. [References](#10-references)

---

## 1\. Introduction

### 1.1 Purpose of This Document

The base Reference Architecture (v1.1) defines four components that describe a delegation: who the parties are (Relationship), why the delegator may delegate (Authority), what the specific task is (Obligations), and within what bounds it must be performed (Constraints). The Privacy-Preserving Profile defines how those credentials are handled when the delegation involves vulnerable populations.

This document addresses the **execution layer** — Components 5 (Delegated Capabilities) and 6 (Delegated Access) — which are the runtime patterns that translate the abstract delegation into concrete authorization decisions and credential issuances. Unlike Components 1–4, the execution layer is primarily about **integration with existing authorization infrastructure**, not about defining new credential formats.

### 1.2 Status

This is a v0.1 working draft that records the current thinking. Several of its claims are tentative and several open questions remain unresolved. The intent is to provide a foundation for pressure-testing the deployment model, not to specify a finished architecture. Wire-protocol details are deliberately deferred; this document describes *what* parties do and *which patterns* govern their interaction, not *how exactly* messages are framed on the wire.

### 1.3 Relationship to Other Specifications

This document references and builds upon:

- **Delegated Authorization Reference Architecture v1.1** — the four-component model this layer executes  
- **Privacy-Preserving Profile v1.0** — the layered profile that constrains what flows through the execution layer when vulnerable populations are involved  
- **OpenID AuthZen (OpenID Authorization API 1.0)** — the authorization decision API used at the Component 5 layer  
- **OAuth 2.0 Transaction Tokens** *(IETF draft)* — the within-trust-domain workload-chain pattern  
- **OpenID for Verifiable Credential Issuance (OID4VCI)** — a candidate inspiration for the Credential Manager's issuance protocol  
- **RFC 8693** — OAuth 2.0 Token Exchange, considered but not adopted as the Credential Manager's primary pattern (see §5.4)

---

## 2\. Framing: Integration Patterns, Not New Credentials

### 2.1 What the Execution Layer Is Not

The execution layer is not a new credential format. It does not introduce a new JWT type, a new claim vocabulary, or a new wire protocol that competes with existing standards. Components 5 and 6 are not new things to issue; they are existing things, used in patterns shaped by Components 1–4.

This framing is deliberate. The IETF and OpenID Foundation ecosystems already provide rich machinery for runtime authorization decisions (AuthZen, OPA, XACML, Cedar) and for credential issuance (OAuth 2.0, OID4VCI, VC Data Model). The execution layer's contribution is to define how the abstract delegation expressed in Components 1–4 plugs into that machinery — not to replace it.

### 2.2 What the Execution Layer Is

The execution layer specifies:

- **Roles** that operate at runtime: AuthZen PDP, Credential Manager, Transaction Token Service, PEP variants  
- **Patterns** for how delegation evidence (Components 1–4) is presented to those roles  
- **Trust relationships** between the delegator, the delegatee, and the various execution-layer parties  
- **Sequence flows** showing how a delegated action proceeds end-to-end, from initiation to resource access  
- **Audit obligations** at the execution layer that complement the audit trail held by the verification intermediary in privacy-preserving deployments

### 2.3 Possible Small Additions

Although the bias is toward zero new JSON, two small structures may emerge as useful:

- A **Delegation Evidence Bundle** — a transport wrapper that packages the relevant Components 1–4 credentials together with integrity protection, for presentation to AuthZen or to a Credential Manager. This is not a new credential type; it is a JWS-signed envelope around existing credentials.  
- A **Delegation Context Object** for AuthZen — a structured representation of the delegation evidence as an AuthZen `context` field value, so that PDPs receive the evidence in a predictable, parseable form.

Neither of these is normatively committed to in this draft; both are candidates for resolution after pressure-testing.

---

## 3\. Architectural Roles in the Execution Layer

This section enumerates the roles that operate at runtime. Each role is described in terms of its **trust properties** and **functions**, not its physical deployment — a single regulated entity may play multiple roles, and a single role may be implemented across multiple deployed components.

### 3.1 The Delegatee Workload

The party that actually performs the delegated task. May be:

- A natural person operating through a client application  
- An autonomous AI agent  
- A human-controlled tool (a clinician's order entry system)  
- A multi-agent orchestrator that further delegates to sub-agents

The delegatee workload holds standing IAM permissions provisioned by its enterprise (or by the delegator, in consumer cases) and presents delegation evidence to obtain runtime authorization for delegated actions.

### 3.2 The AuthZen Policy Decision Point (PDP)

A runtime authorization decision service conformant with OpenID AuthZen. Receives a structured request describing the subject, resource, action, and context (including delegation evidence) and returns a permit/deny decision with optional obligations.

The PDP evaluates policy that combines:

- The delegatee's standing entitlements (what the IAM system says they may do)  
- The delegation evidence (what Components 1–4 say is currently authorized)  
- Resource-side policy (what the resource server requires for this specific action)  
- Environmental context (time, location, threat level, etc.)

The PDP is conceptually trust-domain-internal: it serves the trust domain in which it is deployed. Cross-trust-domain authorization typically involves *multiple* PDPs, each evaluating the portion of the decision that falls within its domain.

### 3.3 The Policy Enforcement Point (PEP)

The runtime component that intercepts an action and consults a PDP before allowing it to proceed. PEPs may live in three architectural locations (§6); the choice depends on deployment realities.

### 3.4 The Transaction Token Service

Within a single trust domain, the Transaction Token Service mints short-lived, immutable Transaction Tokens that propagate the original request context through a chain of microservice calls. Per the IETF draft, Transaction Tokens carry the original request's authorization claims and prevent privilege escalation as work flows through the system.

In our model, when a delegated action arrives at a trust domain boundary, an API Gateway (acting as the entry-point PEP) validates the delegation evidence, exchanges it for a Transaction Token via the Transaction Token Service, and the Transaction Token then propagates through downstream calls. Downstream services trust the Transaction Token as evidence that the originating request was authorized; they do not re-validate the full delegation evidence at each hop.

### 3.5 The Credential Manager

A delegator-domain role that holds (or has the authority to mint) credentials for the delegator's accounts on third-party systems, and issues purpose-bound short-lived credentials to delegated workloads upon presentation of valid delegation evidence.

The Credential Manager is described as a **role** rather than a single deployment. In different scenarios it may be:

- A wallet application on the delegator's device (consumer travel-agent case)  
- A cloud service the delegator has authorized (a personal "credential vault")  
- A component of an enterprise identity platform when the delegator is acting in a work capacity  
- Multiple co-operating parties (e.g., a device-resident passkey provider for high-assurance credentials, plus a cloud vault for lower-assurance ones)

What unifies these deployments is the trust relationship: the Credential Manager acts under the delegator's control, with the delegator's authority to release credentials, and the delegator can revoke its authority at any time.

The Credential Manager is distinct from the Verification Intermediary defined in the Privacy-Preserving Profile, which sits in the regulated/oversight trust domain. In some deployments (especially within a single regulated entity acting as both delegator and intermediary), they may be co-located; in consumer agentic-AI deployments they are clearly distinct.

### 3.6 The Verification Intermediary (privacy-preserving deployments only)

When the deployment uses the Privacy-Preserving Profile, the verification intermediary handles the relying-party-facing presentation of delegation evidence. At the execution layer, this means the AuthZen PDP and the upstream resource server may receive evidence in the form of presentation tokens issued by the verification intermediary, rather than the raw Component 1–4 credentials. The PEP and PDP must be able to validate intermediary-issued presentations as well as raw credentials, depending on which deployment profile is in effect.

### 3.7 The Resource Server (Upstream Account Holder)

The third-party system holding the resource the delegated action targets — the hotel's loyalty API, the patient's medical record system, the bank's account API. This party knows nothing about the delegation as such; it simply receives a credential at its API and decides whether to honor it.

The credential it receives may be:

- An access token issued by its own authorization server (in cases where the Credential Manager performs token exchange against that AS)  
- A Verifiable Presentation derived from a credential it previously issued to the delegator  
- A bearer credential that the delegator pre-provisioned (an API key held in the Credential Manager)  
- A cryptographically scoped capability token in a non-OAuth format

In all cases, the resource server's view is conventional: it sees a credential at its API, validates it per its existing rules, and either honors or rejects the request. The resource server does not need to understand the delegation framework.

---

## 4\. Component 5: Delegated Capabilities — The AuthZen Pattern

### 4.1 What Component 5 Is

Component 5 — Delegated Capabilities — is the runtime authorization decision that combines the delegatee's standing entitlements with the contextual delegation evidence to produce a permit/deny decision for a specific action.

The delegatee already has *some* IAM permissions: an enterprise has provisioned its AI orchestrator with the ability to call certain APIs; a clinician's order system has standing access to the EHR; a consumer's installed agent has been granted certain device permissions. These standing entitlements are necessary but not sufficient for delegated action: the workload must additionally demonstrate that *this specific action* is authorized under *this specific delegation*.

The natural mechanism for this combination is a runtime authorization decision API. AuthZen is the OpenID Foundation's specification for such an API, and we adopt it as the reference pattern.

### 4.2 The AuthZen Request, Extended for Delegation

A standard AuthZen evaluation request has four top-level fields: `subject`, `resource`, `action`, and `context`. The delegation evidence flows through the `context` field as a structured `delegation` sub-object:

{

  "subject": {

    "type": "workload",

    "id": "spiffe://travel-co.example/agent-7291"

  },

  "resource": {

    "type": "hotel\_reservation",

    "id": "marriott-boston-7842"

  },

  "action": {

    "name": "create"

  },

  "context": {

    "delegation": {

      "relationship\_jwt": "\<base spec Relationship JWT\>",

      "authority\_jwt": "\<base spec Authority JWT\>",

      "task\_jwt": "\<base spec Task JWT\>"

    },

    "request\_time": "2026-04-23T14:32:00Z",

    "request\_origin": {

      "ip": "203.0.113.42",

      "user\_agent": "Travel-Agent/2.1"

    }

  }

}

The PDP evaluates a policy that:

1. Verifies the binding chain among the three JWTs (per §8.1 of the base spec)  
2. Validates each JWT's signature and lifecycle  
3. Checks revocation status (with caching per §8.3)  
4. Evaluates the delegatee against the Relationship's eligibility criteria  
5. Evaluates the requested action against the Authority's scope domain and the Task's obligations  
6. Evaluates the request context against the Task's constraints  
7. Combines all of the above with the workload's standing IAM entitlements  
8. Returns permit or deny, with optional obligations (e.g., "log this action with the audit identifier")

### 4.3 The Decision Response

The AuthZen response carries the decision, optional obligations, and (if denied) optional reason information. For delegation-driven decisions, the response should include an `audit_id` that ties back to the verification intermediary's audit trail when the privacy-preserving profile is in effect:

{

  "decision": "permit",

  "obligations": \[

    {

      "id": "log\_delegated\_action",

      "audit\_id": "ai-9c5d1e3f4a6b",

      "scope": "urn:scope:guardianship:platform-account-management"

    }

  \]

}

In privacy-preserving deployments, the `audit_id` is the only identifier the PEP and resource server need to retain; full reconstruction of who acted under what authority is recoverable only by the intermediary.

### 4.4 What the PDP Is Not Asked to Decide

A clean separation of concerns: the AuthZen PDP decides whether the *current request* falls within the *delegation envelope* defined by Components 1–4. It does not decide:

- Whether the resource server should ultimately honor the request — that is the resource server's concern, and may involve its own additional checks (anti-fraud, rate limiting, account-specific rules)  
- Whether the workload has the *credentials* needed to access the resource — that is Component 6's concern, handled by the Credential Manager  
- Whether the delegation itself was legitimately formed — that is a Component 1–2 issuance concern, predating runtime

The PDP's scope is narrow and precise: given a valid delegation, valid runtime context, and a specific requested action, does the delegation envelope permit it.

---

## 5\. Component 6: Delegated Access — The Credential Manager Pattern

### 5.1 What Component 6 Is

Component 6 — Delegated Access — is the issuance of a credential that a delegated workload can present to an upstream account holder (the resource server) to access the delegator's data or perform the delegator's actions on third-party systems. It addresses the question: "Now that the delegated action is authorized, how does the workload actually present itself to the resource server?"

The delegator typically already has a credential at the resource server — a hotel loyalty account, a healthcare portal login, a bank account, an email account. Naively, the workload would need a copy of that credential to act. This is a poor pattern: it gives the workload long-lived, broad access to the delegator's account, beyond what any specific task requires.

The Credential Manager pattern replaces this with **purpose-bound, short-lived credential issuance**: the workload presents the delegation evidence to the Credential Manager, which validates the evidence and issues a credential narrowly scoped to the specific task. The workload uses this scoped credential at the resource server and discards it.

### 5.2 The Conceptual Model

sequenceDiagram

    participant Delegator

    participant Workload as Delegatee Workload

    participant CM as Credential Manager\<br/\>(delegator-domain)

    participant RS as Resource Server\<br/\>(third party)

    Note over Delegator,CM: Establishment time:\<br/\>delegator authorizes CM\<br/\>with their credentials

    Delegator-\>\>CM: Authorize CM to hold/mint\<br/\>credentials for resource server

    Note over Workload,RS: Run time

    Workload-\>\>CM: Present delegation evidence\<br/\>(Relationship \+ Authority \+ Task JWTs)\<br/\>+ requested credential scope

    CM-\>\>CM: Validate evidence\<br/\>Evaluate request against\<br/\>delegator's policy

    CM-\>\>Workload: Purpose-bound credential\<br/\>(short-lived, scoped to task)

    Workload-\>\>RS: Present credential at API

    RS-\>\>RS: Validate credential per RS's\<br/\>existing rules

    RS-\>\>Workload: Resource access granted

### 5.3 The Credential Manager's Trust Properties

What makes the Credential Manager different from a generic OAuth Authorization Server is its trust relationship to the delegator:

- **The Credential Manager acts as the delegator's authorized issuer**, not as a federated identity provider. It issues credentials that *represent the delegator's authority* over their own accounts.  
- **The Credential Manager validates delegation evidence as the gating mechanism for issuance**, not user interaction. In contrast to OID4VCI's typical flow (where the user interactively consents at the issuer), the Credential Manager issues based on the validated delegation that already encodes the user's prior consent and bounds.  
- **The Credential Manager applies the delegator's policy** — the delegator may configure their Credential Manager with rules like "this AI agent gets read-only access to the loyalty account, never write access" or "no credentials issued for spending over $1000 without out-of-band confirmation." This is a layer of policy *additional to* the constraints in the Task JWT, expressing the delegator's preferences about credential issuance specifically.

### 5.4 Why Not Token Exchange?

OAuth 2.0 Token Exchange (RFC 8693\) is the closest existing pattern, and the natural question is whether the Credential Manager is simply an Authorization Server that does Token Exchange with the delegation evidence as the `subject_token`. The answer is "partly, but not really," for three reasons:

**Token Exchange is trust-domain-internal by design.** It assumes the AS has visibility into both the subject token's issuer and the resulting access token's audience. The Credential Manager pattern crosses trust domains: the delegation evidence is issued by the delegator's identity systems, but the resource server (where the issued credential is honored) is a third party that has no relationship with the delegator's identity systems.

**Token Exchange produces OAuth tokens.** The Credential Manager may need to produce diverse credential formats: OAuth tokens for some resource servers, Verifiable Presentations for others, API keys, scoped sessions, even passkey-derived credentials. A Token Exchange-only model is too narrow.

**Token Exchange's policy model is thin.** Token Exchange specifies the wire format and the basic security model but leaves issuance policy entirely to the AS. The Credential Manager pattern needs explicit treatment of how the delegation evidence drives issuance decisions, which Token Exchange does not provide.

That said, **Token Exchange is a reasonable wire-protocol implementation** for the Credential Manager when the delegation crosses domains in a way the resource server's AS supports. The Credential Manager pattern subsumes Token Exchange as one possible implementation, alongside OID4VCI-style flows, VC issuance, and bespoke patterns.

### 5.5 What the Credential Manager Issues

The credentials issued by the Credential Manager are characterized by:

- **Short-lived** — typically minutes to hours, never longer than the Task JWT's `exp`  
- **Purpose-bound** — scoped to the specific obligation in the Task, not the full breadth of the delegator's account access  
- **Audience-bound** — tied to a specific resource server, not transferable  
- **Audit-traceable** — issuance is logged with a reference to the underlying delegation evidence  
- **Format-appropriate** — whatever format the resource server accepts (OAuth token, VP, API key, etc.)

The Credential Manager does not issue credentials for delegations whose evidence fails validation, whose Task expires before the requested credential lifetime, or whose constraints are not satisfied by the request context.

### 5.6 Audit and Revocation

The Credential Manager records every issuance in an audit log accessible to the delegator. This log shows what credentials were issued, to which workload, under which delegation, for what purpose, and at what time. Delegators retain the right to revoke individual credentials, classes of credentials, or the underlying delegation entirely.

When a delegation is revoked at the Component 1–2 level, the Credential Manager MUST stop issuing credentials against it within the enforcement window defined in base spec §8.3 (24 hours by default). Already-issued short-lived credentials may continue to function until their natural expiry; the bound on damage is the credential's lifetime, which is why short lifetimes matter.

For higher-assurance deployments, the Credential Manager may issue credentials that themselves carry a `revocation_endpoint` allowing the resource server to check liveness in real time — at the cost of online dependency at access time.

---

## 6\. Three AuthZen Deployment Models

The PEP that consults the AuthZen PDP can live in three architectural locations. These are not competing patterns; they correspond to different deployment realities, and a single end-to-end flow may touch all three.

### 6.1 Model A: Resource-Server-Side PEP

The API the delegated workload is calling enforces authorization. The workload presents its credential and the delegation evidence (or a reference to it); the resource server's PEP consults the PDP before allowing the action.

sequenceDiagram

    participant Workload as Delegatee Workload

    participant RS as Resource Server\<br/\>(with PEP)

    participant PDP as AuthZen PDP

    Workload-\>\>RS: API request with credential\<br/\>+ delegation evidence

    RS-\>\>PDP: Evaluate(subject, resource,\<br/\>action, delegation context)

    PDP-\>\>RS: Permit / Deny

    alt Permit

        RS-\>\>Workload: Resource response

    else Deny

        RS-\>\>Workload: 403 Forbidden

    end

**When this applies:** The resource server is engineered for delegation-aware authorization. This is most natural when the resource server is operated by the same organization as the PDP, or when the resource server is a participant in a federation that has standardized on delegation-aware AuthZen.

**Trade-offs:** The resource server bears the burden of receiving and validating delegation evidence, which not all third-party APIs will support. The PDP and resource server must share a trust relationship.

### 6.2 Model B: Workload-Side PEP (Sidecar)

The delegatee workload's runtime intercepts outbound calls, consults a local PDP, and only proceeds if the PDP permits. The resource server sees a conventional credential and is unaware that the workload performed any pre-flight authorization check.

sequenceDiagram

    participant Logic as Workload Logic

    participant Sidecar as Workload Sidecar PEP

    participant PDP as AuthZen PDP

    participant CM as Credential Manager

    participant RS as Resource Server

    Logic-\>\>Sidecar: I want to call RS to do X

    Sidecar-\>\>PDP: Evaluate(workload, RS, X,\<br/\>delegation context)

    PDP-\>\>Sidecar: Permit

    Sidecar-\>\>CM: Need credential for X at RS

    CM-\>\>Sidecar: Purpose-bound credential

    Sidecar-\>\>RS: Conventional API call\<br/\>with credential

    RS-\>\>Sidecar: Resource response

    Sidecar-\>\>Logic: Result

**When this applies:** The resource server is a third-party API that does not support delegation-aware authorization. The workload's organization places the enforcement responsibility on the workload itself, ensuring it never makes outbound calls beyond its delegation envelope.

**Trade-offs:** The PEP is on the workload's side of the trust boundary, so it must be assumed honest — a compromised workload could bypass its own sidecar. This is mitigated by deploying the sidecar in an isolated execution context (a separate process, container, or enclave) so that compromise of the workload logic does not automatically compromise the sidecar.

### 6.3 Model C: API Gateway PEP with Transaction Tokens

The trust domain's perimeter (the API Gateway) is the primary PEP. Inbound requests are validated once at the gateway, exchanged for a Transaction Token, and the Transaction Token propagates through downstream microservices that trust it.

sequenceDiagram

    participant External as External Caller\<br/\>(workload from another domain)

    participant GW as API Gateway PEP

    participant PDP as AuthZen PDP

    participant TTS as Transaction Token Service

    participant SvcA as Internal Service A

    participant SvcB as Internal Service B

    External-\>\>GW: API request with delegation evidence

    GW-\>\>PDP: Evaluate(external, resource,\<br/\>action, delegation context)

    PDP-\>\>GW: Permit

    GW-\>\>TTS: Mint Txn Token\<br/\>(carrying delegation context)

    TTS-\>\>GW: Txn Token

    GW-\>\>SvcA: Forward request with Txn Token

    SvcA-\>\>SvcA: Validate Txn Token

    SvcA-\>\>SvcB: Internal call with Txn Token

    SvcB-\>\>SvcB: Validate Txn Token

    SvcB-\>\>SvcA: Response

    SvcA-\>\>GW: Response

    GW-\>\>External: Response

**When this applies:** The receiving organization has multiple internal services that participate in fulfilling the delegated request. The Transaction Token preserves authorization context as work flows through the system, without requiring each microservice to independently validate the delegation evidence.

**Trade-offs:** The Transaction Token Service becomes a critical trust component. The pattern works only within a single trust domain; cross-domain workload chains require either re-entering through another gateway or using the Credential Manager pattern to obtain credentials that the next domain accepts.

### 6.4 Combining Models

A single end-to-end flow often combines all three models. A consumer's AI Travel Agent (Model B sidecar protecting outbound calls) might call into an aggregator's API (Model C gateway with internal Transaction Tokens), which in turn calls a hotel chain's API (Model A resource-server-side PEP for its loyalty system). Each domain enforces its own boundary; the delegation evidence is the consistent thread that runs through all three.

---

## 7\. The Two Execution Flows

### 7.1 Flow X: Cross-Trust-Domain Personal-Data Access

When a delegated workload needs to access the delegator's data on a third-party system, the Credential Manager pattern handles the credential issuance. The high-level flow:

sequenceDiagram

    participant Workload as Delegatee Workload

    participant PEP as Sidecar / Gateway PEP

    participant PDP as AuthZen PDP

    participant CM as Credential Manager\<br/\>(delegator-domain)

    participant RS as Resource Server\<br/\>(third party)

    Workload-\>\>PEP: Request to call RS

    PEP-\>\>PDP: Authorize with delegation evidence

    PDP-\>\>PEP: Permit

    PEP-\>\>CM: Request credential for RS\<br/\>(presenting delegation evidence)

    CM-\>\>CM: Validate evidence\<br/\>+ apply delegator policy

    CM-\>\>PEP: Purpose-bound credential

    PEP-\>\>RS: Call with credential

    RS-\>\>PEP: Response

    PEP-\>\>Workload: Result

The defining property: the credential the workload presents to the resource server is *not* the workload's own enterprise credential, nor a long-lived copy of the delegator's credential, but a freshly minted, purpose-bound credential issued by the delegator-domain Credential Manager.

### 7.2 Flow Y: Within-Trust-Domain Workload Chain

When a delegated action results in work flowing through multiple services *within a single trust domain*, the Transaction Token pattern handles propagation. The high-level flow:

sequenceDiagram

    participant Origin as Origin Workload

    participant GW as API Gateway PEP

    participant PDP as AuthZen PDP

    participant TTS as Txn Token Service

    participant Svc1 as Service 1

    participant Svc2 as Service 2

    participant Svc3 as Service 3

    Origin-\>\>GW: Delegated request with evidence

    GW-\>\>PDP: Authorize

    PDP-\>\>GW: Permit

    GW-\>\>TTS: Exchange evidence for Txn Token

    TTS-\>\>GW: Txn Token

    GW-\>\>Svc1: Request \+ Txn Token

    Svc1-\>\>Svc2: Sub-request \+ Txn Token

    Svc2-\>\>Svc3: Sub-request \+ Txn Token

    Svc3-\>\>Svc2: Result

    Svc2-\>\>Svc1: Result

    Svc1-\>\>GW: Result

    GW-\>\>Origin: Result

The defining property: the original delegation evidence is validated *once* at the trust domain boundary; downstream services trust the Transaction Token as proof that the originating request was authorized.

### 7.3 When Flows X and Y Combine

Realistic scenarios combine the two flows. A consumer's AI Travel Agent that needs to book a hotel might:

1. (Flow Y in the agent's home environment) The agent's sidecar PEP authorizes the outbound call against the agent's standing entitlements \+ the delegation evidence  
2. (Flow X) The sidecar requests a credential from Bob's Credential Manager for the hotel chain's loyalty system  
3. (Flow Y in the hotel chain's environment) The credential is presented at the hotel chain's API Gateway; the gateway PEP validates it, mints an internal Transaction Token, and the booking flows through the chain's internal services  
4. (Audit) The AuthZen PDP in Bob's environment, the Credential Manager, and the hotel chain's audit infrastructure each record their own portions of the audit trail; together they reconstruct the full picture if needed

This composition is the realistic deployment pattern. The two flows are not alternatives but complementary patterns used at different points in the same end-to-end path.

### 7.4 Honest Acknowledgement

The two-flow framing is a starting hypothesis, not a settled architecture. Specifically:

- We have not pressure-tested what happens when the trust boundary itself is fuzzy (e.g., a workload running in a SaaS provider that is contractually but not architecturally separate from the delegator's domain)  
- We have not addressed how the AuthZen PDP and the Credential Manager coordinate their policy decisions — currently each has its own policy, and a delegation may be permitted by one and denied by the other; the resolution semantics need work  
- We have not considered cases where the delegator does not have a Credential Manager (e.g., low-tech consumers or institutional delegators that delegate without holding the underlying credentials themselves)

These are addressed in §9 (Open Questions) and require pressure-testing before they harden.

---

## 8\. Pressure-Test Scenarios

This section walks through five scenarios, each chosen to stress a different aspect of the execution layer model. The intent is to expose where the model holds and where it breaks.

### 8.1 Scenario 1: AI Travel Agent (cross-domain credential issuance)

**Setup:** Bob delegates to his AI Travel Agent (operated by Travel Co.) the authority to book a hotel for a conference. Bob has hotel loyalty accounts at Marriott and Hilton, held in his personal Credential Manager. Bob is a consumer; the agent is a third-party SaaS.

**Components 1–4 (recap):**

- Relationship: Bob → Travel Agent (service-client type)  
- Authority: agentic profile, scope \= `https://schemas.example.com/scope/travel`  
- Task: hotel booking, Boston, May 12–15  
- Constraints: $400/night, walking distance to Hynes, Marriott or Hilton

**Execution layer flow:**

1. Bob's Travel Agent receives the Task JWT. It identifies that the task requires booking at a Marriott or Hilton property, both of which Bob has accounts at.  
2. The Travel Agent's sidecar PEP validates the delegation evidence against Travel Co.'s AuthZen PDP. Travel Co.'s PDP confirms that the agent is permitted to act under this delegation (the delegation falls within the agent's scope\_domain, the Task is well-formed, the constraints are evaluable).  
3. The agent contacts Bob's Credential Manager (running on Bob's phone, app-mediated). It presents the delegation evidence and requests a credential for the Marriott loyalty API.  
4. Bob's Credential Manager validates the evidence. It applies Bob's policy: "AI agents acting under travel delegation may receive read-write loyalty credentials capped at the constraint amount." Validation passes.  
5. Bob's Credential Manager issues a purpose-bound OAuth access token, scoped to "create reservation, max value $1200, valid for 30 minutes," via an OID4VCI-style flow (where the delegation evidence plays the role normally played by user interactive consent).  
6. The agent presents this token at Marriott's API. Marriott's API sees a conventional OAuth token (issued by Bob's authorized Credential Manager, which Marriott federates with). It validates the token per its existing rules and creates the reservation.  
7. The credential expires 30 minutes later. The reservation persists; the agent's ability to make further calls under this credential does not.

**What the model handles well:** Cross-domain personal-data access works cleanly. The agent never holds a long-lived copy of Bob's loyalty credentials. Bob's policy is applied at issuance time. The credential's short lifetime bounds the damage from compromise.

**What this scenario stresses:** The federation between Bob's Credential Manager and Marriott's authorization server is non-trivial. Marriott has to recognize Bob's Credential Manager as an authorized issuer, which requires either Bob having pre-registered his Credential Manager with Marriott (a federation step that today's loyalty programs do not support) or a broader trust framework for Credential Manager federations. This is a real-world friction point that is genuinely unresolved in the industry.

### 8.2 Scenario 2: Multi-Agent Orchestration (within-trust-domain Txn Token chain)

**Setup:** A corporate AI orchestrator at Acme Co. receives a complex task: "prepare a sales proposal for Customer X." It delegates sub-tasks to specialist agents: a research agent, a proposal-drafting agent, a pricing agent, and a legal-review agent. All are within Acme Co.'s trust domain.

**Components 1–4:**

- Relationship: Sales VP → Orchestrator (employer-employee type, with role-tuple group at the orchestrator level allowing it to further delegate)  
- Authority: corporate profile, scope \= `https://acme.example/scope/sales-operations`, further\_delegation allowed with max\_depth=2  
- Task: prepare sales proposal for Customer X, with specific obligations and constraints

**Execution layer flow:**

1. The orchestrator presents the Task JWT to Acme's API Gateway. The gateway PEP validates the evidence against Acme's AuthZen PDP. Permit.  
2. The gateway exchanges the delegation evidence for a Transaction Token via Acme's Transaction Token Service. The Txn Token carries the delegation context, the original task identifier, and the orchestrator's identity.  
3. The orchestrator dispatches sub-tasks to the research, proposal-drafting, pricing, and legal-review agents, each receiving the Txn Token.  
4. Each specialist agent operates under the same Txn Token. When the pricing agent calls the pricing service, the pricing service validates the Txn Token, sees that this is a delegated sales-proposal action, and provides pricing data within the constraints.  
5. When the legal-review agent calls the contract template service, the same Txn Token authorizes it.  
6. The aggregated output flows back to the orchestrator, which produces the final proposal.

**What the model handles well:** The full delegation evidence is validated once at the gateway. Internal services trust the Txn Token; they don't re-validate the underlying JWTs. The Txn Token is short-lived and bounded to the duration of the original task. Audit traces every internal call back to the originating delegation.

**What this scenario stresses:** The further\_delegation semantics from base spec Component 2 must be operationally honored at runtime. When the orchestrator dispatches to specialist agents, those agents are nominally sub-delegates; the Txn Token is acting as the wire-format proof of their authority. We have not specified how the Txn Token represents the further\_delegation chain — does it include the sub-agent's identity? Does it allow downstream services to verify that the sub-agent was permitted under the original further\_delegation rules? This is a real gap.

### 8.3 Scenario 3: Healthcare Narcotic Administration (high-assurance with audit)

**Setup:** Dr. Smith's Task JWT (from earlier scenarios) authorizes on-call nurses to administer 5mg morphine IV PRN q4h to patient MRN-7842915. A nurse is now ready to administer.

**Execution layer flow:**

1. The nurse's clinical workstation presents the delegation evidence to the hospital's medication dispensing system.  
2. The dispensing system's PEP (Model A — resource-server-side) consults the hospital's AuthZen PDP.  
3. The PDP validates the evidence, confirms the nurse is in the policy-defined group (Cedar policy evaluation at the eligibility tier), evaluates the conditional constraint (current pain score), confirms the count constraint hasn't been exhausted, and permits.  
4. The dispensing system releases the medication. The action is logged with the audit identifier and the count constraint is decremented.  
5. No external Credential Manager is involved — the entire flow is within the hospital's trust domain, and the "credential" here is the dispensing system's internal authorization to release the medication, not a third-party API token.

**What the model handles well:** High-assurance delegation, layered constraints, in-trust-domain enforcement, full audit. The model's clean separation of authorization (Component 5\) from credential issuance (Component 6\) means the absence of Component 6 in this scenario is natural rather than awkward — there's no third-party resource server involved.

**What this scenario stresses:** This scenario reveals that **Component 6 is not always required**. Some delegated actions are entirely within the trust domain that owns both the workload and the resource. The execution layer model needs to make explicit that Component 5 is universally relevant and Component 6 is conditional on cross-domain personal-data access.

### 8.4 Scenario 4: Cross-Trust-Domain Workload Chain (the hard one)

**Setup:** Bob's AI Travel Agent (Travel Co.) needs to coordinate a complex booking that includes a hotel (Marriott), a restaurant reservation (OpenTable), a rental car (Avis), and a flight (Delta). Travel Co. has its own internal sub-agents for each domain. Each external service has its own trust domain.

**This is the cross-trust-domain scenario you flagged as raising many questions. It does.**

**Execution layer flow (one possible interpretation):**

1. Bob's Task JWT authorizes "trip planning for Boston, May 12–15," with constraints across all four domains (hotel under $400, restaurant under $150, car under $80/day, flight under $600).  
2. Travel Co.'s API Gateway validates the evidence against Travel Co.'s PDP. Permit. Mints a Travel Co.-internal Txn Token.  
3. The orchestrator dispatches sub-tasks to Travel Co.'s hotel-agent, restaurant-agent, car-agent, flight-agent.  
4. Each sub-agent operates under the Travel Co. Txn Token *within Travel Co.'s domain*. But each needs to call out to its respective external service.  
5. **Here's the hard question.** When the hotel-agent calls Marriott's API, what does it present?

**Option A:** The hotel-agent goes to Bob's Credential Manager directly, presenting the original delegation evidence (not the Travel Co. Txn Token, because Bob's Credential Manager doesn't trust Travel Co.'s Txn Token Service). It receives a Marriott-scoped credential and proceeds. Same for restaurant-agent → OpenTable, car-agent → Avis, etc.

**Option B:** Travel Co.'s Txn Token Service is itself authorized by Bob to act on his behalf for outbound calls. It exchanges the internal Txn Token for purpose-bound credentials from Bob's Credential Manager, on behalf of the sub-agents. The sub-agents present a Marriott credential they got from Travel Co.'s gateway, which got it from Bob's Credential Manager.

**Option C:** Bob's Credential Manager has issued Travel Co. a "session credential" at the start of the trip-planning task that empowers Travel Co.'s gateway to mint Marriott/OpenTable/Avis/Delta credentials on Bob's behalf for the duration of the task.

**What this scenario stresses (heavily):**

- **Option A** is cleanest from a trust standpoint (Bob's CM only ever issues credentials to parties presenting valid delegation evidence) but operationally awkward — every sub-agent needs direct connectivity to Bob's CM, and Bob's CM may receive many parallel issuance requests.  
- **Option B** is operationally cleaner (one connection between Travel Co. and Bob's CM) but introduces a "delegation of credential issuance authority" concept that we have not specified. Travel Co.'s gateway is now a *secondary* Credential Manager, which has its own trust implications.  
- **Option C** is the slickest but requires Bob to have configured an explicit "session credential" relationship with Travel Co. up front, which conflates the delegation with credential pre-authorization.

**This scenario shows that cross-trust-domain workload chains are genuinely under-specified in our model.** The combinations of "where is the credential manager," "who holds whose Txn Tokens," and "how does delegation evidence cross trust boundaries" are not yet resolved. The honest answer is: this is the area requiring the most pressure-testing, and any of options A, B, or C may be appropriate for different deployments. We should describe the trade-offs and let deployments choose, while flagging the design space as open.

### 8.5 Scenario 5: Child-Welfare Platform Account Setup (privacy-preserving profile)

**Setup:** Per the example in the Privacy-Preserving Profile (§9), a residential care home manager is setting up a 12-year-old looked-after child's social media platform account. The verification intermediary holds the rich credentials.

**Execution layer flow:**

1. The manager initiates the account setup. The platform contacts the verification intermediary requesting authorization for "data processing consent" for this account.  
2. The verification intermediary acts as the Credential Manager *and* the Verification Intermediary in this deployment (they are co-located when the deployment is fully regulated).  
3. The intermediary validates the underlying rich credentials, evaluates the action against the Task's constraints, and issues a presentation token (per the Privacy-Preserving Profile) that the platform can validate.  
4. The platform receives only the presentation token. It does not see the underlying delegation evidence.  
5. The platform's PEP validates the presentation token (using the intermediary's published keys, federated through the regulator's trust framework). Permit.  
6. Account setup proceeds. The platform records its part of the audit trail (presentation received, decision made) but does not retain anything that would allow it to learn the underlying delegation context.

**What the model handles well:** The Privacy-Preserving Profile slots cleanly into the execution layer. The intermediary plays both the Verification Intermediary role and the Credential Manager role, simplifying the architectural picture. The platform receives a credential (the presentation token) that is functionally similar to other Credential Manager outputs but bound to the intermediary's trust framework.

**What this scenario stresses:** When the deployment uses the Privacy-Preserving Profile, the AuthZen PDP at the platform's side does not see the underlying delegation evidence. Its policy can evaluate the presentation token but not the rich constraints. This means **the AuthZen PDP's role is reduced** when the privacy-preserving profile is in effect — most of the authorization decision happens inside the intermediary, and the platform PDP simply validates that an authorized intermediary issued a presentation. We should make this explicit in the Privacy-Preserving Profile companion document, because it affects how implementers think about PDP design.

---

## 9\. Open Questions

The execution layer is genuinely earlier-stage than Components 1–4. The following questions need resolution before this draft hardens.

### 9.1 Cross-Trust-Domain Credential Issuance

Scenario 4 (§8.4) exposes a fundamental open question: when a workload chain crosses trust domains, where does credential issuance happen and who is authorized to chain credential requests on whose behalf? The three options sketched (direct-to-CM, gateway-as-secondary-CM, pre-authorized-session) each have trade-offs and none is clearly correct. This needs concrete pressure-testing with practitioners building real cross-domain agentic systems.

### 9.2 Credential Manager Federation

For the Credential Manager pattern to work, resource servers must accept credentials issued by Credential Managers they federate with. Today's consumer ecosystem does not have such federation. A hotel loyalty program does not currently know how to validate a token issued by "Bob's personal Credential Manager." Solving this requires either (a) explicit federation agreements, (b) reusing existing identity federation patterns where the Credential Manager registers as an identity provider, or (c) a new trust framework specifically for delegator-domain Credential Managers. This is a genuinely hard ecosystem problem, separable from the technical specification.

### 9.3 Coordination Between AuthZen PDP and Credential Manager

A delegation may be permitted by the AuthZen PDP but denied by the Credential Manager (or vice versa). For example, the PDP may say "this delegation authorizes the workload to book hotels" while the Credential Manager says "but the delegator's policy does not authorize issuing loyalty-API credentials at the requested scope." We have not specified the resolution semantics. Likely: deny-overrides-permit (any party in the chain can deny), but the user-experience implications need thought (which party explains the failure to the user?).

### 9.4 Transaction Token Representation of Further-Delegation Chains

Scenario 2 (§8.2) exposes that when the orchestrator dispatches sub-tasks to specialist agents, the Transaction Token must somehow carry the further\_delegation chain (per base spec §5.5) to enable downstream verification. The Transaction Tokens IETF draft does not currently specify how delegation chains are represented. We need either an extension to that draft or a specific encoding for our use case. This is a concrete piece of standards-track work.

### 9.5 Delegators Without Credential Managers

Some delegators do not personally hold the underlying credentials being delegated against — institutional delegators (a local authority, a corporate signing authority) who delegate without holding personal credentials, low-tech consumers who don't have Credential Managers, or delegators who hold credentials in formats that don't fit the Credential Manager model. The execution layer should describe how delegations work in these cases. The likely answer is "the delegator's organization provides the Credential Manager role on their behalf" but this needs explicit treatment.

### 9.6 PDP-PDP Federation

When a request crosses trust domains, multiple PDPs may need to make portions of the decision. Domain A's PDP authorizes the egress; Domain B's PDP authorizes the ingress; both must agree for the request to proceed. We have not specified PDP-PDP federation patterns. AuthZen does not currently address this; we may need to extend it or describe a federation pattern that uses AuthZen as a building block.

### 9.7 Audit Reconciliation Across Domains

When an end-to-end flow touches Domain A's PDP, Bob's Credential Manager, Domain B's gateway, and Domain B's resource server, each retains its own audit fragment. Reconstructing the full trail for a dispute or oversight inquiry requires correlating identifiers across domains. The Privacy-Preserving Profile specifies that the verification intermediary holds the canonical trail, but in non-privacy-preserving deployments there is no single party that can reconstruct the full picture. The audit story needs explicit treatment.

### 9.8 Wire Protocol Specification for the Credential Manager

This document deliberately defers the wire protocol for the Credential Manager pattern. Once the deployment model has been pressure-tested, the wire protocol should be specified — likely as a profile of OID4VCI, possibly with extensions for delegation-evidence-as-gating. This is concrete future work but should not happen before the architectural questions above are resolved.

### 9.9 Performance and Online Dependencies

The Credential Manager pattern introduces an online dependency at the moment of credential issuance. For high-frequency delegated actions (e.g., a continuously-running monitoring agent) this may be unacceptable. Patterns for caching, pre-issuance, or session credentials need treatment, balanced against the privacy implications (longer-lived credentials are more linkable).

### 9.10 Composability with Existing OAuth Deployments

Many existing OAuth deployments will not change quickly. The execution layer model needs to describe how delegations can be exercised against pre-existing OAuth-only resource servers without forcing them to become delegation-aware. The Credential Manager's ability to mint OAuth tokens that look conventional to the resource server is the key adaptation pattern; this needs concrete examples.

---

## 10\. References

### 10.1 Companion Documents

- *Delegated Authorization Reference Architecture v1.1* — the four-component model this layer executes  
- *Privacy-Preserving Profile v1.0* — the layered profile that constrains execution-layer behavior for vulnerable populations

### 10.2 Normative References

- **OpenID AuthZen (Authorization API 1.0)** — [https://openid.net/specs/authorization-api-1\_0.html](https://openid.net/specs/authorization-api-1_0.html)  
- **OAuth 2.0 Transaction Tokens** *(IETF draft)* — [https://datatracker.ietf.org/doc/draft-ietf-oauth-transaction-tokens/](https://datatracker.ietf.org/doc/draft-ietf-oauth-transaction-tokens/)  
- **RFC 8693** — OAuth 2.0 Token Exchange  
- **RFC 9396** — OAuth 2.0 Rich Authorization Requests  
- **OpenID for Verifiable Credential Issuance (OID4VCI)** — OpenID Foundation

### 10.3 Informative References

- **W3C Verifiable Credentials Data Model 2.0**  
- **SPIFFE / SVID** — workload identity standard  
- **OpenID Federation 1.0** — for trust framework patterns  
- **Open Policy Agent (OPA)** and **Cedar** — policy languages for AuthZen PDPs

---

*End of working draft v0.1. This document is intended as a basis for pressure-testing the execution layer model. Substantial revision is expected as the open questions in §9 are resolved.*  
