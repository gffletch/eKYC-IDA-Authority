# Stress Test: Child Guardianship & The Delegated Authorization Reference Architecture

*Evaluating Components 1–4 of the Reference Architecture against the OIDF Delegated Authority Policy Framework for Child Rights-Respecting Digital Environments (O'Connell & Curtis, April 2026).*

---

## 1\. Why This Document Is the Right Stress Test

The O'Connell & Curtis framework is the hardest test case our architecture can be put through. It combines, in a single domain, every dimension that strains a delegation model:

- **Multi-party authority chains that change frequently** — court → state → local authority → care home → designated key worker, with handovers happening on placement changes, court order modifications, reunification, and age transitions  
- **Polymorphic delegators and delegatees** — natural persons (parents, carers), legal entities (local authorities, hospitals), institutional roles (Director of Social Welfare Standards, IRO), and groups (on-call key workers)  
- **Privacy as a hard architectural constraint, not a soft preference** — the entire framework hinges on a relying-party (platform) being able to receive a boolean authorization without inferring care status, cardinality, role, or legal basis. This is closer to a zero-knowledge requirement than to standard data-minimization  
- **Layered, multi-author constraints** — the Care Plan layers in court-ordered restrictions, social work decisions, statutory requirements (GDPR Art. 8, COPPA, AMLD), platform safety rules, and the child's own evolving capacity  
- **Hard temporal/state-change semantics** — placement transitions, age-of-majority transitions (revocation event), court order amendments, statutory review cycles  
- **Cross-regulatory coordination** — the same delegation must satisfy data protection, online safety, financial crime, social care, and AI safety regulators simultaneously  
- **An explicit legal mandate that the framework already articulates** — the document itself names *six core components* of an authority chain (Section 4.2 of their framework) that map onto our six-component model

If our architecture handles this, it handles essentially everything else.

---

## 2\. What Holds Up Strongly

### 2.1 The four-component decomposition itself

The O'Connell & Curtis framework independently arrives at a remarkably similar decomposition. Their Section 4.2 lists six core components: authority claim, verification against authoritative sources, representation of authority chains, time-boundedness and review, revocation and transfer, and consent dashboard / audit trail. The mapping to our architecture is direct:

| O'Connell & Curtis component | Our architecture component |
| :---- | :---- |
| Authority claim | Component 2 — Authority (derivative authority section) |
| Verification against authoritative sources | Component 2 — Authority (source authority \+ verified\_claims envelope) |
| Representation of authority chains | Component 1 — Relationship \+ Component 2 — Authority (`prior_authority_ref`) |
| Time-boundedness and review | Component 1 — Relationship (`exp`, lifecycle); Component 4 — Constraints (temporal predicates) |
| Revocation and transfer | Component 1 — Relationship (`rev_endpoint`, `rel_status`) — but with notable gaps; see §3.3 |
| Consent dashboard and audit trail | Out of scope for Components 1–4; flagged for execution layer (Components 5–6) |

This convergence is encouraging. It suggests our componentization is not arbitrary but reflects something structural about how delegation actually decomposes in real-world authority chains.

### 2.2 Layered scoping (Authority → Obligations → Constraints)

The framework's distinction between the Care Plan as the **authoritative source of digital permissions** (broad domain) and the specific permissions encoded into a delegated authority credential (narrow scope) maps cleanly onto our coarse-Authority-to-fine-Obligations layering.

For example, the residential care manager's authority over a 14-year-old's digital life is a **scope domain** (`https://cityhospital.example/scope/digital-guardianship/<placement_id>`), expressed in Component 2\. The specific operational decisions — "approve TikTok for educational research only, daily 30-minute window, ages-13-15 peer matching, no monetization features" — are Obligations \+ Constraints in our Component 3/4. The Care Plan itself is the underlying policy document referenced via Tier 2 `policy_ref`.

This is exactly the layering our Reference Architecture proposed.

### 2.3 Polymorphic identity and groups

The framework requires representing:

- A natural-person foster carer (OIDC `sub` \+ `iss`)  
- A care home manager *as the role* rather than as an individual (organizational role identity)  
- A "group of on-call key workers credentialed for digital guardianship" (policy-defined group, exactly our Section 4.5 example)  
- A corporate parent (legal entity, GLEIF/equivalent identifier)  
- An AI agent acting under an "AI Intention Mandate" (workload, SPIFFE/DID)

Our polymorphic identity model with `fmt` discriminator and three group types (static, role-based, policy-defined) handles all of these without modification. The `min_members` quorum field handles the framework's two-key-worker scenarios (where statutory practice requires consensus among multiple authorized adults for Tier 3 strategic decisions).

### 2.4 Constraint provenance

The framework explicitly identifies multi-author constraint layering — court orders, social workers, platforms, regulators (GDPR Art. 8, COPPA, AMLD), and the Independent Reviewing Officer all contribute. Our Component 4 `source` field (delegator | authority | relationship | platform | regulatory) was designed for exactly this. The framework adds one source class we should consider: **judicial** (court-ordered constraints, distinct from regulatory). This is a small extension, not a structural change.

### 2.5 Two-tier constraint expression

The framework's mix of operational-bright-line rules ("no school uniforms in photos during contact visits") and policy-engine evaluations (Cedar-style policies determining who counts as "authorized key worker on shift") is exactly what our Tier 1 / Tier 2 constraint architecture supports. The framework would not work with predicates alone or policy-language-only — it needs both, exactly as our model provides.

### 2.6 RFC 9396 RAR-aligned obligations

Concrete Obligations from the framework — "approve a new platform," "grant data processing consent under GDPR Art. 8," "authorize a payment from a minor's account," "configure age-band matchmaking" — fit cleanly as RAR types. The framework's Tier 1/Tier 2/Tier 3 decision classification (routine / significant / strategic) maps to required-assurance metadata on the Obligation, not to its structural shape.

---

## 3\. Where the Stress Test Reveals Genuine Gaps

The framework forces us to take seriously several issues that our Reference Architecture either handled too lightly or did not handle at all.

### 3.1 Privacy as a binding architectural constraint, not a property of good implementation

This is the most important gap. Our Reference Architecture treats privacy as something that good deployments will achieve through careful field selection. The O'Connell & Curtis framework treats it as a **non-negotiable architectural constraint** with six binding technical requirements (R1–R6 in their Section 4.2):

| Requirement | What it forbids |
| :---- | :---- |
| **R1 — Cardinality hiding** | Credential MUST NOT reveal how many children a guardian is linked to (a care home manager with 16 children and a parent with one must produce indistinguishable credentials at the relying-party layer) |
| **R2 — Legal-basis abstraction** | Credential MUST NOT reveal the legal basis for authority (foster care, kinship, residential placement) |
| **R3 — Role anonymisation** | Professional role and organizational affiliation MUST NOT appear in any platform-visible attribute |
| **R4 — Tier-label privacy** | Risk tiers MUST NOT correlate with guardianship type (the highest assurance applies to all high-risk account types regardless of arrangement) |
| **R5 — Audit trail custody** | Audit trail held by regulated intermediary, NOT the platform; platform receives only timestamped authorization proofs |
| **R6 — Purpose-scoped, unlinkable credentials** | Credentials presented to Platform A and Platform B MUST NOT be cryptographically correlatable |

Our current Reference Architecture **fails several of these as written**. Specifically:

- The `derivative_authority.permission.scope_domain` is a free-form URI. If a deployment uses `https://cityhospital.example/scope/foster-care/...`, that URI itself violates R2.  
- Our `delegator.group.policy_uri` reveals group structure to anyone who sees the credential, violating R1 (the policy URI may differ between a one-child parent and a sixteen-child care home manager).  
- The Authority JWT's `prior_authority_ref` reveals chain depth and prior authority structure to relying parties, violating R3 and R5.  
- We have no provision for unlinkable credentials across relying parties (R6) — a single Task JWT presented to multiple platforms is correlatable.

**Recommendation:** Add a new Section 8.4 to our Reference Architecture: *"Privacy-Preserving Profile."* This profile defines:

1. **Two-token issuance model** — a "rich" internal credential held by a regulated verification intermediary that contains the full chain, the source authority, the group structure, and the audit metadata; and a "minimum-disclosure" presentation credential issued by the intermediary to relying parties that contains only the boolean authorization, the purpose scope, and the expiry.  
2. **Selective disclosure mechanism** — the rich credential supports SD-JWT or equivalent selective disclosure so the intermediary can derive presentation credentials per relying party.  
3. **Cardinality-hidden group descriptors** — when the privacy-preserving profile is in effect, the `delegatee.group` field MUST be reduced to an opaque commitment (e.g., a hash with blinding factor) at the relying-party layer, even if the rich form is used internally.  
4. **Unlinkability requirements** — presentation credentials issued for different relying parties MUST be cryptographically distinct in a way that prevents cross-platform correlation. This pushes us toward BBS+, ZKP-based credential systems, or anonymous credentials as a profile-level requirement.  
5. **Scope-domain abstraction rules** — under the privacy-preserving profile, scope\_domain URIs MUST be drawn from a registered, abstract vocabulary (e.g., `urn:scope:guardianship:platform-management`) rather than deployment-specific URIs that leak context.

This profile sits alongside the legal/healthcare and agentic profiles already in our model. The framework's case is that **for any delegation where the delegatee population includes a vulnerable group (children, adults under guardianship, asylum seekers), the privacy-preserving profile is mandatory**, not optional.

### 3.2 The "rich internal / flat external" architectural pattern

The framework explicitly states (Section 4.2): "the structured authority chain, however rich internally, must be flattened to a boolean credential before it crosses to the platform. The chain is for the verification intermediary and the regulated audit record; it is not for the platform to see or infer."

This is a pattern our architecture doesn't currently name, but should. It implies a **third party** in the architecture — the **regulated verification intermediary** — that sits between the delegator/delegatee and the relying party. The intermediary holds the rich credential; the relying party receives only the flattened presentation.

This is structurally similar to the OpenID for Verifiable Presentations holder/verifier model, but with stronger constraints: the intermediary is **regulated** (operating under statutory confidentiality equivalent to social care records, per the framework), and the audit trail is held by the intermediary, not by the relying party.

**Recommendation:** Add to our Section 8 a description of the **verification intermediary role**, distinct from the delegator (who holds source authority) and the relying party (who receives the presentation). The intermediary:

- Holds the rich JWTs (Relationship, Authority, Task in their full form)  
- Issues presentation credentials per relying party  
- Maintains the audit trail  
- Operates under statutory confidentiality obligations defined by the deployment profile  
- Is the only party that can correlate cross-platform activity (and is bound not to do so for commercial purposes)

This may belong in Components 5–6 (execution layer) rather than the establishment-layer components, but it must be flagged in Components 1–4 because the privacy properties of the establishment-layer JWTs depend on this role existing.

### 3.3 Revocation cascade and transfer semantics

Section 10.3 of our Reference Architecture flags revocation as an open question. The framework provides concrete requirements that we should adopt:

- **Revocation enforceable within 24 hours** of a guardianship change  
- **Cascade behavior**: when a Relationship is revoked, all Authority and Task JWTs bound to it MUST be revoked  
- **Transfer with continuity**: during guardianship transfer, the outgoing guardian's credential remains valid until the incoming guardian's credential is issued, preventing a protection gap  
- **Age-of-majority as automatic revocation**: at age 18 (or jurisdictional equivalent), guardianship credentials expire automatically and irrevocably; no extension is possible without the now-adult individual's express consent

These are not abstract design questions — they are operational requirements with hard deadlines. Our current `rev_endpoint` reference is too loose. We need:

1. A **revocation cascade rule** in §8.4 (new): revocation of a Relationship cascades to all Authority and Task JWTs binding to it  
2. A **transfer protocol** sketched at the Relationship layer: a new field `successor_rel_jti` that, when present, indicates that this Relationship has been superseded by another, and that during a transition window both are valid  
3. **Trigger-based expiry** as a first-class temporal constraint type: not just `exp` (calendar), but `expires_on_event` referencing an event type (age-of-majority, statutory review, court order termination) and an evaluation endpoint

The framework also exposes a privacy issue with revocation: **revocation reasons must not leak to platforms.** A platform should see "credential lapsed; new credential issued" without learning whether the cause was placement change, court order, or reunification. This is an additional privacy-preserving-profile constraint.

### 3.4 The child's evolving capacity — graduated authority

The framework introduces a concept that our current architecture does not handle: the delegatee themselves (the child) progressively acquires authority as they age, in a graduated way:

- 0–7: full DDAH authority; child's preferences noted but not determinative  
- 8–12: child consulted; DDAH retains authority but must record dissents  
- 13–15: child participates actively; DDAH retains data-processing consent authority (GDPR Article 8 threshold)  
- 16–17: presumption of digital autonomy unless the Care Plan states otherwise  
- 18+: full revocation of guardianship credential; child holds full authority

This is a **graduated revocation curve**, not a single revocation event. Modeling it requires either:

- **Multiple Relationships** — one per age band, each with its own validity window and authority scope, that automatically activate/deactivate as the child crosses age thresholds, OR  
- **A single Relationship with age-band-keyed permission shifts** — the `eligibility.tier1` carries a temporal predicate keyed to the child's age, and the `derivative_authority.permission.scope_domain` shifts accordingly

The first is cleaner architecturally but requires the issuing intermediary to manage a sequence of credentials. The second is more compact but bakes a non-trivial state machine into a single JWT.

**Recommendation:** Add to Component 1 a discussion of **graduated authority over time** as a deployment pattern, with both implementation options described. Do not normatively prefer one — the framework's text seems compatible with either, and the choice depends on the issuing intermediary's operational model. This should be flagged in Section 10 (Open Questions) of the Reference Architecture.

### 3.5 The two-key-worker / consensus pattern

The framework distinguishes three decision tiers (routine / significant / strategic) where strategic decisions require **full consensus** among multiple authorized parties. Our Component 1 quorum (`min_members`) handles the structural requirement, but the framework adds two refinements:

1. **Different quorum thresholds for different decision types** — routine decisions need only one authorized DDAH, strategic decisions need consensus among four to seven roles (social worker, psychologist, care home manager, biological parent, child, IRO, digital safety practitioner). This means the quorum configuration is **decision-tier-keyed**, not a single relationship-wide value.  
     
2. **Consensus participants drawn from heterogeneous role types** — not a list of identities, but a list of *roles*, each of which must be filled by *some* authorized individual at evaluation time.

Our current architecture handles (1) imperfectly — we'd need to express it as multiple Relationships (one per decision tier) or as Tier 2 policy\_ref. (2) is handled cleanly by our policy-defined group model, but only if we extend it to express **structured role-tuples** rather than single-role policies.

**Recommendation:** In Component 1, extend the policy-defined group descriptor to support a **role-tuple** sub-type, where membership requires presenting credentials for *each* of N specified roles. The Tier 1 vocabulary stays simple; the Tier 2 escape hatch absorbs the more complex consensus-protocol cases.

### 3.6 The cardinality-hiding requirement is the biggest single gap

Of all the issues raised by this framework, R1 (cardinality hiding) is the one that most directly contradicts something we currently do. Our Authority JWT and Task JWT both currently *do* leak cardinality. A care home manager with 16 children would need to issue 16 Task JWTs per day; the platform's logs would obviously show that the same `iss` (the care home's order entry system) issues hundreds of Task JWTs naming the same `delegatee` (the manager) for distinct child principals. Even without inspecting any field, the platform infers institutional-scale guardianship from issuance volume and pattern.

Closing this gap requires either:

- **Per-platform issuer keys**, so that the manager's actions on Platform A and Platform B are not linkable via shared `iss` — but this is operationally hard and may itself be observable as a key-rotation pattern  
- **A pseudonymous broker** (the regulated verification intermediary), which is the framework's preferred answer  
- **Anonymous-credential-style issuance**, where the platform receives a presentation that carries no `iss` linkable to the underlying issuer — possible with BBS+ or ZKP credentials but not with vanilla JWTs

This is genuinely hard and probably requires a **deferred privacy-preserving profile that uses a different credential format than vanilla JWT** for the platform-facing presentation. The internal (intermediary-held) credentials can stay JWTs. We should be candid about this in the Reference Architecture: vanilla JWT presentation cannot meet R1 in adversarial threat models.

### 3.7 GDPR Article 9 and special-category data risk

The framework correctly notes (Section 4.2) that a credential or architectural pattern that allows a platform to *infer* a child's care status from credential attributes, account structure, or request patterns is **likely to be non-compliant with GDPR Article 25 (data protection by design and by default) and Article 5(1)(c) (data minimization)**, regardless of whether any explicit special-category data is held.

This is a legal concern that bears directly on our Reference Architecture: a deployment using our framework as written, without the privacy-preserving profile, may be legally non-compliant in EU jurisdictions for any delegation involving children in care. This is not a soft "should consider" concern; it is a hard regulatory requirement under existing law.

**Recommendation:** Add to Section 5.6 (Authority profiles) a note that for any delegatee population that may include vulnerable individuals (children, adults under guardianship, asylum seekers, persons in coercive control situations), the privacy-preserving profile MUST be used, and that conformance to GDPR Article 25 is a required deployment constraint, not an optional enhancement.

---

## 4\. Where the Framework Reveals Issues for Components 5 and 6

The document gives us advance warning of issues we'll face when designing the execution layer:

### 4.1 The Permissions Dashboard is Component 5/6 territory

The framework's "Permissions Dashboard" — held by a regulated intermediary, accessible to the verified guardian, the IRO, and the child in age-appropriate format, with timestamped authorization proofs — is exactly the kind of artifact that lives at the execution layer. It is not a JWT; it is a stateful, queryable record. Our Components 5 (Delegated Capabilities) and 6 (Delegated Access) should be designed with this dashboard concept as a reference use case, and the `consent_dashboard_ref` field in the Authority should be specified at that point.

### 4.2 AI Intention Mandates as a Component 5 concept

The framework's "AI Intention Mandate" (defined in Appendix A) is a structured declaration setting out the purposes, limits, and safeguards under which an AI system may act on behalf of a child or guardian. It is "cryptographically bound to the credentials or tokens used by each AI system" — i.e., it lives at the execution layer, where capabilities are actually issued to AI workloads. This is a strong fit for our Component 5 design and validates our approach of separating establishment (Components 1–4) from execution (Components 5–6).

### 4.3 Dispute and chargeback mechanisms

The framework describes "dispute resolution and chargebacks where harm or misuse occurs" as a required execution-layer capability. Our Component 5 must support ergonomic revocation, and Component 6 must support credentialed dispute processes. This is more concrete operational guidance than we had before and should be incorporated into the Component 5/6 design phase.

---

## 5\. Updated Open Questions

Based on this stress test, the §10 open questions list in our Reference Architecture should be updated:

| Open question | Status after stress test |
| :---- | :---- |
| **Revocation cascade semantics** | Now concrete: cascade is required; 24-hour enforcement window; transfer continuity required; reasons must not leak |
| **Constraint conflict resolution** | Now concrete: deny-overrides-allow is correct; judicial/regulatory sources override delegator; framework provides a precedence ordering |
| **Cross-profile interaction** | Newly important: a privacy-preserving profile must be defined that supersedes both legal/healthcare and agentic profiles when delegatees may include vulnerable populations |
| **Chain reconstruction for multi-step delegation** | Stays open, but the framework's "rich internal / flat external" pattern suggests reconstruction is the *intermediary's* concern, not the relying party's |
| **Group identity privacy** | Now critical: cardinality hiding (R1), legal-basis abstraction (R2), and role anonymisation (R3) are binding requirements, not future work |
| **NEW: Graduated authority over time** | Added: how to model the child's evolving capacity (delegatee progressively acquires authority) |
| **NEW: Verification intermediary role** | Added: the architecture needs to formally describe the regulated intermediary that sits between delegator/delegatee and relying party |
| **NEW: Unlinkable presentation credentials** | Added: the platform-facing presentation may need a non-JWT format (BBS+, anonymous credentials) for adversarial threat models |
| **NEW: Trigger-based expiry as first-class** | Added: `expires_on_event` referencing event types (age-of-majority, statutory review, court order termination) |

---

## 6\. Net Assessment

**The architecture is fundamentally sound but incomplete.** The four-component decomposition holds up — the framework's authors independently arrive at essentially the same shape — and the layered scoping (Authority → Obligations → Constraints) maps directly onto the real-world distinction between a Care Plan's broad authority and the operational permissions enforced at platforms.

However, the framework exposes that **privacy is not a property of well-implemented deployments; it is a binding architectural constraint** that requires specific structural choices our current Reference Architecture does not enforce. The six R-requirements (cardinality hiding, legal-basis abstraction, role anonymisation, tier-label privacy, audit trail custody, purpose-scoped unlinkable credentials) are non-negotiable for any deployment involving vulnerable populations, and our architecture currently fails several of them as written.

The fix is additive, not destructive. The four-component model stays. We add:

1. A **privacy-preserving profile** alongside the existing legal/healthcare and agentic profiles  
2. A formal description of the **regulated verification intermediary role** as a third party between delegator/delegatee and relying party  
3. **Concrete revocation, cascade, and transfer semantics** with operational requirements (24-hour enforcement, continuity during transfer, reason-hiding)  
4. **Trigger-based expiry** as a first-class temporal constraint type  
5. **Graduated-authority-over-time** as a documented deployment pattern in Component 1  
6. **Cardinality-hidden group descriptors** for privacy-preserving profile usage  
7. An honest acknowledgement that the platform-facing presentation may require non-JWT credential formats (BBS+, ZKP credentials) for adversarial threat models, with vanilla JWT remaining the internal/intermediary-held format

These additions should be incorporated into the Reference Architecture before we proceed to Components 5 and 6, because the privacy-preserving profile requirements ripple into the execution layer's design.

The strongest validation from this stress test, beyond the technical findings, is this: **a working group of children's rights lawyers, social-care regulators, and frontline practitioners, operating independently of our IETF/OIDF technical work, has independently arrived at a six-component decomposition of delegated authority that matches ours almost exactly.** That convergence is the kind of empirical evidence one cannot manufacture, and it strongly suggests that the structural shape of the Reference Architecture reflects something real about how delegated authority decomposes in practice.

---

*End of stress test analysis.*  
