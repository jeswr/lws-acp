# LWS-APL Unified Datalog Core Signature

This document specifies an abstract Datalog-style core that can encode a broad range of access control paradigms (ACL, RBAC, ABAC, ReBAC, OrBAC, capabilities, lattice/MAC, risk-based, etc.) for comparison and potential implementation atop LWS-APL CLv2 predicates.

## 1. Base Domains (Entity Sorts)
Constants implicitly come from typed domains (or are IRIs in RDF):
- Subjects: `S`
- Resources: `R`
- Actions: `A`
- Applications / Clients: `App`
- Capabilities / Tokens: `Cap`
- Roles / Groups: `Role`, `G`
- Evidence identifiers: `Ev`
- Issuers / Verification Methods: `Iss`
- Time instants: `T`
- IP addresses / network identifiers: `IP`
- Classification lattice levels: `Level`
- Numeric scores: integers / reals for risk.

## 2. Extensional (Input) Predicates
Facts imported from the Assertion Graph (AG) or environment after Trust Policy admission:
```
hasAttr(S, AttrName, AttrValue)
resAttr(R, AttrName, AttrValue)
action(A)
purpose(App, Purpose)
edge(RelType, X, Y)                  % generic relationship edge (membership, ownership, delegation)
roleAssign(S, Role)
roleHierarchy(Senior, Junior)
permission(RoleOrGroup, R, A)
aclEntry(R, A, S)
capability(S, Cap)
capAuthorizes(Cap, R, A)
capDelegates(CapParent, CapChild)
capCaveat(Cap, CaveatType, Value)    % expiry, pathPrefix, depthLimit, purpose, etc.
clearance(S, Level)
classification(R, Level)
dominates(LevelHigh, LevelLow)       % lattice order
riskScore(S, R, A, Score)
threshold(ScoreType, Value)
evidence(Ev, Type)
issuedBy(Ev, Iss)
trustedIssuer(Iss)
attrCert(Ev, S, Role)                 % attribute/role certificate
binds(Ev, BindingType, Target)        % subject/resource/action/app bindings
notRevoked(Ev)
fresh(Ev)
contextNow(T)
contextIP(IP)
ipInCIDR(IP, CIDR)                    % can also be external pure check
viewOf(View, R)                       % OrBAC view abstraction
activityOf(Activity, A)
groupMember(S, G)
groupNesting(GParent, GChild)
devicePosture(Device, Posture)
subjectDevice(S, Device)
segmentRequiresRole(Segment, Role)
segment(Segment)
conflictClass(R, ConflictClass)       % Chinese Wall
accessedConflictClass(S, ConflictClass)
revoked(Cap)                          % capability revocation
rootCap(Cap)
```

## 3. Derived / Helper Predicates
Rules may define these intermediates:
```
reachable(RelType, X, Y)
inRole(S, Role)
canUseCap(S, R, A)
caveatsOk(Cap, S, R, A, T)
delegationDepthLe(Cap, Depth)
basePermit(S, R, A)
permit(S, R, A)
deny(S, R, A)
permitRead(S, R)
permitWrite(S, R)
abstractPermit(Role, Activity, View)
inGroup(S, G)
```

## 4. Generic Recursion & Hierarchies
```
reachable(Rel, X, Y) :- edge(Rel, X, Y).
reachable(Rel, X, Z) :- edge(Rel, X, Y), reachable(Rel, Y, Z).

inRole(S, Role) :- roleAssign(S, Role).
inRole(S, RoleHi) :- roleAssign(S, RoleLo), roleHierarchy(RoleHi, RoleLo).

delegationDepthLe(Cap, 0) :- rootCap(Cap).
delegationDepthLe(Cap, D) :- capDelegates(Parent, Cap), delegationDepthLe(Parent, Dp), Dp < D.
```

## 5. Capability Validity & Attenuation
```
caveatsOk(Cap, S, R, A, T) :-             % all caveats must hold (illustrative)
  (not capCaveat(Cap, expires, Exp) ; T < Exp),
  (not capCaveat(Cap, depthLimit, DL) ; delegationDepthLe(Cap, DL)),
  (not capCaveat(Cap, pathPrefix, Pfx) ; resAttr(R, path, P), prefix(Pfx, P)),
  (not capCaveat(Cap, purpose, P) ; purpose(App, P)).

canUseCap(S, R, A) :- capability(S, Cap), capAuthorizes(Cap, R, A), contextNow(T), caveatsOk(Cap, S, R, A, T), not revoked(Cap).
```
`prefix/2` assumed as an external pure predicate (maps to `lws:ExternalCheck`).

## 6. Lattice (Confidentiality / Integrity)
```
permitRead(S, R) :- clearance(S, C), classification(R, L), dominates(C, L).
permitWrite(S, R) :- clearance(S, C), classification(R, L), dominates(L, C).  % Bell-LaPadula: no write down
```

## 7. Chinese Wall (Conflict of Interest)
```
deny(S, R, read) :- conflictClass(R, CC), accessedConflictClass(S, CCp), CC != CCp.
```

## 8. ACL
```
permit(S, R, A) :- aclEntry(R, A, S).
```

## 9. RBAC (with Hierarchies)
```
permit(S, R, A) :- inRole(S, Role), permission(Role, R, A).
```

## 10. ABAC (Example Conjunctive Policy)
```
permit(S, R, A) :- hasAttr(S, dept, "finance"), resAttr(R, type, "report"),
                   hasAttr(S, clearance, C), classification(R, L), dominates(C, L).
```

## 11. ReBAC (Graph Reachability)
```
permit(S, R, read) :- reachable(ownerOf, S, R).
permit(S, R, write) :- reachable(collaborator, S, R), hasAttr(S, status, "active").
```

## 12. OrBAC (Abstract -> Concrete)
```
abstractPermit(Role, Activity, View) :- orbacRule(Role, Activity, View, Ctx), contextHolds(Ctx).
permit(S, R, A) :- roleAssign(S, Role), viewOf(View, R), activityOf(Activity, A), abstractPermit(Role, Activity, View).
```
`contextHolds` reduces to attribute/time/network predicates.

## 13. PERMIS (Attribute Certificates)
```
roleAssign(S, Role) :- attrCert(Ev, S, Role), issuedBy(Ev, Iss), trustedIssuer(Iss), fresh(Ev), notRevoked(Ev).
permit(S, R, A) :- inRole(S, Role), permission(Role, R, A).
```

## 14. Capability-Based
```
permit(S, R, A) :- canUseCap(S, R, A).
```

## 15. Risk-Based Augmentation
```
basePermit(S, R, A) :- hasAttr(S, acctStatus, "good"), resAttr(R, tier, "low").
permit(S, R, A) :- basePermit(S, R, A), riskScore(S, R, A, Score), threshold(allowRisk, Th), Score =< Th.
```

## 16. Identity Driven Networking (Segment Access)
```
permit(S, Seg, connect) :- subjectDevice(S, Dev), devicePosture(Dev, "healthy"),
                           hasAttr(S, role, Role), segmentRequiresRole(Seg, Role),
                           contextIP(IP), ipInCIDR(IP, CIDRAllowed).
```
`CIDRAllowed` may be enumerated via additional facts.

## 17. Apache Fortress / ARBAC Constraints (Sketch)
```
deny(S, R, A) :- inRole(S, R1), inRole(S, R2), staticConflict(R1, R2).
permit(S, R, A) :- inRole(S, Role), permission(Role, R, A), not deny(S, R, A).
```

## 18. Zanzibar-like Tuple Relations
Assuming relation tuples edge(rel, (Object,Relation), (Subject,User)) flattened appropriately:
```
permit(S, R, view) :- reachable(rel, (S, "user"), (R, "object") ).
```

## 19. Composition & Combining Algorithms
A higher-level combining algorithm (deny-overrides, permit-overrides, first-applicable, risk-minimizing) is applied after computing candidate `permit/3` and `deny/3` derivations. In pure Datalog this can be represented by deriving intermediate intents and a final decision predicate.

## 20. Mapping to LWS-APL CLv2
Each body literal corresponds to a predicate node (`lws:Predicate`) implemented by a SPARQL ASK over the AG or an external function / SHACL shape. Conjunction becomes `lws:allOf`. Disjunction uses multiple rules or `lws:anyOf`. Negation (if stratified) uses `lws:not`. Recursive paths can be materialized prior to evaluation or expressed via SPARQL property paths where sufficient (`(rel:edge)+`). Capability caveats & risk computations map to `lws:ExternalCheck` with deterministic guarantees.

## 21. Expressivity Notes
- Non-recursive fragments cover ACL/RBAC (flat)/simple ABAC.
- Recursion enables ReBAC, nested groups, delegation chains.
- Order predicates (`dominates/2`) capture lattice models.
- Numeric comparison and external pure predicates handle risk and caveats.

## 22. Safety & Determinism Alignment
All external predicates must be pure, terminating, and side-effect free to satisfy LWS-APL §2.3. Missing facts yield rule failure, aligning with closed-world deny-by-default semantics.

---
This core can serve as a reference for formal comparison and for generating CLv2 predicate vocabularies.
