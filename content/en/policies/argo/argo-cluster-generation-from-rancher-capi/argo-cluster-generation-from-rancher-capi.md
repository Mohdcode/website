---
title: "Argo Cluster Secret Generation From Rancher CAPI Secret"
category: Argo
version: 1.7.0
subject: Secret
policyType: "generate"
description: >
    This policy generates and synchronizes Argo CD cluster secrets from Rancher  managed cluster.provisioning.cattle.io/v1 resources and their corresponding CAPI secrets. In this solution, Argo CD integrates with Rancher managed clusters via the central Rancher authentication proxy which shares the network endpoint of the Rancher API/GUI. The policy implements work-arounds for Argo CD issue https://github.com/argoproj/argo-cd/issues/9033 "Cluster-API cluster auto-registration" and Rancher issue https://github.com/rancher/rancher/issues/38053 "Fix type and labels Rancher v2 provisioner specifies when creating CAPI Cluster Secret".
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//argo/argo-cluster-generation-from-rancher-capi/argo-cluster-generation-from-rancher-capi.yaml" target="-blank">/argo/argo-cluster-generation-from-rancher-capi/argo-cluster-generation-from-rancher-capi.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: argo-cluster-generation-from-rancher-capi
  annotations:
    policies.kyverno.io/title: Argo Cluster Secret Generation From Rancher CAPI Secret
    policies.kyverno.io/category: Argo
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Secret
    kyverno.io/kyverno-version: 1.7.1
    policies.kyverno.io/minversion: 1.7.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/description: >-
      This policy generates and synchronizes Argo CD cluster secrets from Rancher 
      managed cluster.provisioning.cattle.io/v1 resources and their corresponding CAPI secrets.
      In this solution, Argo CD integrates with Rancher managed clusters via the central
      Rancher authentication proxy which shares the network endpoint of the Rancher API/GUI.
      The policy implements work-arounds for Argo CD issue https://github.com/argoproj/argo-cd/issues/9033
      "Cluster-API cluster auto-registration" and Rancher issue https://github.com/rancher/rancher/issues/38053
      "Fix type and labels Rancher v2 provisioner specifies when creating CAPI Cluster Secret".
spec:
  generateExisting: true
  rules:
  - name: source-rancher-non-local-cluster-and-capi-secret
    match:
      all:
      - resources:
          kinds:
          - provisioning.cattle.io/v1/Cluster
    exclude:
      any:
      - resources:
          namespaces:
          - fleet-local
    context:
    - name: clusterName
      variable:
        value: "{{request.object.metadata.name}}"
        jmesPath: 'to_string(@)'
    - name: clusterPrefixedName
      variable:
        value: "{{ join('-', ['cluster', clusterName]) }}"
        jmesPath: 'to_string(@)'
    - name: kubeconfigName
      variable:
        value: "{{ join('-', [clusterName, 'kubeconfig']) }}"
        jmesPath: 'to_string(@)'
    - name: extraLabels
      variable:
        value:
          argocd.argoproj.io/secret-type: cluster
          clusterId: "{{ clusterName }}"
    - name: metadataLabels
      variable:
        jmesPath: request.object.metadata.labels
        default: {}
    - name: metadataLabels
      variable:
        jmesPath: merge(metadataLabels, extraLabels)
    - name: kubeconfigData
      apiCall:
        urlPath: "/api/v1/namespaces/{{request.object.metadata.namespace}}/secrets/{{kubeconfigName}}"
        jmesPath: 'data | to_string(@)'
    - name: serverName
      variable:
        value: "{{ kubeconfigData | parse_yaml(@).value | base64_decode(@) | parse_yaml(@).clusters[0].cluster.server }}"
        jmesPath: 'to_string(@)'
    - name: bearerToken
      variable:
        value: "{{ kubeconfigData | parse_yaml(@).token | base64_decode(@) }}"
        jmesPath: 'to_string(@)'
    - name: caData
      variable:
        value: "{{ kubeconfigData | parse_yaml(@).value | base64_decode(@) | parse_yaml(@).clusters[0].cluster.\"certificate-authority-data\" }}"
        jmesPath: 'to_string(@)'
    - name: dataConfig
      variable:
        value: |
          {
            "bearerToken": "{{ bearerToken }}",
            "tlsClientConfig": {
              "insecure": false,
              "caData": "{{ caData }}"
            }
          }
        jmesPath: 'to_string(@)'
    generate:
      synchronize: true
      apiVersion: v1
      kind: Secret
      name: "{{ clusterPrefixedName }}"
      namespace: argocd
      data:
        metadata:
          labels:
            "{{ metadataLabels }}"
        type: Opaque
        data:
          name: "{{ clusterPrefixedName | base64_encode(@) }}"
          server: "{{ serverName | base64_encode(@) }}"
          config: "{{ dataConfig | base64_encode(@) }}"

```
