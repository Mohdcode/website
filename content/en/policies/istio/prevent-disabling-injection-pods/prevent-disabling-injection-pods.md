---
title: "Prevent Disabling Istio Sidecar Injection"
category: Istio
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    One way sidecar injection in an Istio service mesh may be accomplished is by defining an annotation at the Pod level. Pods not receiving a sidecar cannot participate in the mesh thereby reducing visibility. This policy ensures that Pods cannot set the annotation `sidecar.istio.io/inject` to a value of `false`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//istio/prevent-disabling-injection-pods/prevent-disabling-injection-pods.yaml" target="-blank">/istio/prevent-disabling-injection-pods/prevent-disabling-injection-pods.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: prevent-disabling-injection-pods
  annotations:
    policies.kyverno.io/title: Prevent Disabling Istio Sidecar Injection
    policies.kyverno.io/category: Istio
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.8.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.24"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      One way sidecar injection in an Istio service mesh may be accomplished is by defining
      an annotation at the Pod level. Pods not receiving a sidecar cannot participate in the mesh
      thereby reducing visibility. This policy ensures that Pods cannot set the annotation
      `sidecar.istio.io/inject` to a value of `false`.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: prohibit-inject-annotation
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "Pods may not disable sidecar injection by setting the annotation sidecar.istio.io/inject to a value of false."
      pattern:
        metadata:
          =(annotations):
            =(sidecar.istio.io/inject): "!false"
```
