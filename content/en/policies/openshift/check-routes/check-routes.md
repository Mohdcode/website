---
title: "Require TLS routes in OpenShift"
category: OpenShift
version: 1.6.0
subject: Route
policyType: "validate"
description: >
    HTTP traffic is not encrypted and hence insecure. This policy prevents configuration of OpenShift HTTP routes.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//openshift/check-routes/check-routes.yaml" target="-blank">/openshift/check-routes/check-routes.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: check-routes
  annotations:
    policies.kyverno.io/title: Require TLS routes in OpenShift
    policies.kyverno.io/category: OpenShift
    policies.kyverno.io/severity: high
    kyverno.io/kyverno-version: 1.6.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.20"
    policies.kyverno.io/subject: Route
    policies.kyverno.io/description: |-
      HTTP traffic is not encrypted and hence insecure. This policy prevents configuration of OpenShift HTTP routes.
spec:
  validationFailureAction: Enforce
  background: true
  rules:
    - name: require-tls-routes
      match:
        any:
        - resources:
            kinds:
              - route.openshift.io/v1/Route
      preconditions:
        all:
        - key: "{{ request.operation || 'BACKGROUND' }}"
          operator: NotEquals
          value: ["DELETE"]
      validate:
        message: >-
          HTTP routes are not allowed. Configure TLS for secure routes.
        deny:
          conditions:
            all:
            - key: "{{ keys(request.object.spec) | contains(@, 'tls') }}"
              operator: Equals
              value: false


```
