# Access Control Paradigm Encodings in the Datalog Core

This document gives concise encodings of multiple paradigms using the unified core (see `datalog-core.md`). Each encoding shows how canonical policy semantics reduce to normalized rule forms suitable for LWS-APL CLv2 predicates.

## Legend
`permit(S,R,A)` and `deny(S,R,A)` are intermediate derivations prior to effect combining. Closed-world + deny-overrides assumed.

## 1. ACL
```
permit(S, R, A) :- aclEntry(R, A, S).
```
Mapping: Each ACL line becomes an `lws:Predicate` (ASK query) or direct triple in AG; rule condition is a single predicate node.

## 2. RBAC (Flat)
```
permit(S, R, A) :- roleAssign(S, Role), permission(Role, R, A).
```
Mapping: `roleAssign` + `permission` are ASK patterns; conjunction via `lws:allOf`.

## 3. RBAC with Hierarchy
```
inRole(S, Role) :- roleAssign(S, Role).
inRole(S, Senior) :- roleAssign(S, Junior), roleHierarchy(Senior, Junior).
permit(S, R, A) :- inRole(S, Role), permission(Role, R, A).
```
Option: Pre-compute closure into AG or express with a property path producing derived `inRole` assertions.

## 4. AGDLP (Windows Pattern)
```
inGroup(S,G) :- groupMember(S,G).
inGroup(S, Gp) :- groupMember(S, Gc), groupNesting(Gp, Gc).
permit(S, R, A) :- inGroup(S, Global), mapGlobalToDomainLocal(Global, DL), permission(DL, R, A).
```
`mapGlobalToDomainLocal` realized as triples or ASK over AG.

## 5. ABAC (Attribute Conjunction)
```
permit(S, R, read) :- hasAttr(S, dept, "finance"), resAttr(R, type, "report"), hasAttr(S, clearance, C), classification(R, L), dominates(C, L).
```
Each attribute check a predicate; ordering check another predicate.

## 6. ReBAC (Ownership / Collaboration Path)
```
reachable(Rel, X, Y) :- edge(Rel, X, Y).
reachable(Rel, X, Z) :- edge(Rel, X, Y), reachable(Rel, Y, Z).
permit(S, R, read) :- reachable(ownerOf, S, R).
permit(S, R, write) :- reachable(collaborator, S, R), hasAttr(S, status, "active").
```
Option: Materialize reachable facts into AG; or use SPARQL property paths `(rel:ownerOf)+`.

## 7. OrBAC
```
abstractPermit(Role, Activity, View) :- orbacRule(Role, Activity, View, Ctx), contextHolds(Ctx).
permit(S, R, A) :- roleAssign(S, Role), viewOf(View, R), activityOf(Activity, A), abstractPermit(Role, Activity, View).
```
Context predicates become separate ASK queries (time, risk, network, etc.).

## 8. PERMIS
```
roleAssign(S, Role) :- attrCert(Ev, S, Role), issuedBy(Ev, Iss), trustedIssuer(Iss), fresh(Ev), notRevoked(Ev).
permit(S, R, A) :- inRole(S, Role), permission(Role, R, A).
```
Credential validation occurs in Trust Admission prior to role assertions; or as predicates inside the policy if dynamic.

## 9. Capability-Based
```
canUseCap(S,R,A) :- capability(S, Cap), capAuthorizes(Cap,R,A), contextNow(T), caveatsOk(Cap,S,R,A,T), not revoked(Cap).
permit(S,R,A) :- canUseCap(S,R,A).
```
`caveatsOk` expands to function predicates (`pathPrefix`, `depthLimit`, etc.).

## 10. Capability Delegation Depth
```
delegationDepthLe(Cap,0) :- rootCap(Cap).
delegationDepthLe(Cap,D) :- capDelegates(Parent,Cap), delegationDepthLe(Parent,Dp), Dp < D.
```
If recursion disallowed in evaluation phase, pre-compute maximum depth offline and assert as attributes.

## 11. Lattice (Bell-LaPadula)
```
permitRead(S,R) :- clearance(S,C), classification(R,L), dominates(C,L).
```
Integrate with action mapping: if A=read then require `permitRead`.

## 12. Chinese Wall
```
deny(S,R,read) :- conflictClass(R,CC), accessedConflictClass(S,CCp), CC != CCp.
permit(S,R,read) :- permitRead(S,R), not deny(S,R,read).
```
Conflict classes recorded as assertions upon first access.

## 13. Risk-Based Augmentation
```
basePermit(S,R,A) :- hasAttr(S, acctStatus, "good"), resAttr(R, tier, "low").
permit(S,R,A) :- basePermit(S,R,A), riskScore(S,R,A,Score), threshold(allowRisk,Th), Score =< Th.
```
`riskScore` may be produced by ExternalCheck; threshold static.

## 14. Identity Driven Networking
```
permit(S,Seg,connect) :- subjectDevice(S,Dev), devicePosture(Dev,"healthy"), hasAttr(S,role,Role), segmentRequiresRole(Seg,Role), contextIP(IP), ipInCIDR(IP,CIDR).
```
Segment membership and posture imported via admission.

## 15. Apache Fortress (RBAC + SoD)
```
deny(S,R,A) :- inRole(S,R1), inRole(S,R2), staticConflict(R1,R2).
permit(S,R,A) :- inRole(S,Role), permission(Role,R,A), not deny(S,R,A).
```
Dynamic SoD adds session predicates.

## 16. Zanzibar (Tuple Relations)
Simplified tuple representation:
```
reachable(rel, X, Y) :- edge(rel, X, Y).
reachable(rel, X, Z) :- edge(rel, X, Y), reachable(rel, Y, Z).
permit(S,R,view) :- reachable(rel, (S,"user"), (R,"object")).
```
Object-typed pairs flattened to concrete constants or reified triples.

## 17. Composition / Combining Algorithms
Expressed by deriving `decision(S,R,A,Effect)` after analyzing `permit/deny` sets:
```
decision(S,R,A,deny) :- deny(S,R,A).
decision(S,R,A,permit) :- permit(S,R,A), not deny(S,R,A).
```
Alternate strategies add priority or ordering predicates.

## 18. Mapping to CLv2
For each rule body literal create a `lws:Predicate` IRI; implement via:
- SPARQL ASK (attribute, relationship, lattice order, membership, permission)
- SHACL shape (structural validation cases)
- External function (caveats, riskScore, ipInCIDR, prefix)
Conjunction: `lws:allOf` list. Negation: `lws:not`. Multiple rules for same head correspond to separate `Rule` resources combined under permit-overrides or aggregated predicate `anyOf`.

## 19. Safety Considerations
Only total, deterministic external functions allowed; recursion either pre-materialized or constrained (no function calls inside recursion) to preserve termination.

---
These encodings provide a basis for comparative analysis and direct translation to LWS-APL documents.
