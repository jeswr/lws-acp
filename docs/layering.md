# Layered Architecture for a Universal Web Authorization Language ("Permissioned Web")

Goal: Define an abstract, paradigm-neutral authorization language that can express all major access control models; layer concrete instantiations (protocol, evidence, crypto) beneath it so agents, applications, and browsers can interoperate over Linked Web Storage (LWS) resources.

## How this fits with Linked Web Storage (LWS)

This layering gives us a theoretical grounding for how I (Jesse) am thinking about authZ; for the current version of LWS I suggest we actually define layers 0-7 (including spec documents describing e.g. what the UMA profile looks like). But eventually move layers 4-7 to a separate spec that is upgradable in the same manner as upgrading other layers of the OSI stack.

## Overview Diagram (Conceptual)
```
+-----------------------------------------------------------+
| 14 Evolution & Extension / Version Negotiation            |
+-----------------------------------------------------------+
| 13 Threat, Safety & Performance Controls                  |
+-----------------------------------------------------------+
| 12 Interop & Conformance (Profiles, Test Suites)          |
+-----------------------------------------------------------+
| 11 Observability & Accountability (Logs, Audit Graph)     |
+-----------------------------------------------------------+
| 10 Administration & Governance (Lifecycle, Delegation)    |
+-----------------------------------------------------------+
|  9 Enforcement Architecture (PEPs, Caches, Edge, Client)  |
+-----------------------------------------------------------+
|  8 Transport & Caching (HTTP, CoAP, ETags, Cache Rules)   |
+-----------------------------------------------------------+
|  7 Cryptographic Binding (DI Proofs, JOSE/COSE, mTLS, SD) |
+-----------------------------------------------------------+
|  6 Session & Tokenization (Tokens, Capabilities, DPoP)    |
+-----------------------------------------------------------+
|  5 Protocol Bindings (UMA, GNAP, OIDC/OAuth2, GNAP Ext)   |
+-----------------------------------------------------------+
|  4 Evidence & Admission (VCs, JWTs, ZCAP, Headers)        |
+-----------------------------------------------------------+
|  3 Domain Vocabularies (Subjects, Actions, Attr Schemas)  |
+-----------------------------------------------------------+
|  2 Paradigm Profiles (ACL, RBAC, ABAC, ReBAC, OrBAC, ...) |
+-----------------------------------------------------------+
|  1 Core Policy Meta-Model (Target, Condition, Effect...)  |
+-----------------------------------------------------------+
|  0 Logical Semantics (Datalog/RDF CLv2 Denotation)        |
+-----------------------------------------------------------+
```
(Layers 0–5 are semantic foundations; 6–9 operational; 10–14 governance & evolution.)

## 0. Logical Semantics Layer
Defines the abstract evaluation model (Assertion Graph, Targets, Conditions with predicates / quantifiers, Combining algorithms, closed-world deny). Technology-neutral (can be written in denotational form or Datalog-style core). Ensures every higher layer maps to the same truth conditions.

## 1. Core Policy Meta-Model
RDF vocabulary (LWS-APL CLv2): PolicySet, Policy, Rule, Target, Condition, Predicate, Query, ShapeConstraint, ExternalCheck, Effect, Obligation. Purely structural; no paradigm-specific assumptions.

## 2. Paradigm Profiles
Constraint subsets mapping classic models:
- ACL Profile: only principal listing predicates.
- RBAC Profile: role assignment predicates + optional hierarchy.
- ABAC Profile: attribute predicates (no recursion).
- ReBAC Profile: relationship reachability (recursive path predicates).
- Capability Profile: capability validity predicate family (delegation, caveats).
- Lattice Profile: ordering predicates (clearance/classification).
- OrBAC Profile: roles / views / activities abstractions.
- Risk Profile: external deterministic risk scoring predicate + threshold.
- Zanzibar Profile: tuple relations + rewrite expansion.
Each profile = (a) predicate vocabulary subset, (b) structural constraints (depth, recursion), (c) safety limits. Profiles are composable.

## 3. Domain Vocabularies
Concrete IRIs for actions, resource types, attribute schemas (e.g., storage:Read, storage:Write, mediaType, owner, group, purpose). Independent registries allow sector-specific extension (health, finance). Shapes (SHACL) validate domain data imported as assertions.

## 4. Evidence & Admission Layer
Trust Policy governs which external artifacts become Input Assertions:
- Evidence Types: VC, VP, JWT, MTLS identity, ZCAP, PrivacyPass, HTTP-Signature, local DB row.
- Admission checks: freshness, revocation, binding (subject/app/resource), shape/namespace constraints, attribution.
- Output: normalized RDF assertions + provenance/integrity metadata into Assertion Graph.
Decouples semantics from credential format diversity.

## 5. Protocol Bindings Layer
Bindings that deliver requests, policies, and evidence:
- UMA 2.0 resource protection & permission ticket flows.
- GNAP authorization grant & subject-bound access token acquisition.
- OIDC/OAuth2 token issuance (ID Token, Access Token introspection → assertions).
- Direct HTTP interactions (Verifiable Presentation over HTTP header, ZCAP invocation).
Binding spec defines: how to present evidence, correlate with request context, and retrieve policy references.

## 6. Session & Tokenization Layer
Represents cached authority post initial protocol interaction:
- Access tokens (JWT, CBOR, encrypted).
- Capability tokens / ZCAP chains / macaroons (caveats map to predicates).
- DPoP / mutual TLS channel binding / proof-of-possession.
- Token introspection → predicate-satisfying assertions.

## 7. Cryptographic Binding Layer
Standardizes proof formats and integrity:
- Data Integrity proofs (e.g., Ed25519Signature2020, BBS+ selective disclosure).
- JOSE / COSE signatures & encryption.
- Hashlinks / Merkle proofs for inclusion.
- Channel bindings (TLS exporter, mTLS identities) as admission evidence.
Provides deterministic verification results for Layer 4 admission predicates.

## 8. Transport & Caching Layer
Optimizes evaluation & distribution:
- HTTP semantics (ETag / If-None-Match for policies & predicate definitions).
- CDN / edge caches for static predicates & shapes.
- Cache invalidation signaling (policy version IRIs, revision tokens (cf. Zanzibar zookies)).
- Content negotiation for policy documents (Turtle, JSON-LD).

## 9. Enforcement Architecture Layer
Placement & coordination:
- PEP at storage server / gateway.
- Edge policy evaluation caches (predicate materialization, reachability pre-compute).
- Client-side preview/prefetch evaluation (optional, advisory only).
- Delegated enforcement (object capabilities carrying attenuated rights).
Defines latency budgets, consistency strategies (snapshot vs eventual). 

## 10. Administration & Governance Layer
Lifecycle and authority to change policy:
- Policy authorship & delegation (who can define or update which policies).
- Versioning & deprecation semantics.
- Administrative RBAC / capability for policy management.
- Audit trail of changes (PROV).

## 11. Observability & Accountability Layer
Runtime transparency:
- Obligations (mustLog) produce structured audit entries (who, what, predicate set satisfied, evidence digests).
- Decision explanation graph (minimal justification subset) for user agents.
- Privacy controls (redaction of sensitive assertion content while retaining proof hashes).

## 12. Interoperability & Conformance Layer
- Conformance classes = subsets of profiles + protocol bindings.
- Test vectors: sample policies, evidence artifacts, expected decisions.
- Negative tests (revoked, expired, malformed) for safety.
- Performance thresholds (max evaluation time, recursion depth).

## 13. Threat, Safety & Performance Controls
Cross-cutting constraints:
- Determinism & totality for predicates (timeouts, failure ⇒ false).
- Recursion depth & result set cardinality limits (avoid blow-up).
- Side-effect boundaries (only ExternalCheck endpoints whitelisted).
- Policy static analysis (unreachable rules, shadowed denies, redundancy, complexity scoring).
- Mitigations: incremental evaluation, predicate result caching, cost-based abort.

## 14. Evolution & Extension Layer
- Version IRIs for core vocabulary & profiles (semantic versioning constraints).
- Extension points: new predicate families, new evidence types, new combining algorithms.
- Capability negotiation (client indicates supported profiles & proof suites).
- Deprecation channel (policy metadata marking sunset dates).

## Rationale for Layering
1. Separation of concerns: semantic policy logic (Layers 0–2) stable while credential & protocol ecosystems evolve (Layers 4–7).
2. Interop: Agents implement the core meta-model once; additional paradigms are profile constraints, not distinct languages.
3. Security: Admission & cryptographic layers isolate parsing/verification from evaluation core; simplifies TCB.
4. Evolvability: New evidence or signature formats only affect Layer 4/7; existing policies remain valid.
5. Optimization: Transport & enforcement layers can cache compiled predicates without altering semantics.
6. Governance & audit layers enable regulated scenarios (compliance, explainability) without contaminating core logic.

## Evaluation of Suitability
- Completeness: Cataloged paradigms (ACL, RBAC, ABAC, ReBAC, OrBAC, Capability, Lattice, Risk, Zanzibar) map cleanly to Profiles Layer.
- Orthogonality: Evidence formats (VC vs JWT) do not alter predicate truth conditions (only admission success); good separation.
- Practicality: Aligns with existing standards layering (e.g., OAuth core vs token formats vs resource server logic).
- Extensibility: Clear extension axes (new predicate types, new flows, new proof suites) without redefining core semantics.
- Risk: Need strong conformance guidance to prevent fragmentation through uncontrolled profile proliferation.

## Minimal Viable Stack (MVS)
1. Core Semantics + Meta-Model (0–1)
2. ABAC + ReBAC Profiles (2)
3. Basic Domain Vocabulary (3)
4. JWT + VC Admission + Trust Policy (4)
5. OAuth2 / GNAP Binding (5)
6. Access Tokens + DPoP (6)
7. JOSE + Data Integrity (7)
8. HTTP Transport & Predicate Caching (8)

## Browser / Agent Interaction Path (Example)
1. Browser requests resource with stored token.
2. Server Admission validates token / VC, builds Assertion Graph.
3. Policy evaluation (profiles: ReBAC + Capability) produces Permit with mustLog.
4. Obligation triggers structured log entry (hash-linked to decision provenance).
5. Response returned with cache hints (policy revision token). 

## Standardization Strategy
- Publish Core Meta-Model & Semantics (WG Note / Spec).
- Define Initial Profile Set + Conformance.
- Specify Admission & Evidence Type Registry (IANA/W3C style).
- Protocol Binding Specs (UMA, GNAP, OIDC) referencing core, not re-inventing semantics.
- Produce Test Suite & Reference Implementations (open-source) for evaluation & admission.
- Iterate with performance & security evaluations (cost model, DoS analysis).

## Key Open Questions
- Predicate recursion limits vs expressivity (formal tractability bounds).
- Canonical explanation representation (proof tree minimality criteria).
- Privacy-preserving logging (zero-knowledge or hashed assertion subsets).
- Harmonizing selective disclosure credentials with caching & replay protections.
- Unified revocation semantics spanning capabilities, credentials, and policies.

---
This layering is suitable: it cleanly separates enduring semantic constructs from evolving protocol & cryptographic mechanisms, enabling a universal authorization substrate for the "permissioned web" while supporting incremental adoption. Further refinement can add concrete JSON-LD frames per profile and a Trust Policy shape.
