---
title: "Require StorageClass in CEL expressions"
category: Other, Multi-Tenancy in CEL
version: 
subject: PersistentVolumeClaim, StatefulSet
policyType: "validate"
description: >
    PersistentVolumeClaims (PVCs) and StatefulSets may optionally define a StorageClass to dynamically provision storage. In a multi-tenancy environment where StorageClasses are far more common, it is often better to require storage only be provisioned from these StorageClasses. This policy requires that PVCs and StatefulSets containing volumeClaimTemplates define the storageClassName field with some value.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/require-storageclass/require-storageclass.yaml" target="-blank">/other-cel/require-storageclass/require-storageclass.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-storageclass
  annotations:
    policies.kyverno.io/title: Require StorageClass in CEL expressions
    policies.kyverno.io/category: Other, Multi-Tenancy in CEL 
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: PersistentVolumeClaim, StatefulSet
    kyverno.io/kyverno-version: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      PersistentVolumeClaims (PVCs) and StatefulSets may optionally define a StorageClass
      to dynamically provision storage. In a multi-tenancy environment where StorageClasses are
      far more common, it is often better to require storage only be provisioned from these
      StorageClasses. This policy requires that PVCs and StatefulSets containing
      volumeClaimTemplates define the storageClassName field with some value.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: pvc-storageclass
    match:
      any:
      - resources:
          kinds:
          - PersistentVolumeClaim
          operations:
          - CREATE
          - UPDATE
    validate:
      cel:
        expressions:
          - expression: "object.spec.?storageClassName.orValue('') != ''"
            message: "PersistentVolumeClaims must define a storageClassName."
  - name: ss-storageclass
    match:
      any:
      - resources:
          kinds:
          - StatefulSet
          operations:
          - CREATE
          - UPDATE
    validate:
      cel:
        expressions:
          - expression: >-
              !has(object.spec.volumeClaimTemplates) || 
              object.spec.volumeClaimTemplates.all(volumeClaimTemplate, 
              volumeClaimTemplate.spec.?storageClassName.orValue('')  != '')
            message: "StatefulSets must define a storageClassName."


```
