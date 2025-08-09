# Layering Strategy Applied to Linked Web Storage (LWS)

This note captures how the proposed authorization layering maps to LWS implementation and standardization phases.

## 1. Purpose
Provide a coherent theoretical grounding (Layers 0–14) while scoping the *initial* LWS specifications to the semantic & protocol core (Layers 0–5). Higher operational / governance layers are acknowledged but left for subsequent documents. Layers 4–5 (Evidence/Admission & Protocol Bindings) are intentionally designed to be *swappable / upgradable* similarly to how transport layers evolve in the OSI model.

## 2. Immediate Scope for Current LWS Release (Layers 0–5)
- Layer 0 Logical Semantics: Normative denotational semantics + Datalog/RDF CLv2 mapping.
- Layer 1 Core Policy Meta-Model: LWS-APL vocabulary (PolicySet, Rule, Predicate, Condition, etc.).
- Layer 2 Paradigm Profiles: Define at least ABAC, ReBAC, Capability, and a Minimal (ACL) profile; formal SHACL shapes and profile conformance conditions.
- Layer 3 Domain Vocabularies: Base action/resource/attribute IRIs for storage (read, write, append, list, control), plus common subject & capability predicates.
- Layer 4 Evidence & Admission: Trust Policy vocab (anchors, issuers, admission rules). Specify minimal evidence set: JWT access token, Verifiable Credential (VC) presentation, ZCAP (capability) invocation, mTLS identity. Admission processing pipeline, freshness/ revocation / binding checks.
- Layer 5 Protocol Bindings: UMA profile (permission ticket→token→assertion pipeline), GNAP profile (grant types, subject binding), OAuth2/OIDC minimal mapping (introspection → assertions), Direct HTTP VC header exchange. Each binding normatively references Layers 0–4 semantics.

Deliverables:
1. Core Semantics Spec (0–1)
2. Profiles & Shapes Spec (2)
3. Domain Vocabulary Registry (3)
4. Trust & Admission Spec (4)
5. Protocol Bindings Spec (5) — includes UMA Profile section

## 3. Rationale for Deferring Layers 6–14
- Layers 6–7 (Session/Tokenization, Cryptographic Binding) rapidly evolve (new selective disclosure, post-quantum). Keeping them out of the initial locked core allows independent upgrade cycles.
- Layers 8–9 (Transport & Enforcement) include performance-specific tactics that differ between deployments (edge vs origin); premature standardization risks constraining innovation.
- Layers 10–14 (Administration, Observability, Conformance, Threat, Evolution) require operational experience to finalize realistic constraints and best practices.

## 4. Upgrade Strategy for Layers 4–5
Although initially specified inside the same suite, plan a **future extraction**:
- Version 1.x: Monolithic spec set (0–5) for simplicity.
- Version 2.0: Split into Core (0–3) + Admission (4) + Bindings (5). Introduce explicit version IRIs and negotiation (client signals supported binding versions in an Accept-Profile header or link relation).
- Admission Extension Registry: IANA/W3C-style registry for Evidence Types and Admission Check classes (freshness, revocation method kinds, binding methods, shape families).
- Deprecation Policy: Provide overlap period where old binding versions must be supported for N months.

## 5. Parallels to OSI Stack
- Core Semantics (0–1) ≈ Application-independent logic foundation (analogous to a language runtime).
- Profiles (2) ≈ Application-layer protocol options negotiated by capabilities.
- Evidence/Admission (4) ≈ Presentation / session establishment (normalizing heterogeneous artifacts).
- Protocol Bindings (5) ≈ Application protocols (UMA, GNAP) sitting atop normalized semantics, similar to HTTP atop TCP/IP.
Decoupling ensures improvements in Evidence representation or binding flows do not change policy truth conditions.

## 6. Migration & Compatibility Principles
1. **Semantic Stability:** Changes to Layers 4–5 MUST NOT alter evaluation results for unchanged Assertions + Policies.
2. **Additive Extensions:** New evidence types or binding profiles default to optional; policies can express required evidence classes using predicates without breaking older clients.
3. **Capability Negotiation:** Clients may send a header (e.g., `Accept-Auth-Bindings: uma-v1, gnap-v1`) allowing servers to pick a supported binding.
4. **Graceful Deprecation:** Mark outdated binders with sunset metadata; clients receive warnings in response headers before removal.
5. **Deterministic Downgrade:** If a preferred binding unsupported, server may fall back to a basic direct evidence submission mode (e.g., VC-in-HTTP) while preserving semantics.

## 7. Interactions with Policy Caching
- Predicate & Policy caching (Layers 8–9 future) rely on revision tokens independent of binding version.
- Admission results (which evidence produced which assertions) can be cached keyed by (evidence hash, trust policy version) allowing binding changes without re-verifying underlying credentials if cryptographic layer unchanged.

## 8. Implementation Phases
Phase A (Prototype): Implement 0–3 with simple in-memory admission (JWT only) and direct HTTP binding.
Phase B (Alpha): Add Trust Policy & VC/ZCAP admission (4), UMA & GNAP (5), risk-limited recursion controls.
Phase C (Beta): Introduce capability attenuation, explanation graph, conformance test harness.
Phase D (1.0): Freeze 0–5; publish registry drafts; begin extraction planning for 4–5.
Phase E (2.0 Planning): Formal negotiation mechanism, extension registry governance, optional post-quantum signature suites (Layer 7 future work).

## 9. Success Metrics
- Interop: Independent implementations achieve identical decisions on test corpus across at least 4 paradigms using same core policy documents.
- Extensibility: Add a new evidence type (e.g., Privacy Pass token) without modifying any Core Semantics text.
- Performance:  p95 evaluation latency under target (e.g., <10ms) for typical policy graphs (≤200 predicates) after warm cache.
- Stability: No breaking semantic changes required to integrate a new protocol binding.

## 10. Open Coordination Tasks
- Define Accept-Profile / negotiation header syntax (future extraction).
- Draft initial Evidence Type registry entries (JWT, VC, ZCAP, MTLS, HTTPSig).
- Specify normative JSON-LD frames for each profile to support signing & packaging.
- Decide minimum mandatory-to-implement binding set for 1.0 (likely: OAuth2 Introspection + UMA Profile).

## 11. Risks & Mitigations
| Risk | Impact | Mitigation |
|------|--------|------------|
| Over-conflation of layers in early spec | Hard to decouple later | Explicit modular section numbering & stable IRIs now |
| Profile proliferation | Fragmented ecosystem | Conformance classes & registry review process |
| Admission complexity (DoS) | Latency / resource exhaustion | Cost limits, caching, admission pre-validation, size caps |
| Predicate recursion abuse | Non-termination / blow-up | Static depth & result cardinality limits, per-predicate cost metrics |
| Evidence privacy leakage | Data minimization failures | Use selective disclosure & hashed assertion references |

## 12. Summary
We proceed with a *semantic-first* specification (Layers 0–5) while architecting for eventual extraction and independent evolution of Evidence & Protocol layers. This preserves long-term agility (new protocols, credentials) without destabilizing core authorization semantics central to a universal, permissioned Web.
