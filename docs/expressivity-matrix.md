# Expressivity & Feature Matrix for Access Control Paradigms

This matrix compares key expressive / structural features across paradigms, using the unified core (see `datalog-core.md`). Y = native, + = trivial encoding, (ext) = requires extension (recursion, external predicate, numeric), • = not typical / awkward.

| Feature / Model | ACL | RBAC | AGDLP | ABAC | ReBAC | OrBAC | Cap-based | PERMIS | Lattice (BLP) | Chinese Wall | Risk-based | IDN | Apache Fortress | Zanzibar |
|-----------------|-----|------|-------|------|-------|-------|----------|--------|---------------|--------------|------------|-----|-----------------|----------|
| Direct principal listing | Y | + | + | + | + | + | + (via cap subject) | + | + | + | + | + | + | + |
| Roles abstraction | • | Y | Y | + (attr) | + (rel) | Y | + (cap attr) | Y | + | + | + | + | Y | + |
| Attribute predicates | • | (ext) | (ext) | Y | + | Y | + (caveats) | Y | + | + | + | Y | (ext) | + |
| Relationship paths (reachability) | • | (hierarchy) | (groups) | (ext) | Y | + (views) | Y (delegation) | + (deleg) | + (order as rel) | + | + | + | + | Y |
| Delegation / attenuation | • | Hierarchy only | Hierarchy only | (ext) | + (rel) | (ext) | Y | Y (attribute issuers) | • | • | + | • | (admin roles) | + |
| Capability possession semantics | • | • | • | • | • | • | Y | • | • | • | + (risk cap) | • | • | • |
| Lattice level ordering | • | + | + | + | + | + | + | + | Y | + | + | + | + | + |
| Conflict of interest control | • | + (SoD) | + | (policy) | (policy) | (context) | (policy) | + | + | Y | + | + | Y (SoD) | (policy) |
| Risk / numeric score use | • | (ext) | (ext) | (ext) | (ext) | (ext) | (ext) | (ext) | (ext) | (ext) | Y | + | (ext) | (ext) |
| Temporal constraints | • | (ext) | (ext) | + | + | + (context) | + (caveat) | + | + | + | + | + | + | + |
| External pure functions needed | Low | Low | Low | Med | Med | Med | High | Med | Low | Low | High | Med | Med | Med |
| Recursion required (typical) | No | Optional (hier) | Yes (nest) | No | Yes | No | Yes (deleg depth) | Yes (deleg) | No | No | No | No | Optional | Yes |
| Revocation granularity | ACL entry | Role assignment | Group membership | Attribute change | Relation removal | Context change | Capability revocation | Cert revocation | Label change | Access history gating | Threshold change | Attribute/device change | Role/Sod rule | Tuple removal |
| Least privilege support | Weak | Moderate | Moderate | Good | Good | Good | Strong (attenuation) | Good | Moderate | Good (conflict walls) | Contextual | Good | Moderate | Good |
| Formal analysis tractability | High | High | High | Med | Med (recursion) | Med | Med | Med | High | Med | Var (prob) | Med | Med | Med |

## Informal Containment / Fragment Hierarchy
- Non-recursive Horn fragment: ACL, flat RBAC, simple ABAC.
- + Role hierarchy recursion: RBAC(H), AGDLP.
- + General recursion (graph reachability & delegation): ReBAC, Cap-based (deleg chains), Zanzibar, PERMIS delegation graphs.
- + Lattice order predicates: Mandatory access control (Bell-LaPadula, Biba), Chinese Wall (with inequality + history).
- + Numeric comparison & external functions: Risk-based, capability caveats, temporal / IP / purpose checks.
- + Probabilistic / scoring semantics (optional): advanced adaptive & risk models.

## Dimensions Explained
- Relationship paths: need for transitive closure / recursive path evaluation.
- Delegation / attenuation: ability to derive authority via chains with potential restriction (depth, caveats).
- Lattice: partial order comparison central to decision.
- Conflict of interest: dependence on historic access sets or mutually exclusive labels.
- Risk / numeric: use of quantitative scoring beyond pure boolean.
- External pure functions: reliance on side-effect-free computations not trivially expressible in pure Horn clauses (time window parsing, CIDR math, cryptographic caveat validation).
- Recursion: syntactic necessity for typical deployments of the model.
- Least privilege: granularity & attenuation support to minimize excess rights.
- Formal tractability: relative ease of static analysis (safety, reachability, conflict detection).

## Mapping to LWS-APL CLv2
Each feature dimension maps to predicate families:
- Attributes, roles, groups → `lws:Predicate` via `lws:implementedByQuery`.
- Paths / reachability → either pre-materialize edges as assertions or use SPARQL property paths in predicate queries.
- Delegation / capability caveats → chain validation as external pure function (`lws:ExternalCheck`) plus supporting assertions.
- Lattice comparisons & conflicts → ASK queries over ordering / history facts.
- Risk & numeric thresholds → deterministic external function computing score + simple comparison query.
- Temporal / IP constraints → external functions (time window, CIDR) or SHACL shapes.

## Safety Alignment
Recursion confined to pure path closure and delegation depth; external predicates must be total & deterministic per §2.3 of LWS-APL. Closed-world default deny maintains consistent reduction across paradigms.

---
This matrix facilitates explicit comparison of expressive needs and informs which CLv2 predicate capabilities must be implemented for interoperability.
