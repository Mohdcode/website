---
title: "Metadata Matches Regex"
category: Other
version: 
subject: Pod, Label
policyType: "validate"
description: >
    Rather than a simple check to see if given metadata such as labels and annotations are present, in some cases they need to be present and the values match a specified regular expression. This policy illustrates how to ensure a label with key `corp.org/version` is both present and matches a given regex, in this case ensuring semver is met.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/metadata-match-regex/metadata-match-regex.yaml" target="-blank">/other/metadata-match-regex/metadata-match-regex.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: metadata-match-regex
  annotations:
    policies.kyverno.io/title: Metadata Matches Regex
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod, Label
    policies.kyverno.io/description: >-
      Rather than a simple check to see if given metadata such as labels and annotations are present,
      in some cases they need to be present and the values match a specified regular expression. This
      policy illustrates how to ensure a label with key `corp.org/version` is both present and matches
      a given regex, in this case ensuring semver is met.
spec:
  validationFailureAction: Audit
  background: false
  rules:
  - name: check-for-regex
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: >-
        The label `corp.org/version` is required and must match the specified regex: ^v[0-9].[0-9].[0-9]$
      deny:
        conditions:
          all:
          - key: "{{ regex_match('^v[0-9].[0-9].[0-9]$','{{request.object.metadata.labels.\"corp.org/version\" || 'empty'}}') }}"
            operator: Equals
            value: false

```
