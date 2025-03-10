---
title: "Restrict Ingress Host with Wildcards in CEL expressions"
category: Other in CEL
version: 1.11.0
subject: Ingress
policyType: "validate"
description: >
    Ingress hosts optionally accept a wildcard as an alternative to precise matching. In some cases, this may be too permissive as it would direct unintended traffic to the given Ingress resource. This policy enforces that any Ingress host does not contain a wildcard character.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/restrict-ingress-wildcard/restrict-ingress-wildcard.yaml" target="-blank">/other-cel/restrict-ingress-wildcard/restrict-ingress-wildcard.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-ingress-wildcard
  annotations:
    policies.kyverno.io/title: Restrict Ingress Host with Wildcards in CEL expressions
    policies.kyverno.io/category: Other in CEL 
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.11.0
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/subject: Ingress
    policies.kyverno.io/description: >-
      Ingress hosts optionally accept a wildcard as an alternative
      to precise matching. In some cases, this may be too permissive as it
      would direct unintended traffic to the given Ingress resource. This
      policy enforces that any Ingress host does not contain a wildcard
      character.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: block-ingress-wildcard
      match:
        any:
        - resources:
            kinds:
              - Ingress
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
            - expression: "!object.spec.?rules.orValue([]).exists(rule, has(rule.host) && rule.host.contains('*'))"
              message: "Wildcards are not permitted as hosts."


```
