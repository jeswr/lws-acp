# Access Control Policies for Linked Web Storage

## Executive Summary

This document provides a comprehensive analysis of access control mechanisms across major cloud storage providers and existing Solid specifications (WAC and ACP), proposing a unified access control model for the Linked Web Storage (LWS) specification that can be adopted by all drive providers while maintaining compatibility with existing systems.

## Table of Contents

1. [Introduction](#introduction)
2. [Analysis of Existing Access Control Systems](#analysis-of-existing-access-control-systems)
3. [Common Patterns and Requirements](#common-patterns-and-requirements)
4. [Proposed LWS Access Control Model](#proposed-lws-access-control-model)
5. [Implementation Considerations](#implementation-considerations)
6. [Migration Path](#migration-path)
7. [Security Considerations](#security-considerations)
8. [Appendices](#appendices)

## Introduction

The Linked Web Storage (LWS) specification aims to provide a standardized interface for personal cloud storage that can be implemented by existing providers while supporting advanced features like semantic data sharing and agentic access. A critical component of this specification is a unified access control model that:

1. Can be implemented by major cloud storage providers (Google Drive, Microsoft OneDrive, Dropbox, AWS S3, etc.)
2. Supports the rich access control features of Solid (WAC and ACP)
3. Enables fine-grained permissions suitable for agentic access
4. Maintains backward compatibility with existing systems
5. Supports credential storage use cases

## Analysis of Existing Access Control Systems

### 1. Google Drive

**Key Features:**
- Role-based permissions: owner, organizer, fileOrganizer, writer, commenter, reader
- File and folder-level permissions
- Domain-wide sharing capabilities
- Link sharing with expiration dates
- Permission inheritance from parent folders
- Supports both user and group principals

**API Model:**
```json
{
  "kind": "drive#permission",
  "id": "string",
  "type": "user|group|domain|anyone",
  "role": "owner|organizer|fileOrganizer|writer|commenter|reader",
  "emailAddress": "string",
  "domain": "string",
  "allowFileDiscovery": boolean,
  "expirationTime": "datetime"
}
```

### 2. Microsoft OneDrive (Graph API)

**Key Features:**
- Role-based permissions: owner, write, read
- Sharing links with various permission levels
- Inheritance from parent containers
- Integration with Azure AD for enterprise scenarios
- Support for anonymous sharing via links
- Granular permissions through SharePoint integration

**API Model:**
```json
{
  "id": "string",
  "roles": ["read", "write", "owner"],
  "grantedTo": {
    "user": {
      "id": "string",
      "displayName": "string"
    }
  },
  "link": {
    "type": "view|edit|embed",
    "scope": "anonymous|organization|users"
  }
}
```

### 3. Dropbox

**Key Features:**
- Member-based permissions: owner, editor, viewer
- Folder and file-level access control
- Shared folder membership
- Link sharing with passwords and expiration
- Team folders with inherited permissions
- Viewer info restrictions

**API Model:**
```json
{
  "access_type": {
    ".tag": "owner|editor|viewer"
  },
  "user": {
    "account_id": "string",
    "email": "string"
  },
  "permissions": ["can_edit", "can_share", "can_view_metadata"],
  "is_inherited": boolean
}
```

### 4. AWS S3

**Key Features:**
- IAM policies with fine-grained actions
- Bucket policies for resource-based permissions
- Access Control Lists (ACLs) for legacy support
- Condition-based access control
- Cross-account access
- Temporary credentials via STS

**Policy Model:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow|Deny",
    "Principal": {"AWS": ["arn:aws:iam::123456789012:user/username"]},
    "Action": ["s3:GetObject", "s3:PutObject"],
    "Resource": ["arn:aws:s3:::bucket-name/*"],
    "Condition": {
      "StringEquals": {
        "s3:x-amz-acl": "public-read"
      }
    }
  }]
}
```

### 5. Solid Web Access Control (WAC)

**Key Features:**
- RDF-based access control
- Mode-based permissions: Read, Write, Append, Control
- Agent classes: Agent, AuthenticatedAgent, Origin
- Inheritance through default rules
- Resource and container-level controls

**RDF Model:**
```turtle
@prefix acl: <http://www.w3.org/ns/auth/acl#>.

<#authorization>
    a acl:Authorization;
    acl:agent <https://alice.example/profile#me>;
    acl:mode acl:Read, acl:Write;
    acl:accessTo <./resource.ttl>.
```

### 6. Solid Access Control Policy (ACP)

**Key Features:**
- Policy-based access control
- Matcher-based agent selection
- Allow/deny semantics
- Context-aware access control
- Verifiable Credentials support
- Policy composition and reuse

**RDF Model:**
```turtle
@prefix acp: <http://www.w3.org/ns/solid/acp#>.

<#policy>
    a acp:Policy;
    acp:allow acl:Read;
    acp:anyOf [
        acp:agent <https://alice.example/profile#me>
    ].
```

## Common Patterns and Requirements

### Universal Concepts

1. **Principals/Agents**
   - Individual users (authenticated by various means)
   - Groups/teams
   - Applications/services
   - Anonymous/public access
   - Authenticated but unspecified users

2. **Permissions/Modes**
   - Read (view content)
   - Write (modify content)
   - Delete (remove content)
   - Share (grant permissions to others)
   - Control/Admin (modify access control)

3. **Resources**
   - Individual files/documents
   - Folders/containers
   - Metadata
   - Access control resources themselves

4. **Inheritance**
   - All systems support some form of permission inheritance
   - Parent-to-child propagation is standard
   - Override mechanisms vary

### Divergent Approaches

1. **Grant Model**
   - Role-based (Google, OneDrive, Dropbox)
   - Action-based (AWS S3)
   - Mode-based (Solid WAC/ACP)

2. **Policy Evaluation**
   - Implicit deny (most providers)
   - Explicit allow/deny (AWS S3, ACP)
   - Default inheritance rules vary

3. **Sharing Mechanisms**
   - Direct user grants
   - Shareable links
   - Token-based access
   - Credential-based access

## Proposed LWS Access Control Model

### Design Principles

1. **Compatibility First**: The model must be implementable as a facade over existing provider APIs
2. **Semantic Clarity**: Use RDF/Linked Data for interoperability
3. **Progressive Enhancement**: Support basic features universally, advanced features optionally
4. **Security by Default**: Fail closed, explicit grants required
5. **Auditability**: Clear authorization paths

### Core Vocabulary

```turtle
@prefix lws: <http://www.w3.org/ns/lws#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

# Core Classes
lws:Authorization a rdfs:Class ;
    rdfs:label "Authorization" ;
    rdfs:comment "A grant of access to a resource" .

lws:Policy a rdfs:Class ;
    rdfs:label "Policy" ;
    rdfs:comment "A reusable access control policy" .

lws:Principal a rdfs:Class ;
    rdfs:label "Principal" ;
    rdfs:comment "An entity that can be granted access" .

lws:Resource a rdfs:Class ;
    rdfs:label "Resource" ;
    rdfs:comment "A resource that can be accessed" .

# Permission Modes (Core Set)
lws:Read a lws:PermissionMode ;
    rdfs:label "Read" ;
    rdfs:comment "Permission to read/view resource content" .

lws:Write a lws:PermissionMode ;
    rdfs:label "Write" ;
    rdfs:comment "Permission to modify resource content" .

lws:Append a lws:PermissionMode ;
    rdfs:label "Append" ;
    rdfs:comment "Permission to add to resource content" ;
    rdfs:subClassOf lws:Write .

lws:Delete a lws:PermissionMode ;
    rdfs:label "Delete" ;
    rdfs:comment "Permission to remove resources" .

lws:Control a lws:PermissionMode ;
    rdfs:label "Control" ;
    rdfs:comment "Permission to modify access control" .

# Properties
lws:grantedTo a rdf:Property ;
    rdfs:domain lws:Authorization ;
    rdfs:range lws:Principal .

lws:grants a rdf:Property ;
    rdfs:domain lws:Authorization ;
    rdfs:range lws:PermissionMode .

lws:target a rdf:Property ;
    rdfs:domain lws:Authorization ;
    rdfs:range lws:Resource .

lws:inheritsFrom a rdf:Property ;
    rdfs:domain lws:Authorization ;
    rdfs:range lws:Authorization .

lws:validFrom a rdf:Property ;
    rdfs:domain lws:Authorization ;
    rdfs:range xsd:dateTime .

lws:validUntil a rdf:Property ;
    rdfs:domain lws:Authorization ;
    rdfs:range xsd:dateTime .
```

### Authorization Model

#### Basic Authorization

```turtle
# Simple read access grant
<#auth1> a lws:Authorization ;
    lws:grantedTo <https://alice.example/profile#me> ;
    lws:grants lws:Read ;
    lws:target </storage/documents/report.pdf> .

# Group write access with expiration
<#auth2> a lws:Authorization ;
    lws:grantedTo <https://example.org/groups/editors> ;
    lws:grants lws:Read, lws:Write ;
    lws:target </storage/documents/> ;
    lws:validUntil "2024-12-31T23:59:59Z"^^xsd:dateTime .
```

#### Provider Mappings

| LWS Mode | Google Drive | OneDrive | Dropbox | AWS S3 | WAC | ACP |
|----------|--------------|----------|---------|---------|-----|-----|
| Read | reader | read | viewer | s3:GetObject | acl:Read | acp:Read |
| Write | writer | write | editor | s3:PutObject | acl:Write | acp:Write |
| Append | writer | write | editor | s3:PutObject | acl:Append | acp:Append |
| Delete | writer | write | editor | s3:DeleteObject | acl:Write | acp:Write |
| Control | owner | owner | owner | s3:PutBucketAcl | acl:Control | acp:Control |

### Advanced Features

#### 1. Conditional Access

```turtle
<#conditional-auth> a lws:Authorization ;
    lws:grantedTo lws:AuthenticatedAgent ;
    lws:grants lws:Read ;
    lws:target </storage/public/> ;
    lws:condition [
        lws:ipRange "192.168.1.0/24" ;
        lws:validDuring lws:BusinessHours
    ] .
```

#### 2. Delegated Access

```turtle
<#delegated-auth> a lws:Authorization ;
    lws:grantedTo <https://app.example/> ;
    lws:grants lws:Read, lws:Write ;
    lws:target </storage/app-data/> ;
    lws:onBehalfOf <https://alice.example/profile#me> ;
    lws:scope "calendar.read calendar.write" .
```

#### 3. Policy-Based Access

```turtle
<#policy1> a lws:Policy ;
    rdfs:label "Company Editors Policy" ;
    lws:grants lws:Read, lws:Write ;
    lws:condition [
        lws:memberOf <https://example.org/groups/employees> ;
        lws:emailDomain "example.org"
    ] .

<#auth3> a lws:Authorization ;
    lws:usePolicy <#policy1> ;
    lws:target </storage/company-docs/> .
```

### Access Evaluation Algorithm

```text
function evaluateAccess(principal, resource, mode):
    authorizations = findAuthorizations(resource)
    
    for auth in authorizations:
        if matchesPrincipal(auth, principal) and 
           grantsMode(auth, mode) and
           isValid(auth):
            if meetsConditions(auth, context):
                return ALLOW
    
    # Check inherited authorizations
    if parent = getParent(resource):
        return evaluateAccess(principal, parent, mode)
    
    return DENY
```

## Implementation Considerations

### 1. Provider Adapters

Each storage provider needs an adapter that:
- Translates LWS authorization to native permissions
- Maps native permissions to LWS model
- Handles capability differences gracefully

Example adapter interface:
```typescript
interface LWSAdapter {
  // Convert LWS authorization to provider-specific format
  toProviderAuth(auth: LWSAuthorization): ProviderPermission;
  
  // Convert provider permissions to LWS format
  fromProviderAuth(perm: ProviderPermission): LWSAuthorization;
  
  // Check if provider supports specific LWS features
  supportsFeature(feature: LWSFeature): boolean;
  
  // Apply authorization changes
  applyAuthorization(auth: LWSAuthorization): Promise<void>;
}
```

### 2. Capability Negotiation

Providers should advertise their capabilities:

```turtle
<https://storage.example/> a lws:StorageProvider ;
    lws:supportsMode lws:Read, lws:Write, lws:Delete, lws:Control ;
    lws:supportsFeature lws:ConditionalAccess, lws:GroupPrincipals ;
    lws:maxPrincipalsPerResource 100 ;
    lws:supportsInheritance true .
```

### 3. Performance Optimization

- Cache authorization decisions
- Batch permission checks
- Use provider-native bulk operations where available
- Implement efficient inheritance resolution

### 4. Audit and Compliance

```turtle
<#audit-entry> a lws:AuditEntry ;
    lws:timestamp "2024-01-15T10:30:00Z"^^xsd:dateTime ;
    lws:principal <https://alice.example/profile#me> ;
    lws:action lws:Read ;
    lws:resource </storage/documents/sensitive.pdf> ;
    lws:result lws:Allowed ;
    lws:authorization <#auth1> .
```

## Migration Path

### Phase 1: Basic Compatibility (Months 1-3)
- Implement core permission modes (Read, Write, Delete)
- Support user and group principals
- Basic resource-level permissions

### Phase 2: Enhanced Features (Months 4-6)
- Add Append and Control modes
- Implement inheritance
- Support time-based access

### Phase 3: Advanced Capabilities (Months 7-12)
- Conditional access
- Policy-based authorization
- Verifiable credentials integration

### Migration Tools

1. **Permission Mapper**: Converts existing provider permissions to LWS format
2. **Compatibility Checker**: Validates LWS policies against provider capabilities
3. **Batch Migrator**: Bulk permission migration utilities

## Security Considerations

### 1. Principle of Least Privilege
- Default to minimal access
- Require explicit grants
- Regular permission audits

### 2. Defense in Depth
- Multiple authorization checks
- Provider-level and LWS-level validation
- Fail-safe defaults

### 3. Attack Vectors
- **Privilege Escalation**: Prevented by Control mode separation
- **Inheritance Abuse**: Limited inheritance depth
- **Timing Attacks**: Consistent evaluation time
- **Delegation Chains**: Maximum delegation depth

### 4. Privacy Considerations
- Minimal principal information exposure
- Anonymous access options
- Audit log access controls

## Appendices

### Appendix A: Complete LWS Access Control Vocabulary

[Full RDF vocabulary definition would go here]

### Appendix B: Provider-Specific Mapping Tables

[Detailed mapping tables for each provider]

### Appendix C: Example Scenarios

#### Scenario 1: Personal Document Sharing
```turtle
# Alice shares a document with Bob for review
<#share1> a lws:Authorization ;
    lws:grantedTo <https://bob.example/profile#me> ;
    lws:grants lws:Read ;
    lws:target </alice/documents/proposal.pdf> ;
    lws:validUntil "2024-02-01T00:00:00Z"^^xsd:dateTime ;
    lws:note "For review - comments welcome" .
```

#### Scenario 2: Application Data Access
```turtle
# Calendar app gets access to calendar data
<#app-auth> a lws:Authorization ;
    lws:grantedTo <https://calendar.app/> ;
    lws:grants lws:Read, lws:Write ;
    lws:target </alice/data/calendar/> ;
    lws:onBehalfOf <https://alice.example/profile#me> ;
    lws:scope "calendar.read calendar.write" .
```

#### Scenario 3: Organization-Wide Policy
```turtle
# All employees can read company handbook
<#company-policy> a lws:Policy ;
    rdfs:label "Employee Handbook Access" ;
    lws:grants lws:Read ;
    lws:principalClass [
        a lws:GroupMembership ;
        lws:group <https://acme.corp/groups/all-employees>
    ] .

<#handbook-auth> a lws:Authorization ;
    lws:usePolicy <#company-policy> ;
    lws:target </company/handbook/> ;
    lws:inheritable true .
```

### Appendix D: Reference Implementation

[Link to reference implementation]

### Appendix E: Test Suite

[Description of conformance test suite]

## Conclusion

The proposed LWS Access Control Model provides a unified approach that:
1. Maps cleanly to existing provider models
2. Supports rich Solid-style access control
3. Enables progressive enhancement
4. Maintains security and privacy
5. Provides clear migration paths

This model balances the need for standardization with the reality of existing provider implementations, creating a foundation for interoperable personal data storage with sophisticated access control suitable for both human users and autonomous agents.