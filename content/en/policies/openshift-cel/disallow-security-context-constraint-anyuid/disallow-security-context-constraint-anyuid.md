---
title: "Disallow use of the SecurityContextConstraint (SCC) anyuid in CEL expressions"
category: Security in CEL
version: 1.11.0
subject: Role,ClusterRole,RBAC
policyType: "validate"
description: >
    Disallow the use of the SecurityContextConstraint (SCC) anyuid which allows a pod to run with the UID as declared in the image instead of a random UID
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//openshift-cel/disallow-security-context-constraint-anyuid/disallow-security-context-constraint-anyuid.yaml" target="-blank">/openshift-cel/disallow-security-context-constraint-anyuid/disallow-security-context-constraint-anyuid.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-security-context-constraint-anyuid
  annotations:
    policies.kyverno.io/title: Disallow use of the SecurityContextConstraint (SCC) anyuid in CEL expressions
    policies.kyverno.io/category: Security in CEL 
    policies.kyverno.io/severity: high
    kyverno.io/kyverno-version: 1.11.0
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/subject: Role,ClusterRole,RBAC
    policies.kyverno.io/description: >-
      Disallow the use of the SecurityContextConstraint (SCC) anyuid which allows a pod to run with the UID as declared in the image instead of a random UID
spec:
  validationFailureAction: Enforce
  background: true
  rules:
  - name: check-security-context-constraint
    match:
      any:
      - resources:
          kinds:
          - ClusterRole
          - Role
          operations:
          - CREATE
          - UPDATE
    validate: 
      cel:
        expressions:
          - expression: "!object.?rules.orValue([]).exists(rule, 'anyuid' in rule.resourceNames && ('use' in rule.verbs || '*' in rule.verbs))"
            message: >-
              Use of the SecurityContextConstraint (SCC) anyuid is not allowed
  - name: check-security-context-roleref
    match:
      any:
      - resources:
          kinds:
          - ClusterRoleBinding
          - RoleBinding
          operations:
          - CREATE
          - UPDATE
    validate:
      cel:
        expressions:
          - expression: "object.roleRef.name != 'system:openshift:scc:anyuid'"
            message: >-
              Use of the SecurityContextConstraint (SCC) anyuid is not allowed


```
