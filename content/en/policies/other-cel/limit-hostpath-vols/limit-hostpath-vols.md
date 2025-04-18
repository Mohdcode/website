---
title: "Limit hostPath Volumes to Specific Directories in CEL expressions"
category: Other in CEL
version: 1.11.0
subject: Pod
policyType: "validate"
description: >
    hostPath volumes consume the underlying node's file system. If hostPath volumes are not to be universally disabled, they should be restricted to only certain host paths so as not to allow access to sensitive information. This policy ensures the only directory that can be mounted as a hostPath volume is /data. It is strongly recommended to pair this policy with a second to ensure readOnly access is enforced preventing directory escape.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/limit-hostpath-vols/limit-hostpath-vols.yaml" target="-blank">/other-cel/limit-hostpath-vols/limit-hostpath-vols.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: limit-hostpath-vols
  annotations:
    policies.kyverno.io/title: Limit hostPath Volumes to Specific Directories in CEL expressions
    policies.kyverno.io/category: Other in CEL
    policies.kyverno.io/severity: medium
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kyverno-version: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      hostPath volumes consume the underlying node's file system. If hostPath volumes
      are not to be universally disabled, they should be restricted to only certain
      host paths so as not to allow access to sensitive information. This policy ensures
      the only directory that can be mounted as a hostPath volume is /data. It is strongly
      recommended to pair this policy with a second to ensure readOnly
      access is enforced preventing directory escape.
spec:
  background: false
  validationFailureAction: Audit
  rules:
  - name: limit-hostpath-to-slash-data
    match:
      any:
      - resources:
          kinds:
          - Pod
          operations:
          - CREATE
          - UPDATE
    celPreconditions:
      - name: "has-host-path-volume"
        expression: "object.spec.?volumes.orValue([]).exists(volume, has(volume.hostPath))"
    validate:
      cel:
        expressions:
          - expression: "object.spec.volumes.all(volume, !has(volume.hostPath) || volume.hostPath.path.split('/')[1] == 'data')"
            message: hostPath volumes are confined to /data.


```
