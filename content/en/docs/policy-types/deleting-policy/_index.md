---
title: DeletingPolicy
description: >-
  Deletes pre-existing resources from cluster on scheduled time
weight: 20
---

`DeletingPolicy` enables the definition of label/field-matching rules that are evaluated on a schedule using CEL (Common Expression Language) expressions. If the conditions match, actions such as deletion can be performed. This resource is useful for garbage collection, automatic cleanup, or enforcement of lifecycle policies for Kubernetes resources.

## Introduction

`DeletingPolicy` is a custom resource designed to work similarly to admission policies but is triggered on a defined schedule rather than on admission events. It supports the following capabilities:

- Matching resources based on labels and field selectors

- CEL expressions for conditional logic

- Scheduled execution via cron syntax

- Reusable variable expressions

## Spec Fields
`schedule` 
The cron-formatted string that defines when the policy runs.
```yaml
schedule: "0 0 * * *" #everyday at midnight
```


`matchConditions` 
A list of CEL expressions that must evaluate to `true` for a resource to match.
```yaml
matchConditions:
  - name: isTestNamespace
    expression: "object.metadata.namespace.startsWith('test-')"
```

`namespaceSelector / objectSelector`
Label selectors to narrow down the scope of namespaces or objects the policy applies to.
```yaml
namespaceSelector:
  matchLabels:
    team: dev
```

`resourceRules`
Defines what API groups, versions, operations, and resources this policy matches.
```yaml
resourceRules:
  - apiGroups: ["apps"]
    apiVersions: ["v1"]
    operations: ["DELETE"]
    resources: ["deployments"]
    scope: "Namespaced"
```

`variables`
Reusable CEL expressions that can be referred in match conditions or actions.
```yaml
variables:
  - name: isOld
    expression: "now().getFullYear() > object.metadata.creationTimestamp.getFullYear() + 1"
```

`status`
This field contains the latest execution metadata and readiness of the policy

```yaml
status:
  conditionStatus:
    ready: true
    message: "Successfully applied"
    conditions:
      - type: Ready
        status: "True"
        reason: "AllMatched"
        message: "All conditions satisfied"
        lastTransitionTime: "2025-06-18T15:04:05Z"
```

## Example
```yaml
apiVersion: kyverno.io/v1alpha1
kind: DeletingPolicy
metadata:
  name: cleanup-old-test-pods
spec:
  schedule: "0 1 * * *" # Run daily at 1 AM
  namespaceSelector:
    matchLabels:
      environment: test
  matchConditions:
    - name: isOld
      expression: "now() - object.metadata.creationTimestamp > duration('72h')"
  resourceRules:
    - apiGroups: [""]
      apiVersions: ["v1"]
      operations: ["DELETE"]
      resources: ["pods"]
      scope: "Namespaced"
```