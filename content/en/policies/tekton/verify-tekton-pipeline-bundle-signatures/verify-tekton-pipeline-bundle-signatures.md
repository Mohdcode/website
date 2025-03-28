---
title: "Require Signed Tekton Pipeline"
category: Tekton
version: 1.7.0
subject: PipelineRun
policyType: "verifyImages"
description: >
    A signed bundle is required
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//tekton/verify-tekton-pipeline-bundle-signatures/verify-tekton-pipeline-bundle-signatures.yaml" target="-blank">/tekton/verify-tekton-pipeline-bundle-signatures/verify-tekton-pipeline-bundle-signatures.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: signed-pipeline-bundle
  annotations:
    policies.kyverno.io/title: Require Signed Tekton Pipeline
    policies.kyverno.io/category: Tekton
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: PipelineRun
    kyverno.io/kyverno-version: 1.7.2
    policies.kyverno.io/minversion: 1.7.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/description: >- 
      A signed bundle is required
spec:
  validationFailureAction: Enforce
  webhookTimeoutSeconds: 30
  rules:
  - name: check-signature
    match:
      resources:
        kinds:
        - PipelineRun
    imageExtractors:
      PipelineRun:
        - name: "pipelineruns"
          path: /spec/pipelineRef
          value: "bundle"
          key: "name"
    verifyImages:
    - imageReferences:
      - "*"
      attestors:
      - entries:
        - keys: 
            publicKeys: |-
              -----BEGIN PUBLIC KEY-----
              MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEahmSvGFmxMJABilV1usgsw6ImcQ/
              gDaxw57Sq+uNGHW8Q3zUSx46PuRqdTI+4qE3Ng2oFZgLMpFN/qMrP0MQQg==
              -----END PUBLIC KEY-----
```
