# Delegated Authorization Reference Architecture

**Working draft v2.5 — Relationship, Authority, Objective, Obligations, and Constraints**

*A framework for expressing delegated authorization across human, organizational, and AI workload contexts, building on IETF and OpenID Foundation specifications.*

*This file replaces the prior `reference_architecture_v1.1.md`. Going forward, the version is carried in this document header and in the per-revision changelog below; the filename is stable across revisions.*

**Changelog (v2.4 → v2.5):**
- **Structural reorganization — the Purpose Object definition moved from §6 (Obligations) to §9 (Objective); no normative or wire change.** The structured `purpose` object's canonical shape definition and base purpose-class vocabulary were added at v1.3 as **§6.6 "Purpose Classes"** while purpose was a field of Obligations. When v2.0 relocated `purpose` onto the Objective, that definition physically stayed in Section 6, with Section 9 pointing back to it — leaving the Objective's own purpose field defined in a different component's section. This revision moves the definition into a rewritten **§9.4 "The Purpose Object"** (consolidated with the former §9.4 relocation note), making Section 9 the single self-contained home for the Objective and everything it carries. The `kind`/`params`/`display` shape, the five base purpose classes, and the inert posture (§9.2) are **unchanged** — only the location changed.
- **Renumbering:** former **§6.6 "Purpose Classes" removed**; former **§6.7 "Terminal Conditions on Obligations" renumbered to §6.6**. All body cross-references swept — pointers to the terminal-conditions section (§6.1, §6.2, §8.3, §9.4, §9.6) now read §6.6; pointers to the purpose shape (§6.1, §6.2, §9.1, §11.9) now read §9.4. Historical changelog entries retain the section numbers valid at their revision.
- **Version:** no component added or removed, no JWT wire shape or normative rule changed — a document-structure change with no behavioral effect. Stamped a **minor** bump for cross-reference traceability, since section numbers are the cross-document reference interface (CLAUDE.md §4.6).

**Changelog (v2.3 → v2.4):**
- **New §5.5.1 — Scope Matching Semantics (normative; addresses review finding H2).** v2.3 and earlier left `permission.scope_domain` a free-form URI while every verification flow said to confirm an act was "within scope," with no defined rule for what "within" meant — so the outermost authority boundary was effectively unenforceable as specified. §5.5.1 resolves this by **demoting `scope_domain` to an advisory categorization/audit label** that a verifier MUST NOT match lexically (no prefix, substring, or path-containment), and **moving the enforceable "within authority" test onto a typed vocabulary with defined comparison** — the RFC 9396 RAR type registry (§6.3): an Obligation is within the derivative authority when its RAR `type` is admitted by the authority under **exact registered-identifier equality** (value-space, not lexical). `further_delegation.sub_scopes` is the enumerated delegable subset (same equality); for the delegatee's own exercise the enforceable bound remains Obligations + Constraints (validated at issuance, re-checked at the commit boundary), and any per-type gate ahead of a formed Obligation is a profile concern. §8.1 step 7 and the §10.1/§10.2 scenario "within scope" steps were updated to the type-admission rule. The `scope_override` interaction (§4.8, Pattern B) is flagged as unreconciled and left to the separate M1 decision (§12.2). **Value-space discipline** follows the Mission-Bound Authorization suite's Common Constraint subset/intersection approach (§13). No wire-structure change (a normative clarification of matching semantics; `scope_domain` and `sub_scopes` are unchanged in shape) → **minor** bump.
- **Source:** Karl McGuinness's review of Reference Architecture v2.0 (`external-analysis/karl_ref_arch_v2_feedback.txt`), finding **H2** (`scope_domain` has no matching semantics). Direction was recorded as settled (typed-vocabulary/value-space with `scope_domain` advisory) in the v2.3 return note; this revision makes the edit.

**Changelog (v2.2 → v2.3):**
- **New §12 — Security Considerations (consolidated threat model).** Prior revisions scattered trust assumptions through the prose. §12 consolidates them into a single section, structured as: the trusted base (each role — the three issuers, the regulated intermediary, the verifier/PDP, the commit-time status sources, the transparency/receipts substrate — with what it must achieve, what it assumes, and how its compromise degrades the guarantees); role-scoped multi-issuer trust anchors; commit-time source integrity; cross-cutting assumptions; what the model does and does not guarantee; and an adversary-move / addressed-by / residual coverage table. Informational-style consolidation; it pulls together the already-normative pieces (the §9.2 inert-Objective injection rationale, §8.4 reason-hiding, the §8.1-step-8 and §8.7 availability/bearer rulings) and adds two normative rules (below). The structure follows the trusted-base-plus-adversary-coverage form used by the Mission-Bound Authorization suite's Security Model (§13).
- **Role-scoped multi-issuer trust anchors (normative; addresses review finding H3).** §8.2 and §12.2 now state that under multi-issuer signing a verifier MUST establish a trust anchor for *each* issuer in the chain, and that trust is **role-scoped** — an issuer trusted to sign Relationships is not thereby trusted to sign Authorities or Task JWTs. The *discovery mechanism* for anchors is deferred to profiles. This frames (but does not fix) the cross-issuer scope-escalation concern tracked as an open question (`scope_override`, PROJECT_MEMORY §10.2.17).
- **Commit-time source integrity and fail-closed availability (normative; addresses review finding M2).** All commit-time external status sources MUST be authenticated and integrity-protected — closing the gap where §8.4 required this of `rev_endpoint` but §4.5 (`group.evaluation_endpoint`) and §7.7 (`event_source.evaluation_endpoint`) did not. §12.3 states the unifying availability rule: an **authority-conferring or authority-removing** source that cannot be reached **fails closed** (the act is denied), whereas an **authority-irrelevant (inert)** artifact fails *safe* (recorded, not denied — the §8.1-step-8 standalone-Objective ruling). §4.5 and §7.7 are updated to require authentication and to define fail-closed behavior on an unreachable endpoint, with §7.7's existing `grace_period_then_revocation` as the per-trigger softening.
- **Renumbering:** former §12 (References) → §13. Added a §12 entry to the Table of Contents; the three body cross-references to the References section were updated (§12 → §13). §11.2 (Signing Models) notes that §12.2 now states the base multi-issuer trust-anchor requirement it previously only gestured at.
- **Source:** Karl McGuinness's two-pass review of Reference Architecture v2.0 (`external-analysis/karl_ref_arch_v2_feedback.txt`), findings **H3** (no consolidated threat model; multi-issuer trust establishment) and **M2** (inconsistent commit-time endpoint authentication). Additive new section plus a normative hardening of previously-unauthenticated endpoints; no component added or removed and no JWT wire-structure change → minor bump. Finding **M1** (`scope_override` layering inversion) is named as a threat in §12.2 but deliberately not fixed here; it remains a tracked open question pending a structural decision.

**Changelog (v2.1 → v2.2):**
- **New §8.7 — Presentation Binding (Holder Binding and Audience).** The binding chain (§8.1) proves issuance and integrity but not *possession*: nothing binds a component to a holder key (`cnf`) or a target audience (`aud`), so a component reaching a relying party is, as this base spec defines it, a **bearer artifact**. §8.7 now states this explicitly and normatively: presentation binding is an **execution-layer concern** (Permissioned Capabilities / Protected Access, §11.1), and until a profile or the execution layer provides it, the delegation components **MUST NOT be treated as bearer-safe** — a deployment MUST either derive a holder-bound, audience-restricted execution-layer credential before honoring an act, or keep capture-and-replay out of its threat model by construction. The mechanism is deliberately *not* mandated on the Task JWT here (same rationale as the profile-dependent signing model, §8.2): committing a `cnf`/`aud` shape on the durable delegation artifact would prematurely couple it to a presentation model that belongs on the derived, short-lived credential the delegatee presents. This is a binding-*model* statement, not a new mechanism; it addresses the "no holder binding / PoP / `aud`" gap (the highest-severity finding in K. McGuinness's review of v2.0) at the level the review asked for. §10.1 and §10.2 gain a one-line note that their "presents the Task JWT" step is the delegation-layer view; §11.1 adds presentation binding to the execution-layer work items. Additive and normative but touches no JWT structure and no existing mechanism, hence a minor bump.

**Changelog (v2.0 → v2.1):**
- **§8.1 step 8 — standalone-Objective integrity now distinguishes a *tampered* record from an *unreachable* one (the only normative change in this revision).** Prior text made any failure of step 8 invalidate the delegation; because checking `objective_hash` requires *retrieving* the standalone `objective+jwt`, an unreachable Objective failed step 8 and denied an act the enforceable components already permitted — turning intent-record availability into a denial condition for an authority-irrelevant artifact (and, in an injected-attacker model, into a denial lever). Step 8 now splits the two: a *tampered* Objective (retrieved, but hash/signature/upstream-reference mismatch) is an **integrity failure** and still invalidates the delegation; an *unreachable* standalone Objective is an **audit-availability failure** that MUST be surfaced and recorded but MUST NOT deny a properly-authorized act. This imports the integrity-failure vs. audit-failure distinction the Mission-Bound Authorization suite uses for its audit/transparency and completion profiles, and applies the suite's "commit the descriptor, keep evaluated/availability state out of the decision" discipline to an inert artifact (mirror case: because the Objective grants no authority, an availability failure fails *safe*, not closed). Surfaced by K. McGuinness's review of v2.0.
- *Editorial honesty fixes from the same review:* §8.3 now states that of the obligation-level commit-boundary signals it lists, only `terminal_when` has a specified mechanism in this version and the others are illustrative of the principle; §8.4 frames the 24h/5-min enforcement window as a target conditioned on both source-propagation lag *and* status-response cache lifetime, not a guarantee; §9.1 no longer implies the inert `action` *bounds* generated Obligations (the bound is the enforceable Authority/Constraints layer) and notes that `beneficiary` is descriptive only and MUST NOT scope whose records may be accessed; §11.9 upgrades the beneficiary sub-question from a detail to a load-bearing descriptive-vs-scoping decision; §12 corrects the McGuinness author initial (A. → K.) and adds the Mission completion companion as an informative reference.

**Changelog (v1.4 → v2.0):**
- Added the **Objective** (provisional name) as a new delegation component (§9): an *inert declared-intent layer* carrying the umbrella `action`, the `purpose`, and an optional `beneficiary`. This is a structural change (a new component), hence a major version bump per the versioning convention. Naming note: "Objective" is provisional (renamed from "Charge" 2026-06-27; candidate alternatives: Commission, Charter); deliberately *not* "Mission" (collision with the Mission-Bound Authorization for OAuth suite) or "Intent" (collision with agentic generated intent).
- §9.2 establishes the defining normative property: the Objective is **inert with respect to authority** — a verifier MUST NOT use any Objective field to derive, widen, or gate authority. Enforceable authority remains solely with Relationship, Authority, Obligations, and Constraints. Rationale is prompt-injection resistance (an authority-bearing intent field is an attack surface; an inert one is not). Mirrors the inert posture of the Mission-Bound Authorization suite's `goal`/`purpose`/`success_criteria`.
- Relocated the structured `purpose` object from Obligations (§6.6) to the Objective (§9.4), with no change to its `kind`/`params`/`display` wire shape. **Retired** the v1.3 governance-shaping vs. audit-only param distinction and the rule that "ignoring governance-shaping params is equivalent to dropping a constraint," as a direct consequence of the inert property. Purpose is now audit-and-consent material; resource-side policy MAY consult it out of band (e.g., retention regimes) but it does not gate this delegation's authority. This reverses the v1.3 purpose-as-governance-shaping direction.
- Added the domain-separated **intent anchor** (`objective_hash`, distinct from `auth_hash`), following the dual-anchor (`intent_hash`/`authority_hash`) pattern of the Mission-Bound Authorization suite. Extended the §8.1 binding-chain procedure with an integrity-only Objective step (verified for integrity, never evaluated for scope) and updated the §8.1 chain diagram.
- Updated §1.1 (component count is now seven total: five delegation + two execution), §1.2 (component table gains an Objective row; "four delegation components" → "five"), §3 (Component Overview gains the Objective as a third logical grouping; overview diagram updated).
- Renumbered former §9 (End-to-End Scenarios) → §10, §10 (Open Questions and Future Work) → §11, §11 (References) → §12 to make room for §9 (Objective). Added §11.9 recording the Objective's remaining open sub-questions (final name, beneficiary modeling, `action` class vocabulary, inline-vs-standalone guidance, execution-layer interaction).
- *Editorial (2026-06-27):* the provisional component name was changed from **Charge** to **Objective** (still provisional; no structural change). All prose references, the `objective_*` / `objective+jwt` claim and type names, and the §9 naming note were updated accordingly.

**Changelog (v1.3 → v1.4):**
- Extended the Obligations/Constraints cleavage rule (§6.1) with a third category: **completion definition** — elements that specify when an obligation is satisfied or discharged belong in Obligations, not in Constraints. `terminal_when` is the first use of this category.
- Added `terminal_when` to §6.2 obligation item fields.
- Added §6.7: Terminal Conditions on Obligations. Defines `terminal_when` as an optional sub-field on obligation items, reusing the §7.7 event vocabulary with `behaviour_on_event` omitted (behavior is fixed: obligation discharged). Distinguishes obligation discharge (task complete) from revocation (task invalid) in both evaluation semantics and audit interpretation. Explains composition-logic interaction (`and` vs. `or`). Includes the Maya/Theo vaccination-record-disclosure example as the canonical illustration. Cross-referenced from §6.1, §6.2, and §8.3.
- Updated §8.3 (Status signals beyond revocation) to name `terminal_when` as a fourth instance of the commit-boundary status-signal pattern, alongside revocation, trigger-based expiry, and constraint tightening by external authority.

**Changelog (v1.2 → v1.3):**
- Changed §6.2: `purpose` field upgraded from optional string to optional structured object with three sub-fields: `kind` (URI identifying the purpose class; required when `purpose` is present), `params` (optional governance-shaping parameters), and `display` (optional human-readable description; replaces the former string value). The string form is deprecated; conformant implementations SHOULD use the structured form. Deployments that issued Task JWTs with a `purpose` string under v1.1/v1.2 SHOULD treat that string as equivalent to `{ "display": "<string>" }` with no `kind` or `params`.
- Added §6.6: Purpose Classes. Defines the structured purpose object, purpose class URIs, the governance-shaping vs. audit-only params distinction, and the purpose catalog concept. Notes the relationship to §10.2.9 (runtime semantic test) and §8.3 (commit-boundary evaluation). Includes a base purpose class vocabulary and two examples.
- Updated §6.5 (hotel booking example) and §9.1 (AI Travel Agent scenario) to use the structured purpose object.
- Updated §6.1 with a cross-reference to §6.6.

**Changelog (v1.1 → v1.2):**
- Added §3.1: The Task JWT — Obligations and Constraints Bound Together. Articulates the four facets of the binding (cryptographic, structural, scope, commit-boundary) so reviewers and implementers do not have to infer the binding from `applies_to_obligations` and the JWT envelope. Surfaces the property that the Task JWT is the smallest unit at which an act is authorized at runtime, and that the bundle — not either component in isolation — is what the verifier honors or denies.
- Tightened §7.2: `applies_to_obligations` is now normatively scoped to the same Task JWT; cross-Task references are forbidden and verifiers MUST reject Task JWTs whose constraints reference undefined obligation_ids.
- Added cross-references from §6.1 and §7.1 pointing to §3.1 for the binding semantics.
- Added §8.3: Commit-Boundary Validity as the architectural principle generalizing the v1.1 Relationship-rooted revocation cascade. Each component independently carries (or references) a status signal that the verifier MUST re-check at the commit boundary, not only at issuance. Revocation is the canonical instance; tightening constraints from external sources and silently-lapsing authorities are the cases that motivate the generalization. The implementation lives in Permissioned Capabilities; the property is established here.
- Renumbered §8.3 (Revocation) → §8.4; §8.4 (Transfer with Continuity) → §8.5; §8.5 (Hash Object Format) → §8.6. §8.4 lightly reframed to position revocation as one instance of the §8.3 pattern.
- Updated §8.1 binding-chain procedure to mark step 6 (status-signal check) as a commit-time check, not solely an issuance-time check, and to apply symmetrically to Relationship, Authority, and Task JWTs (Task JWTs MAY now carry their own `rev_endpoint`).
- Updated cross-references in §1.2, §4.8, and §7.7 to the new §8 numbering.

**Changelog (v1.0 → v1.1):**
- Added §4.8: Graduated authority over time as a Relationship deployment pattern
- Added §4.9: Role-tuple group sub-type for consensus decisions
- Added §7.7: Trigger-based expiry (`expires_on_event`) as a first-class temporal constraint type
- Expanded §8.3 (now §8.4) and §8.4 (now §8.5): Concrete revocation cascade, transfer-with-continuity, and reason-hiding requirements
- Updated §10: Open questions list refined based on stress test analysis
- Companion document introduced: *Privacy-Preserving Profile* layers the six privacy R-requirements on top of this base specification
- Editorial pass: De-numbered component references (use names not "Component N") and propagated execution-layer rename (Delegated Capabilities → Permissioned Capabilities, Delegated Access → Protected Access)

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Design Principles](#2-design-principles)
3. [Component Overview](#3-component-overview)
4. [Relationship](#4-relationship)
5. [Authority](#5-authority)
6. [Obligations](#6-obligations)
7. [Constraints](#7-constraints)
8. [Cryptographic Binding and Commit-Boundary Validity](#8-cryptographic-binding-and-commit-boundary-validity)
9. [Objective (Declared Intent Layer)](#9-objective-declared-intent-layer)
10. [End-to-End Scenarios](#10-end-to-end-scenarios)
11. [Open Questions and Future Work](#11-open-questions-and-future-work)
12. [Security Considerations](#12-security-considerations)
13. [References](#13-references)

---

## 1. Introduction

### 1.1 Motivation

Existing authorization frameworks were designed for a model where a human end-user directly authorizes an application to act on their behalf within a single trust domain. Modern authorization scenarios are richer in three dimensions that strain that model:

- **Authority**: Acts may be authorized not by the immediate user but by a chain of legal, institutional, or relational instruments (powers of attorney, corporate signing authorities, healthcare credentialing, parental rights).
- **Delegation depth**: Tasks are increasingly performed by autonomous workloads (AI agents, orchestrators) that further delegate sub-tasks to other workloads, creating multi-step delegation chains.
- **Scope precision**: Real-world delegations require fine-grained bounding (time, place, amount, brand, recipient, conditional logic) that exceed what OAuth scopes were designed to express.

This reference architecture decomposes delegated authorization into seven coherent components, each with a defined JSON / JWT representation, that together address all three dimensions while remaining interoperable with existing IETF and OpenID Foundation specifications. Five of these are **delegation components** that describe the delegation itself (Relationship, Authority, Objective, Obligations, Constraints); two are **execution-layer components** (Permissioned Capabilities, Protected Access) specified in a companion document.

### 1.2 Scope of This Document

This document specifies the five **delegation components** — those that describe the delegation itself:

| Component | Establishes | Lifecycle |
|-----------|-------------|-----------|
| Relationship | Who can delegate to whom, under what eligibility | Establishment-time, long-lived |
| Authority | Why the delegator has the right; the broadest scope of the delegation | Establishment-time, long-lived |
| Objective *(inert)* | What act the delegation is for, and why (declared intent; never gates authority) | Delegation-time, declared once |
| Obligations | What specific task is to be performed | Run-time, task-specific |
| Constraints | The bounds within which the task must be performed | Run-time, task-specific |

The **Objective** (§9) is a new component as of v2.0. It is *inert with respect to authority*: it records the declared umbrella act, purpose, and optional beneficiary for consent and audit, but a verifier MUST NOT use it to derive, widen, or gate authority (§9.2). It is provisionally named; see the §9 naming note.

The execution-layer components — **Permissioned Capabilities** and **Protected Access** — are out of scope for this revision and will be addressed in a subsequent draft.

### 1.3 Relationship to Existing Specifications

This work builds upon and extends:

- **RFC 7519** — JSON Web Token (JWT). All component objects are profiled as signed JWTs.
- **RFC 8693** — OAuth 2.0 Token Exchange. The expected mechanism for translating these objects into operational access tokens.
- **RFC 9396** — OAuth 2.0 Rich Authorization Requests (RAR). The Obligations component reuses the RAR `type` registry directly.
- **OpenID Connect Core / eKYC IDA** — Verified claims envelope used by Authority for high-assurance profiles.
- **OpenID Authority Claims Extension** *(working draft)* — The basis for Authority's source authority structure, with extensions for entity-to-entity and credentialing/affiliation cases.

This document does not duplicate definitions from those specifications. Where this work makes additions or modifications, it does so explicitly and provides rationale.

A companion document, the **Privacy-Preserving Profile**, layers six binding privacy requirements (cardinality hiding, legal-basis abstraction, role anonymization, tier-label privacy, audit-trail custody, and unlinkable purpose-scoped credentials) on top of this base specification. The Privacy-Preserving Profile is REQUIRED when the delegatee population may include vulnerable individuals (children, adults under guardianship, asylum seekers, persons in coercive control situations) and is RECOMMENDED for any deployment where the delegation type or relationship structure is itself sensitive.

---

## 2. Design Principles

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

For machine-evaluable conditions (eligibility, constraints), the framework offers a typed predicate vocabulary in the base specification (Tier 1) and a `policy_ref` extension point for richer policy languages like Cedar, Rego, or XACML (Tier 2). Implementations support Tier 1 by default; Tier 2 is opt-in for deployments needing full policy-engine expressivity.

### 2.7 Provenance for layered authorship

Constraints in particular are often layered in by parties other than the delegator. Each constraint may carry a `source` field identifying its author (delegator, authority, relationship, platform, regulatory), enabling auditors to trace responsibility for any element of the constraint surface.

---

## 3. Component Overview

The five delegation components form two enforceable pairs plus one inert declared-intent layer:

**Establishment pair (long-lived, infrequent change):**
- *Relationship* — defines the parties and their eligibility to participate in the delegation
- *Authority* — defines why the delegator may delegate and the broadest scope thereof

**Declared-intent layer (declared once, inert):**
- *Objective* — declares the umbrella act of the delegation, its purpose, and an optional beneficiary. It is committed and rendered for consent and audit but is *inert with respect to authority*: it never derives, widens, or gates what is permitted (§9). It sits, on the binding chain, between Authority and the runtime task.

**Runtime pair (task-specific, ephemeral):**
- *Obligations* — defines the essential task
- *Constraints* — defines the bounds within which the task must be performed

Obligations and Constraints are co-located in a single signed JWT (`task+jwt`). Relationship and Authority are separate JWTs, each independently issuable. The Objective may be a separate JWT (`objective+jwt`) when one declared act underwrites many tasks, or an inline claim within the Task JWT for the simple single-task case (§9.3).

```
┌─────────────────────────────────────────────────────────────────────┐
│                        ESTABLISHMENT LAYER                          │
│                                                                     │
│  ┌────────────────────────┐         ┌───────────────────────────┐   │
│  │   Relationship JWT     │ ◄────── │     Authority JWT         │   │
│  │   typ: relationship+jwt│         │     typ: authority+jwt    │   │
│  │                        │ rel_jti │                           │   │
│  │  delegator             │ rel_hash│  source_authority         │   │
│  │  delegatee             │         │  derivative_authority     │   │
│  │  eligibility           │         │                           │   │
│  └────────────────────────┘         └───────────────────────────┘   │
│             ▲                                  ▲                    │
│             │ rel_jti / rel_hash               │ auth_jti/auth_hash │
└─────────────┼──────────────────────────────────┼────────────────────┘
              │                                  │
┌─────────────┼──────────────────────────────────┼────────────────────┐
│             │   DECLARED-INTENT LAYER (inert)  │                    │
│             │                                  │                    │
│         ┌───┴──────────────────────────────────┴────┐               │
│         │            Objective (optional)           │               │
│         │            typ: objective+jwt             │               │
│         │                                           │               │
│         │  action      (umbrella act)               │               │
│         │  purpose     (kind / params / display)    │               │
│         │  beneficiary (optional)                   │               │
│         │  -- inert: never gates authority (§9.2) --│               │
│         └───────────────────────────────────────────┘               │
│                          ▲                                          │
│                          │ objective_jti / objective_hash           │
└──────────────────────────┼──────────────────────────────────────────┘
                           │
┌──────────────────────────┼──────────────────────────────────────────┐
│            RUNTIME LAYER  │                                          │
│                           │                                          │
│         ┌─────────────────┴─────────────────────────┐               │
│         │              Task JWT                     │               │
│         │            typ: task+jwt                  │               │
│         │                                           │               │
│         │  objective    (inline, or ref to above)   │               │
│         │  obligations  (composition_logic + items) │               │
│         │  constraints  (tier1 + tier2)             │               │
│         └───────────────────────────────────────────┘               │
└─────────────────────────────────────────────────────────────────────┘
```

The Task JWT additionally carries the `rel_jti`/`auth_jti` references shown in the establishment layer; the Objective, when present as a standalone `objective+jwt`, carries the same upstream references and is itself referenced by the Task via `objective_jti`/`objective_hash`. A single Relationship + Authority pair typically underwrites many Task JWTs over its lifetime, and a single Objective may underwrite many Task JWTs when the delegatee generates the specific Obligations (§9.3).

### 3.1 The Task JWT — Obligations and Constraints Bound Together

Obligations and Constraints are not two independently-issued artifacts that the execution layer reassembles at runtime. They are sub-claims within a single signed JWT (`typ: task+jwt`) and travel as a unit. This specification deliberately binds them at four distinct layers, so that no implementation path can evaluate one without also evaluating the other.

**Cryptographic binding.** Obligations and Constraints are claims within the same JWT envelope and signed together. There is no "Obligations JWT" and no "Constraints JWT" in this specification — only the composite Task JWT. Presenting an Obligation without its issued Constraints (or vice versa) is not possible without breaking the signature. The smallest issuable unit of run-time delegation is the Task JWT, not its constituents.

**Structural binding.** A Task JWT MUST contain both an `obligations` claim and a `constraints` claim. The `constraints` claim MAY be an empty array (a Task with no bounds beyond those inherited from the Authority and Relationship), but the structural placeholder MUST be present so that downstream verifiers cannot mistake "no constraints" for "constraints were lost in transit." `obligation_id` values are scoped to the Task JWT in which they appear; a constraint's `applies_to_obligations` references MUST refer to obligation_ids defined in the same Task JWT, and MUST NOT reach across Task JWTs.

**Scope binding.** Within the Task JWT, each Constraint either applies to specific Obligations (via `applies_to_obligations`) or to all Obligations (when `applies_to_obligations` is absent). Both forms are scoped to the Task JWT only. Constraints layered in by parties other than the delegator (platform, regulatory, judicial, per §7.5) are carried inside the same Task JWT and bound by the same signature as the delegator-authored Obligations; there is no separate "regulatory constraints" artifact that the verifier composes at runtime.

**Commit-boundary binding.** Per §8.3, the verifier MUST re-check the Task JWT's status signal at the commit boundary, not only at issuance. This re-evaluation operates on the bundle: the runtime authorization decision is not "is this Obligation valid AND are these Constraints valid" considered independently, but "do these Obligations within these Constraints still authorize this act, right now." A Constraint whose underlying basis has shifted in flight (a court order narrowed scope after issuance; a regulatory cap lowered a budget; a trigger event per §7.7 fired) invalidates the Task JWT even if the Obligation considered in isolation still looks well-formed. The bundle is what the verifier honors or denies — never one half of it.

The cumulative effect is that the Task JWT is the smallest unit at which an act can be authorized at runtime. There is no specification path by which an Obligation is honored while its bounding Constraints are bypassed, and no path by which Constraints are evaluated without the Obligations they scope. The execution layer (companion document, *Delegated Authorization Execution Layer*) is responsible for performing the bundle-level re-evaluation; this specification ensures the structure of the Task JWT makes any other implementation choice impossible.

A Task JWT MAY additionally carry an Objective (§9), inline or by reference, declaring the umbrella act and purpose the bundle executes under. The Objective is bound to the Task (inline under the same signature, or by `objective_jti`/`objective_hash`) and is verified for integrity, but it is **inert**: it is never part of the bundle-level authorization decision described above. The four-facet binding governs Obligations and Constraints, the enforceable bundle; the Objective rides alongside as committed, integrity-checked declared intent (§9.2).

---

## 4. Relationship

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
|----------|---------|
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

```json
{
  "fmt": "oidc",
  "iss": "https://idp.example",
  "sub": "user-12345"
}
```

Supported formats include `oidc`, `did`, `spiffe`, `x509`, and `org-uri`. Implementations MUST reject identity objects with unknown `fmt` values rather than attempt heuristic interpretation.

### 4.5 Group Descriptor

When a party is a group rather than a single identity, a `group` sub-object is present:

```json
{
  "group_type": "policy",
  "policy_uri": "https://policy.example/narcotic-delegation/v2",
  "policy_hash": { "alg": "sha-256", "digest": "..." },
  "policy_lang": "cedar",
  "evaluation_endpoint": "https://authz.example/evaluate",
  "min_members": 1
}
```

The `group_type` field takes one of `static`, `role`, or `policy`. The `min_members` field expresses quorum requirements (e.g., two-nurse co-authorization for high-risk medications).

For a `policy` group, `evaluation_endpoint` is a **commit-time authority-conferring source**: whether the presenting party is a member of the eligible group — and therefore whether the delegation applies to it at all — is decided by dereferencing it at the moment of action. It is therefore subject to the commit-time source-integrity rule of §12.3: the endpoint MUST be authenticated and its responses integrity-protected, and a verifier that cannot reach it, or cannot validate its response, within the deployment's freshness bound MUST **fail closed** — it MUST NOT treat unresolved membership as membership, and MUST deny the act rather than proceed on a stale or unverifiable evaluation. The `policy_hash` binds the policy document the endpoint is expected to evaluate, so a substituted policy is detectable independently of transport authentication.

### 4.6 Eligibility Criteria

Tier 1 (typed predicates) supports the following base vocabulary, all of which may be combined via `combination` predicates with `and`/`or` operators:

- **temporal** — date ranges, time-of-day windows, day-of-week, timezone
- **geographic** — jurisdictions, regions, proximity to a point
- **credential** — required credential type, issuer, minimum assurance level
- **role** — required organizational role, with org_id and org_uri
- **quorum** — minimum number of members required (links to group descriptor)

Tier 2 (`policy_ref`) carries `policy_uri`, `policy_hash`, `policy_lang`, `policy_version`, and optional `evaluation_endpoint`. The two tiers may be present simultaneously; verifiers MUST satisfy both to honor the relationship.

### 4.7 Example: Doctor-Nurse Narcotic Delegation

```json
{
  "iss": "https://credentialing.cityhospital.example/delegation",
  "iat": 1714000000,
  "nbf": 1714000000,
  "exp": 1745536000,
  "jti": "urn:uuid:a3f2c1d0-8e4b-4f7a-9c2d-1b0e3f5a7c9d",
  "rel_type": "urn:ietf:params:delegation:principal-agent",
  "rel_version": "1.0",
  "rel_status": "active",
  "rev_endpoint": "https://credentialing.cityhospital.example/delegation/revoke",
  "delegator": {
    "fmt": "oidc",
    "iss": "https://idp.cityhospital.example",
    "sub": "dr-jane-smith-7291"
  },
  "delegatee": {
    "fmt": "oidc",
    "iss": "https://idp.cityhospital.example",
    "group": {
      "group_type": "policy",
      "policy_uri": "https://policy.cityhospital.example/narcotic-delegation/v2",
      "policy_hash": {
        "alg": "sha-256",
        "digest": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
      },
      "policy_lang": "cedar",
      "evaluation_endpoint": "https://authz.cityhospital.example/evaluate",
      "min_members": 1
    }
  },
  "eligibility": {
    "tier1": {
      "credential": {
        "type": "DEA-COR",
        "issuer": "https://www.deadiversion.usdoj.gov",
        "schedules": ["II", "III"],
        "min_assurance": "LOA3"
      },
      "role": {
        "role_id": "on-call-nurse",
        "org_id": "cityhospital",
        "org_uri": "https://idp.cityhospital.example/org/nursing"
      },
      "combination": {
        "op": "and",
        "conditions": ["credential", "role"]
      }
    }
  }
}
```

This Relationship asserts that Dr. Jane Smith (delegator) may delegate to any individual who is simultaneously (a) holding an active DEA registration for Schedule II and III substances and (b) holding the `on-call-nurse` role at City Hospital. The Cedar policy at `policy_uri` is the authoritative definition of group membership and is dereferenced and evaluated at run-time.

### 4.8 Graduated Authority Over Time

Some delegations are not single-event grants but **progressive transfers** in which authority shifts gradually between parties as circumstances change. The canonical example is a child's evolving capacity: a guardian holds full digital authority for a young child, shares decision-making with an older child, and ultimately retains only safeguarding authority for an adolescent before fully ceding control at the age of majority. The same pattern arises in capacity-graded medical decision-making, in escalating organizational responsibility, and in time-boxed corporate succession.

The framework supports two implementation patterns for graduated authority. Deployments may choose either based on the operational characteristics of the issuing intermediary; the base specification does not normatively prefer one.

**Pattern A — Sequential Relationships.** A series of Relationship JWTs are issued, each scoped to a defined life stage or capacity band, with non-overlapping `nbf`/`exp` windows. The issuing intermediary manages the sequence, ensuring that exactly one Relationship is active at any given time. Transitions between stages are revocation-and-issuance events. This pattern is operationally simpler to verify (each verifier sees one Relationship at a time) but requires the intermediary to actively manage the schedule.

**Pattern B — Single Relationship with Stage-Keyed Eligibility.** A single Relationship JWT carries a `stage_schedule` field within `eligibility.tier1`, defining the authority bands and their boundary conditions. Verifiers evaluate the schedule against the runtime context (e.g., the child's current age) to determine which band is active. This pattern is more compact but bakes a state machine into the credential.

When using Pattern B, the schedule structure is:

```json
"stage_schedule": {
  "schedule_type": "age_banded",
  "subject_age_attribute": "delegatee_age",
  "stages": [
    {
      "stage_id": "early",
      "min_age": 0,
      "max_age": 7,
      "scope_override": "https://schemas.example.com/scope/full-guardianship"
    },
    {
      "stage_id": "consultative",
      "min_age": 8,
      "max_age": 12,
      "scope_override": "https://schemas.example.com/scope/consultative-guardianship"
    },
    {
      "stage_id": "shared",
      "min_age": 13,
      "max_age": 15,
      "scope_override": "https://schemas.example.com/scope/shared-decision"
    },
    {
      "stage_id": "safeguarding-only",
      "min_age": 16,
      "max_age": 17,
      "scope_override": "https://schemas.example.com/scope/safeguarding-only"
    }
  ],
  "terminal_event": {
    "event_type": "age_of_majority_reached",
    "behaviour": "automatic_revocation"
  }
}
```

The `scope_override` field interacts with the bound Authority JWT: when the active stage carries a scope_override, that scope (rather than the Authority's `derivative_authority.permission.scope_domain`) is the operative scope at the moment of evaluation. This allows a single Authority instrument to underwrite a graduated chain of operative scopes without requiring re-issuance of the Authority itself.

The `terminal_event` field defines the boundary condition that ends the Relationship. Terminal events are typically irrevocable (an adult cannot have their majority undone) and trigger automatic, non-cascading revocation per §8.4.

### 4.9 Role-Tuple Group Sub-Type

The base group descriptor (§4.5) supports `static`, `role`, and `policy` group types. For consensus-driven decisions in which authority requires the simultaneous participation of *several distinct roles*, a fourth sub-type is defined: **role-tuple**.

A role-tuple group requires that, at the moment of evaluation, at least one credentialed individual is presented for *each* of the named roles. This is structurally distinct from the `min_members` quorum field, which requires N members from a single role pool. Role-tuple groups model situations like a child welfare strategic decision requiring the joint participation of a social worker, a psychologist, a residential care manager, and an Independent Reviewing Officer.

```json
"group": {
  "group_type": "role_tuple",
  "required_roles": [
    {
      "role_id": "social_worker",
      "org_id": "lambeth-childrens-services",
      "min_assurance": "LOA3"
    },
    {
      "role_id": "psychologist",
      "credential_type": "HCPC-registered",
      "min_assurance": "LOA3"
    },
    {
      "role_id": "care_home_manager",
      "credential_type": "ofsted-registered",
      "min_assurance": "LOA3"
    },
    {
      "role_id": "independent_reviewing_officer",
      "credential_type": "iro-appointed",
      "min_assurance": "LOA3"
    }
  ],
  "consensus_mode": "all_required"
}
```

The `consensus_mode` field accepts:

- `all_required` — every named role must be filled (default)
- `n_of_m` — at least N of the M named roles must be filled, with N specified by an additional `min_roles_filled` field
- `weighted` — each role carries a weight, and the sum of present roles' weights must meet a `min_weight` threshold; useful where a senior role can substitute for two junior roles

Role-tuple groups compose with other group sub-types: a delegatee may be defined as a static group whose members are themselves role-tuples, supporting nested consensus structures. Verifiers MUST be able to resolve at least the simple `all_required` form to comply with the base specification; `n_of_m` and `weighted` modes are optional features that profiles may require.

---

## 5. Authority

### 5.1 Purpose

The Authority JWT carries two related but distinct authority assertions, either or both of which may be present:

- **Source authority** — proof that the delegator had the right to delegate at all (e.g., a power of attorney, a corporate director appointment, a hospital credentialing instrument).
- **Derivative authority** — the rights as they flow to the delegatee, expressed at the *coarsest meaningful scope* (e.g., "travel domain," "narcotic administration domain"). Specific tasks and their bounds are expressed in Obligations and Constraints.

### 5.2 Relationship to OpenID Authority Claims Extension

Authority builds directly on the OpenID Connect Authority claims extension (working draft) for the source authority structure. The `applies_to`, `permission`, and `granted_by` sub-elements are reused with the following modifications:

- Extension of `applies_to` to support entity-to-entity (workload) and affiliation cases not explicitly in scope of the OpenID draft.
- Reduction of `permission` to a minimal set on the *derivative* authority (scope domain + further-delegation rules), since validity, budget, audience, and function are properly expressed as Constraints in this framework.
- Addition of an `inherent` value for `granted_by.method` to express self-authority of natural persons over their own affairs.
- Addition of a `prior_authority_ref` field on derivative authority to support summary-level chain-of-authority semantics.

### 5.3 Structure

An Authority JWT contains:

1. **JOSE header** with `typ: "authority+jwt"`
2. **Registered claims**
3. **Relationship binding** — `rel_jti`, `rel_hash`
4. **Optional verified-claims envelope** (eKYC IDA) — required for high-assurance profiles
5. **Optional `source_authority`** — full OpenID Authority structure
6. **Optional `derivative_authority`** — coarse scope domain + further-delegation rules + optional prior-authority reference

At least one of `source_authority` or `derivative_authority` MUST be present.

### 5.4 Source Authority

The source authority section follows the OpenID Authority structure:

- `applies_to` — the entity the authority covers (organization, natural person, or workload)
- `permission` — `role`, `may_delegate` (boolean), and optional `validity` and `function`
- `granted_by` — `method` (`delegated`, `appointed`, `self-asserted`, `inherent`), `granting_body`, `reason`

### 5.5 Derivative Authority

The derivative authority section is intentionally minimal:

- `permission.scope_domain` — a free-form URI naming the coarsest meaningful scope (e.g., `https://schemas.example.com/scope/travel`)
- `permission.further_delegation` — an object with `allowed`, `max_depth`, `sub_scopes`, and `constraints_inheritance` (`strict` | `additive`)
- `prior_authority_ref` (optional) — `authority_uri`, `authority_hash`, `link_method` for summary-level chain references

#### 5.5.1 Scope Matching Semantics (normative)

`scope_domain` is a free-form URI with no globally-defined subsumption relation: two issuers may coin overlapping or nested scope URIs with no agreed rule for when one contains another. A verifier therefore MUST NOT decide whether an act is authorized by lexically matching it against `scope_domain` — not by prefix, substring, or hierarchical-path containment. `scope_domain` is **advisory**: it names the authority's coarsest scope for **categorization, discovery, and audit legibility**, and does not by itself gate an act.

The enforceable "within authority" test is expressed instead over a **typed vocabulary with defined comparison** — the RFC 9396 RAR type registry that Obligations already draw on (§6.3). An Obligation is **within the derivative authority** when its RAR `type` is admitted by the authority, and admission is decided by **exact registered-identifier equality** — value-space, not lexical: two identifiers that denote the same registered type match regardless of serialization, and two that denote different types do not, whatever their string similarity. Any mapping from a coarser scope identifier to a set of registered types is a deployment or profile concern, out of scope of the base comparison rule.

The set of admitted types is enumerated by the authority, not inferred from `scope_domain`. `further_delegation.sub_scopes`, when present, enumerates the scopes or types the delegatee MAY **delegate onward**; each entry is compared by the same registered-identifier equality (a coarser scope identifier, as in §5.7, resolves to its registered type set by the deployment's mapping before comparison), and a scope or type absent from that enumeration MUST NOT be delegated onward. For the delegatee's **own** exercise, the enforceable bound is the Obligations and Constraints validated against this authority at issuance (§8.1) and re-checked at the commit boundary (§8.3); `scope_domain` contributes categorization only. A deployment that needs to gate an action by admitted type *before* an Obligation is formed (for example, at a Permissioned Capabilities PDP, §11.1) carries that enumeration in a profile — the base specification fixes the comparison rule, not a wire field for it.

This keeps Authority's run-time role real without resting it on undefined string matching: the authority still bounds *what may be delegated onward* (an explicit, value-space-checkable enumeration) and *what basis the delegator holds* (source authority, §5.4), while the free-form `scope_domain` is demoted to the advisory label it can be relied on to be. The value-space discipline follows the approach the Mission-Bound Authorization suite takes for its Common Constraint subset and intersection rules — every registered entry defines its own value-space comparison so independent deployments compute the same result (§13) — applied here by pushing the enforceable test onto the registered RAR type vocabulary rather than onto free-form URIs.

The graduated-authority Pattern B `scope_override` mechanism (§4.8) interacts with this reframing and is **not** reconciled here: §4.8 currently designates a stage's `scope_override` as "the operative scope," which reads as an enforcement lever, whereas this section makes `scope_domain` advisory. Squaring the two — whether `scope_override` should move an advisory label, gate value-space type admission, or be replaced by sequenced Authorities — is part of the separate structural decision tracked as a known open item in §12.2, and is deliberately left open.

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

```json
{
  "iss": "https://wallet.bob.example",
  "iat": 1714000000,
  "exp": 1745536000,
  "jti": "urn:uuid:b7e2f1a4-3c8d-4e2a-9f1b-2c5d8e0f3a91",
  "rel_jti": "urn:uuid:9d4f8b2c-1e3a-4b5c-8d7e-6f0a1b2c3d4e",
  "rel_hash": {
    "alg": "sha-256",
    "digest": "f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2"
  },
  "derivative_authority": {
    "permission": {
      "scope_domain": "https://schemas.example.com/scope/travel",
      "further_delegation": {
        "allowed": true,
        "max_depth": 1,
        "sub_scopes": [
          "https://schemas.example.com/scope/travel/booking",
          "https://schemas.example.com/scope/travel/payment"
        ],
        "constraints_inheritance": "strict"
      }
    }
  }
}
```

Source authority is omitted; under the agentic profile this signals self-authority of a natural person over their own affairs.

### 5.8 Example B: High-assurance — Hospital → Dr. Smith

```json
{
  "iss": "https://credentialing.cityhospital.example",
  "iat": 1714000000,
  "exp": 1745536000,
  "jti": "urn:uuid:c8d3f9e7-2b4a-5c6d-7e8f-9a0b1c2d3e4f",
  "rel_jti": "urn:uuid:a3f2c1d0-8e4b-4f7a-9c2d-1b0e3f5a7c9d",
  "rel_hash": {
    "alg": "sha-256",
    "digest": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2"
  },
  "verified_claims": {
    "verification": {
      "trust_framework": "us-healthcare-credentialing-framework",
      "time": "2026-01-15T10:30:00Z",
      "verification_process": "urn:uuid:7f8a9b0c-1d2e-3f4a-5b6c-7d8e9f0a1b2c"
    },
    "claims": {
      "given_name": "Jane",
      "family_name": "Smith",
      "birthdate": "1978-03-22",
      "source_authority": {
        "applies_to": {
          "organization_name": "City Hospital",
          "registration_number": "CH-2001-NY",
          "legal_jurisdiction": "US-NY",
          "organization_identifiers": [{
            "identifier_type": "NPI",
            "identifier": "1234567890",
            "issuer": "CMS"
          }]
        },
        "permission": {
          "role": "Attending Physician",
          "may_delegate": true,
          "validity": { "start": "2024-07-01", "end": "2027-06-30" },
          "function": "narcotic-prescription-and-administration"
        },
        "granted_by": {
          "method": "appointed",
          "granting_body": "City Hospital Medical Staff Credentialing Committee",
          "reason": "Active medical staff appointment with Schedule II prescribing privileges"
        }
      }
    }
  },
  "derivative_authority": {
    "permission": {
      "scope_domain": "https://cityhospital.example/scope/narcotic-administration",
      "further_delegation": { "allowed": false }
    }
  }
}
```

Both source and derivative authority are present. Source authority is wrapped in the `verified_claims` envelope and asserts Dr. Smith's credentialed status. Derivative authority asserts that the delegation covers narcotic administration but cannot be further delegated by the delegatee.

---

## 6. Obligations

### 6.1 Purpose

The Obligations section of the Task JWT defines the *essential* elements of the task — the verb and its essential complements without which the task is incomplete. The cleavage rule:

> An element belongs in Obligations if removing it would render the task undefined. An element belongs in Constraints if removing it would merely make the task less restrictive.

Thus: a hotel booking's destination and dates are Obligations. Its budget, brand allowlist, and proximity to a landmark are Constraints.

A third category — **completion definition** — also belongs in Obligations: elements that specify *when an obligation is satisfied or discharged*, rather than bounding how it may be performed. Removing a completion definition changes the task from a bounded, purpose-scoped action into an open-ended grant, which is a change in meaning, not merely a relaxation of constraints. See §6.6.

Obligations and Constraints share the same Task JWT envelope and are bound together cryptographically, structurally, by scope, and at the commit boundary, as specified in §3.1. The runtime authorization decision is on the bundle, not on either component in isolation. The *why* of the bundle — its declared purpose, and the umbrella act it executes under — is carried by the Objective (§9), not by Obligations. As of v2.0 the structured `purpose` object has moved from Obligations to the Objective (§9.4), where its shape and the base purpose-class vocabulary are now defined.

### 6.2 Structure

The `obligations` object contains:

- `composition_logic` — `and` (all items must be performed) or `or` (any one item satisfies the task)
- `items` — array of obligation entries

> **Moved in v2.0.** The `purpose` field is no longer carried on `obligations`. It is now a sub-claim of the Objective (§9.4). A Task JWT that needs to declare purpose carries it under an inline `objective` claim (§9.3, §9.6) or by reference to a standalone `objective+jwt`. Deployments that emitted `obligations.purpose` under v1.1–v1.4 SHOULD treat that field as an inline Objective `purpose` with no change in wire shape. The object's structure is defined in §9.4.

Each obligation entry contains:

- `obligation_id` — unique within the Task JWT, used as the target of constraints
- `type` — RFC 9396 RAR type identifier (shared registry with OAuth RAR)
- Type-specific fields per the RAR type definition
- `terminal_when` (optional) — completion event definition specifying when this obligation is satisfied and no further authorization may be granted under it; see §6.6

### 6.3 RAR Type Reuse

This framework deliberately reuses the RFC 9396 RAR type registry. RAR-aware authorization servers can interpret an obligation's type-specific fields without learning a new vocabulary. Where this framework requires a type not yet in the RAR registry, the expectation is to register it through the standard IANA process rather than create a parallel registry.

### 6.4 Sequencing

For composite tasks like "find hotel, then book it," sequencing is implicit in `and` composition; the agent or orchestrator handles execution order based on data dependencies. Explicit `pre_conditions` are not part of the base specification but may be added by extension if deployment experience shows a need.

### 6.5 Example: Hotel Booking

```json
"obligations": {
  "composition_logic": "and",
  "items": [
    {
      "obligation_id": "obl-1",
      "type": "hotel_booking",
      "locations": [
        { "city": "Boston", "country": "US", "region": "MA" }
      ],
      "check_in": "2026-05-12",
      "check_out": "2026-05-15",
      "guests": { "adults": 1, "children": 0 },
      "rooms": 1
    }
  ]
}
```

Each field on the obligation is essential: the locations, dates, guest count, and room count cannot be omitted without making the task undefined. (Prior to v2.0 this `obligations` object also carried a `purpose` field; as of v2.0 that purpose is declared in the Objective — see §9.6 and Scenario A, §10.1 — and no longer sits on `obligations`.)

### 6.6 Terminal Conditions on Obligations

Some obligations have an inherent completion event — a point at which the task is semantically done regardless of calendar expiry or any externally-imposed constraint. For these obligations, the completion event is part of the obligation's *definition*, not a performance bound on it. The optional `terminal_when` sub-field on an obligation item captures this.

**Why completion definition belongs in Obligations.** The cleavage rule (§6.1) identifies two categories: elements whose removal renders the task undefined (Obligations) and elements whose removal merely makes the task less restrictive (Constraints). Completion definition is a third category: removing it does not leave the task undefined, but it changes the task's meaning from a bounded, purpose-scoped action into an open-ended grant. "Release vaccination records for school enrollment" is semantically different from "release vaccination records indefinitely." The terminal condition is not a constraint on how the task may be performed; it is what makes the delegation bounded in time and purpose.

**Relationship to §7.7 trigger-based expiry.** Obligation-level terminal conditions and constraint-level trigger-based expiry are complementary, not overlapping:

| | `terminal_when` (§6.6 — Obligation item) | `expires_on_event` (§7.7 — Constraint) |
|---|---|---|
| Authored by | Delegator, as part of task definition | Any constraint author: delegator, platform, regulatory, judicial |
| Semantics | Obligation *satisfied* — task complete | Obligation *voided* — task invalid |
| Omittable by issuer? | Harder: embedded in the obligation item itself | Yes: a separately-authored constraint |
| Behavior on fire | Obligation discharged; no further authorization under it | Task JWT invalidated per configured `behaviour_on_event` |
| Audit interpretation | Completion record | Revocation or lapse record |

A Task JWT MAY carry both: `terminal_when` on the obligation (enrollment complete → task done) and `expires_on_event` on a constraint (if enrollment runs past a regulatory deadline → task void). These fire independently; either closes authorization under the affected obligation.

**Structure.** `terminal_when` reuses the event vocabulary of §7.7, with `behaviour_on_event` omitted because the behavior is fixed:

- `event_type` — a registered event identifier from the §7.7 base vocabulary, or a profile-extended type
- `event_source` — how to determine whether the event has occurred; same structure as §7.7 (`source_type`, `source_uri`, `evaluation_endpoint`)
- `polling_interval_max` — maximum acceptable staleness of event evaluation, as ISO 8601 duration

**Evaluation semantics.** Verifiers MUST check `terminal_when` status at the commit boundary (§8.3), subject to the same staleness rules as other status signals. When the configured event fires:

- The obligation is **discharged**: further authorization requests citing that `obligation_id` MUST be denied.
- Under `composition_logic: "and"`: once all obligations in the Task JWT are discharged, no further actions may be authorized under the Task JWT.
- Under `composition_logic: "or"`: remaining non-discharged obligations continue to be honorable; only the discharged obligation is closed.
- Discharge differs from revocation in audit semantics: the Task JWT is complete, not invalid. Issuers SHOULD reflect this distinction in audit records and, where applicable, in status responses.

**Example: Maya / Theo vaccination record disclosure.** Maya has been granted authority to release Theo's vaccination records to a school records office in connection with an enrollment application. The `terminal_when` field makes the bounded character of the release explicit in the task definition itself, rather than relying on a separately-configured trigger-based expiry constraint that an issuer could accidentally omit.

```json
"obligations": {
  "composition_logic": "and",
  "items": [
    {
      "obligation_id": "obl-1",
      "type": "record_disclosure",
      "record_type": "vaccination_records",
      "recipient": {
        "type": "school_records_office",
        "ref": "ENR-2026-04582"
      },
      "terminal_when": {
        "event_type": "enrollment_completed",
        "event_source": {
          "source_type": "governmental_registry",
          "source_uri": "https://enrollment.example.gov/ENR-2026-04582",
          "evaluation_endpoint": "https://enrollment.example.gov/ENR-2026-04582/status"
        },
        "polling_interval_max": "PT1H"
      }
    }
  ]
}
```

The declared purpose of this release (record-disclosure for school enrollment) travels in the accompanying inline Objective shown in §9.6, not on `obligations` (the relocation is described in §9.4). Note the division of labor the Objective makes explicit: the *enforceable* bounding — discharge of the obligation when enrollment completes — is the Obligation's `terminal_when` below; the *inert* record of why the release was authorized is the Objective's purpose. When enrollment ENR-2026-04582 transitions to `completed` or `cancelled`, the obligation is discharged and no further release of records may be authorized under this Task JWT, regardless of the JWT's cryptographic validity or the status of any other components in the binding chain.

---

## 7. Constraints

### 7.1 Purpose

The Constraints section of the Task JWT bounds the acceptable performance of the obligations. Constraints are typically more numerous and more diverse in authorship than Obligations.

Constraints are bound to the Obligations they scope within the same Task JWT: they share a signature, share an envelope, reference obligation_ids that exist only in that envelope, and are re-evaluated together as a bundle at the commit boundary. The four facets of this binding are specified in §3.1. Constraints in this specification do not exist as standalone artifacts; they are always part of a Task JWT.

### 7.2 Structure

The `constraints` object contains:

- `tier1` — array of typed predicate constraints
- `tier2` (optional) — array of policy-language references for constraints not expressible in the Tier 1 vocabulary

Each Tier 1 constraint contains:

- `category` — one of the registered categories (see 7.3)
- Category-specific fields
- `applies_to_obligations` (optional) — array of `obligation_id` values; absent means applies to all Obligations in the same Task JWT. Referenced obligation_ids MUST exist within the same Task JWT; cross-Task references are not permitted (see §3.1 for the binding rationale). Verifiers MUST reject a Task JWT whose constraints reference obligation_ids not defined in that Task JWT.
- `source` (optional) — `delegator`, `authority`, `relationship`, `platform`, or `regulatory`; recorded when the constraint is not authored by the delegator
- `source_ref` (optional) — URI identifying the underlying policy or rule the constraint derives from

### 7.3 Constraint Categories

The base Tier 1 vocabulary:

| Category | Purpose | Key fields |
|----------|---------|------------|
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

```json
{
  "tier2": [
    {
      "policy_uri": "https://policy.cityhospital.example/medication-rules/v3",
      "policy_hash": { "alg": "sha-256", "digest": "..." },
      "policy_lang": "cedar",
      "policy_version": "3.2.1",
      "evaluation_endpoint": "https://authz.cityhospital.example/evaluate",
      "applies_to_obligations": ["obl-1"],
      "source": "platform"
    }
  ]
}
```

### 7.5 Constraint Provenance

The `source` field is the primary auditability mechanism for layered authorization. When constraints come from multiple authors (delegator, platform, regulatory), the source field allows verifiers and auditors to:

- Trace the responsible party for any element of the constraint surface
- Determine whether the platform or regulatory constraints have been tampered with by the delegator
- Diagnose constraint failures by identifying which authority's rule was violated

### 7.6 Example: Layered Constraints

```json
"constraints": {
  "tier1": [
    {
      "category": "monetary",
      "max_per_night": { "amount": "400.00", "currency": "USD" },
      "max_total": { "amount": "1200.00", "currency": "USD" },
      "source": "delegator"
    },
    {
      "category": "geographic",
      "proximity": {
        "anchor": { "lat": 42.3601, "lon": -71.0589, "label": "Hynes Convention Center" },
        "max_distance_km": 1.5,
        "mode": "walking"
      },
      "source": "delegator"
    },
    {
      "category": "resource",
      "rule": "deny",
      "resources": [
        { "type": "vendor", "value": "ofac_sanctioned" }
      ],
      "source": "regulatory",
      "source_ref": "https://sanctions.treasury.gov/ofac"
    },
    {
      "category": "data_minimization",
      "permitted_fields": ["full_name", "email", "loyalty_number"],
      "data_subject": "delegator",
      "source": "platform",
      "source_ref": "https://platform.example/policy/booking-pii-minimization"
    }
  ]
}
```

### 7.7 Trigger-Based Expiry

The base temporal constraint vocabulary defined in §7.3 supports calendar-based bounds (`valid_from`, `valid_until`, `time_windows`). Many real-world delegations require expiry tied not to a calendar date but to the **occurrence of an external event**: a court order termination, the next statutory review, the child reaching the age of majority, the completion of a probate proceeding, the closure of an investigation. These cannot be reliably expressed as calendar dates because the date is unknown at issuance time.

Trigger-based expiry is defined as a first-class temporal constraint sub-category, `expires_on_event`, with the following structure:

```json
{
  "category": "temporal",
  "expires_on_event": {
    "event_type": "court_order_terminated",
    "event_source": {
      "source_type": "court_registry",
      "source_uri": "https://courts.example/orders/2026-FAM-04582",
      "evaluation_endpoint": "https://courts.example/orders/2026-FAM-04582/status"
    },
    "behaviour_on_event": "immediate_revocation",
    "polling_interval_max": "PT1H"
  }
}
```

The fields are:

- `event_type` — a registered event identifier from a base vocabulary that includes `court_order_terminated`, `court_order_amended`, `statutory_review_completed`, `placement_changed`, `age_of_majority_reached`, `capacity_assessment_changed`, `employment_terminated`, `proceeding_closed`. Profiles may register additional types.
- `event_source` — describes how to determine whether the event has occurred. The `source_type` field discriminates among `court_registry`, `governmental_registry`, `organizational_registry`, `webhook_subscription`, and `manual_attestation`. The `evaluation_endpoint` is dereferenceable to obtain current event status.
- `behaviour_on_event` — one of `immediate_revocation` (the constraint and bound JWTs become invalid the moment the event occurs), `grace_period_then_revocation` (with a `grace_period` duration), or `transfer_required` (the credential remains valid only if a successor credential is presented; see §8.5).
- `polling_interval_max` — the maximum acceptable staleness of event evaluation, expressed as ISO 8601 duration. Verifiers MUST re-check the event source at intervals no greater than this.

The `event_source.evaluation_endpoint` is a **commit-time authority-removing source**: it is consulted to learn whether an event has fired that would invalidate the credential, so an adversary who can suppress or spoof its responses can keep an event-revoked credential alive or kill a valid one. It is therefore subject to the commit-time source-integrity rule of §12.3 on the same footing as the revocation endpoint (§8.4): the `evaluation_endpoint` MUST be authenticated and its responses integrity-protected. Because the source *removes* authority, an unreachable or unverifiable event source **fails closed**: a verifier that cannot establish current event status within `polling_interval_max` MUST treat the credential as if the terminating event may have fired and MUST NOT honor the act — it MUST NOT assume "not yet fired" from the absence of a reachable answer. Where a deployment cannot tolerate a hard stop on a transient source outage, the softening is the existing `behaviour_on_event: grace_period_then_revocation`, which bounds how long an unconfirmed credential may continue before it is treated as revoked; a deployment MUST NOT instead fail open. (This is the opposite default from the inert Objective of §8.1 step 8, which fails *safe* on unavailability precisely because it confers and removes no authority; the contrast is drawn in §12.3.)

Trigger-based expiry interacts with calendar expiry: if both are present on a constraint, the credential becomes invalid at the *earlier* of the two boundaries. A credential with `valid_until: 2027-01-01` and `expires_on_event: court_order_terminated` is invalid as soon as the court order ends or the calendar date passes, whichever comes first.

Trigger-based expiry is itself one form of the commit-boundary status signal generalized in §8.3: the fact that the trigger event has fired is the status signal, evaluated at commit time. Trigger-based expiry MAY be applied to any layer of the binding chain: a Relationship may carry a trigger-based expiry, in which case revocation of the Relationship cascades to bound Authority and Task JWTs per §8.4. Constraints on a Task JWT may carry trigger-based expiry independently of the Relationship's lifecycle.

Privacy note: the `event_type` and `event_source` fields are inherently revealing — knowing that a credential expires when "placement_changed" is itself a strong signal of care-status. Deployments operating under privacy-preserving requirements MUST use the layering rules defined in the *Privacy-Preserving Profile* companion document, which specifies how trigger-based expiry is wrapped in opaque tokens at the relying-party boundary.

---

## 8. Cryptographic Binding and Commit-Boundary Validity

### 8.1 Binding Chain

The components form an integrity chain through hash references. The Objective (§9), when present, participates in the chain for integrity only:

```
Relationship JWT  (rel_jti, sha256 = rel_hash)
        ▲
        │ Authority JWT references via rel_jti + rel_hash
        │
Authority JWT  (auth_jti, sha256 = auth_hash)
        ▲
        │ Objective (optional) references via rel_jti+rel_hash AND auth_jti+auth_hash
        │ Task JWT references the Objective via objective_jti + objective_hash (intent anchor)
        │
Objective (optional)  (objective_jti, sha256 = objective_hash)   ── inert (§9.2)
        ▲
        │ Task JWT references via rel_jti+rel_hash, auth_jti+auth_hash, objective_jti+objective_hash
        │
Task JWT
```

A verifier presented with a Task JWT MUST:

1. Validate the Task JWT signature using the issuer's published key.
2. Retrieve or accept (as cached) the referenced Relationship JWT and verify `rel_hash` matches its sha-256 digest.
3. Retrieve or accept (as cached) the referenced Authority JWT and verify `auth_hash` matches its sha-256 digest.
4. Verify that the Authority JWT's `rel_jti` and `rel_hash` likewise reference the same Relationship JWT.
5. Validate the signatures on all three JWTs (Relationship, Authority, Task).
6. **Perform the commit-boundary status-signal check on each of the three enforceable-chain components per §8.3.** This is repeated at the moment of action, not only at issuance, and applies symmetrically to Relationship, Authority, and Task — each MAY carry a `rev_endpoint` (or equivalent status reference) and the verifier MUST honor a not-active status on any of them.
7. Evaluate eligibility criteria (Relationship), authority scope (Authority — per the §5.5.1 matching rule: the act's Obligation RAR `type` MUST be admitted by the derivative authority under value-space registered-identifier equality; the free-form `scope_domain` MUST NOT be lexically matched), and constraints (Task) against the runtime context.

If an Objective is present (inline in the Task JWT, or referenced via `objective_jti`/`objective_hash`), the verifier MUST additionally, **for integrity only**:

8. For an Objective the verifier holds or can retrieve, verify `objective_hash` matches the Objective's sha-256 digest (for a standalone `objective+jwt`), validate its signature, and confirm its `rel_jti`/`auth_jti` references resolve to the same Relationship and Authority as the Task JWT. The verifier MUST NOT evaluate the Objective for authorization scope, and MUST NOT permit or deny the act on the basis of the Objective's `action`, `purpose`, or `beneficiary` (§9.2). This step has two distinct failure modes, which the verifier MUST NOT conflate:

   - **Integrity failure** — the Objective is present but its hash, signature, or upstream references do not check out. A corrupted or substituted record of intent is not acceptable, and the delegation MUST NOT be honored.
   - **Audit-availability failure** — a standalone `objective+jwt` cannot be retrieved at all (the referenced artifact is unreachable), so the verifier cannot perform the integrity check. Because the Objective is inert and contributes nothing to the authorization decision (§9.2), an unreachable Objective MUST NOT be treated as an integrity failure: the verifier MUST NOT deny an act that steps 1–7 already permit, and MUST instead record the missing-intent-record condition for audit. (This mode cannot arise for an inline Objective, which travels under the Task JWT signature and is covered by step 1.)

   An Objective that passes integrity contributes nothing to the authorization decision beyond the consent/audit record. The split between a committed, integrity-checked *descriptor* and an *availability/evaluated* condition kept out of the deny decision follows the discipline the Mission-Bound Authorization suite applies in its audit/transparency and completion profiles (§13); the direction here is its mirror image — because the Objective grants no authority, the conservative action on an availability failure is to fail *safe*, not closed, whereas a signal that *removes* authority fails closed.

If any of steps 1–7, or step 8's *integrity* check, fails, the delegation MUST NOT be honored. A step-8 *audit-availability* failure is recorded but is not, by itself, grounds for denial.

Steps 1–5 and 8's integrity checks are cryptographic and may be cached for the lifetime of the JWTs. Step 6 is a liveness check that, by design, may not be cached beyond the bounds the status response itself permits. Step 7 must be re-evaluated whenever the runtime context changes materially. The Objective carries no commit-boundary status signal of its own, because an inert component has no liveness that could change the authorization outcome.

### 8.2 Issuer Identity and Profile-Dependent Signing

The signing model for each JWT is profile-dependent and intentionally left out of scope for the base specification. Deployment profiles MAY require:

- Single-issuer signing (delegator signs all components)
- Multi-issuer signing (Relationship signed by an identity federation, Authority by a credentialing body, Task by the delegator)
- Notary or co-signature requirements for high-assurance Authority instruments

While the concrete signing profile is out of scope, one trust property is not, because it is a correctness requirement rather than a profile choice: **trust anchors are role-scoped.** Under multi-issuer signing a verifier MUST establish a trust anchor for each distinct issuer in the chain, and an issuer trusted to sign one component's role is not thereby trusted to sign another's — an issuer accepted for Relationships is not, by that fact, accepted for Authorities or Task JWTs. A verifier MUST NOT honor a component whose signer it does not trust *for that component's role*, and MUST NOT infer trust in an issuer merely because that issuer is named inside another signed component. The mechanism by which a verifier discovers and configures these per-role anchors is deferred to profiles; the requirement that trust be role-scoped is normative here. The security rationale, and the cross-issuer escalation this rule is meant to prevent, are given in §12.2.

### 8.3 Commit-Boundary Validity

The binding chain (§8.1) establishes that the components are cryptographically tied together at the moment of issuance. But issuance is not the moment of action. Between when the components are issued and the moment a delegated act actually commits at a relying party, any of the following may change:

- The Relationship may be revoked or superseded (an employment ends, a guardianship is transferred, a court order terminates the underlying arrangement).
- The Authority may lapse silently (a credentialing body withdraws a privilege without an explicit revocation event; an organizational delegation of signing authority expires; an inherent self-authority ceases at end-of-life or loss of capacity).
- The Constraints may tighten in flight (a court order narrows a guardian's authority after the Task was issued; a regulatory cap reduces a previously authorized budget; an external trigger event per §7.7 fires; a sanctions list is updated).
- The Obligations may be overtaken by events that render them undefined or already discharged (the underlying request has already been completed by another party; the resource the Obligation targets no longer exists; a precondition the Obligation implicitly relied on has been invalidated).

The architectural principle of this specification is that **each component independently carries (or references) a status signal that the verifier MUST re-check at the commit boundary**, not only at issuance. The runtime authorization decision is not "are these JWTs valid as issued" but "do the four enforceable components, as they stand right now, still authorize this act" (the Objective, being inert, does not enter this decision; §9.2).

Three properties follow from this principle:

**Per-component status signals.** Relationship, Authority, and Task JWTs each MAY carry a `rev_endpoint` claim (or, in deployment profiles that prefer them, an equivalent status reference such as a status list URL or a transparency-service receipt). The status signal is the component's own — the Relationship's `rev_endpoint` is consulted for the Relationship's liveness, not for the Authority's. The cascade rules in §8.4 then compose these per-component signals into the overall delegation's validity. A component without a `rev_endpoint` is treated as having no in-flight liveness signal beyond its own `exp` and the cascade from its upstream components; this is permitted but signals to operators that no run-time state change can independently invalidate that component.

**Commit-time, not issuance-time.** A verifier that checked status only at issuance and cached the result for the lifetime of the cryptographic binding would defeat the purpose of the status signal. Verifiers MUST consult the status signal at the moment of action, subject to the cache bounds the status response itself declares. For high-stakes profiles (life-safety, large-value financial signing, child-welfare safeguarding) the cache bounds are tight; for low-stakes profiles they may be looser. §8.4 specifies the maximum enforcement window.

**Status signals beyond revocation.** Revocation — an explicit, issuer-signed assertion that a component is no longer valid — is the simplest status signal and is specified concretely in §8.4. The principle covers more than revocation. Trigger-based expiry (§7.7) is a status signal embedded in a Constraint that may fire without any action by the issuer. Tightening of Constraints by an external authority (regulatory body, court) is a status signal expressed as a layered overlay rather than as a revocation. Silent lapse of the underlying Authority (an employment ending, a credentialing body withdrawing a privilege) may be expressed either as a revocation event from the issuer or as an out-of-band status reference the verifier consults. Obligation discharge via `terminal_when` (§6.6) is a fourth instance: a status signal embedded in an Obligation item that fires when the task's inherent completion event occurs, independently of any constraint tightening or issuer revocation action. Discharge differs from the other instances in audit semantics — it signals task completion rather than task invalidity — but the verifier's commit-boundary obligation is identical: check the signal, and if it has fired, deny the request.

Among the *obligation-level* signals, only `terminal_when` (§6.6) has a concrete observation mechanism specified in this version. The other obligation-level cases listed earlier in this section (the targeted resource no longer exists, an implicit precondition was invalidated, the act was already completed by another party) are illustrative of the commit-boundary principle rather than fully specified mechanisms; the means by which a verifier observes them is deliberately left to profiles and to the *Execution Layer*, and the principle should not be read as a claim that the base specification already defines how each is surfaced.

The specification accommodates all of these by making the status signal a per-component property and letting profiles choose the concrete mechanism.

The implementation of commit-boundary checks is performed by Permissioned Capabilities — the runtime authorization decision (companion document, *Execution Layer*). The property that a component carries a status signal at all is established by this base specification; the property that the signal is consulted at commit time is established by Permissioned Capabilities. Both layers are needed.

### 8.4 Revocation: Mechanisms, Cascade, and Enforcement Window

Revocation is the canonical instance of the commit-boundary status-signal pattern defined in §8.3: an explicit, issuer-signed assertion that a previously valid component is no longer valid. This section specifies the concrete mechanism and the cascade rules that compose per-component revocation statuses into overall delegation validity.

Relationship, Authority, and Task JWTs each MAY carry a `rev_endpoint` claim referencing a revocation status service. Verifiers consult this endpoint as part of binding-chain validation (§8.1, step 6). The base specification defines three concrete behaviors that any conformant deployment MUST implement.

**Revocation mechanisms.** A revocation endpoint accepts the JWT's `jti` and returns one of the statuses `active`, `suspended`, `revoked`, or `superseded`. The endpoint MUST be authenticated and SHOULD support short-lived signed status tokens (analogous to OAuth 2.0 token introspection responses) to enable verifier caching with bounded staleness.

**Revocation cascade.** Revocation of an upstream component cascades to all components bound to it downstream. Specifically:

- A Task JWT whose `rel_jti` references a revoked Relationship MUST be treated as invalid, regardless of the Task JWT's own status.
- An Authority JWT whose `rel_jti` references a revoked Relationship MUST be treated as invalid.
- Revocation of an Authority JWT cascades to all Task JWTs bound to it via `auth_jti`.
- Revocation does not cascade upward: revoking a Task JWT does not revoke the Relationship or Authority.

Cascade is a consequence of the §8.3 commit-boundary principle, not a separate mechanism. A verifier walking the binding chain at commit time consults each component's status signal in turn; an upstream `revoked` status surfaces as the cause of the denial, and the downstream signals are not relied upon. The cascade rule is binding on verifiers, not optional. Verifiers MUST evaluate the full binding chain at every validation; caching of revocation status is permitted only within the validity window of the cached status response.

**Enforcement window.** When a guardianship, employment, capacity, or other underlying authority status changes, the revocation system MUST propagate the change such that downstream verifiers stop honoring the affected credentials within **24 hours** of the change. Profiles for high-stakes domains (healthcare narcotic administration, financial signing authority, child welfare) MAY mandate shorter enforcement windows, down to **5 minutes** for life-safety-critical scenarios.

The 24-hour window is a maximum, not a target. Issuers and intermediaries SHOULD aim for near-real-time propagation when the underlying authoritative source supports it. Where the authoritative source is itself slow (e.g., a court registry that updates weekly), the enforcement window is bounded by the source's propagation lag plus the revocation system's own cycle time.

The window is also bounded below by the **status-response cache lifetime**. A signed status token (above) that is valid for hours cannot reflect a revocation that occurs mid-window, so a profile that targets a tight enforcement window MUST set status-token TTLs no longer than that window — the 5-minute life-safety target implies status TTLs of at most 5 minutes, with the attendant re-fetch frequency and the scale and availability cost that follows. The enforcement window is therefore a **target conditioned jointly on source-propagation lag and caching policy**, not a guarantee the base specification can make on its own; a deployment that needs the stated window must provision both the propagation path and the cache policy to honor it.

**Revocation reason hiding.** The reason for revocation — placement change, court order modification, capacity assessment, employment termination, age of majority — is not transmitted to relying parties. The revocation status response carries only the status enumeration (`revoked`, `superseded`, etc.) and an optional opaque `revocation_id` for audit correlation. Reasons are recorded only in the audit trail held by the issuing intermediary, accessible to authorized auditors and oversight bodies but not to the platform receiving the credential.

This is a base-specification requirement, not a privacy-preserving-profile feature. Reason-hiding is the default for all profiles, because the reason for revocation is almost always more sensitive than the fact of revocation.

### 8.5 Transfer with Continuity

When authority transfers between parties — a placement change, a guardian succession, a corporate signing authority handover — the digital representation MUST update without creating a protection gap during the transition. Two transfer patterns are defined.

**Pattern 1 — Successor reference (preferred).** The outgoing Relationship JWT, at the moment of revocation, transitions to status `superseded` and the revocation response (per §8.4) carries an additional field `successor_rel_jti` referencing the incoming Relationship JWT. Verifiers receiving a `superseded` status with a successor reference MUST attempt to retrieve and validate the successor Relationship; if validation succeeds, the verifier treats the *successor* as the operative Relationship for the binding chain. The outgoing Relationship is not honored after supersession.

This pattern requires that the successor Relationship be issued and active *before* the outgoing one is superseded. The issuing intermediary is responsible for ensuring this ordering; if the successor is not yet active, supersession MUST be deferred and the outgoing Relationship remains in `active` status.

**Pattern 2 — Overlapping validity windows.** The outgoing Relationship JWT carries a `transition_window` claim defining a period during which both the outgoing and incoming Relationships are simultaneously valid. The outgoing Relationship's `exp` is the end of the transition window. During the window, verifiers may honor either credential. After the window, only the incoming Relationship is valid. This pattern is operationally simpler but creates an interval during which authorization decisions could be made under either authority — appropriate for transfers between cooperating parties (e.g., outgoing and incoming care home managers during a planned handover) but not for adversarial transfers (e.g., emergency removal of authority from a compromised actor).

**Hard transfer (no continuity).** Some transfers are intentionally discontinuous: emergency removal of authority due to safeguarding concerns, court orders barring contact, capacity loss requiring immediate substitution. In these cases, the outgoing Relationship is revoked with status `revoked` (not `superseded`) and no successor reference is provided. A separate Relationship may be issued to a new authorized party but does not inherit the binding chain of the revoked Relationship; bound Authority and Task JWTs are invalidated and must be re-issued under the new Relationship.

**Reason hiding extends to transfers.** A relying party observing a `superseded` status learns only that the credential was replaced; it does not learn whether the cause was scheduled handover, emergency reassignment, or any other underlying reason.

### 8.6 Hash Object Format

All hash references use the format:

```json
{
  "alg": "sha-256",
  "digest": "hex-encoded-digest-string"
}
```

The digest is computed over the entire compact JWS serialization of the referenced JWT (header.payload.signature).

### 8.7 Presentation Binding (Holder Binding and Audience)

The binding chain (§8.1) establishes that the components are cryptographically tied to one another and to their issuers. It does **not**, on its own, establish *who may present* a delegation artifact or *to which relying party*. Nothing in §8.1–§8.6 binds a Task JWT (or any other component) to a holder key or to a target audience: the signatures prove issuance and integrity, not possession. A component that reaches a relying party carrying only these bindings is therefore, as far as this base specification defines it, a **bearer artifact** — anyone who obtains it could in principle present it, at any relying party that accepts its issuer, until it expires.

This is deliberate, but it MUST be stated rather than left implicit. **Presentation binding — proof-of-possession of a holder key (e.g., a `cnf` confirmation key) and audience restriction (e.g., an `aud` naming the intended relying party) — is an execution-layer concern**, defined by Permissioned Capabilities and Protected Access (§11.1, companion *Execution Layer*), not by the delegation components. The delegation layer answers "who may act on whose behalf, for what task, within what bounds"; the execution layer answers "and this specific presenter, holding this key, is that delegatee, acting at this specific resource." The two are separable, and this specification separates them on purpose: the delegation artifacts are designed to be long-lived, cacheable, and independently verifiable, which is at odds with binding each to a single presentation.

Until a profile or the execution layer defines presentation binding for a given deployment, **the delegation components MUST NOT be treated as bearer-safe**: a relying party MUST NOT rely on possession of a component as evidence that the presenter is the authorized delegatee, and a deployment MUST either (a) obtain presentation binding from the execution layer (a holder-bound, audience-restricted Permissioned Capabilities credential derived from the delegation) before honoring an act, or (b) constrain the transport and custody of the components such that capture-and-replay is not in the deployment's threat model (for example, components that never leave a confidential channel between cooperating trusted services). A deployment that does neither, and that treats a delegation artifact as sufficient authority merely because it verifies, has a confused-deputy and replay exposure this specification does not close at the delegation layer.

The mechanism is left to the execution layer rather than mandated here for the same reason the signing model is (§8.2): it is profile-dependent, and committing a specific `cnf`/`aud` shape on the Task JWT now would prematurely couple the durable delegation artifact to a presentation model that properly belongs on the derived, short-lived credential the delegatee actually presents. This mirrors the posture of adjacent work: in the Mission-Bound Authorization suite (§13), the durable governance object is not itself the presented credential; presentation is carried by sender-constrained (proof-of-possession) tokens, with the high-consequence sender-constraint key held by a mediating enforcement point rather than by the agent. The binding lives on the presented credential, not on the governance record — which is the division this section draws between the execution layer and the delegation components.

---

## 9. Objective (Declared Intent Layer)

> **Naming note.** "Objective" is a *provisional* name for this component, adopted in v2.0 pending community input (earlier v2.0 drafts named this component *Charge*; it was renamed to *Objective* on 2026-06-27). Candidate alternatives under consideration are *Commission* and *Charter*. It is deliberately **not** named "Mission" (to avoid collision with the Mission-Bound Authorization for OAuth suite, whose constructs differ from these) or "Intent" (to avoid collision with the "generated intent" of agentic task generation, which refers to the *opposite* end of the lifecycle). The name may change in a later revision; the concept and its normative properties (§9.2) are stable.

### 9.1 Purpose

The enforceable components (Relationship, Authority, Obligations, Constraints) describe *who may act, why they had the right, what is to be done, and within what bounds*. They do not, on their own, capture two facts that delegations routinely carry: the **umbrella act** the delegation exists to accomplish, and the **purpose** for which it exists.

When a delegator says "handle the family renewals this week" or "book my anniversary trip," the umbrella act ("renewals," "the trip") is not itself an Obligation a verifier can check; it is the descriptive umbrella under which specific Obligations are authored, or, in agentic deployments, generated. Because the Objective is inert (§9.2), `action` does not itself *bound* those Obligations: the enforceable bound on what may be authored or generated is the Authority's scope (§5.5) and the Constraints (§7), not the declared `action`. The purpose ("for the anniversary," "for GDPR-compliant marketing analysis") is not a bound; it is the reason the delegation exists at all. The entity *for whom* the act is performed (a dependent whose records are released) may be neither the delegator nor the delegatee.

The **Objective** is the component that carries these declared facts:

- `action` — the umbrella act the delegation is to accomplish. Structured as a `kind` URI (following the namespace convention of relationship types, §4.3) plus an optional human-readable `display`.
- `purpose` — the structured purpose object; its `kind`/`params`/`display` shape and the base purpose-class vocabulary are defined in §9.4.
- `beneficiary` (optional) — the entity for whom the delegated act is performed, where this differs from both delegator and delegatee. Expressed using the identity object of §4.4. `beneficiary` is **descriptive only**: like every Objective field it is inert (§9.2), and it MUST NOT be used to scope *whose* records or resources may be accessed. Where the identity of the data subject narrows what may be touched (release *this* dependent's records, not another's), that scoping is an enforceable Constraint or Obligation concern (§7.3), not a property of the inert Objective. This boundary is load-bearing; see §11.9.

### 9.2 The Inert Property (normative)

The defining property of the Objective is that **it is inert with respect to authority**:

> A verifier MUST NOT use any field of the Objective to derive, widen, or gate the authority of a delegation. Enforceable authority is determined solely by the Relationship, Authority, Obligations, and Constraints. The Objective is committed and rendered for consent and audit; it never decides what is permitted.

Concretely:

- The Objective MUST NOT be consulted to *expand* what the delegatee may do beyond what the enforceable components already permit.
- A request that the enforceable components permit MUST NOT be denied solely because an evaluator judges it to have drifted from the declared `action` or `purpose`.
- A request that the enforceable components forbid MUST NOT be permitted because its declared `purpose` reads as benign.

**Rationale — prompt-injection resistance.** In agentic deployments the agent's inputs are reachable by an attacker (a poisoned document, an injected instruction). If the declared `action` or `purpose` were authority-bearing, an attacker who could rewrite them could talk the delegation into enlarging its own authority. Keeping the Objective inert means injected text has nothing to push on: it can corrupt the *record of intent* (which the integrity check of §8.1 step 8 detects) but it cannot move authority. This mirrors the inert posture of the Mission-Bound Authorization suite's `goal`/`purpose`/`success_criteria` (see §13 References).

The same suite supplies an independent confirmation of this rule at a *different* layer. Its issuance profile keeps declared intent inert at delegation time, as above; its runtime profile, addressing the case where a deployment pairs the authority layer with an inspection layer that scores an agent's intent against its task, constrains any such semantic intent-alignment signal to be **advisory and deny-only** — it MAY contribute to refusing an action but MUST NOT widen, grant, or refresh authority. That two practitioners arrived at "intent never widens authority" at both the issuance boundary and the runtime boundary strengthens the rationale: the inert property is not an artifact of where intent is declared, but a general consequence of intent being attacker-reachable. The Objective takes the conservative position at the layer this specification governs — declared intent is committed and inert; if a deployment additionally scores intent at runtime, that signal too may only deny, never widen.

**Out-of-band consultation is not gating.** Nothing prevents a *resource-side* policy from consulting the audit-recorded `purpose` for the resource's own decisions — for example, a retention regime that keeps order data for 13 months under a marketing purpose and 10 years under a tax purpose. Such consultation is a property of the resource's policy, downstream of and separate from this delegation's authority. It is not the Objective gating the delegation, and it does not make the Objective authority-bearing within the meaning of this section.

### 9.3 Lifecycle and Binding

The Objective is **declared at delegation time** by the delegator (or the delegating authority), on the delegator's clock. A single Objective MAY underwrite many Obligations: in agentic deployments where the delegatee *generates* the specific Obligations within delegator-set bounds, the Objective is the stable declared umbrella under which those generated Obligations sit, while each generated Task JWT carries its own Obligations and Constraints.

The Objective MAY be expressed in either of two forms (the choice is a deployment decision; wire-format guidance will be tightened as deployment experience accrues, per §11.9):

- **Standalone** (`typ: objective+jwt`) — a separate JWT in the binding chain, positioned between Authority and the Task JWT. Used when one declared act underwrites multiple Task JWTs (the agentic generated-Obligations case). A standalone Objective references its Relationship and Authority exactly as the Task JWT does (`rel_jti`/`rel_hash`, `auth_jti`/`auth_hash`); each Task JWT then references the Objective via `objective_jti`/`objective_hash`.
- **Inline** — an `objective` claim within the Task JWT, used for the common single-declared-task case. An inline Objective occupies the role the `obligations.purpose` field held prior to v2.0 and travels under the Task JWT signature.

**Domain-separated integrity anchor.** The hash by which a Task JWT commits to a standalone Objective (`objective_hash`) is the framework's **intent anchor**, deliberately kept distinct from the authority anchor (`auth_hash`). This domain separation — an integrity commitment over the declared intent that is separate from the commitment over the derived authority — follows the dual-anchor pattern (`intent_hash` vs. `authority_hash`) of the Mission-Bound Authorization suite. It lets an auditor verify *what was declared* and *what was authorized* as two independently-committed facts, while the inert property (§9.2) guarantees the former never determines the latter.

**Binding-chain participation** is specified in §8.1 (step 8): the Objective is verified for integrity (hash match, signature, consistent upstream references) and never evaluated for authorization scope.

### 9.4 The Purpose Object

The `purpose` object characterizes the declared intent of a delegation. Like every Objective field it is **inert** (§9.2): a verifier MUST NOT use it to derive, widen, or gate authority. It is audit-and-consent material — what an auditor reads to understand *why* a delegation was authorized, and what a consent interface renders. This section is its canonical definition; it defines the object's shape and the base purpose-class vocabulary.

When present, `purpose` is an optional structured object that MUST contain `kind` and MAY contain `params` and `display`:

- `kind` — a URI identifying the purpose class. Purpose class URIs follow the same namespace convention as relationship types (§4.3) and MAY be registered under the `urn:ietf:params:delegation:purpose:` prefix for common classes, or under deployment-specific namespaces for domain-specific cases. A purpose class URI MAY be dereferenced to retrieve a **purpose catalog entry** describing the expected `params` schema and the meaning of each parameter for consent rendering and audit.

- `params` (optional) — an object carrying purpose-instance parameters, described by the `kind`'s catalog entry. All params are audit-and-consent material; none gate this delegation's authority (§9.2). A resource-side policy MAY consult them out of band for its own decisions, but that is the resource's policy, not this delegation's authority.

- `display` (optional) — a human-readable description for audit trails and consent interfaces. Deployments that issued Task JWTs with a `purpose` string under v1.1 or v1.2 SHOULD treat that string as `{ "display": "<string>" }` with no `kind` or `params`.

**Base purpose class vocabulary.** The following purpose class URIs are pre-registered for common delegation scenarios. The registry is extensible; profiles and deployments MAY register additional classes under their own namespaces, with promotion to the base registry following the same process as relationship types (§4.3).

| Class URI | Meaning | Example params (audit/consent) |
|-----------|---------|-----------------------------------|
| `urn:ietf:params:delegation:purpose:travel-booking` | Travel arrangement on behalf of delegator | `scope` (accommodation / transport / all), `event_type` |
| `urn:ietf:params:delegation:purpose:financial-transaction` | Payment or financial operation | `transaction_class`, `counterparty_type` |
| `urn:ietf:params:delegation:purpose:record-disclosure` | Release of records to a third party | `record_type`, `recipient_context`, `disclosure_event` |
| `urn:ietf:params:delegation:purpose:medical-administration` | Clinical task in a patient care context | `care_domain`, `patient_context` |
| `urn:ietf:params:delegation:purpose:scheduling` | Calendar or appointment management | `scope`, `subject_context` |

**Not a commit-boundary gate.** Because purpose is inert (§9.2), it is not a commit-boundary signal. Commit-boundary validity (§8.3) is decided by the status signals of the enforceable components: Relationship/Authority liveness, Constraint satisfaction and trigger-based expiry (§7.7), and Obligation discharge via `terminal_when` (§6.6). Where a delegation must become inert when its declared purpose no longer holds, the bounding mechanism is an enforceable one — a `terminal_when` on the Obligation (§6.6) or an `expires_on_event` Constraint (§7.7) — not the purpose object. The Maya/Theo record disclosure of §9.6 is the worked example: the release is bounded by the Obligation's `terminal_when`, while the purpose records only *why* it was authorized.

**Relocation and retired posture (v2.0).** Prior to v2.0 the structured `purpose` object was a field of the `obligations` object; as of v2.0 it is a sub-claim of the Objective, and this section is its home. This is a *relocation*, not a redefinition: the `kind`/`params`/`display` structure and the base vocabulary above are unchanged in shape. What changed is the enforcement posture, as a consequence of the inert property (§9.2): the v1.3 distinction between *governance-shaping* and *audit-only* purpose params — and the rule that "ignoring governance-shaping params is equivalent to dropping a constraint" — are **retired**; no purpose param gates the delegation's authority. Deployments that issued Task JWTs with an `obligations.purpose` field under v1.1–v1.4 SHOULD treat that field as an inline Objective `purpose` with no change in wire shape. This reverses the v1.3 direction on purpose-as-governance-shaping and resolves the long-standing purpose-as-runtime-test question in favor of a *structured-but-inert* purpose, on the prompt-injection-resistance grounds of §9.2.

### 9.5 Example: Sam's Household Renewals (standalone Objective)

```json
{
  "iss": "https://wallet.sam.example",
  "iat": 1717200000,
  "exp": 1717804800,
  "jti": "urn:uuid:1f0a7c2e-9b3d-4a51-8e6f-0c2d4b6a8e10",
  "typ": "objective+jwt",
  "rel_jti": "urn:uuid:9d4f8b2c-1e3a-4b5c-8d7e-6f0a1b2c3d4e",
  "rel_hash": { "alg": "sha-256", "digest": "..." },
  "auth_jti": "urn:uuid:b7e2f1a4-3c8d-4e2a-9f1b-2c5d8e0f3a91",
  "auth_hash": { "alg": "sha-256", "digest": "..." },
  "action": {
    "kind": "urn:ietf:params:delegation:action:household-administration",
    "display": "Handle the family renewals this week"
  },
  "purpose": {
    "kind": "urn:ietf:params:delegation:purpose:scheduling",
    "display": "Keep the household's recurring obligations current"
  }
}
```

Sam's assistant generates each specific renewal (DMV registration, property tax, insurance) as its own Task JWT, each referencing this Objective via `objective_jti`/`objective_hash`, each carrying its own Obligations and Constraints. The Objective records *what Sam asked for and why*; the generated Task JWTs carry *what is enforceably authorized*. An injected instruction that rewrote the Objective's `display` to "and also wire $5,000 to this account" would be caught as an integrity break on `objective_hash`; and even undetected it could authorize nothing, because the wire is within no Obligation or Authority the enforceable components grant.

### 9.6 Example: Inline Objective (single declared task)

For Maya's vaccination-record release (§6.6), no umbrella-act layer is needed; the Objective is inline and carries only the purpose:

```json
"objective": {
  "purpose": {
    "kind": "urn:ietf:params:delegation:purpose:record-disclosure",
    "params": {
      "record_type": "vaccination_records",
      "recipient_context": "school_enrollment",
      "disclosure_event": "ENR-2026-04582"
    },
    "display": "Release vaccination records for Theo's school enrollment"
  }
}
```

This is byte-for-byte the object that appeared as `obligations.purpose` prior to v2.0; only its location (under `objective` rather than `obligations`) and its enforcement posture (inert, §9.2) have changed. The bounded-in-time behavior of the release is enforced by the Obligation's `terminal_when` (§6.6), not by this purpose.

---

## 10. End-to-End Scenarios

### 10.1 Scenario A: AI Travel Agent

**Setup**: Bob, a natural person, wants to delegate hotel booking for an upcoming conference to an AI travel agent.

**Component instances**:

1. **Relationship JWT** — Issued by Bob's wallet, type `service-client`, delegator is Bob (OIDC subject), delegatee is the travel agent (DID), eligibility constrained to Bob's authorized agent fleet by policy reference.

2. **Authority JWT** — Issued by Bob's wallet under the agentic profile. No source authority (Bob has inherent self-authority). Derivative authority's `scope_domain` is `https://schemas.example.com/scope/travel`, `further_delegation` allows one-level sub-delegation to booking and payment specialists.

3. **Task JWT** — Issued when Bob initiates the trip planning. Carries an inline Objective (§9.6) whose `purpose` is class `urn:ietf:params:delegation:purpose:travel-booking` with params `scope: "accommodation"` and `event_type: "conference"`; the purpose is inert (§9.2), recorded for consent and audit, and does not gate the booking. Single obligation of type `hotel_booking` with Boston, May 12–15, one adult, one room. Constraints: $400/night max, $1,200 total max, walking distance to convention center, allowed brands (Marriott, Hilton, Hyatt), validity expires May 10. All constraints sourced from the delegator.

**Flow** (the delegation-layer view; the "presents the Task JWT" step is shown at the delegation layer for clarity — in a deployment where capture-and-replay is in scope, the agent would present a holder-bound, audience-restricted Permissioned Capabilities credential derived from this delegation rather than the Task JWT itself, per §8.7):
1. The travel agent presents the Task JWT to a hotel booking API.
2. The API verifies the Task JWT's signature, retrieves the Relationship and Authority JWTs (cached or via referenced URIs), validates the binding chain.
3. The API confirms the travel agent's identity is permitted under the Relationship's eligibility.
4. The API confirms the obligation's RAR `type` (`hotel_booking`) is admitted by the derivative authority under value-space equality (§5.5.1); the `scope_domain` (`.../scope/travel`) is treated as an advisory category label, not matched lexically.
5. The API evaluates the obligation and constraints against candidate hotels and presents acceptable options or a confirmed booking.

**Validation of the model**: All four enforceable components are exercised, with the declared purpose carried inertly in an inline Objective (§9). Each plays a distinct role. The constraint surface is small but layered (and could trivially absorb a regulatory constraint, e.g., OFAC, without modifying the delegator's intent).

### 10.2 Scenario B: Hospital Narcotic Administration

**Setup**: Dr. Jane Smith is on duty and needs an on-call nurse to administer pain medication to a post-operative patient.

**Component instances**:

1. **Relationship JWT** — Issued by the hospital's credentialing authority. Delegator is Dr. Smith. Delegatee is a *policy-defined group* of nurses who hold (a) DEA Schedule II registration and (b) the on-call-nurse role. Group membership is evaluated at run-time via Cedar policy.

2. **Authority JWT** — High-assurance profile. Verified-claims envelope with hospital trust framework. Source authority asserts Dr. Smith's status as Attending Physician with Schedule II prescribing privileges, granted by appointment from the Medical Staff Credentialing Committee. Derivative authority restricts scope to narcotic administration with no further delegation permitted.

3. **Task JWT** — Issued by the hospital's order entry system at the time the order is written. Single obligation of type `medication_administration` with patient MRN, drug code, dose, route, frequency. Constraints layered from three sources:
   - **Delegator**: temporal validity (24 hours), max 6 administrations, only-if pain score ≥ 4
   - **Platform**: deny concurrent benzodiazepine (FDA black-box warning)
   - **Regulatory**: HIPAA minimum-necessary data fields permitted

**Flow** (as in Scenario A, the Task JWT is shown presented directly for clarity; a deployment in which the credential could be captured in transit would present a holder-bound, audience-restricted execution-layer credential derived from this delegation, per §8.7):
1. The on-call nurse, when ready to administer, presents the Task JWT to the medication dispensing system.
2. The system validates the binding chain.
3. The system evaluates the Relationship's eligibility policy: is this nurse currently on-call AND credentialed for Schedule II? Cedar evaluation says yes.
4. The system confirms the obligation's RAR `type` (`medication_administration`) is admitted by the derivative authority under value-space equality (§5.5.1); the narcotic-administration `scope_domain` is an advisory label, not a lexical match.
5. The system evaluates the layered constraints. The current pain score is 6 (≥ 4, satisfies delegator's conditional). No concurrent benzodiazepines on the patient's record (satisfies platform). Administration is logged with only the permitted data fields (satisfies regulatory).
6. The dispensing system releases the medication and records the administration. The Task JWT's `count` constraint is decremented.

**Validation of the model**: This scenario exercises high-assurance features (verified claims, full source authority, policy-defined groups, layered constraints with provenance) and shows how multiple authorities contribute to a single delegation without conflict. The constraint provenance is essential here — if the dispensing fails, knowing whether it was the delegator's pain-score rule, the platform's drug-interaction rule, or the regulatory data rule that failed is critical for clinical workflow.

### 10.3 Scenario C: Corporate Director Delegation

**Setup**: A company director (Bob Smith) delegates expense approval up to £5,000 to the head of finance for the duration of the director's two-week vacation.

**Component instances**:

1. **Relationship JWT** — Issued by the company's identity provider. Delegator is Bob Smith (OIDC). Delegatee is the head-of-finance role (role-based group, currently held by a single named individual). Eligibility includes credential constraint (must hold valid corporate identity at LOA2+).

2. **Authority JWT** — High-assurance profile. Verified-claims envelope with the corporate trust framework. Source authority is the OpenID Authority example structure: applies_to is the company (with LEI, registration number, jurisdiction), permission is `Director` with `may_delegate: true`, granted_by is `appointed` by Companies House. Derivative authority's scope domain is `https://example-co.example/scope/expense-approval`, no further delegation.

3. **Task JWT** — Issued by Bob at the start of vacation. Single obligation of type `expense_approval`. Constraints: temporal (specific two-week window), monetary (max £5,000 per approval), conditional (only-if expense category is in {travel, vendor, equipment}, except-when total exceeds £15,000 for the period — in which case approval reverts to Bob via remote process).

**Validation of the model**: This shows the OpenID Authority pattern fully expressed, with real GLEIF/Companies House identifiers, and demonstrates how the existing OpenID work integrates seamlessly into our broader delegation framework.

---

## 11. Open Questions and Future Work

### 11.1 Execution Layer (Permissioned Capabilities and Protected Access)

The following are deferred to a subsequent draft:

- **Permissioned Capabilities** — How the abstract delegation translates into operational IAM permissions for the delegatee workload. Likely uses RFC 8693 token exchange to obtain capability tokens scoped by the Authority's admitted types (§5.5.1) and bounded by the Task's Obligations and Constraints. This is also the natural home for a profile-level enumeration of admitted types for own exercise, should a PDP need to gate an action by type ahead of a formed Obligation (§5.5.1).
- **Protected Access** — Credentials needed to access protected personal data (e.g., loyalty accounts, medical records, financial accounts) on behalf of the delegator. Potentially modeled as wrapped OAuth access tokens or capability-based credentials.
- **Presentation binding** — Holder binding (proof-of-possession, e.g. a `cnf` key) and audience restriction (e.g. `aud`) for the credential the delegatee actually presents. §8.7 states the binding *model* (presentation binding is an execution-layer concern, and the delegation components are not bearer-safe until it is provided); the concrete mechanism — likely a `cnf`/`aud`-bearing Permissioned Capabilities credential derived from the delegation via RFC 8693 token exchange — is defined here.

The Permissions Dashboard concept identified during stress-testing (a regulated-intermediary-held audit and authorization-status dashboard accessible to delegators, oversight bodies, and the data subject in age-appropriate form) is a strong candidate use case for the execution-layer design.

### 11.2 Signing Models and Trust Frameworks

The base specification leaves the concrete signing *model* out of scope. Standardization or recommendation of common signing profiles (single-issuer, multi-issuer, notarized) would aid interoperability. The Privacy-Preserving Profile companion document defines an intermediary-as-issuer pattern that addresses this for vulnerable-population deployments. What is now *in* scope, as of v2.3, is the trust-anchor *requirement* that any multi-issuer profile must satisfy: trust anchors are role-scoped and per-issuer (§8.2, §12.2). The open work here is therefore narrower than before — the *discovery and configuration mechanism* for those anchors (e.g., federation metadata, an issuer-role registry), not the requirement itself.

### 11.3 Constraint Conflict Resolution

When constraints from multiple sources conflict (e.g., delegator allows a vendor that the platform denies), the base specification adopts **deny-overrides-allow** as the default resolution rule, with the following precedence ordering when conflicts arise: judicial > regulatory > platform > authority > relationship > delegator. A judicial constraint always wins over a delegator constraint regardless of category. Profiles MAY define more nuanced override semantics; the base rule is conservative and predictable.

### 11.4 Chain-of-Authority Reconstruction

The current model supports summary-level chain references via `prior_authority_ref` but does not specify how to walk a multi-step chain end-to-end. For deep delegation chains (court → executor → executor's law firm → individual lawyer), an explicit chain-walking protocol may be needed. The intermediary pattern introduced in the Privacy-Preserving Profile suggests that chain-walking is the *intermediary's* responsibility, with the relying party seeing only the flattened presentation; this needs to be formalized.

### 11.5 Cross-Profile Interaction

When a delegation chain crosses profile boundaries (e.g., a high-assurance medical authority delegates to an agentic AI workflow), the rules for downgrading or upgrading assurance are not yet specified. The general principle is that downstream assurance cannot exceed upstream assurance, but the precise semantics — particularly when a privacy-preserving-profile credential is presented in a context that does not require privacy preservation — need explicit treatment.

### 11.6 Multi-Decision-Tier Quorum

Section 4.9 introduces role-tuple groups for consensus decisions. Extending this to support **multi-decision-tier quorum** (where routine decisions require one role, significant decisions require three, and strategic decisions require all named roles) is left to a future revision. The current Tier 2 policy_ref escape hatch can express this for deployments that need it.

### 11.7 Disputes, Appeals, and Override Mechanisms

Real-world delegations occasionally need to be disputed or appealed by the delegatee or by an affected third party (e.g., the data subject in a guardianship case). The base specification does not yet define a dispute protocol. This is closely related to the Permissioned Capabilities and Protected Access work, since disputes typically operate at the execution layer where capabilities are exercised.

### 11.8 Cross-Border Recognition

A delegation established under one jurisdiction's legal framework may need to be honored in another jurisdiction. The framework supports this in principle (the Relationship and Authority can be presented across jurisdictions), but the legal recognition rules are out of scope. Trust framework operators are expected to handle cross-border recognition through bilateral or multilateral agreements.

### 11.9 Objective: Remaining Sub-Questions

The Objective component (§9) is newly introduced in v2.0 and its concept and inert property are settled, but several details are deliberately left open pending community input and deployment experience:

- **Final name.** "Objective" is provisional (§9 naming note). *Commission* and *Charter* are the leading alternatives; the choice will be made after the LinkedIn community discussion and any IETF/OIDF feedback.
- **`action` class vocabulary.** §9.1 defines `action.kind` as a URI following the §4.3 namespace convention, but no base `action` class registry is yet defined (unlike the base purpose-class vocabulary of §9.4). Whether the umbrella act needs a registry, or whether free-form `display` plus a deployment-specific `kind` suffices, is open.
- **Beneficiary modeling (load-bearing — descriptive vs. scoping).** This is more than a detail; it is a correctness question about whether the inert layer stays inert. If `beneficiary` is purely *descriptive* (a record of for-whom, rendered for consent and retained for audit), it belongs in the Objective and stays inert. But if a deployment ever lets `beneficiary` *scope* whose records or resources may be touched (release *this* dependent's vaccination records and not another's), it becomes enforcement-relevant and **cannot** be inert — placing it in the Objective would re-import the exact authority-bearing-intent trap the inert property (§9.2) exists to close, the same trap `purpose` was just rescued from (§9.4). The framework's current position is that `beneficiary` is descriptive only (§9.1); any scoping role MUST be expressed as an enforceable Constraint or Obligation (§7.3). Whether the beneficiary additionally needs richer *descriptive* structure (role relative to the act, data-subject status, age-appropriate handling under the Privacy-Preserving Profile) is the secondary, non-blocking part of this question. Surfaced by K. McGuinness's review of v2.0.
- **Inline vs. standalone guidance.** §9.3 permits both forms; concrete guidance on when a standalone `objective+jwt` is required (e.g., a normative threshold tied to delegatee-generated Obligations) versus when inline suffices is deferred until generated-Obligations deployment patterns are clearer.
- **Execution-layer interaction.** How Permissioned Capabilities surfaces the inert Objective in consent and audit rendering, and how the `objective_hash` intent anchor composes with Action Receipts and any transparency-service substrate, is execution-layer work (companion document).

---

## 12. Security Considerations

Earlier sections state trust assumptions where they arise — the inert-Objective injection rationale (§9.2), reason-hiding (§8.4), the integrity-vs-availability split (§8.1 step 8), presentation binding (§8.7), commit-boundary re-checking (§8.3). This section consolidates them into a single trust model: the components that must be trusted and to what degree, the assumptions that hold across the whole system, what the model does and does not guarantee, and the adversary moves it is designed to bound. It introduces two normative rules not stated elsewhere — role-scoped multi-issuer trust anchors (§12.2) and commit-time source integrity with a fail-closed availability default (§12.3) — and otherwise points at the mechanisms the referenced sections define normatively.

### 12.1 The Trusted Base

For each role: what it must achieve, what it assumes of the others, and how its compromise degrades the guarantees. Not every role is present in every deployment; the regulated intermediary and the transparency substrate are profile-dependent, and the delegatee's trust level is set by profile (§2.5).

**Relationship issuer, Authority issuer, Task issuer (delegator).** Each signs the component of its role and is the root of trust for that component. Each must issue faithfully and protect its signing key; each assumes the others are authenticated and role-scoped (§12.2). The compromise of an issuer's signing key lets an attacker forge components *of that issuer's role only* — a compromised Relationship issuer can assert eligibility it should not, but cannot thereby widen Authority scope, because trust anchors are role-scoped (§12.2) and the enforceable bound on an act is the conjunction of all components (§8.3). This role-scoping is what bounds the blast radius of a single key compromise; it is the security purpose of decomposition.

**Regulated intermediary (profile-dependent).** In the Privacy-Preserving Profile and the Permissions-Dashboard deployments, an intermediary issues or re-wraps components, holds the audit trail, and performs reason-hiding (§8.4) and cardinality hiding (Privacy-Preserving Profile R1). It must render faithfully what it holds and disclose reasons only to authorized auditors. Its compromise exposes the very material the profile exists to protect — the reasons behind status changes and the subject's relationship graph — and is the single most consequential compromise in a privacy-preserving deployment.

**Verifier / Policy Decision Point.** Evaluates the binding chain and the enforceable components at the commit boundary and renders permit or deny. It must fail closed (§12.3) and must re-check status at commit time, not rely on issuance-time validity (§8.3). It assumes the inputs it is handed are authentic and the sources it consults are integrity-protected (§12.3). A compromised verifier can honor an act the components deny; the framework does not prevent this, and receipts (execution layer) make it detectable after the fact, not in the moment.

**Commit-time status sources — revocation endpoint (§8.4), policy-group evaluation endpoint (§4.5), event source (§7.7).** Each reports, at the moment of action, a fact the authorization decision turns on: whether a component is revoked, whether the presenter is in the eligible group, whether a terminating event has fired. Each must be authenticated and integrity-protected (§12.3) and must report within its declared freshness bound. A compromised or spoofed source can report a false fact — `active` for a revoked component, membership for a non-member, "not fired" for a fired event — and the fail-closed rule (§12.3) bounds only *unavailability*, not a source that is trusted and lying. An integrity-protected but compromised source is a trusted-base residual.

**Transparency service / receipts substrate (execution layer, profile-dependent).** Where Action Receipts and a transparency service are deployed (companion *Execution Layer*), the substrate makes the delegation's evidence tamper-evident. It must be append-only and non-equivocating. Its compromise degrades after-the-fact accountability, not the run-time decision; a single service equivocates only per-service, so a deployment needing that property registers with more than one.

**Delegatee (profile-dependent trust).** Under the agentic profile the delegatee (an AI agent) is assumed compromisable and prompt-injectable, and every guarantee below is stated relative to that; under the high-assurance profile the delegatee is a credentialed party whose authentication is established out of band. The framework's guarantees against a compromised delegatee rest on two structural facts: the enforceable authority is fixed by the issuers, not by the delegatee (§8.3), and the declared intent the delegatee can influence is inert (§9.2).

### 12.2 Role-Scoped Multi-Issuer Trust Anchors

§8.2 states the normative rule; this is its rationale. Under multi-issuer signing the components of one delegation are signed by different parties — a Relationship by an identity federation, an Authority by a credentialing body, a Task by the delegator. A verifier therefore holds trust anchors for several distinct issuers per delegation, and the security of the whole decomposition depends on those anchors being **role-scoped**: an issuer trusted to sign Relationships MUST NOT thereby be trusted to sign Authorities or Task JWTs.

The attack this prevents is **cross-issuer scope escalation**: if a verifier would honor an Authority-scoping claim signed by whichever issuer signed the Relationship, then the Relationship issuer — who is only supposed to establish *who* may act and *when* — could enlarge *what* may be done, which is the Authority issuer's role. Role-scoped anchors close the general case: a claim that moves Authority scope is honored only if signed by an issuer the verifier trusts *for Authority*.

One concrete path in the current specification still violates this rule and is **not** closed by v2.3: the Relationship-side `scope_override` in the graduated-authority Pattern B (§4.8) lets a stage schedule carried in the Relationship override the Authority's `scope_domain`, which under multi-issuer signing lets the Relationship issuer override the Authority issuer's scope. This is a known open item, tracked as a structural decision (it changes a published component behavior and so is deferred pending an explicit design choice); §12.2's rule is the standard against which it will be fixed. Until it is, a deployment using Pattern B `scope_override` under multi-issuer signing MUST treat the Relationship and Authority issuers as a single trust domain, or MUST NOT use `scope_override` to widen Authority scope.

The mechanism by which per-role anchors are discovered and configured — federation metadata, an issuer-role registry, static configuration — is deferred to profiles (§11.2).

### 12.3 Commit-Time Source Integrity and the Fail-Closed Rule

The commit-boundary principle (§8.3) requires the verifier to consult, at the moment of action, one or more external sources for facts the decision turns on. Those sources are attack surface, and the specification treats them uniformly:

**Integrity.** Every commit-time external status source MUST be authenticated and its responses integrity-protected. v2.3 closes an asymmetry in which §8.4 required this of the revocation endpoint but §4.5 (the policy-group `evaluation_endpoint`) and §7.7 (the `event_source.evaluation_endpoint`) did not; all three now carry the requirement. A source whose response cannot be authenticated is treated as unavailable.

**Availability — the fail-closed rule.** The behavior on an *unavailable* source depends on what the source contributes to the decision:

- A source that is **authority-conferring or authority-removing** — the revocation endpoint (removes), the policy-group evaluation endpoint (confers membership, hence applicability), the event source (removes on a terminating event) — **fails closed** when it cannot be reached or validated within the deployment's freshness bound: the verifier MUST deny the act rather than proceed on a stale or unverifiable answer. It MUST NOT read the absence of a reachable answer as the answer it would prefer (membership present, revocation absent, event not fired).
- A source that is **authority-irrelevant** — a standalone inert Objective (§8.1 step 8) — **fails safe**: its unavailability is recorded for audit but does not deny an act the enforceable components already permit, precisely because it confers and removes no authority.

The dividing line is whether the source can change the authorization outcome. This unifies the rulings the specification already makes: the standalone-Objective availability ruling (§8.1 step 8) and the trigger and revocation availability behavior (§7.7, §8.4) are the two sides of this one rule. The consequence is deliberate: because authority-bearing sources fail closed, an attacker who degrades a status source, policy endpoint, or event source converts the attack into denial of service — work stoppage — rather than into unauthorized action. A deployment provisions the availability of these sources accordingly, and where a hard stop on a transient outage is unacceptable for a specific trigger, the softening is the bounded `grace_period_then_revocation` of §7.7, never failing open.

### 12.4 Cross-Cutting Assumptions

Four assumptions hold across the whole model:

- **Cryptographic soundness.** The signature and hash primitives are sound and issuer signing keys are protected by their holders. A forged signature or a broken hash voids the guarantees of the affected component.
- **Fail closed on authority-bearing uncertainty; fail safe on inert.** Wherever a verifier cannot establish a fact that could change the authorization outcome, it denies (§12.3). The single deliberate exception is the inert Objective, which fails safe because it cannot change the outcome.
- **Inert intent does not move authority.** The declared `action`, `purpose`, and `beneficiary` (§9.2), and any other field an attacker-reachable delegatee can influence, are inert: they can be corrupted (the §8.1-step-8 integrity check detects tampering) but cannot derive, widen, or gate authority.
- **Trust is role-scoped.** Per §12.2, a verifier trusts each issuer only for the component role it is anchored for, and never infers trust from a name appearing inside another signed component.

### 12.5 What the Model Does and Does Not Guarantee

Given an intact trusted base, the model guarantees that an act is bounded by the enforceable components — Relationship eligibility, Authority scope, Obligations, and Constraints — **as they stand at the commit boundary**, not merely as issued (§8.3); that a compromised or injected delegatee cannot widen authority by influencing inert intent (§9.2); that a revocation, transfer, or terminating event propagates to the commit boundary within the enforcement window (§8.4, §7.7); that the *reason* for a status change is not disclosed to relying parties (§8.4); and that the delegation is auditable end to end, including multi-party constraint authorship (§7.5) and, under the Privacy-Preserving Profile, without exposing the subject's relationship graph.

It does not guarantee the following, and a deployment owns each residual:

- **Bearer capture-and-replay before presentation binding.** Until the execution layer binds presentation (holder key + audience, §8.7), a captured component is replayable at any relying party that trusts its issuer. The delegation layer does not close this; a deployment either derives a holder-bound execution-layer credential or keeps capture out of its threat model by construction (§8.7).
- **Cross-issuer scope escalation via `scope_override`.** The role-scoped-anchor rule (§12.2) closes the general case, but the specific Pattern B `scope_override` path (§4.8) remains open pending a structural fix; until then it carries the constraint stated in §12.2.
- **Semantic correctness of the act.** The framework bounds *what* is authorized, not whether the delegatee does the right thing within those bounds. Grounding and intent verification are out of scope; pair with a grounding layer where needed.
- **Compromise of a trusted-base component.** A compromised issuer forges within its role; a compromised intermediary exposes reasons and correlation; a compromised verifier can ignore a deny; a trusted-but-lying status source reports false facts. These are not prevented; receipts make some detectable after the fact.
- **Availability.** Because authority-bearing sources fail closed (§12.3), degrading them yields denial of service, not privilege escalation — a trade the model makes on purpose for the regulated and safeguarding domains it targets.

### 12.6 Adversary Model and Coverage

The adversary is assumed to control a delegatee (agentic profile), to reach the content and inputs the delegatee processes (prompt injection), and to capture components in transit. The adversary is assumed not to break the cryptographic primitives, not to forge a trusted issuer's signing key, and not to compromise a trusted-base component; those last two are the trusted-base residuals of §12.1, not moves this table covers. The residual column is what the deploying party still owns.

| Adversary move | Addressed by | Residual: not stopped |
|---|---|---|
| Compromised/injected delegatee acts beyond the task | Enforceable components re-checked at the commit boundary (§8.3) | Misuse *within* the granted scope |
| Prompt injection tries to widen authority via declared intent | Inert Objective — `action`/`purpose`/`beneficiary` never derive, widen, or gate authority (§9.2) | Injected text can still drive acts already in scope |
| Component captured and replayed at another relying party (confused deputy) | **Not closed at the delegation layer** — presentation binding is an execution-layer concern (§8.7) | Full replay exposure until the execution layer binds holder + audience; a deployment residual |
| Stale authority used after revocation or transfer | Commit-boundary re-check + revocation cascade (§8.3, §8.4) | A window up to the enforcement window / status-cache TTL (§8.4) |
| Terminating event fired, credential still used | Trigger check at commit time, fail-closed on an unreachable event source (§7.7, §12.3) | A bounded `grace_period` window where configured (§7.7) |
| MITM, suppression, or spoof of a commit-time source | Source authentication + integrity + fail-closed availability (§12.3) | A source that is *trusted and lying* (compromised, not merely MITM'd) — a trusted-base residual |
| Relationship issuer enlarges Authority scope (cross-issuer escalation) | Role-scoped trust anchors (§8.2, §12.2) | The Pattern B `scope_override` path (§4.8) is a currently-open instance, not yet fixed |
| Disclosure of the *reason* for a status change | Reason-hiding is the default for all profiles (§8.4) | The intermediary that holds the reason is trusted (§12.1) |
| Disclosure of the subject's relationship existence or graph | Privacy-Preserving Profile layering and cardinality hiding (R1) | Correlation by a colluding set of relying parties (Privacy-Preserving Profile threat model) |
| Compromise of one issuer's signing key | Role-scoped anchors bound the forgery to that issuer's role (§12.2) | Within-role forgery until the key is rotated or revoked |
| Delegatee fabricates results or acts on false data | Not addressed | Full — semantic/grounding verification is out of scope |
| A trusted-base component is compromised | Not prevented; receipts detect some cases after the fact | Degrades the specific guarantee named in §12.1 |

Three residuals are worth naming on their own, because they are the limits most easily overstated away: **bearer replay** until the execution layer binds presentation (§8.7); the **`scope_override`** cross-issuer path (§4.8), open pending a structural decision; and **availability**, which the fail-closed rule converts attacks into, by design.

---

## 13. References

### 13.1 Normative References

- **RFC 7519** — Jones, M., Bradley, J., and N. Sakimura, "JSON Web Token (JWT)", May 2015.
- **RFC 8693** — Jones, M., Nadalin, A., Campbell, B., Bradley, J., and C. Mortimore, "OAuth 2.0 Token Exchange", January 2020.
- **RFC 9396** — Lodderstedt, T., Richer, J., and B. Campbell, "OAuth 2.0 Rich Authorization Requests", May 2023.
- **OpenID Connect Core 1.0** — Sakimura, N., Bradley, J., Jones, M., de Medeiros, B., and C. Mortimore, November 2014.

### 13.2 Informative References

- **OpenID Connect Authority Claims Extension** *(working draft)* — Haine, M. and J. White, eKYC & IDA Working Group, 2026. https://openid.bitbucket.io/ekyc/openid-authority.html
- **draft-mcguinness-oauth-actor-profile** — McGuinness, K., "OAuth 2.0 Actor Profile", IETF Internet-Draft.
- **draft-mcguinness-oauth-mission** — McGuinness, K., "Mission-Bound Authorization for OAuth 2.0", IETF Internet-Draft (suite: issuance core plus companion profiles). Its inert `goal`/`purpose`/`success_criteria` posture and its dual `intent_hash`/`authority_hash` integrity-anchor pattern informed the v2.0 Objective (§9.2, §9.3).
- **draft-mcguinness-oauth-mission-completion** — McGuinness, K., "Mission Completion for OAuth 2.0", IETF Internet-Draft (OPTIONAL companion in the Mission suite). Commits the completion *descriptor* to the authority anchor while keeping *fired status* (evaluated state) out of it; this descriptor-vs-evaluated-state discipline is the pattern §8.1 step 8 mirrors for the inert Objective's integrity-vs-availability split.
- **draft-aap-oauth-profile-01** — "AI Agent Profile for OAuth 2.0", IETF Internet-Draft.
- **draft-mora-oauth-entity-profiles-01** — Mora, F., "OAuth 2.0 Entity Profiles", IETF Internet-Draft.
- **draft-hardt-aauth-protocol-01** — Hardt, D., "Agent Authorization Protocol", IETF Internet-Draft.
- **ISO 17442-1:2020** — Financial services — Legal entity identifier (LEI).
- **ISO 5009** — Financial services — Official organizational roles.
- **GLEIF Registration Authorities List** — Global Legal Entity Identifier Foundation.
- **Cedar Policy Language** — https://www.cedarpolicy.com/
- **Open Policy Agent (Rego)** — https://www.openpolicyagent.org/
- **SPIFFE / SVID** — https://spiffe.io/

---

*End of working draft.*
