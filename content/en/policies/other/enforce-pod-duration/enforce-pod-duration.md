---
title: "Enforce pod duration"
category: Sample
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    This validation is valuable when annotations are used to define durations, such as to ensure a Pod lifetime annotation does not exceed some site specific max threshold. Pod lifetime annotation can be no greater than 8 hours.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/enforce-pod-duration/enforce-pod-duration.yaml" target="-blank">/other/enforce-pod-duration/enforce-pod-duration.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: pod-lifetime
  annotations:
    policies.kyverno.io/title: Enforce pod duration
    policies.kyverno.io/category: Sample
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      This validation is valuable when annotations are used to define durations,
      such as to ensure a Pod lifetime annotation does not exceed some site specific max threshold.
      Pod lifetime annotation can be no greater than 8 hours.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: pods-lifetime
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "Pod lifetime exceeds limit of 8h"
      deny:
        conditions:
          any:
          - key: "{{ request.object.metadata.annotations.\"pod.kubernetes.io/lifetime\" || '0s' }}"
            operator: GreaterThan
            value: "8h"

```
