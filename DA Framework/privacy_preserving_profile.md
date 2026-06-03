# Privacy-Preserving Profile for Delegated Authorization

**Working draft v1.0 — Companion Profile to the Delegated Authorization Reference Architecture v1.1**

*A layered profile that makes minimum-disclosure of delegation structure binding, for deployments involving vulnerable populations or sensitive relationship types.*

---

## Table of Contents

1. [Introduction](#1-introduction)  
2. [Scope and Applicability](#2-scope-and-applicability)  
3. [The Verification Intermediary Role](#3-the-verification-intermediary-role)  
4. [The Six Privacy Requirements (R1–R6)](#4-the-six-privacy-requirements-r1r6)  
5. [The Two-Token Architecture](#5-the-two-token-architecture)  
6. [Profile-Specific Field Constraints](#6-profile-specific-field-constraints)  
7. [Cryptographic Unlinkability (Optional Extension)](#7-cryptographic-unlinkability-optional-extension)  
8. [Compliance and Audit](#8-compliance-and-audit)  
9. [Worked Example: Care Home Manager → Platform](#9-worked-example-care-home-manager--platform)  
10. [Open Questions](#10-open-questions)  
11. [References](#11-references)

---

## 1\. Introduction

### 1.1 Why This Profile Exists

The base Reference Architecture defines a clean four-component model for delegated authorization. For most deployments — agentic AI delegation, corporate signing authority, healthcare credentialing among professional peers — the base specification is sufficient.

For deployments where the delegatee population includes **vulnerable individuals** or where the **type of delegation is itself sensitive**, the base specification is necessary but not sufficient. A platform receiving a base-specification credential could observe enough of its structure to infer information that the delegation system was meant to protect: the cardinality of a guardian's responsibilities, the legal basis for their authority, their professional role, or the membership pattern of authorized groups.

This profile addresses that gap. It does so by **layering** privacy requirements on top of the base specification rather than modifying the base specification's credentials or claims. Implementers conform to the base spec to interoperate with the broader ecosystem; deployments that handle vulnerable populations additionally conform to this profile.

### 1.2 Design Principle: Layering, Not Replacement

This profile changes nothing in the base specification's wire format, claim definitions, or binding model. Authority JWTs, Relationship JWTs, and Task JWTs as defined in the Reference Architecture are issued, bound, and validated identically. What this profile changes is:

1. **Who issues the credentials that platforms see.** Under this profile, platforms do not see the raw credentials defined in the base specification. They see *presentation credentials* derived from those raw credentials by a regulated verification intermediary.  
2. **What fields appear in the platform-facing presentation.** The presentation is constrained to a narrow, abstract set of attributes that does not allow the relying party to reconstruct the underlying delegation structure.  
3. **How revocation, transfer, and audit are routed.** The intermediary handles all platform-facing lifecycle events, so platforms never observe the underlying state changes that would otherwise leak context.

The base-specification credentials continue to exist and to do their job — they live inside the intermediary, where they support audit, dispute resolution, and oversight. They simply do not cross the relying-party boundary.

### 1.3 Relationship to the Stress Test Findings

The six privacy requirements R1–R6 in §4 of this profile are drawn directly from the *OIDF Delegated Authority Policy Framework for Child Rights-Respecting Digital Environments* (O'Connell & Curtis, April 2026), §4.2 of that document. The framework identified these as binding architectural requirements for any deployment involving children. This profile generalizes them to apply to any deployment involving vulnerable populations and adopts them as MUSTs for conforming implementations.

---

## 2\. Scope and Applicability

### 2.1 When This Profile Is REQUIRED

A deployment MUST conform to this profile if any of the following conditions applies:

- The delegatee population includes or may include children (under the age of majority in the relevant jurisdiction)  
- The delegatee population includes or may include adults under formal guardianship, conservatorship, or capacity-related legal arrangements  
- The delegatee population includes or may include asylum seekers, refugees, or persons whose civil status is contested or fragile  
- The delegation type itself reveals sensitive information about the delegator or delegatee (e.g., medical condition, immigration status, criminal proceeding involvement)  
- The deployment is operating in a jurisdiction whose data protection regulator has determined that the underlying delegation context constitutes special-category data under GDPR Article 9 or equivalent

### 2.2 When This Profile Is RECOMMENDED

A deployment SHOULD conform to this profile if:

- The delegation occurs in a context where the delegator or delegatee has a reasonable expectation of contextual privacy beyond the immediate transaction (e.g., financial signing authority for a private trust)  
- The deployment supports one-to-many delegations where cardinality could itself be sensitive  
- The deployment crosses jurisdictional boundaries with non-uniform privacy regimes

### 2.3 When This Profile Is NOT REQUIRED

The base Reference Architecture is sufficient (and this profile is not required) when:

- The delegation is between professional peers in a single trust domain (e.g., physician-to-nurse within a hospital)  
- The delegation is self-delegation by a competent adult to an agentic workload over their own affairs  
- The delegation type and the parties' roles are public information by nature (e.g., publicly-disclosed corporate signing authority)

### 2.4 Conformance

A deployment claims conformance to this profile by:

1. Operating a regulated verification intermediary as defined in §3  
2. Implementing all six privacy requirements R1–R6 as MUSTs (§4)  
3. Issuing platform-facing credentials only through the two-token architecture defined in §5  
4. Honoring the field constraints in §6  
5. Producing audit records in the form specified in §8

Cryptographic unlinkability (§7) is an OPTIONAL extension; conformance to this profile does not require it, but deployments operating under heightened threat models SHOULD implement it.

---

## 3\. The Verification Intermediary Role

### 3.1 Definition

A **Verification Intermediary** is a regulated party that sits between the delegator/delegatee and the relying party (platform). The intermediary:

- Holds the rich base-specification credentials (Relationship, Authority, Task JWTs in their full form)  
- Issues *presentation credentials* to relying parties on demand  
- Maintains the authoritative audit trail of authorization events  
- Manages revocation, transfer, and lifecycle events without leaking reasons to relying parties  
- Operates under statutory confidentiality obligations equivalent to or stronger than those governing the underlying delegation context

The intermediary is the only architectural party that has access to the full delegation structure. Delegators and delegatees provide their authority instruments to the intermediary; relying parties receive only the flattened presentation. No other architectural party has visibility into both sides.

### 3.2 Regulatory Status

This profile does not specify the legal form of the intermediary's regulation; that is determined by the deployment's jurisdiction and use case. Examples of suitable regulatory frameworks include:

- **In the EU**: an entity authorized as a Qualified Trust Service Provider under eIDAS 2.0, with additional sectoral authorization for the delegation domain (e.g., authorization under national social-care law for child guardianship delegations)  
- **In the UK**: an organization registered with the Information Commissioner's Office and additionally regulated by the relevant sectoral body (e.g., Ofsted for child welfare, the FCA for financial signing authority)  
- **In the US**: a state-licensed entity operating under HIPAA-equivalent confidentiality obligations for healthcare contexts, or under state child welfare law for guardianship contexts  
- **Cross-border**: an entity participating in a multilateral trust framework that imposes equivalent confidentiality obligations across the participating jurisdictions

The intermediary's regulatory status MUST be discoverable by relying parties, delegators, and delegatees through a published trust framework. The intermediary's authorizations MUST be bound to specific delegation domains; an intermediary licensed for child guardianship is not automatically authorized to handle financial signing authority.

### 3.3 Required Functions

A conforming intermediary MUST implement all of the following functions:

- **Credential intake** — accepts Relationship, Authority, and Task JWTs from authorized issuers; validates their conformance to the base specification; stores them with integrity protection  
- **Presentation issuance** — derives presentation credentials per relying party request; ensures presentations conform to the field constraints in §6  
- **Lifecycle management** — propagates revocation, transfer, and expiry events to active presentations; ensures the 24-hour enforcement window from §8.3 of the base specification is met  
- **Audit trail maintenance** — records every presentation issuance event with cryptographic integrity, retains audit records for the period mandated by the deployment's regulatory framework  
- **Authorized access** — provides the delegator, the delegatee (where appropriate to capacity), oversight bodies, and the data subject (where appropriate) with access to the audit trail in age-and-capacity-appropriate form  
- **Regulator interface** — supports inspection by the deployment's regulatory authorities under defined access protocols

### 3.4 Prohibited Functions

A conforming intermediary MUST NOT:

- Disclose to relying parties any information about the underlying credentials beyond what the field constraints in §6 permit  
- Use audit trail data for any commercial purpose, including profiling, marketing, advertising, or sale to third parties  
- Correlate activity across relying parties for any purpose other than fulfilling its regulatory functions  
- Hold relying-party-correlatable data in a form that could be subpoenaed without statutory protection (this MAY require segregating relying-party-specific data into per-relying-party encrypted partitions)  
- Charge the data subject (the individual the delegation concerns, e.g., the child) for access to their own audit trail or for exercise of any right granted under applicable privacy law

### 3.5 Trust Establishment with Relying Parties

A relying party trusts an intermediary's presentation credentials by:

1. Validating that the intermediary is registered in a recognized trust framework for the relevant delegation domain  
2. Validating the intermediary's signing key chains to a trust framework root  
3. Caching trust framework state with bounded staleness to detect intermediary deauthorization

The base-specification's `iss` claim on the underlying credentials is *not* visible to the relying party; the relying party sees only the intermediary's `iss`. This is a deliberate consequence of R3 (role anonymization) — the underlying issuer's identity is itself a context-leak.

---

## 4\. The Six Privacy Requirements (R1–R6)

These requirements are MUSTs for any deployment conforming to this profile. They apply to the credentials and metadata visible to relying parties; they do not apply to the rich credentials held inside the intermediary.

### 4.1 R1 — Cardinality Hiding

The presentation credential MUST NOT reveal how many distinct delegations a delegatee participates in, how many distinct delegators have authorized the delegatee, or how many distinct subjects (e.g., children) the delegator has authority for.

A care home manager authorized for sixteen children and a parent authorized for one child MUST produce presentation credentials that are structurally indistinguishable to the relying party. Cardinality is held internally by the intermediary; it does not appear in any field, header, signature, or pattern observable to the relying party.

This requirement is satisfied by the **intermediary-as-issuer pattern** (§3): all presentations are signed by the intermediary, so the relying party cannot link issuance volume to a specific delegator. Per-delegation issuance keys at the intermediary level MAY be used to further reduce correlatability.

For deployments operating under adversarial threat models (§7), cryptographic unlinkability extensions provide stronger cardinality hiding.

### 4.2 R2 — Legal-Basis Abstraction

The presentation credential MUST NOT reveal the legal basis under which the delegation was established. Specifically, the presentation MUST NOT contain or allow inference of:

- The category of underlying authority (foster care order, kinship arrangement, residential placement, court-appointed guardianship, special guardianship, voluntary care agreement, etc.)  
- The specific instrument identifier (court order number, care plan reference, employment contract reference)  
- The granting body (which court, which local authority, which corporate entity)

Where the underlying delegation requires distinguishing among legal bases for evaluation (e.g., a particular permission only applies under court-order-based authority), that distinction is made *inside the intermediary* during presentation derivation. The relying party receives only the resulting authorization decision, not the legal basis that led to it.

### 4.3 R3 — Role Anonymization

The presentation credential MUST NOT reveal the professional or institutional role of the delegatee. A presentation credential authorizing "this individual to manage this child's account" must not reveal whether the individual is a parent, foster carer, kinship carer, residential care manager, social worker, or court-appointed guardian.

Where the delegatee's role is *strictly required* for the relying party's authorization decision (e.g., a regulated medical procedure that only specific roles may authorize), the role is conveyed at a sufficiently abstract granularity that it does not identify the delegation type. "Authorized digital guardian" is acceptable; "residential care manager" is not.

The intermediary MAY issue different presentation credentials for different relying-party needs, with role abstraction tuned to each context. A medical platform receives "authorized clinical decision-maker"; a social media platform receives "authorized digital guardian." These are not the same credential, but neither reveals the delegatee's specific role.

### 4.4 R4 — Tier-Label Privacy

Where a deployment uses risk tiers (e.g., the high/highest assurance tiers in the base specification's profiles), the tier assignment MUST NOT correlate with the delegation type in a way that allows inference. Specifically:

- The highest assurance tier MUST be applicable to *any* delegation type that warrants it on independent risk grounds (account type, monetary value, age band, sensitivity of action) — not exclusively to delegations involving vulnerable populations.  
- Tier labels MUST NOT use names or identifiers that themselves reveal context (e.g., a tier named "alternative-care-tier" violates R4; a tier named "high-assurance" does not).

This requirement follows from a structural observation: if the highest assurance tier is *only ever* applied to delegations involving children in care, then the mere fact of a presentation carrying that tier label is itself a leak. Tiers must be defined by intrinsic risk properties of the delegation, not by the delegatee population.

### 4.5 R5 — Audit-Trail Custody

The authoritative audit trail of authorization events MUST be held by the intermediary, not by relying parties. Relying parties MAY hold their own logs of presentations they have validated, but those logs MUST NOT be the system of record for the delegation's authorization history.

Specifically:

- The intermediary holds a complete record of every presentation issuance, including the underlying credentials referenced, the relying party served, the timestamp, and the audit identifier  
- The relying party holds at most a record of the presentation it received and the authorization decision it made, sufficient for its own compliance purposes  
- The intermediary's audit record is the artifact a regulator inspects when reviewing the delegation's history; the relying party's records are inspected only for the relying party's own conduct

This separation ensures that the intermediary's regulated confidentiality obligations apply to the most sensitive aspect of the delegation (its history) and that subpoenaing a relying party does not reveal the underlying delegation structure.

### 4.6 R6 — Purpose-Scoped, Unlinkable Credentials

A presentation credential issued for use with one relying party MUST NOT be presentable to a different relying party, and MUST NOT be cryptographically linkable to presentations issued for different relying parties.

The minimum implementation of R6 is **per-relying-party presentation issuance**: the intermediary issues a fresh presentation credential each time a relying party requires one, scoped (via audience claim, key binding, or both) to that specific relying party. A presentation issued for Platform A cannot be replayed at Platform B because Platform B's validation rules will reject it.

This minimum implementation satisfies R6 against passive observers but does not satisfy it against colluding relying parties: if Platform A and Platform B compare their presentations, they may observe identifiers (such as the delegatee's pseudonymous identifier) that allow correlation. Defending against colluding relying parties requires cryptographic unlinkability (§7), which is an optional extension to this profile.

For deployments where colluding relying parties are not a credible threat (e.g., heavily regulated platforms with strong contractual prohibitions on cross-platform data sharing), the minimum implementation is sufficient. For deployments where collusion is plausible (e.g., consumer-facing platforms operating under weaker enforcement regimes), the optional extension SHOULD be implemented.

---

## 5\. The Two-Token Architecture

### 5.1 The Rich Token (Internal)

The rich token is the base-specification credential set: Relationship JWT, Authority JWT, Task JWT, with full claims, full binding, and full identifiability. It is held by the intermediary and is not visible to relying parties.

The rich token continues to be the canonical artifact for:

- Audit trail evidence (an oversight body inspecting the delegation sees the rich token)  
- Dispute resolution (a delegatee challenging an authorization decision can reference the rich token)  
- Inter-intermediary handoff (when a delegation is transferred between intermediaries, the rich token travels)

### 5.2 The Presentation Token (Relying-Party-Facing)

The presentation token is a derived credential issued by the intermediary, scoped to a specific relying party and a specific authorization decision. It is the only artifact the relying party sees.

The presentation token's structure is:

{

  "iss": "\<intermediary identifier\>",

  "aud": "\<relying party identifier\>",

  "iat": \<timestamp\>,

  "exp": \<timestamp\>,

  "jti": "\<unique presentation identifier\>",

  "sub": "\<pseudonymous subject identifier scoped to relying party\>",

  "delegatee\_pseudonym": "\<pseudonymous delegatee identifier scoped to relying party\>",

  "authorization": {

    "scope\_class": "\<abstract scope identifier from registered vocabulary\>",

    "tier": "\<assurance tier from base spec\>",

    "valid\_until": \<timestamp\>,

    "purpose": "\<abstract purpose identifier\>"

  },

  "audit\_id": "\<opaque audit correlation identifier\>"

}

The presentation token is signed by the intermediary. Its `sub` and `delegatee_pseudonym` are pseudonymous identifiers that are unique to (this delegation, this relying party); the same delegation presented to a different relying party uses different pseudonyms.

### 5.3 What the Relying Party Cannot Determine

From the presentation token alone, the relying party cannot determine:

- The identity of the underlying delegator  
- The legal basis of the delegation (R2)  
- The professional or institutional role of the delegatee (R3)  
- How many other delegations the delegatee participates in (R1)  
- Whether this is a healthcare, child-welfare, or other sensitive delegation type

The relying party can determine:

- That the delegation has been validated by a recognized intermediary  
- The abstract scope class of the authorization (e.g., "platform account management")  
- The assurance tier of the delegation  
- The duration of the authorization

This is sufficient for the relying party to enforce its own access control rules without inferring the delegation context.

### 5.4 Issuance Flow

A presentation token is issued through the following flow:

1. The delegatee initiates an action at the relying party (e.g., setting up a parental control on a platform).  
2. The relying party requests an authorization from the intermediary, naming the abstract action class and the audience identifier.  
3. The intermediary validates the underlying rich credentials, evaluates the action against the rich credentials' obligations and constraints, and either issues a presentation token or denies the request.  
4. The intermediary returns the presentation token to the relying party, either directly or via the delegatee acting as a pass-through.  
5. The relying party validates the presentation token's signature against the intermediary's published keys, validates the audience binding, and proceeds with the action.  
6. The intermediary records the issuance in its audit trail.

The relying party never speaks to the delegator's wallet, never sees the rich credentials, and never learns the delegation context.

---

## 6\. Profile-Specific Field Constraints

These constraints apply to the platform-facing presentation token and to any other artifacts that may be visible to the relying party. They do not apply to the rich credentials inside the intermediary.

### 6.1 Scope Domain Abstraction

Under the base specification, `derivative_authority.permission.scope_domain` is a free-form URI. Under this profile, the scope domain visible to the relying party MUST be drawn from a registered abstract vocabulary.

The base abstract scope vocabulary includes:

- `urn:scope:guardianship:platform-account-management` — generic delegated authority over a subject's platform account  
- `urn:scope:guardianship:data-processing-consent` — authority to grant or refuse data processing consent on behalf of a subject  
- `urn:scope:guardianship:financial-decision` — authority over a subject's financial transactions and account configuration  
- `urn:scope:guardianship:medical-decision` — authority over a subject's medical decisions  
- `urn:scope:guardianship:platform-content-controls` — authority to configure content and contact restrictions for a subject

Profiles for specific deployment domains MAY register additional abstract scopes through a designated authority. Concrete deployment-specific URIs (e.g., `https://example-care-home.example/scope/foster-care-management`) MUST NOT appear in presentation tokens.

### 6.2 Group Descriptor Hiding

Where the underlying Relationship JWT contains a group descriptor (e.g., a policy-defined group of authorized key workers), the presentation token MUST NOT contain the group descriptor. The group is resolved inside the intermediary at presentation issuance time; the resulting presentation simply names a single pseudonymous delegatee. The relying party does not learn that the delegatee is a member of a group, nor the size or composition of the group.

### 6.3 Prior Authority Reference Hiding

The base specification's `prior_authority_ref` (Authority JWT, derivative\_authority section) is part of the rich credential and MUST NOT appear in the presentation token. Chain depth and prior-authority structure are visible only to the intermediary.

### 6.4 Verified Claims Envelope Stripping

The base specification's `verified_claims` envelope (used for high-assurance Authority profiles) carries trust framework details, verification timestamps, and verification process identifiers. These MUST NOT appear in the presentation token; the assurance level is conveyed via the abstract `tier` field in the presentation.

### 6.5 Trigger-Based Expiry Opacification

Where a constraint in the rich credential carries a trigger-based expiry (base spec §7.7), the presentation token's `valid_until` MUST be a calendar timestamp, not an event reference. The intermediary computes a calendar expiry that bounds the trigger event with reasonable confidence (e.g., the next statutory review date, the child's next birthday) and re-issues a fresh presentation when the trigger event is approached or has occurred.

The relying party never sees the event type. A care home manager whose authority will end at the child's age of majority sees a presentation with `valid_until: 2027-04-15`; the platform does not learn that the date is the child's eighteenth birthday.

### 6.6 Revocation Status Opacification

Per base spec §8.3, revocation reasons are not transmitted to relying parties. Under this profile, the relying party MUST NOT receive any field that allows inference of the reason. A presentation that has been superseded due to a placement change and a presentation that has been superseded due to a planned handover are indistinguishable at the relying-party boundary.

### 6.7 Stage Schedule Hiding

Where the underlying Relationship uses Pattern B (single Relationship with stage-keyed eligibility, base spec §4.8), the presentation token MUST NOT contain the stage schedule. The intermediary evaluates the active stage at presentation issuance and emits a presentation reflecting only the currently-active scope.

---

## 7\. Cryptographic Unlinkability (Optional Extension)

### 7.1 When This Extension Applies

The minimum-implementation R6 (per-relying-party presentation issuance) is sufficient against passive observers but not against colluding relying parties. Deployments operating under threat models that include colluding relying parties — e.g., advertising-network platforms that exchange data, or platforms whose operators are subject to coercive demands by third parties — SHOULD implement this extension.

### 7.2 Approach: BBS+ or Equivalent Anonymous Credentials

Under this extension, the platform-facing presentation is not a JWT but an anonymous-credential presentation generated from a BBS+ signature (or equivalent unlinkable signature scheme such as CL-signatures or accumulator-based credentials).

The intermediary holds a long-lived BBS+ signing key. When a presentation is needed:

1. The intermediary derives a presentation that proves possession of a valid signature over a specified set of attributes, without revealing the underlying signature itself  
2. The presentation is bound to the relying party's audience identifier through a fresh proof-of-possession  
3. Two presentations issued for two different relying parties are cryptographically unlinkable: an observer comparing them cannot determine that they were derived from the same underlying signature

### 7.3 Trade-offs

This extension provides strong privacy guarantees but at significant operational cost:

- **Implementation complexity** — BBS+ libraries are less mature than vanilla JWT libraries; integration requires custom work at every relying party  
- **Verification cost** — anonymous credential verification is computationally heavier than JWT signature verification  
- **Standardization status** — BBS+ for VCs is being standardized at IETF and W3C; the specifications are still maturing as of 2026  
- **Interoperability** — relying parties that don't support the underlying credential format cannot consume the presentation; deployments may need to support both formats during transition

For most deployments that need this profile, the minimum implementation (per-relying-party JWT presentations from a regulated intermediary) is operationally adequate; cryptographic unlinkability is reserved for deployments where the threat model genuinely requires it.

### 7.4 Hybrid Architectures

A hybrid architecture is permitted: the intermediary may issue JWT presentations for relying parties that do not support anonymous credentials, and BBS+ presentations for relying parties that do. The choice is made per relying party based on the threat model that relying party operates under.

---

## 8\. Compliance and Audit

### 8.1 Required Audit Records

A conforming intermediary MUST maintain the following audit records for each delegation:

- A complete record of all rich credentials received from issuers, with cryptographic integrity protection  
- A record of every presentation issuance, including:  
  - The underlying rich credential identifiers  
  - The audience (relying party)  
  - The abstract scope and tier of the issued presentation  
  - The timestamp  
  - The opaque audit identifier returned to the relying party  
- A record of every revocation, transfer, and lifecycle event affecting the delegation  
- A record of every authorized access to the audit trail (who accessed it, when, what they viewed)

Audit records are retained for the period mandated by the deployment's regulatory framework. The retention period MUST NOT be shorter than the longest applicable statute of limitations for disputes arising from the delegation context.

### 8.2 Authorized Audit Trail Access

The audit trail is accessible to:

- The delegator (full visibility into delegations they have authorized)  
- The delegatee (visibility into authorizations they have exercised)  
- The data subject — i.e., the individual the delegation concerns, where distinct from the delegatee — in age-and-capacity-appropriate form, and in line with applicable privacy law (GDPR data subject rights, equivalent rights in other jurisdictions)  
- Designated oversight bodies (e.g., Independent Reviewing Officers for child welfare, conservatorship monitors for adult guardianship, financial regulators for signing authority)  
- The relevant regulator under defined inspection protocols  
- Law enforcement under valid legal process, with notice to affected parties where permitted

The intermediary MUST NOT make audit trail data accessible to any other party, including the relying party that received the original presentation.

### 8.3 GDPR Article 25 Conformance Assertion

A deployment claiming conformance to this profile MAY assert conformance to GDPR Article 25 (data protection by design and by default) on the basis of the structural privacy guarantees provided by the profile. Specifically:

- R1–R6 collectively implement the data minimization principle (Article 5(1)(c))  
- The two-token architecture implements purpose limitation (Article 5(1)(b))  
- The intermediary's audit trail custody implements integrity and confidentiality (Article 5(1)(f))  
- The pseudonymous identifiers and per-relying-party scoping implement pseudonymization as a safeguard (Article 25(1))

This assertion is not a substitute for a data protection impact assessment. Deployments processing special category data or operating at scale MUST conduct a DPIA per Article 35\.

### 8.4 Data Subject Rights

The intermediary MUST support the following data subject rights for the individual the delegation concerns:

- **Right of access** — the data subject may request a complete view of the authorizations that have been exercised under delegations naming them  
- **Right of rectification** — where a delegation contains inaccurate information about the data subject, the data subject may request correction  
- **Right to erasure** — at the conclusion of the delegation (e.g., on reaching age of majority), the data subject may request deletion of audit records, subject to the regulatory retention requirements in §8.1  
- **Right to portability** — the data subject may request export of the audit trail in a machine-readable form

These rights are exercised against the intermediary, not against the relying party. The relying party's own records (per §4.5) are subject to the relying party's own data protection obligations.

---

## 9\. Worked Example: Care Home Manager → Platform

### 9.1 Setup

- A 12-year-old child is in residential care, under a court order assigning parental responsibility to the local authority.  
- The local authority's care plan designates the residential care home manager as the Delegated Digital Authority Holder (DDAH) for the child's online activities.  
- The child wishes to use a major social media platform.  
- The platform requires verified parental consent under GDPR Article 8\.

### 9.2 Rich Credentials (Inside the Intermediary)

The regulated verification intermediary (a child-welfare-authorized trust service operating under the framework of the local authority) holds:

**Relationship JWT** — type `legal-representative`, delegator is the local authority, delegatee is the care home manager (specifically, a role-tuple group requiring "registered care home manager \+ designated by current placement"), eligibility constrained by Pattern B graduated stage schedule keyed to the child's age, with stage transitions at 8, 13, 16, and 18\.

**Authority JWT** — high-assurance profile with verified-claims envelope. Source authority asserts the local authority's parental responsibility under the court order (with concrete order number, court, and date — all visible only inside the intermediary). Derivative authority's scope domain is `urn:scope:guardianship:platform-account-management`, no further delegation permitted.

**Task JWT** — issued when the manager initiates the platform account setup. Single obligation of type `data_processing_consent`. Constraints include:

- Temporal: `valid_until` is calendar-bounded by the next statutory review date (six months out)  
- Trigger-based: `expires_on_event: statutory_review_completed`  
- Data-minimization: only specified GDPR Article 8 fields may be processed  
- Conditional: only-if the child's age is within the current Care Plan's permitted band

### 9.3 What the Platform Sees

When the manager initiates the account setup, the intermediary issues the following presentation token to the platform:

{

  "iss": "https://trust.example/intermediary/childcare-12",

  "aud": "https://platform.example",

  "iat": 1714000000,

  "exp": 1729800000,

  "jti": "urn:uuid:f9e8d7c6-5b4a-3210-9876-543210fedcba",

  "sub": "ps-7a3b9c1d2e4f",

  "delegatee\_pseudonym": "dp-8b4c0d2e3f5a",

  "authorization": {

    "scope\_class": "urn:scope:guardianship:data-processing-consent",

    "tier": "high-assurance",

    "valid\_until": 1729800000,

    "purpose": "urn:purpose:platform-account-setup"

  },

  "audit\_id": "ai-9c5d1e3f4a6b"

}

The platform validates this against the intermediary's published signing keys and proceeds with account setup.

### 9.4 What the Platform Does Not See

The platform does not learn:

- That the child is in residential care (R2 — legal basis abstraction)  
- That the delegatee is a care home manager (R3 — role anonymization)  
- That the manager is responsible for fifteen other children at the same home (R1 — cardinality hiding)  
- The existence of the court order, its number, or the issuing court  
- The Care Plan or its review schedule  
- That the credential's actual expiry is bounded by a statutory review event (R6.5 — trigger-based expiry opacification)  
- The graduated stage schedule that means the delegatee's authority will change as the child ages (R7 — stage schedule hiding)

A parent in a stable home setting up the same platform account for their own 12-year-old, and the care home manager setting up the account for the looked-after 12-year-old, produce structurally identical presentation tokens. The only differences would be in audience-bound pseudonyms and timestamps.

### 9.5 Lifecycle Events Without Leakage

Six months later, the child has been moved to a foster placement. The intermediary:

1. Receives notification from the local authority's case management system that the placement has changed  
2. Revokes the underlying Relationship JWT (status: `revoked`, no successor reference because the new foster carer is a different DDAH; this is a hard transfer)  
3. Issues a new Relationship JWT for the new foster carer  
4. When the platform next checks the previous presentation token's audit id, the revocation status is `revoked` with no further detail  
5. The new foster carer presents themselves at the platform, the platform initiates a fresh authorization, the intermediary issues a fresh presentation token

The platform observes only that "the previous credential is no longer valid; a new credential has been presented." It does not learn that this was a placement change. The platform cannot distinguish this case from other cases — a guardian who has decided to use a different device, a guardian whose credential has expired and been renewed, an emergency reassignment of authority. All look the same at the relying-party boundary.

---

## 10\. Open Questions

### 10.1 Intermediary Discovery and Selection

This profile assumes that delegators and delegatees know which intermediary to use. In practice, intermediary selection may be non-trivial: a delegation may span jurisdictions, the underlying authoritative source may be operated by a different party than the deployment's intermediary, and trust-framework recognition across jurisdictions is itself a live topic. A subsequent revision should specify the discovery mechanism by which a delegator finds and authenticates the appropriate intermediary.

### 10.2 Inter-Intermediary Handoff

When a delegation is transferred between jurisdictions or trust frameworks, the rich credentials may need to move from one intermediary to another. The protocol for such handoff — preserving audit trail continuity, ensuring no party loses access to authoritative records — is left for future work.

### 10.3 Performance and Scale

The intermediary-as-issuer pattern adds an online dependency at the moment of authorization. For most current deployments this is operationally acceptable, but high-frequency authorization scenarios (e.g., continuous platform interactions) may warrant caching or pre-issuance patterns. Specifying these patterns without compromising R6 (unlinkability) is non-trivial.

### 10.4 Liability Allocation

A delegation may fail in ways that affect the data subject (e.g., a platform honors a presentation that should have been revoked). Liability allocation among the issuer of the underlying credentials, the intermediary, and the relying party needs to be addressed in the deployment's trust framework. This profile does not specify allocation rules; it provides the audit trail necessary to support such allocation under whatever regulatory framework applies.

### 10.5 Cross-Profile Interaction with the Base Specification

A deployment using this profile internally may receive credentials from a deployment using only the base specification (e.g., an agentic workload from another organization presenting a base-specification Task JWT). The rules for handling such cross-profile presentations — whether to upgrade them, refuse them, or treat them as a different trust class — are left to deployment configuration.

### 10.6 Cryptographic Unlinkability Profile

The optional cryptographic unlinkability extension (§7) is sketched but not fully specified in this draft. A subsequent revision should specify the BBS+ credential format, the key management requirements, the verification protocol, and the migration path from JWT to BBS+ presentations.

---

## 11\. References

### 11.1 Normative References

- **Delegated Authorization Reference Architecture v1.1** — the base specification this profile layers upon  
- **RFC 7519** — JSON Web Token (JWT)  
- **GDPR** — Regulation (EU) 2016/679, particularly Articles 5, 9, 25, and 35  
- **eIDAS 2.0** — Regulation (EU) 2024/1183 on European Digital Identity

### 11.2 Informative References

- **OIDF Delegated Authority Policy Framework for Child Rights-Respecting Digital Environments**, O'Connell & Curtis, OpenID Foundation, April 2026 — the document that identified the six R-requirements and motivated this profile  
- **W3C Verifiable Credentials Data Model 2.0**  
- **BBS+ Signatures** — IETF draft and W3C working group  
- **OpenID for Verifiable Presentations** — OpenID Foundation  
- **SD-JWT (Selective Disclosure JWT)** — IETF draft

---

*End of working draft v1.0.*  
