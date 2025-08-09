# Linked Web Storage Authorization Policy Language (LWS-APL)

## 1. RDF Vocabulary

### Classes
- **lws:PolicySet** — A set of policies evaluated together with a specified combining algorithm.
- **lws:Policy** — A collection of rules with a shared target and optional obligations.
- **lws:Rule** — An atomic decision rule with an effect and optional condition.
- **lws:Target** — Defines the subject(s), resource(s), and action(s) to which a policy or rule applies.
- **lws:Condition** — A boolean expression composed from predicates, booleans and quantifiers.
- **lws:Obligation** — An action the PEP must perform when a policy permits access.
- **lws:Effect** — Possible effects: `lws:Permit` or `lws:Deny`.
- **lws:AssertionGraph** — Request-scoped union of input assertions with provenance and integrity metadata.
- **lws:InputAssertion** — A set of RDF statements and accompanying provenance/integrity metadata.
- **lws:Predicate** — A named boolean that can be implemented by a query, a shape constraint, or an external pure function.
- **lws:Query** — A SPARQL ASK (or boolean SELECT) associated with a predicate.
- **lws:ShapeConstraint** — A SHACL shape treated as a boolean check.
- **lws:ExternalCheck** — A side-effect-free function described via FNO.

### Trust & Admission (new)
- **lws:TrustPolicy** — Server-local configuration defining trust roots and admission rules for input assertions.
- **lws:TrustAnchor** — A root of trust (e.g., DID method, X.509 root CA, JWKS set, or public key fingerprint).
- **lws:Issuer** — An authority expected to issue evidence; linked to one or more verification methods.
- **lws:VerificationMethod** — A concrete verification material (Data Integrity key, JOSE/COSE key, X.509 cert, mTLS identity binding, etc.).
- **lws:EvidenceType** — A named class of evidence (e.g., `JWT`, `VC-PRESENTATION`, `ZCAP`, `MTLS`, `HTTPSIG`).
- **lws:AdmissionRule** — A rule that, when satisfied, authorizes import of some statements into the Assertion Graph.
- **lws:FreshnessPolicy** — Requirements for time validity (max age, clock skew).
- **lws:RevocationSource** — Where to check revocation/deny lists.
- **lws:BindingPolicy** — Requirements that bind evidence to the current request (subject binding, application binding, audience/resource binding, DPoP, certificate pinning, etc.).
- **lws:ShapeWhitelist** — Allowed RDF shapes that admitted assertions must conform to.
- **lws:NamespaceWhitelist** — Allowed predicate/name spaces.
- **lws:Attribution** — How admitted statements are attributed (who said what, under which key/method).

### Properties
- **lws:effect** — Outcome of a rule: Permit or Deny.
- **lws:combiningAlg** — How multiple rules or policies are combined (e.g., deny-overrides).
- **lws:subject** — The principal (person, org, agent) making the request.
- **lws:action** — The operation requested on the resource.
- **lws:resource** — The resource to which the request applies.
- **lws:condition** — The condition that must evaluate to true for the rule to apply.
- **lws:obligation** — An obligation associated with a policy or rule.
- **lws:mustLog** — Indicates that the PEP must log the access if permitted.
- **lws:allOf / lws:anyOf / lws:not** — Boolean combinators.
- **lws:exists / lws:forAll** — Quantifiers over sets returned by a binding query.
- **lws:predicate** — Points from a condition node to a `lws:Predicate`.
- **lws:withParam** — Associates parameter bindings (name/value) to a predicate invocation.
- **lws:implementedByQuery** — Associates a predicate with an ASK/boolean SELECT.
- **lws:implementedByShape** — Associates a predicate with a SHACL shape.
- **lws:implementedByFunction** — Associates a predicate with an FNO-described external function.
- **lws:assertion** — Links the request to one or more `lws:InputAssertion` resources.
- **lws:provenance / lws:integrityProof** — Metadata on `lws:InputAssertion` (aliases to PROV/Data Integrity terms in final namespace).
- **lws:trustPolicy** — Links a server (or realm/tenant) to its `lws:TrustPolicy`.
- **lws:anchor** — Associates a `lws:TrustPolicy` with one or more `lws:TrustAnchor`.
- **lws:acceptsEvidence** — From `lws:TrustPolicy` to `lws:EvidenceType`.
- **lws:issuer** — From `lws:TrustPolicy` to permitted `lws:Issuer`(s).
- **lws:verificationMethod** — From `lws:Issuer` to `lws:VerificationMethod`.
- **lws:admissionRule** — From `lws:TrustPolicy` to `lws:AdmissionRule`.
- **lws:freshness** — From `lws:AdmissionRule` to `lws:FreshnessPolicy`.
- **lws:revocation** — From `lws:AdmissionRule` to `lws:RevocationSource`.
- **lws:binding** — From `lws:AdmissionRule` to `lws:BindingPolicy`.
- **lws:requiresShape** — From `lws:AdmissionRule` to `lws:ShapeWhitelist`.
- **lws:allowsNamespace** — From `lws:AdmissionRule` to `lws:NamespaceWhitelist`.
- **lws:attributeTo** — From admitted `lws:InputAssertion` to `lws:Issuer` or `lws:VerificationMethod`.


### Classes
- **lws:PolicySet** — A set of policies evaluated together with a specified combining algorithm.
- **lws:Policy** — A collection of rules with a shared target and optional obligations.
- **lws:Rule** — An atomic decision rule with an effect and optional condition.
- **lws:Target** — Defines the subject(s), resource(s), and action(s) to which a policy or rule applies.
- **lws:Condition** — A boolean expression composed from predicates, booleans, and quantifiers.
- **lws:Obligation** — An action the PEP must perform when a policy permits access.
- **lws:Effect** — Possible effects: `lws:Permit` or `lws:Deny`.
- **lws:AssertionGraph** — Request-scoped union of input assertions with provenance and integrity metadata.
- **lws:InputAssertion** — A set of RDF statements and accompanying provenance/integrity metadata.
- **lws:Predicate** — A named boolean that can be implemented by a query, a shape constraint, or an external pure function.
- **lws:Query** — A SPARQL ASK (or boolean SELECT) associated with a predicate.
- **lws:ShapeConstraint** — A SHACL shape treated as a boolean check.
- **lws:ExternalCheck** — A side-effect-free function described via FNO.

### Properties
- **lws:effect** — Outcome of a rule: Permit or Deny.
- **lws:combiningAlg** — How multiple rules or policies are combined (e.g., deny-overrides).
- **lws:subject** — The principal (person, org, agent) making the request.
- **lws:action** — The operation requested on the resource.
- **lws:resource** — The resource to which the request applies.
- **lws:condition** — The condition that must evaluate to true for the rule to apply.
- **lws:obligation** — An obligation associated with a policy or rule.
- **lws:mustLog** — Indicates that the PEP must log the access if permitted.
- **lws:allOf / lws:anyOf / lws:not** — Boolean combinators.
- **lws:exists / lws:forAll** — Quantifiers over sets returned by a binding query.
- **lws:predicate** — Points from a condition node to a `lws:Predicate`.
- **lws:withParam** — Associates parameter bindings (name/value) to a predicate invocation.
- **lws:implementedByQuery** — Associates a predicate with an ASK/boolean SELECT.
- **lws:implementedByShape** — Associates a predicate with a SHACL shape.
- **lws:implementedByFunction** — Associates a predicate with an FNO-described external function.
- **lws:assertion** — Links the request to one or more `lws:InputAssertion` resources.
- **lws:provenance / lws:integrityProof** — Metadata on `lws:InputAssertion`.

## 2. Normative Evaluation Semantics

1. **Inputs:** Subject, application, resource, action, environment.
2. **Validation:** Proofs are validated according to the **evidence type** (see §2.1), not tied to any single artifact format.
3. **Policy Discovery:** Gather relevant PolicySets/Policies from PRP matching the request.
4. **Rule Evaluation:** Conditions are evaluated using the **Condition Language v2** (see §2.2), against a request-scoped **Assertion Graph** assembled from one or more evidence sources **admitted under the server's Trust Policy (see §2.4)**.
5. **Combining Algorithm:** Apply per-policy and per-policy set combining algorithms (default: deny-overrides).
6. **Decision & Obligations:** Return Permit/Deny with obligations; obligations must be enforced.
7. **Defaults:** No matching policy → Deny; absent assertions evaluate as false.

### 2.1 Evidence & Assertion Graph
- The PDP constructs an **Assertion Graph** (AG) for each decision. The AG is a union of zero or more **Input Assertions** from heterogeneous sources (e.g., signed HTTP headers, mTLS identities, ACL group catalogs, local account DBs, VCs/VPs, OAuth token introspection responses, ZCAP chains, privacy pass tokens, or server-local config). 
- Each *Input Assertion* MUST carry provenance (**prov:wasDerivedFrom**) and integrity metadata (e.g., a Data Integrity proof, JOSE/COSE signature, mTLS binding, or server trust root). 
- Policies reference **assertions by semantics**, not by credential type. There is no special-case handling of "VC" in the condition model.

### 2.2 Condition Language v2 (CLv2)
CLv2 replaces ad-hoc `lws:path` with three interoperable primitives:

1) **lws:Predicate** — a named, boolean predicate with zero or more parameters. Can be backed by:
   - **lws:Query**: a SPARQL `ASK` (or `SELECT` returning boolean via `EXISTS`) evaluated over the AG + request context.
   - **lws:ShapeConstraint**: a SHACL `NodeShape` whose validation result is treated as boolean.
   - **lws:ExternalCheck**: a pure function via FNO (no side effects), evaluated with concrete parameter values.

2) **Boolean composition** — `lws:allOf` / `lws:anyOf` / `lws:not`.

3) **Quantifiers over sets** — `lws:exists` and `lws:forAll`, with a bound variable sourced from a query binding set.

The CLv2 surface is RDF-first and analyzable: policies reference predicates by IRI, with parameters expressed as RDF terms. Implementations MAY cache compiled predicates and perform static checks that all referenced predicates are well-typed and side-effect free.

### 2.3 Determinism and Safety
- Predicates MUST be **total and deterministic** for given inputs; external checks must declare timeouts and be idempotent.
- Network I/O during evaluation is limited to fetching predicate or shape definitions (cacheable) and to pre-declared *ExternalCheck* endpoints.
- If any referenced predicate cannot be resolved or validated, it evaluates to **false**.

### 2.4 Trust Policy & Admission Control
Before evidence can contribute statements to the AG, the server applies its **Trust Policy**:
- **Trust Anchors**: declare accepted roots (e.g., specific DID methods, JWKS endpoints, CA roots, pinned pubkeys).
- **Issuers & Verification Methods**: enumerate acceptable issuers and their keys/certs/signature suites.
- **Admission Rules**: for each accepted **EvidenceType**, define:
  - **Freshness** (max age, not-before/after windows, clock skew),
  - **Revocation** (CRLs/OCSP, status endpoints, key lists, capability revocation registries),
  - **Binding** to the request (subject/app identifiers, audience/resource constraints, DPoP/mTLS/token-binding),
  - **Shape/Namespace** constraints (what RDF can be imported), and
  - **Attribution** (who is considered to have asserted the imported triples).
- Only statements that pass **all** applicable checks are admitted as `lws:InputAssertion` and linked into the AG with provenance and integrity pointers.


1. **Inputs:** Subject, application, resource, action, environment.
2. **Validation:** Proofs validated according to evidence type, not tied to a single artifact format.
3. **Policy Discovery:** Gather relevant PolicySets/Policies from PRP matching the request.
4. **Rule Evaluation:** Conditions evaluated using CLv2 against the request's Assertion Graph.
5. **Combining Algorithm:** Apply per-policy and per-policy set combining algorithms (default: deny-overrides).
6. **Decision & Obligations:** Return Permit/Deny with obligations; obligations must be enforced.
7. **Defaults:** No matching policy → Deny; absent assertions evaluate as false.

### 2.1 Evidence & Assertion Graph
- The PDP builds an Assertion Graph (AG) from zero or more Input Assertions from heterogeneous sources (e.g., signed HTTP headers, mTLS identities, ACL group catalogs, local DBs, VCs, OAuth tokens, ZCAP chains, Privacy Pass tokens).
- Each Input Assertion MUST carry provenance and integrity metadata.
- Policies reference assertions by semantics, not by credential type.

### 2.2 Condition Language v2 (CLv2)
CLv2 uses:
1. **lws:Predicate** — Boolean predicate implemented by Query, ShapeConstraint, or ExternalCheck.
2. **Boolean composition** — `lws:allOf`, `lws:anyOf`, `lws:not`.
3. **Quantifiers** — `lws:exists`, `lws:forAll` over query bindings.

### 2.3 Determinism & Safety
- Predicates must be total and deterministic; external checks must declare timeouts.
- Network I/O limited to fetching predicate/shape definitions and pre-declared external checks.
- Unresolved predicates evaluate to false.

## 3. Mini Standard Library of Predicates
- **lws:ownsResource** — Subject is owner of resource.
- **lws:memberOfGroup** — Subject is member of group.
- **lws:withinTimeWindow** — Current time within given window.
- **lws:ipInRange** — Request IP is in allowed CIDR.
- **lws:purposeAllowed** — Requested purpose in allowed set.
- **lws:capabilityValid** — Capability present and unexpired.

## 4. Mappings to Existing Systems
- WAC, ACP, Zanzibar, ZCAP-LD map to predicates using Queries or ExternalChecks.

## 5. Formal Denotational Semantics (Excerpt)
Let `AG` be the assertion graph, `ctx` the request context.
- ⟦`Predicate(p)`⟧(AG, ctx) = result of executing bound implementation of `p` over (AG ∪ ctx).
- ⟦`allOf(C1..Cn)`⟧ = true iff ∀i, ⟦Ci⟧ = true.
- ⟦`anyOf(C1..Cn)`⟧ = true iff ∃i, ⟦Ci⟧ = true.
- ⟦`not(C)`⟧ = ¬⟦C⟧.
- ⟦`exists(v, Q, C)`⟧ = true iff ∃binding for `v` from Q over AG where ⟦C⟧ = true.
- ⟦`forAll(v, Q, C)`⟧ = true iff ∀binding for `v` from Q over AG, ⟦C⟧ = true.

## 6. JSON-LD Frame for Policies
```json
{
  "@context": "https://www.w3.org/ns/lws-apl",
  "@type": "PolicySet",
  "policy": {
    "@type": "Policy",
    "target": { "resource": "?resource", "action": "?action" },
    "rule": {
      "@type": "Rule",
      "effect": "Permit",
      "condition": {
        "predicate": "?predicateIRI"
      }
    }
  }
}
```

## 7. Examples
- ABAC via SPARQL ASK predicate.
- ReBAC via ASK predicate.
- Time window via ExternalCheck.
- Quantified collaborator check.
- Capability validity check.

## 8. SHACL Shapes (Excerpt)
Shapes validate that predicates have exactly one implementation, conditions use valid combinators, and effects are Permit/Deny.



## 7. Predicate Standard Library (Draft)
The following predicates are OPTIONAL but RECOMMENDED for interop. All operate over the **Assertion Graph (AG)** and request context.

### 7.1 Identity & Relationship
```turtle
<#isOwner> a lws:Predicate ;
  lws:implementedByQuery [ a lws:Query ; lws:sparql """
    PREFIX rel: <https://example.org/rel/>
    ASK WHERE { ?subject rel:owner ?resource }
  """ ] .

<#memberOf> a lws:Predicate ;
  lws:implementedByQuery [ a lws:Query ; lws:sparql """
    PREFIX org: <http://www.w3.org/ns/org#>
    ASK WHERE { ?subject org:memberOf ?group }
  """ ] .

<#hasRole> a lws:Predicate ;
  lws:implementedByQuery [ a lws:Query ; lws:sparql """
    PREFIX asgn: <https://example.org/assign/>
    ASK WHERE { ?subject asgn:hasRole ?role }
  """ ] .
```

### 7.2 Resource Properties
```turtle
<#pathMatches> a lws:Predicate ;
  lws:implementedByFunction <https://functions.example/pathMatches> ;
  lws:withParam [ :prefix "/secure/" ].

<#mimeTypeIn> a lws:Predicate ;
  lws:implementedByQuery [ a lws:Query ; lws:sparql """
    PREFIX dct: <http://purl.org/dc/terms/>
    ASK WHERE { ?resource dct:format ?mt FILTER (?mt IN ("text/turtle","application/json")) }
  """ ] .
```

### 7.3 Context, Time, Network
```turtle
<#withinTimeWindow> a lws:Predicate ;
  lws:implementedByFunction <https://functions.example/withinTimeWindow> ;
  lws:withParam [ :start "09:00" ; :end "17:00" ; :tz "Europe/London" ] .

<#ipInCIDR> a lws:Predicate ;
  lws:implementedByFunction <https://functions.example/ipInCIDR> ;
  lws:withParam [ :cidr "192.0.2.0/24" ] .
```

### 7.4 Purpose & Retention
```turtle
<#purposeAllowed> a lws:Predicate ;
  lws:implementedByQuery [ a lws:Query ; lws:sparql """
    PREFIX dpv: <https://w3id.org/dpv#>
    ASK WHERE { ?request dpv:hasPurpose ?purpose FILTER (?purpose IN (dpv:Research, dpv:PublicInterest)) }
  """ ] .

<#retentionNotExceeded> a lws:Predicate ;
  lws:implementedByFunction <https://functions.example/retentionNotExceeded> ;
  lws:withParam [ :createdPath :created ; :maxDuration "P90D" ].
```

### 7.5 Capabilities
```turtle
<#capabilityValid> a lws:Predicate ;
  lws:implementedByFunction <https://functions.example/capabilityValid> ;
  lws:withParam [ :maxDelegationDepth 2 ; :notExpired true ].
```

## 8. Formal Semantics (CLv2)
We define evaluation of a condition node `C` as [[C]]_{AG,RC} ∈ {true,false}, where `AG` is the assertion graph and `RC` is the fixed request context (bindings for `?subject`, `?resource`, `?action`, `?application`, `?now`, `?ip`, etc.).

**Predicates.** If `C` is a predicate invocation `P(→t)` and `P` is:
- **implementedByQuery** `Q`: evaluate `ASK Q` over `AG ∪ RC`. Result is the boolean outcome.
- **implementedByShape** `S`: run SHACL validation of the designated focus node(s) drawn from parameters; result is `true` iff `S` conforms.
- **implementedByFunction** `F`: evaluate `F(→t, RC)`; must be pure and total; non-success, timeout, or ill-typed ⇒ `false`.

**Booleans.**
- [[allOf(C1,…,Cn)]] = ∧_i [[Ci]]
- [[anyOf(C1,…,Cn)]] = ∨_i [[Ci]]
- [[not(C)]] = ¬ [[C]]

**Quantifiers.** Let `B` be a **bindings query** associated with the quantifier node (see §8.1). Let `Ω = eval(B, AG ∪ RC)` be the finite set of solution mappings.
- [[exists B . C(→v)]] = (∃ µ ∈ Ω) . [[C(→vµ)]]
- [[forAll B . C(→v)]] = (∀ µ ∈ Ω) . [[C(→vµ)]]
If `Ω = ∅`, then `exists` ⇒ `false`, `forAll` ⇒ `true` (vacuous truth).

**Errors & Unknowns.** Any missing predicate, malformed query/shape/function, or unresolved parameter yields `false`. Policies are evaluated under **closed-world, deny-by-default** semantics.

### 8.1 Quantifier Binding Surface
We add the following vocabulary for quantified conditions:
```turtle
# Vocabulary additions
lws:bindingsQuery a rdf:Property .  # points to lws:Query producing rows
lws:var a rdf:Property .            # names of variables bound by the query
lws:sparql a rdf:Property .         # the query text (ASK/SELECT)
```
Example:
```turtle
<#allCollabs> a lws:Query ;
  lws:sparql """
    PREFIX ex: <https://example.org/>
    SELECT ?c WHERE { ?resource ex:collaborator ?c }
  """ .

# forAll { ?c in allCollabs } . staff(?c)
[] a lws:Condition ;
   lws:forAll [ lws:bindingsQuery <#allCollabs> ; lws:var "c" ;
                lws:predicate [ a lws:Predicate ;
                  lws:implementedByQuery [ a lws:Query ; lws:sparql """
                    PREFIX ex: <https://example.org/>
                    ASK WHERE { ?c a ex:Staff }
                  """ ] ] ] .
```

## 9. Reference JSON-LD Frame (Normalization)
The following frame normalizes a Policy document for portable storage and signature:
```json
{
  "@context": {
    "@vocab": "https://www.w3.org/ns/lws-apl#",
    "lws": "https://www.w3.org/ns/lws-apl#",
    "dpv": "https://w3id.org/dpv#"
  },
  "@type": "Policy",
  "target": {
    "@type": "Target",
    "subject": {},
    "resource": {},
    "action": {}
  },
  "rule": {
    "@type": "Rule",
    "effect": {},
    "condition": {
      "@embed": "@always",
      "predicate": {
        "@type": "Predicate",
        "implementedByQuery": {
          "@type": "Query",
          "sparql": {}
        }
      },
      "allOf": [],
      "anyOf": [],
      "not": {}
    }
  },
  "obligation": {
    "@type": "Obligation",
    "mustLog": {}
  }
}
```
**Notes.**
- Frame is illustrative; implementers MAY extend it to include `implementedByShape/Function`, `exists/forAll` blocks, and provenance on input assertions.
- For signing, use JSON-LD Canonicalization + Data Integrity or JOSE with detached hash over canonized RDF dataset.

