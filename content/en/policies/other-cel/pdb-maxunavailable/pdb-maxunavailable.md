---
title: "PodDisruptionBudget maxUnavailable Non-Zero in CEL expressions"
category: Other in CEL
version: 
subject: PodDisruptionBudget
policyType: "validate"
description: >
    A PodDisruptionBudget which sets its maxUnavailable value to zero prevents all voluntary evictions including Node drains which may impact maintenance tasks. This policy enforces that if a PodDisruptionBudget specifies the maxUnavailable field it must be greater than zero.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/pdb-maxunavailable/pdb-maxunavailable.yaml" target="-blank">/other-cel/pdb-maxunavailable/pdb-maxunavailable.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: pdb-maxunavailable
  annotations:
    policies.kyverno.io/title: PodDisruptionBudget maxUnavailable Non-Zero in CEL expressions
    policies.kyverno.io/category: Other in CEL 
    kyverno.io/kyverno-version: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/subject: PodDisruptionBudget
    policies.kyverno.io/description: >-
      A PodDisruptionBudget which sets its maxUnavailable value to zero prevents
      all voluntary evictions including Node drains which may impact maintenance tasks.
      This policy enforces that if a PodDisruptionBudget specifies the maxUnavailable field
      it must be greater than zero.
spec:
  validationFailureAction: Audit
  background: false
  rules:
    - name: pdb-maxunavailable
      match:
        any:
          - resources:
              kinds:
                - PodDisruptionBudget
              operations:
              - CREATE
              - UPDATE
      validate:
        cel:
          expressions:
            - expression: "int(object.spec.?maxUnavailable.orValue(1)) > 0"
              message: "The value of maxUnavailable must be greater than zero."


```
