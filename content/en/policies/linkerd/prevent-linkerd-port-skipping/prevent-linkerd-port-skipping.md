---
title: "Prevent Linkerd Port Skipping"
category: Linkerd
version: 
subject: Pod
policyType: "validate"
description: >
    Linkerd has the ability to skip inbound and outbound ports assigned to Pods, exempting them from mTLS. This can be important in some narrow use cases but generally should be avoided. This policy prevents Pods from setting the annotations `config.linkerd.io/skip-inbound-ports` or `config.linkerd.io/skip-outbound-ports`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//linkerd/prevent-linkerd-port-skipping/prevent-linkerd-port-skipping.yaml" target="-blank">/linkerd/prevent-linkerd-port-skipping/prevent-linkerd-port-skipping.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: prevent-linkerd-port-skipping
  annotations:
    policies.kyverno.io/title: Prevent Linkerd Port Skipping
    policies.kyverno.io/category: Linkerd
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Linkerd has the ability to skip inbound and outbound ports assigned to Pods, exempting
      them from mTLS. This can be important in some narrow use cases but
      generally should be avoided. This policy prevents Pods from setting
      the annotations `config.linkerd.io/skip-inbound-ports` or `config.linkerd.io/skip-outbound-ports`.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: pod-prevent-port-skipping
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "Pods may not skip ports. The annotations `config.linkerd.io/skip-inbound-ports` or `config.linkerd.io/skip-outbound-ports` must not be set."
      pattern:
        metadata:
          =(annotations):
            X(config.linkerd.io/skip-inbound-ports): "null"
            X(config.linkerd.io/skip-outbound-ports): "null"
```
