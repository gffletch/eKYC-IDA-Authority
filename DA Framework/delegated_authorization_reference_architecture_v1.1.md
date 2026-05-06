# Delegated Authorization Reference Architecture

**Working draft v1.1 — Components 1 through 4**

*A framework for expressing delegated authorization across human, organizational, and AI workload contexts, building on IETF and OpenID Foundation specifications.*

**Changelog (v1.0 → v1.1):**

- Added §4.8: Graduated authority over time as a Component 1 deployment pattern  
- Added §4.9: Role-tuple group sub-type for consensus decisions  
- Added §7.7: Trigger-based expiry (`expires_on_event`) as a first-class temporal constraint type  
- Expanded §8.3 and §8.4: Concrete revocation cascade, transfer-with-continuity, and reason-hiding requirements  
- Updated §10: Open questions list refined based on stress test analysis  
- Companion document introduced: *Privacy-Preserving Profile* layers the six privacy R-requirements on top of this base specification

---

## Table of Contents

1. [Introduction](#1-introduction)  
2. [Design Principles](#2-design-principles)  
3. [Component Overview](#3-component-overview)  
4. [Component 1: Relationship](#4-component-1-relationship)  
5. [Component 2: Authority](#5-component-2-authority)  
6. [Component 3: Obligations](#6-component-3-obligations)  
7. [Component 4: Constraints](#7-component-4-constraints)  
8. [Cryptographic Binding, Revocation, and Transfer](#8-cryptographic-binding-revocation-and-transfer)  
9. [End-to-End Scenarios](#9-end-to-end-scenarios)  
10. [Open Questions and Future Work](#10-open-questions-and-future-work)  
11. [References](#11-references)

---

## 1\. Introduction

### 1.1 Motivation

Existing authorization frameworks were designed for a model where a human end-user directly authorizes an application to act on their behalf within a single trust domain. Modern authorization scenarios are richer in three dimensions that strain that model:

- **Authority**: Acts may be authorized not by the immediate user but by a chain of legal, institutional, or relational instruments (powers of attorney, corporate signing authorities, healthcare credentialing, parental rights).  
- **Delegation depth**: Tasks are increasingly performed by autonomous workloads (AI agents, orchestrators) that further delegate sub-tasks to other workloads, creating multi-step delegation chains.  
- **Scope precision**: Real-world delegations require fine-grained bounding (time, place, amount, brand, recipient, conditional logic) that exceed what OAuth scopes were designed to express.

This reference architecture decomposes delegated authorization into six coherent components, each with a defined JSON / JWT representation, that together address all three dimensions while remaining interoperable with existing IETF and OpenID Foundation specifications.

### 1.2 Scope of This Document

This document specifies **Components 1 through 4** — the components that describe the delegation itself:

| \# | Component | Establishes | Lifecycle |
| :---- | :---- | :---- | :---- |
| 1 | Relationship | Who can delegate to whom, under what eligibility | Establishment-time, long-lived |
| 2 | Authority | Why the delegator has the right; the broadest scope of the delegation | Establishment-time, long-lived |
| 3 | Obligations | What specific task is to be performed | Run-time, task-specific |
| 4 | Constraints | The bounds within which the task must be performed | Run-time, task-specific |

Components 5 (Delegated Capabilities) and 6 (Delegated Access) — the execution layer — are out of scope for this revision and will be addressed in a subsequent draft.

### 1.3 Relationship to Existing Specifications

This work builds upon and extends:

- **RFC 7519** — JSON Web Token (JWT). All component objects are profiled as signed JWTs.  
- **RFC 8693** — OAuth 2.0 Token Exchange. The expected mechanism for translating these objects into operational access tokens.  
- **RFC 9396** — OAuth 2.0 Rich Authorization Requests (RAR). The Obligations component reuses the RAR `type` registry directly.  
- **OpenID Connect Core / eKYC IDA** — Verified claims envelope used by Authority for high-assurance profiles.  
- **OpenID Authority Claims Extension** *(working draft)* — The basis for Component 2's source authority structure, with extensions for entity-to-entity and credentialing/affiliation cases.

This document does not duplicate definitions from those specifications. Where this work makes additions or modifications, it does so explicitly and provides rationale.

A companion document, the **Privacy-Preserving Profile**, layers six binding privacy requirements (cardinality hiding, legal-basis abstraction, role anonymization, tier-label privacy, audit-trail custody, and unlinkable purpose-scoped credentials) on top of this base specification. The Privacy-Preserving Profile is REQUIRED when the delegatee population may include vulnerable individuals (children, adults under guardianship, asylum seekers, persons in coercive control situations) and is RECOMMENDED for any deployment where the delegation type or relationship structure is itself sensitive.

---

## 2\. Design Principles

The framework is shaped by seven principles, each justified by a real-world delegation scenario.

### 2.1 Layered scoping from coarse to fine

Authority establishes the **outermost boundary** of what could ever be done under this delegation ("travel domain"). Obligations define the **specific task** ("book a hotel in Boston for May 12–15"). Constraints carve out the **bounds of acceptable performance** ("under $400/night, walking distance to the convention center"). Each layer narrows the prior; the result is that the delegatee operates within an intersection of all four.

This is not merely an organizational convenience. It allows:

- A single Authority to underwrite many Tasks without re-establishing the relationship each time.  
- Constraints to be layered in by parties other than the delegator (platform, regulatory) without modifying the original mandate.  
- An auditor to reason about the delegation at any granularity, from "what could this agent ever do" down to "what specifically did it do at 14:23 on Tuesday."

### 2.2 Self-contained verifiability

Each component is a signed JWT that can be verified independently using only its issuer's public key and the integrity hashes of its upstream bindings. No central database lookup is required for verification, though revocation endpoints are referenced for liveness checks.

### 2.3 Polymorphic identity

Delegators and delegatees may be natural persons, legal entities, organizational groups, or workloads. The framework supports OIDC subject identifiers, DIDs, SPIFFE SVIDs, X.509 distinguished names, and organizational URIs through a `fmt` discriminator field. Identity format is signaled explicitly rather than inferred from string syntax.

### 2.4 Polymorphic groups

Delegators and delegatees may be groups, not just individuals. Three group models are supported:

- **Static**: a bounded list of identities, inline or by reference  
- **Role-based**: defined by an organizational role ("on-call nurse")  
- **Policy-defined**: membership determined at runtime by policy evaluation ("any clinician credentialed for Schedule II narcotics")

### 2.5 Profile-based assurance

Different deployment contexts have different assurance requirements. Healthcare and legal contexts demand verified-claims envelopes, trust-framework attestation, and notarized authority instruments. Agentic AI contexts need lightweight, high-velocity delegation. The framework supports both through deployment profiles that determine which fields are required versus optional.

### 2.6 Two-tier policy expression

For machine-evaluable conditions (eligibility, constraints), the framework offers a typed predicate vocabulary in the base specification (Tier 1\) and a `policy_ref` extension point for richer policy languages like Cedar, Rego, or XACML (Tier 2). Implementations support Tier 1 by default; Tier 2 is opt-in for deployments needing full policy-engine expressivity.

### 2.7 Provenance for layered authorship

Constraints in particular are often layered in by parties other than the delegator. Each constraint may carry a `source` field identifying its author (delegator, authority, relationship, platform, regulatory), enabling auditors to trace responsibility for any element of the constraint surface.

---

## 3\. Component Overview

The four components form two logical pairs:

**Establishment pair (long-lived, infrequent change):**

- *Relationship* — defines the parties and their eligibility to participate in the delegation  
- *Authority* — defines why the delegator may delegate and the broadest scope thereof

**Runtime pair (task-specific, ephemeral):**

- *Obligations* — defines the essential task  
- *Constraints* — defines the bounds within which the task must be performed

Components 3 and 4 are co-located in a single signed JWT (`task+jwt`). Components 1 and 2 are separate JWTs, each independently issuable.

┌─────────────────────────────────────────────────────────────────────┐  
│                        ESTABLISHMENT LAYER                          │  
│                                                                     │  
│  ┌────────────────────────┐         ┌───────────────────────────┐   │  
│  │   Relationship JWT     │ ◄────── │     Authority JWT         │   │  
│  │   typ: relationship+jwt│         │     typ: authority+jwt    │   │  
│  │                        │ rel\_jti │                           │   │  
│  │  delegator             │ rel\_hash│  source\_authority         │   │

│  │  delegatee             │         │  derivative\_authority     │   │  
│  │  eligibility           │         │                           │   │  
│  └────────────────────────┘         └───────────────────────────┘   │  
│             ▲                                  ▲                    │  
│             │ rel\_jti / rel\_hash               │ auth\_jti/auth\_hash │  
└─────────────┼──────────────────────────────────┼────────────────────┘  
              │                                  │  
┌─────────────┼──────────────────────────────────┼────────────────────┐  
│             │           RUNTIME LAYER          │                    │  
│             │                                  │                    │  
│         ┌───┴──────────────────────────────────┴────┐               │  
│         │              Task JWT                     │               │  
│         │            typ: task+jwt                  │               │  
│         │                                           │               │  
│         │  obligations  (composition\_logic \+ items) │               │  
│         │  constraints  (tier1 \+ tier2)             │               │  
│         └───────────────────────────────────────────┘               │  
└─────────────────────────────────────────────────────────────────────┘

A single Relationship \+ Authority pair typically underwrites many Task JWTs over its lifetime.

---

## 4\. Component 1: Relationship

### 4.1 Purpose

The Relationship JWT defines the two (or more) parties to a delegation and the eligibility criteria that determine who may act in each role. It is the long-lived foundation upon which run-time tasks are issued.

### 4.2 Structure

A Relationship JWT contains five logical sections:

1. **JOSE Header** — `alg`, `kid`, and `typ: "relationship+jwt"`  
2. **Registered claims** — `iss`, `iat`, `nbf`, `exp`, `jti`  
3. **Relationship metadata** — `rel_type`, `rel_version`, `rel_status`, `rev_endpoint`  
4. **Party identity** — polymorphic `delegator` and `delegatee` objects, each optionally carrying a group descriptor  
5. **Eligibility criteria** — two-tier: typed predicates (`tier1`) and/or policy reference (`tier2`)

### 4.3 Relationship Types

The base relationship-type registry includes:

| Type URN | Meaning |
| :---- | :---- |
| `urn:ietf:params:delegation:principal-agent` | General delegation from a principal to an agent |
| `urn:ietf:params:delegation:employer-employee` | Authority arising from employment |
| `urn:ietf:params:delegation:guardian-dependent` | Parental, custodial, or carer relationships |
| `urn:ietf:params:delegation:legal-representative` | Power of attorney, executor, court-appointed representative |
| `urn:ietf:params:delegation:service-client` | Workload acting for a human or organization |
| `urn:ietf:params:delegation:orchestrator-subagent` | AI agent delegating to sub-agents |
| `urn:ietf:params:delegation:peer-delegation` | Lateral delegation between peers |

The registry is extensible; deployments may register custom types under their own namespace.

### 4.4 Identity Object

Every delegator and delegatee identity is represented as an object with a `fmt` discriminator and format-specific fields:

{

  "fmt": "oidc",

  "iss": "https://idp.example",

  "sub": "user-12345"

}

Supported formats include `oidc`, `did`, `spiffe`, `x509`, and `org-uri`. Implementations MUST reject identity objects with unknown `fmt` values rather than attempt heuristic interpretation.

### 4.5 Group Descriptor

When a party is a group rather than a single identity, a `group` sub-object is present:

{

  "group\_type": "policy",

  "policy\_uri": "https://policy.example/narcotic-delegation/v2",

  "policy\_hash": { "alg": "sha-256", "digest": "..." },

  "policy\_lang": "cedar",

  "evaluation\_endpoint": "https://authz.example/evaluate",

  "min\_members": 1

}

The `group_type` field takes one of `static`, `role`, or `policy`. The `min_members` field expresses quorum requirements (e.g., two-nurse co-authorization for high-risk medications).

### 4.6 Eligibility Criteria

Tier 1 (typed predicates) supports the following base vocabulary, all of which may be combined via `combination` predicates with `and`/`or` operators:

- **temporal** — date ranges, time-of-day windows, day-of-week, timezone  
- **geographic** — jurisdictions, regions, proximity to a point  
- **credential** — required credential type, issuer, minimum assurance level  
- **role** — required organizational role, with org\_id and org\_uri  
- **quorum** — minimum number of members required (links to group descriptor)

Tier 2 (`policy_ref`) carries `policy_uri`, `policy_hash`, `policy_lang`, `policy_version`, and optional `evaluation_endpoint`. The two tiers may be present simultaneously; verifiers MUST satisfy both to honor the relationship.

### 4.7 Example: Doctor-Nurse Narcotic Delegation

{

  "iss": "https://credentialing.cityhospital.example/delegation",

  "iat": 1714000000,

  "nbf": 1714000000,

  "exp": 1745536000,

  "jti": "urn:uuid:a3f2c1d0-8e4b-4f7a-9c2d-1b0e3f5a7c9d",

  "rel\_type": "urn:ietf:params:delegation:principal-agent",

  "rel\_version": "1.0",

  "rel\_status": "active",

  "rev\_endpoint": "https://credentialing.cityhospital.example/delegation/revoke",

  "delegator": {

    "fmt": "oidc",

    "iss": "https://idp.cityhospital.example",

    "sub": "dr-jane-smith-7291"

  },

  "delegatee": {

    "fmt": "oidc",

    "iss": "https://idp.cityhospital.example",

    "group": {

      "group\_type": "policy",

      "policy\_uri": "https://policy.cityhospital.example/narcotic-delegation/v2",

      "policy\_hash": {

        "alg": "sha-256",

        "digest": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"

      },

      "policy\_lang": "cedar",

      "evaluation\_endpoint": "https://authz.cityhospital.example/evaluate",

      "min\_members": 1

    }

  },

  "eligibility": {

    "tier1": {

      "credential": {

        "type": "DEA-COR",

        "issuer": "https://www.deadiversion.usdoj.gov",

        "schedules": \["II", "III"\],

        "min\_assurance": "LOA3"

      },

      "role": {

        "role\_id": "on-call-nurse",

        "org\_id": "cityhospital",

        "org\_uri": "https://idp.cityhospital.example/org/nursing"

      },

      "combination": {

        "op": "and",

        "conditions": \["credential", "role"\]

      }

    }

  }

}

This Relationship asserts that Dr. Jane Smith (delegator) may delegate to any individual who is simultaneously (a) holding an active DEA registration for Schedule II and III substances and (b) holding the `on-call-nurse` role at City Hospital. The Cedar policy at `policy_uri` is the authoritative definition of group membership and is dereferenced and evaluated at run-time.

### 4.8 Graduated Authority Over Time

Some delegations are not single-event grants but **progressive transfers** in which authority shifts gradually between parties as circumstances change. The canonical example is a child's evolving capacity: a guardian holds full digital authority for a young child, shares decision-making with an older child, and ultimately retains only safeguarding authority for an adolescent before fully ceding control at the age of majority. The same pattern arises in capacity-graded medical decision-making, in escalating organizational responsibility, and in time-boxed corporate succession.

The framework supports two implementation patterns for graduated authority. Deployments may choose either based on the operational characteristics of the issuing intermediary; the base specification does not normatively prefer one.

**Pattern A — Sequential Relationships.** A series of Relationship JWTs are issued, each scoped to a defined life stage or capacity band, with non-overlapping `nbf`/`exp` windows. The issuing intermediary manages the sequence, ensuring that exactly one Relationship is active at any given time. Transitions between stages are revocation-and-issuance events. This pattern is operationally simpler to verify (each verifier sees one Relationship at a time) but requires the intermediary to actively manage the schedule.

**Pattern B — Single Relationship with Stage-Keyed Eligibility.** A single Relationship JWT carries a `stage_schedule` field within `eligibility.tier1`, defining the authority bands and their boundary conditions. Verifiers evaluate the schedule against the runtime context (e.g., the child's current age) to determine which band is active. This pattern is more compact but bakes a state machine into the credential.

When using Pattern B, the schedule structure is:

"stage\_schedule": {

  "schedule\_type": "age\_banded",

  "subject\_age\_attribute": "delegatee\_age",

  "stages": \[

    {

      "stage\_id": "early",

      "min\_age": 0,

      "max\_age": 7,

      "scope\_override": "https://schemas.example.com/scope/full-guardianship"

    },

    {

      "stage\_id": "consultative",

      "min\_age": 8,

      "max\_age": 12,

      "scope\_override": "https://schemas.example.com/scope/consultative-guardianship"

    },

    {

      "stage\_id": "shared",

      "min\_age": 13,

      "max\_age": 15,

      "scope\_override": "https://schemas.example.com/scope/shared-decision"

    },

    {

      "stage\_id": "safeguarding-only",

      "min\_age": 16,

      "max\_age": 17,

      "scope\_override": "https://schemas.example.com/scope/safeguarding-only"

    }

  \],

  "terminal\_event": {

    "event\_type": "age\_of\_majority\_reached",

    "behaviour": "automatic\_revocation"

  }

}

The `scope_override` field interacts with the bound Authority JWT: when the active stage carries a scope\_override, that scope (rather than the Authority's `derivative_authority.permission.scope_domain`) is the operative scope at the moment of evaluation. This allows a single Authority instrument to underwrite a graduated chain of operative scopes without requiring re-issuance of the Authority itself.

The `terminal_event` field defines the boundary condition that ends the Relationship. Terminal events are typically irrevocable (an adult cannot have their majority undone) and trigger automatic, non-cascading revocation per §8.3.

### 4.9 Role-Tuple Group Sub-Type

The base group descriptor (§4.5) supports `static`, `role`, and `policy` group types. For consensus-driven decisions in which authority requires the simultaneous participation of *several distinct roles*, a fourth sub-type is defined: **role-tuple**.

A role-tuple group requires that, at the moment of evaluation, at least one credentialed individual is presented for *each* of the named roles. This is structurally distinct from the `min_members` quorum field, which requires N members from a single role pool. Role-tuple groups model situations like a child welfare strategic decision requiring the joint participation of a social worker, a psychologist, a residential care manager, and an Independent Reviewing Officer.

"group": {

  "group\_type": "role\_tuple",

  "required\_roles": \[

    {

      "role\_id": "social\_worker",

      "org\_id": "lambeth-childrens-services",

      "min\_assurance": "LOA3"

    },

    {

      "role\_id": "psychologist",

      "credential\_type": "HCPC-registered",

      "min\_assurance": "LOA3"

    },

    {

      "role\_id": "care\_home\_manager",

      "credential\_type": "ofsted-registered",

      "min\_assurance": "LOA3"

    },

    {

      "role\_id": "independent\_reviewing\_officer",

      "credential\_type": "iro-appointed",

      "min\_assurance": "LOA3"

    }

  \],

  "consensus\_mode": "all\_required"

}

The `consensus_mode` field accepts:

- `all_required` — every named role must be filled (default)  
- `n_of_m` — at least N of the M named roles must be filled, with N specified by an additional `min_roles_filled` field  
- `weighted` — each role carries a weight, and the sum of present roles' weights must meet a `min_weight` threshold; useful where a senior role can substitute for two junior roles

Role-tuple groups compose with other group sub-types: a delegatee may be defined as a static group whose members are themselves role-tuples, supporting nested consensus structures. Verifiers MUST be able to resolve at least the simple `all_required` form to comply with the base specification; `n_of_m` and `weighted` modes are optional features that profiles may require.

---

## 5\. Component 2: Authority

### 5.1 Purpose

The Authority JWT carries two related but distinct authority assertions, either or both of which may be present:

- **Source authority** — proof that the delegator had the right to delegate at all (e.g., a power of attorney, a corporate director appointment, a hospital credentialing instrument).  
- **Derivative authority** — the rights as they flow to the delegatee, expressed at the *coarsest meaningful scope* (e.g., "travel domain," "narcotic administration domain"). Specific tasks and their bounds are expressed in Components 3 and 4\.

### 5.2 Relationship to OpenID Authority Claims Extension

Component 2 builds directly on the OpenID Connect Authority claims extension (working draft) for the source authority structure. The `applies_to`, `permission`, and `granted_by` sub-elements are reused with the following modifications:

- Extension of `applies_to` to support entity-to-entity (workload) and affiliation cases not explicitly in scope of the OpenID draft.  
- Reduction of `permission` to a minimal set on the *derivative* authority (scope domain \+ further-delegation rules), since validity, budget, audience, and function are properly expressed as Constraints in this framework.  
- Addition of an `inherent` value for `granted_by.method` to express self-authority of natural persons over their own affairs.  
- Addition of a `prior_authority_ref` field on derivative authority to support summary-level chain-of-authority semantics.

### 5.3 Structure

An Authority JWT contains:

1. **JOSE header** with `typ: "authority+jwt"`  
2. **Registered claims**  
3. **Relationship binding** — `rel_jti`, `rel_hash`  
4. **Optional verified-claims envelope** (eKYC IDA) — required for high-assurance profiles  
5. **Optional `source_authority`** — full OpenID Authority structure  
6. **Optional `derivative_authority`** — coarse scope domain \+ further-delegation rules \+ optional prior-authority reference

At least one of `source_authority` or `derivative_authority` MUST be present.

### 5.4 Source Authority

The source authority section follows the OpenID Authority structure:

- `applies_to` — the entity the authority covers (organization, natural person, or workload)  
- `permission` — `role`, `may_delegate` (boolean), and optional `validity` and `function`  
- `granted_by` — `method` (`delegated`, `appointed`, `self-asserted`, `inherent`), `granting_body`, `reason`

### 5.5 Derivative Authority

The derivative authority section is intentionally minimal:

- `permission.scope_domain` — a free-form URI defining the coarsest meaningful scope (e.g., `https://schemas.example.com/scope/travel`)  
- `permission.further_delegation` — an object with `allowed`, `max_depth`, `sub_scopes`, and `constraints_inheritance` (`strict` | `additive`)  
- `prior_authority_ref` (optional) — `authority_uri`, `authority_hash`, `link_method` for summary-level chain references

### 5.6 Profiles

Two reference profiles are defined:

**High-assurance profile** (legal, healthcare, financial)

- `verified_claims` envelope REQUIRED  
- `source_authority` REQUIRED  
- `granted_by.method` SHOULD NOT be `inherent`  
- `granted_by.granting_body` REQUIRED

**Agentic profile** (AI workload delegation, consumer self-delegation)

- `verified_claims` envelope OPTIONAL  
- `source_authority` OPTIONAL (omission implies inherent self-authority)  
- `derivative_authority` REQUIRED  
- All `granted_by.method` values permitted

### 5.7 Example A: Agentic — Bob → AI Travel Agent

{

  "iss": "https://wallet.bob.example",

  "iat": 1714000000,

  "exp": 1745536000,

  "jti": "urn:uuid:b7e2f1a4-3c8d-4e2a-9f1b-2c5d8e0f3a91",

  "rel\_jti": "urn:uuid:9d4f8b2c-1e3a-4b5c-8d7e-6f0a1b2c3d4e",

  "rel\_hash": {

    "alg": "sha-256",

    "digest": "f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2"

  },

  "derivative\_authority": {

    "permission": {

      "scope\_domain": "https://schemas.example.com/scope/travel",

      "further\_delegation": {

        "allowed": true,

        "max\_depth": 1,

        "sub\_scopes": \[

          "https://schemas.example.com/scope/travel/booking",

          "https://schemas.example.com/scope/travel/payment"

        \],

        "constraints\_inheritance": "strict"

      }

    }

  }

}

Source authority is omitted; under the agentic profile this signals self-authority of a natural person over their own affairs.

### 5.8 Example B: High-assurance — Hospital → Dr. Smith

{

  "iss": "https://credentialing.cityhospital.example",

  "iat": 1714000000,

  "exp": 1745536000,

  "jti": "urn:uuid:c8d3f9e7-2b4a-5c6d-7e8f-9a0b1c2d3e4f",

  "rel\_jti": "urn:uuid:a3f2c1d0-8e4b-4f7a-9c2d-1b0e3f5a7c9d",

  "rel\_hash": {

    "alg": "sha-256",

    "digest": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2"

  },

  "verified\_claims": {

    "verification": {

      "trust\_framework": "us-healthcare-credentialing-framework",

      "time": "2026-01-15T10:30:00Z",

      "verification\_process": "urn:uuid:7f8a9b0c-1d2e-3f4a-5b6c-7d8e9f0a1b2c"

    },

    "claims": {

      "given\_name": "Jane",

      "family\_name": "Smith",

      "birthdate": "1978-03-22",

      "source\_authority": {

        "applies\_to": {

          "organization\_name": "City Hospital",

          "registration\_number": "CH-2001-NY",

          "legal\_jurisdiction": "US-NY",

          "organization\_identifiers": \[{

            "identifier\_type": "NPI",

            "identifier": "1234567890",

            "issuer": "CMS"

          }\]

        },

        "permission": {

          "role": "Attending Physician",

          "may\_delegate": true,

          "validity": { "start": "2024-07-01", "end": "2027-06-30" },

          "function": "narcotic-prescription-and-administration"

        },

        "granted\_by": {

          "method": "appointed",

          "granting\_body": "City Hospital Medical Staff Credentialing Committee",

          "reason": "Active medical staff appointment with Schedule II prescribing privileges"

        }

      }

    }

  },

  "derivative\_authority": {

    "permission": {

      "scope\_domain": "https://cityhospital.example/scope/narcotic-administration",

      "further\_delegation": { "allowed": false }

    }

  }

}

Both source and derivative authority are present. Source authority is wrapped in the `verified_claims` envelope and asserts Dr. Smith's credentialed status. Derivative authority asserts that the delegation covers narcotic administration but cannot be further delegated by the delegatee.

---

## 6\. Component 3: Obligations

### 6.1 Purpose

The Obligations section of the Task JWT defines the *essential* elements of the task — the verb and its essential complements without which the task is incomplete. The cleavage rule:

An element belongs in Obligations if removing it would render the task undefined. An element belongs in Constraints if removing it would merely make the task less restrictive.

Thus: a hotel booking's destination and dates are Obligations. Its budget, brand allowlist, and proximity to a landmark are Constraints.

### 6.2 Structure

The `obligations` object contains:

- `composition_logic` — `and` (all items must be performed) or `or` (any one item satisfies the task)  
- `purpose` (optional) — human-readable intent for audit and consent contexts  
- `items` — array of obligation entries

Each obligation entry contains:

- `obligation_id` — unique within the Task JWT, used as the target of constraints  
- `type` — RFC 9396 RAR type identifier (shared registry with OAuth RAR)  
- Type-specific fields per the RAR type definition

### 6.3 RAR Type Reuse

This framework deliberately reuses the RFC 9396 RAR type registry. RAR-aware authorization servers can interpret an obligation's type-specific fields without learning a new vocabulary. Where this framework requires a type not yet in the RAR registry, the expectation is to register it through the standard IANA process rather than create a parallel registry.

### 6.4 Sequencing

For composite tasks like "find hotel, then book it," sequencing is implicit in `and` composition; the agent or orchestrator handles execution order based on data dependencies. Explicit `pre_conditions` are not part of the base specification but may be added by extension if deployment experience shows a need.

### 6.5 Example: Hotel Booking

"obligations": {

  "composition\_logic": "and",

  "purpose": "Bob's annual conference trip to Boston",

  "items": \[

    {

      "obligation\_id": "obl-1",

      "type": "hotel\_booking",

      "locations": \[

        { "city": "Boston", "country": "US", "region": "MA" }

      \],

      "check\_in": "2026-05-12",

      "check\_out": "2026-05-15",

      "guests": { "adults": 1, "children": 0 },

      "rooms": 1

    }

  \]

}

Each field on the obligation is essential: the locations, dates, guest count, and room count cannot be omitted without making the task undefined.

---

## 7\. Component 4: Constraints

### 7.1 Purpose

The Constraints section of the Task JWT bounds the acceptable performance of the obligations. Constraints are typically more numerous and more diverse in authorship than Obligations.

### 7.2 Structure

The `constraints` object contains:

- `tier1` — array of typed predicate constraints  
- `tier2` (optional) — array of policy-language references for constraints not expressible in the Tier 1 vocabulary

Each Tier 1 constraint contains:

- `category` — one of the registered categories (see 7.3)  
- Category-specific fields  
- `applies_to_obligations` (optional) — array of `obligation_id` values; absent means applies to all  
- `source` (optional) — `delegator`, `authority`, `relationship`, `platform`, or `regulatory`; recorded when the constraint is not authored by the delegator  
- `source_ref` (optional) — URI identifying the underlying policy or rule the constraint derives from

### 7.3 Constraint Categories

The base Tier 1 vocabulary:

| Category | Purpose | Key fields |
| :---- | :---- | :---- |
| `temporal` | Time bounds on task validity | `valid_from`, `valid_until`, `time_windows` |
| `monetary` | Financial limits | `max_per_transaction`, `max_total`, `currency` |
| `geographic` | Location bounds | `allowed_regions`, `jurisdictions`, `proximity` |
| `count` | Quantity limits | `max_n`, `unit` (e.g., bookings, items, calls) |
| `resource` | Allowed/disallowed specific resources | `rule` (allow/deny), `resources` array |
| `data_minimization` | Permitted data fields | `permitted_fields`, `data_subject` |
| `recipient` | Permitted data recipients | `allowed_recipients`, `categories` |
| `conditional` | Run-time predicates | `only_if`, `except_when` with predicate definitions |

### 7.4 Tier 2 Policy Reference

For complex constraint logic exceeding the typed vocabulary, a Tier 2 policy reference may be used:

{

  "tier2": \[

    {

      "policy\_uri": "https://policy.cityhospital.example/medication-rules/v3",

      "policy\_hash": { "alg": "sha-256", "digest": "..." },

      "policy\_lang": "cedar",

      "policy\_version": "3.2.1",

      "evaluation\_endpoint": "https://authz.cityhospital.example/evaluate",

      "applies\_to\_obligations": \["obl-1"\],

      "source": "platform"

    }

  \]

}

### 7.5 Constraint Provenance

The `source` field is the primary auditability mechanism for layered authorization. When constraints come from multiple authors (delegator, platform, regulatory), the source field allows verifiers and auditors to:

- Trace the responsible party for any element of the constraint surface  
- Determine whether the platform or regulatory constraints have been tampered with by the delegator  
- Diagnose constraint failures by identifying which authority's rule was violated

### 7.6 Example: Layered Constraints

"constraints": {

  "tier1": \[

    {

      "category": "monetary",

      "max\_per\_night": { "amount": "400.00", "currency": "USD" },

      "max\_total": { "amount": "1200.00", "currency": "USD" },

      "source": "delegator"

    },

    {

      "category": "geographic",

      "proximity": {

        "anchor": { "lat": 42.3601, "lon": \-71.0589, "label": "Hynes Convention Center" },

        "max\_distance\_km": 1.5,

        "mode": "walking"

      },

      "source": "delegator"

    },

    {

      "category": "resource",

      "rule": "deny",

      "resources": \[

        { "type": "vendor", "value": "ofac\_sanctioned" }

      \],

      "source": "regulatory",

      "source\_ref": "https://sanctions.treasury.gov/ofac"

    },

    {

      "category": "data\_minimization",

      "permitted\_fields": \["full\_name", "email", "loyalty\_number"\],

      "data\_subject": "delegator",

      "source": "platform",

      "source\_ref": "https://platform.example/policy/booking-pii-minimization"

    }

  \]

}

### 7.7 Trigger-Based Expiry

The base temporal constraint vocabulary defined in §7.3 supports calendar-based bounds (`valid_from`, `valid_until`, `time_windows`). Many real-world delegations require expiry tied not to a calendar date but to the **occurrence of an external event**: a court order termination, the next statutory review, the child reaching the age of majority, the completion of a probate proceeding, the closure of an investigation. These cannot be reliably expressed as calendar dates because the date is unknown at issuance time.

Trigger-based expiry is defined as a first-class temporal constraint sub-category, `expires_on_event`, with the following structure:

{

  "category": "temporal",

  "expires\_on\_event": {

    "event\_type": "court\_order\_terminated",

    "event\_source": {

      "source\_type": "court\_registry",

      "source\_uri": "https://courts.example/orders/2026-FAM-04582",

      "evaluation\_endpoint": "https://courts.example/orders/2026-FAM-04582/status"

    },

    "behaviour\_on\_event": "immediate\_revocation",

    "polling\_interval\_max": "PT1H"

  }

}

The fields are:

- `event_type` — a registered event identifier from a base vocabulary that includes `court_order_terminated`, `court_order_amended`, `statutory_review_completed`, `placement_changed`, `age_of_majority_reached`, `capacity_assessment_changed`, `employment_terminated`, `proceeding_closed`. Profiles may register additional types.  
- `event_source` — describes how to determine whether the event has occurred. The `source_type` field discriminates among `court_registry`, `governmental_registry`, `organizational_registry`, `webhook_subscription`, and `manual_attestation`. The `evaluation_endpoint` is dereferenceable to obtain current event status.  
- `behaviour_on_event` — one of `immediate_revocation` (the constraint and bound JWTs become invalid the moment the event occurs), `grace_period_then_revocation` (with a `grace_period` duration), or `transfer_required` (the credential remains valid only if a successor credential is presented; see §8.4).  
- `polling_interval_max` — the maximum acceptable staleness of event evaluation, expressed as ISO 8601 duration. Verifiers MUST re-check the event source at intervals no greater than this.

Trigger-based expiry interacts with calendar expiry: if both are present on a constraint, the credential becomes invalid at the *earlier* of the two boundaries. A credential with `valid_until: 2027-01-01` and `expires_on_event: court_order_terminated` is invalid as soon as the court order ends or the calendar date passes, whichever comes first.

Trigger-based expiry MAY be applied to any layer of the binding chain: a Relationship may carry a trigger-based expiry, in which case revocation of the Relationship cascades to bound Authority and Task JWTs per §8.3. Constraints on a Task JWT may carry trigger-based expiry independently of the Relationship's lifecycle.

Privacy note: the `event_type` and `event_source` fields are inherently revealing — knowing that a credential expires when "placement\_changed" is itself a strong signal of care-status. Deployments operating under privacy-preserving requirements MUST use the layering rules defined in the *Privacy-Preserving Profile* companion document, which specifies how trigger-based expiry is wrapped in opaque tokens at the relying-party boundary.

---

## 8\. Cryptographic Binding, Revocation, and Transfer

### 8.1 Binding Chain

The four components form an integrity chain through hash references:

Relationship JWT  (rel\_jti, sha256 \= rel\_hash)

        ▲

        │ Authority JWT references via rel\_jti \+ rel\_hash

        │

Authority JWT  (auth\_jti, sha256 \= auth\_hash)

        ▲

        │ Task JWT references via rel\_jti \+ rel\_hash AND auth\_jti \+ auth\_hash

        │

Task JWT

A verifier presented with a Task JWT MUST:

1. Validate the Task JWT signature using the issuer's published key.  
2. Retrieve or accept (as cached) the referenced Relationship JWT and verify `rel_hash` matches its sha-256 digest.  
3. Retrieve or accept (as cached) the referenced Authority JWT and verify `auth_hash` matches its sha-256 digest.  
4. Verify that the Authority JWT's `rel_jti` and `rel_hash` likewise reference the same Relationship JWT.  
5. Validate the signatures on both Relationship and Authority JWTs.  
6. Check revocation status for any JWT that carries a `rev_endpoint`.  
7. Evaluate eligibility criteria (Relationship), authority scope (Authority), and constraints (Task) against the runtime context.

If any step fails, the delegation MUST NOT be honored.

### 8.2 Issuer Identity and Profile-Dependent Signing

The signing model for each JWT is profile-dependent and intentionally left out of scope for the base specification. Deployment profiles MAY require:

- Single-issuer signing (delegator signs all four components)  
- Multi-issuer signing (Relationship signed by an identity federation, Authority by a credentialing body, Task by the delegator)  
- Notary or co-signature requirements for high-assurance Authority instruments

### 8.3 Revocation: Mechanisms, Cascade, and Enforcement Window

Each Relationship and Authority JWT MAY carry a `rev_endpoint` claim referencing a revocation status service. Verifiers consult this endpoint as part of binding-chain validation (§8.1, step 6). The base specification defines three concrete behaviors that any conformant deployment MUST implement.

**Revocation mechanisms.** A revocation endpoint accepts the JWT's `jti` and returns one of the statuses `active`, `suspended`, `revoked`, or `superseded`. The endpoint MUST be authenticated and SHOULD support short-lived signed status tokens (analogous to OAuth 2.0 token introspection responses) to enable verifier caching with bounded staleness.

**Revocation cascade.** Revocation of a Relationship cascades to all bound Authority and Task JWTs. Specifically:

- A Task JWT whose `rel_jti` references a revoked Relationship MUST be treated as invalid, regardless of the Task JWT's own status.  
- An Authority JWT whose `rel_jti` references a revoked Relationship MUST be treated as invalid.  
- Revocation of an Authority JWT cascades to all Task JWTs bound to it via `auth_jti`.  
- Revocation does not cascade upward: revoking a Task JWT does not revoke the Relationship or Authority.

This cascade rule is binding on verifiers, not optional. Verifiers MUST evaluate the full binding chain at every validation; caching of revocation status is permitted only within the validity window of the cached status response.

**Enforcement window.** When a guardianship, employment, capacity, or other underlying authority status changes, the revocation system MUST propagate the change such that downstream verifiers stop honoring the affected credentials within **24 hours** of the change. Profiles for high-stakes domains (healthcare narcotic administration, financial signing authority, child welfare) MAY mandate shorter enforcement windows, down to **5 minutes** for life-safety-critical scenarios.

The 24-hour window is a maximum, not a target. Issuers and intermediaries SHOULD aim for near-real-time propagation when the underlying authoritative source supports it. Where the authoritative source is itself slow (e.g., a court registry that updates weekly), the enforcement window is bounded by the source's propagation lag plus the revocation system's own cycle time.

**Revocation reason hiding.** The reason for revocation — placement change, court order modification, capacity assessment, employment termination, age of majority — is not transmitted to relying parties. The revocation status response carries only the status enumeration (`revoked`, `superseded`, etc.) and an optional opaque `revocation_id` for audit correlation. Reasons are recorded only in the audit trail held by the issuing intermediary, accessible to authorized auditors and oversight bodies but not to the platform receiving the credential.

This is a base-specification requirement, not a privacy-preserving-profile feature. Reason-hiding is the default for all profiles, because the reason for revocation is almost always more sensitive than the fact of revocation.

### 8.4 Transfer with Continuity

When authority transfers between parties — a placement change, a guardian succession, a corporate signing authority handover — the digital representation MUST update without creating a protection gap during the transition. Two transfer patterns are defined.

**Pattern 1 — Successor reference (preferred).** The outgoing Relationship JWT, at the moment of revocation, transitions to status `superseded` and the revocation response carries an additional field `successor_rel_jti` referencing the incoming Relationship JWT. Verifiers receiving a `superseded` status with a successor reference MUST attempt to retrieve and validate the successor Relationship; if validation succeeds, the verifier treats the *successor* as the operative Relationship for the binding chain. The outgoing Relationship is not honored after supersession.

This pattern requires that the successor Relationship be issued and active *before* the outgoing one is superseded. The issuing intermediary is responsible for ensuring this ordering; if the successor is not yet active, supersession MUST be deferred and the outgoing Relationship remains in `active` status.

**Pattern 2 — Overlapping validity windows.** The outgoing Relationship JWT carries a `transition_window` claim defining a period during which both the outgoing and incoming Relationships are simultaneously valid. The outgoing Relationship's `exp` is the end of the transition window. During the window, verifiers may honor either credential. After the window, only the incoming Relationship is valid. This pattern is operationally simpler but creates an interval during which authorization decisions could be made under either authority — appropriate for transfers between cooperating parties (e.g., outgoing and incoming care home managers during a planned handover) but not for adversarial transfers (e.g., emergency removal of authority from a compromised actor).

**Hard transfer (no continuity).** Some transfers are intentionally discontinuous: emergency removal of authority due to safeguarding concerns, court orders barring contact, capacity loss requiring immediate substitution. In these cases, the outgoing Relationship is revoked with status `revoked` (not `superseded`) and no successor reference is provided. A separate Relationship may be issued to a new authorized party but does not inherit the binding chain of the revoked Relationship; bound Authority and Task JWTs are invalidated and must be re-issued under the new Relationship.

**Reason hiding extends to transfers.** A relying party observing a `superseded` status learns only that the credential was replaced; it does not learn whether the cause was scheduled handover, emergency reassignment, or any other underlying reason.

### 8.5 Hash Object Format

All hash references use the format:

{

  "alg": "sha-256",

  "digest": "hex-encoded-digest-string"

}

The digest is computed over the entire compact JWS serialization of the referenced JWT (header.payload.signature).

---

## 9\. End-to-End Scenarios

### 9.1 Scenario A: AI Travel Agent

**Setup**: Bob, a natural person, wants to delegate hotel booking for an upcoming conference to an AI travel agent.

**Component instances**:

1. **Relationship JWT** — Issued by Bob's wallet, type `service-client`, delegator is Bob (OIDC subject), delegatee is the travel agent (DID), eligibility constrained to Bob's authorized agent fleet by policy reference.  
     
2. **Authority JWT** — Issued by Bob's wallet under the agentic profile. No source authority (Bob has inherent self-authority). Derivative authority's `scope_domain` is `https://schemas.example.com/scope/travel`, `further_delegation` allows one-level sub-delegation to booking and payment specialists.  
     
3. **Task JWT** — Issued when Bob initiates the trip planning. Single obligation of type `hotel_booking` with Boston, May 12–15, one adult, one room. Constraints: $400/night max, $1,200 total max, walking distance to convention center, allowed brands (Marriott, Hilton, Hyatt), validity expires May 10\. All constraints sourced from the delegator.

**Flow**:

1. The travel agent presents the Task JWT to a hotel booking API.  
2. The API verifies the Task JWT's signature, retrieves the Relationship and Authority JWTs (cached or via referenced URIs), validates the binding chain.  
3. The API confirms the travel agent's identity is permitted under the Relationship's eligibility.  
4. The API confirms the hotel-booking domain is within the derivative authority's scope.  
5. The API evaluates the obligation and constraints against candidate hotels and presents acceptable options or a confirmed booking.

**Validation of the model**: All four components are exercised. Each plays a distinct role. The constraint surface is small but layered (and could trivially absorb a regulatory constraint, e.g., OFAC, without modifying the delegator's intent).

### 9.2 Scenario B: Hospital Narcotic Administration

**Setup**: Dr. Jane Smith is on duty and needs an on-call nurse to administer pain medication to a post-operative patient.

**Component instances**:

1. **Relationship JWT** — Issued by the hospital's credentialing authority. Delegator is Dr. Smith. Delegatee is a *policy-defined group* of nurses who hold (a) DEA Schedule II registration and (b) the on-call-nurse role. Group membership is evaluated at run-time via Cedar policy.  
     
2. **Authority JWT** — High-assurance profile. Verified-claims envelope with hospital trust framework. Source authority asserts Dr. Smith's status as Attending Physician with Schedule II prescribing privileges, granted by appointment from the Medical Staff Credentialing Committee. Derivative authority restricts scope to narcotic administration with no further delegation permitted.  
     
3. **Task JWT** — Issued by the hospital's order entry system at the time the order is written. Single obligation of type `medication_administration` with patient MRN, drug code, dose, route, frequency. Constraints layered from three sources:  
     
   - **Delegator**: temporal validity (24 hours), max 6 administrations, only-if pain score ≥ 4  
   - **Platform**: deny concurrent benzodiazepine (FDA black-box warning)  
   - **Regulatory**: HIPAA minimum-necessary data fields permitted

**Flow**:

1. The on-call nurse, when ready to administer, presents the Task JWT to the medication dispensing system.  
2. The system validates the binding chain.  
3. The system evaluates the Relationship's eligibility policy: is this nurse currently on-call AND credentialed for Schedule II? Cedar evaluation says yes.  
4. The system confirms the Authority's narcotic-administration scope covers the requested action.  
5. The system evaluates the layered constraints. The current pain score is 6 (≥ 4, satisfies delegator's conditional). No concurrent benzodiazepines on the patient's record (satisfies platform). Administration is logged with only the permitted data fields (satisfies regulatory).  
6. The dispensing system releases the medication and records the administration. The Task JWT's `count` constraint is decremented.

**Validation of the model**: This scenario exercises high-assurance features (verified claims, full source authority, policy-defined groups, layered constraints with provenance) and shows how multiple authorities contribute to a single delegation without conflict. The constraint provenance is essential here — if the dispensing fails, knowing whether it was the delegator's pain-score rule, the platform's drug-interaction rule, or the regulatory data rule that failed is critical for clinical workflow.

### 9.3 Scenario C: Corporate Director Delegation

**Setup**: A company director (Bob Smith) delegates expense approval up to £5,000 to the head of finance for the duration of the director's two-week vacation.

**Component instances**:

1. **Relationship JWT** — Issued by the company's identity provider. Delegator is Bob Smith (OIDC). Delegatee is the head-of-finance role (role-based group, currently held by a single named individual). Eligibility includes credential constraint (must hold valid corporate identity at LOA2+).  
     
2. **Authority JWT** — High-assurance profile. Verified-claims envelope with the corporate trust framework. Source authority is the OpenID Authority example structure: applies\_to is the company (with LEI, registration number, jurisdiction), permission is `Director` with `may_delegate: true`, granted\_by is `appointed` by Companies House. Derivative authority's scope domain is `https://example-co.example/scope/expense-approval`, no further delegation.  
     
3. **Task JWT** — Issued by Bob at the start of vacation. Single obligation of type `expense_approval`. Constraints: temporal (specific two-week window), monetary (max £5,000 per approval), conditional (only-if expense category is in {travel, vendor, equipment}, except-when total exceeds £15,000 for the period — in which case approval reverts to Bob via remote process).

**Validation of the model**: This shows the OpenID Authority pattern fully expressed, with real GLEIF/Companies House identifiers, and demonstrates how the existing OpenID work integrates seamlessly into our broader delegation framework.

---

## 10\. Open Questions and Future Work

### 10.1 Components 5 and 6 (Execution Layer)

The following are deferred to a subsequent draft:

- **Delegated Capabilities** — How the abstract delegation translates into operational IAM permissions for the delegatee workload. Likely uses RFC 8693 token exchange to obtain capability tokens scoped by the Authority's `scope_domain` and bounded by the Task's Obligations and Constraints.  
- **Delegated Access** — Credentials needed to access protected personal data (e.g., loyalty accounts, medical records, financial accounts) on behalf of the delegator. Potentially modeled as wrapped OAuth access tokens or capability-based credentials.

The Permissions Dashboard concept identified during stress-testing (a regulated-intermediary-held audit and authorization-status dashboard accessible to delegators, oversight bodies, and the data subject in age-appropriate form) is a strong candidate use case for the Component 5/6 design.

### 10.2 Signing Models and Trust Frameworks

The base specification leaves signing models out of scope. Standardization or recommendation of common signing profiles (single-issuer, multi-issuer, notarized) would aid interoperability. The Privacy-Preserving Profile companion document defines an intermediary-as-issuer pattern that addresses this for vulnerable-population deployments.

### 10.3 Constraint Conflict Resolution

When constraints from multiple sources conflict (e.g., delegator allows a vendor that the platform denies), the base specification adopts **deny-overrides-allow** as the default resolution rule, with the following precedence ordering when conflicts arise: judicial \> regulatory \> platform \> authority \> relationship \> delegator. A judicial constraint always wins over a delegator constraint regardless of category. Profiles MAY define more nuanced override semantics; the base rule is conservative and predictable.

### 10.4 Chain-of-Authority Reconstruction

The current model supports summary-level chain references via `prior_authority_ref` but does not specify how to walk a multi-step chain end-to-end. For deep delegation chains (court → executor → executor's law firm → individual lawyer), an explicit chain-walking protocol may be needed. The intermediary pattern introduced in the Privacy-Preserving Profile suggests that chain-walking is the *intermediary's* responsibility, with the relying party seeing only the flattened presentation; this needs to be formalized.

### 10.5 Cross-Profile Interaction

When a delegation chain crosses profile boundaries (e.g., a high-assurance medical authority delegates to an agentic AI workflow), the rules for downgrading or upgrading assurance are not yet specified. The general principle is that downstream assurance cannot exceed upstream assurance, but the precise semantics — particularly when a privacy-preserving-profile credential is presented in a context that does not require privacy preservation — need explicit treatment.

### 10.6 Multi-Decision-Tier Quorum

Section 4.9 introduces role-tuple groups for consensus decisions. Extending this to support **multi-decision-tier quorum** (where routine decisions require one role, significant decisions require three, and strategic decisions require all named roles) is left to a future revision. The current Tier 2 policy\_ref escape hatch can express this for deployments that need it.

### 10.7 Disputes, Appeals, and Override Mechanisms

Real-world delegations occasionally need to be disputed or appealed by the delegatee or by an affected third party (e.g., the data subject in a guardianship case). The base specification does not yet define a dispute protocol. This is closely related to the Component 5/6 work, since disputes typically operate at the execution layer where capabilities are exercised.

### 10.8 Cross-Border Recognition

A delegation established under one jurisdiction's legal framework may need to be honored in another jurisdiction. The framework supports this in principle (the Relationship and Authority can be presented across jurisdictions), but the legal recognition rules are out of scope. Trust framework operators are expected to handle cross-border recognition through bilateral or multilateral agreements.

---

## 11\. References

### 11.1 Normative References

- **RFC 7519** — Jones, M., Bradley, J., and N. Sakimura, "JSON Web Token (JWT)", May 2015\.  
- **RFC 8693** — Jones, M., Nadalin, A., Campbell, B., Bradley, J., and C. Mortimore, "OAuth 2.0 Token Exchange", January 2020\.  
- **RFC 9396** — Lodderstedt, T., Richer, J., and B. Campbell, "OAuth 2.0 Rich Authorization Requests", May 2023\.  
- **OpenID Connect Core 1.0** — Sakimura, N., Bradley, J., Jones, M., de Medeiros, B., and C. Mortimore, November 2014\.

### 11.2 Informative References

- **OpenID Connect Authority Claims Extension** *(working draft)* — Haine, M. and J. White, eKYC & IDA Working Group, 2026\. [https://openid.bitbucket.io/ekyc/openid-authority.html](https://openid.bitbucket.io/ekyc/openid-authority.html)  
- **draft-mcguinness-oauth-actor-profile** — McGuinness, A., "OAuth 2.0 Actor Profile", IETF Internet-Draft.  
- **draft-aap-oauth-profile-01** — "AI Agent Profile for OAuth 2.0", IETF Internet-Draft.  
- **draft-mora-oauth-entity-profiles-01** — Mora, F., "OAuth 2.0 Entity Profiles", IETF Internet-Draft.  
- **draft-hardt-aauth-protocol-01** — Hardt, D., "Agent Authorization Protocol", IETF Internet-Draft.  
- **ISO 17442-1:2020** — Financial services — Legal entity identifier (LEI).  
- **ISO 5009** — Financial services — Official organizational roles.  
- **GLEIF Registration Authorities List** — Global Legal Entity Identifier Foundation.  
- **Cedar Policy Language** — [https://www.cedarpolicy.com/](https://www.cedarpolicy.com/)  
- **Open Policy Agent (Rego)** — [https://www.openpolicyagent.org/](https://www.openpolicyagent.org/)  
- **SPIFFE / SVID** — [https://spiffe.io/](https://spiffe.io/)

---

*End of working draft.*  
