# Applying Policies in Kubernetes using Open Policy Agent (OPA) Gatekeeper

![Open Policy Agent Gatekeeper](https://kubernetes.io/images/blog/2019-08-06-opa-gatekeeper/v3.png)

## Introduction
This guide will walk you through the process of applying policies in Kubernetes using Open Policy Agent (OPA) Gatekeeper. OPA Gatekeeper is an open-source tool that allows you to enforce policies and validate configurations in a Kubernetes cluster.

## Prerequisites
Before getting started, ensure that you have the following prerequisites:
- A Kubernetes cluster up and running.
- `kubectl` command-line tool installed and configured to connect to your cluster.
- Basic knowledge of Kubernetes concepts and resources.

## Step 1: Install OPA Gatekeeper
To install OPA Gatekeeper, follow these steps:
1. Clone the Gatekeeper repository from GitHub: `git clone https://github.com/open-policy-agent/gatekeeper`.
2. Change directory to the cloned repository: `cd gatekeeper`.
3. Run the installation script: `./gatekeeper.sh install`.

## Step 2: Define Policies
Once OPA Gatekeeper is installed, you need to define the policies you want to enforce. Policies are written in Rego, a declarative policy language used by OPA. Here's an example policy that ensures all pods have resource limits defined:

```rego
package k8srequiredlabels

violation[{"msg": msg, "details": {"missing_labels": missing_labels}}] {
    provided := {label | input.review.object.metadata.labels[label]}
    required := {"app", "env", "tier"}
    missing_labels := required - provided
    count(missing_labels) > 0
    msg := sprintf("Pod must have labels: %v", [missing_labels])
}
```

Save the policy in a file named `require-resource-limits.rego`.

## Step 3: Create Constraint Templates
Constraint Templates define the structure and schema for the policies you want to enforce. Create a Constraint Template YAML file (e.g., `require-resource-limits-template.yaml`) with the following content:

```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        # Schema for the `parameters` field
        openAPIV3Schema:
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels

        violation[{"msg": msg, "details": {"missing_labels": missing_labels}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing_labels := required - provided
          count(missing_labels) > 0
          msg := sprintf("Pod must have labels: %v", [missing_labels])
        }
```

## Step 4: Create Constraint Instances
Constraint Instances are created from Constraint Templates and applied to Kubernetes resources. Create a Constraint Instance YAML file (e.g., `require-resource-limits-instance.yaml`) with the following content:

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-resource-limits
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  parameters:
    labels:
      - app
      - env
      - tier
```

## Step 5: Apply the Policies


To apply the policies, follow these steps:
1. Apply the Constraint Template: `kubectl apply -f require-resource-limits-template.yaml`.
2. Apply the Constraint Instance: `kubectl apply -f require-resource-limits-instance.yaml`.

## Conclusion
You have successfully applied policies in Kubernetes using OPA Gatekeeper. This allows you to enforce rules and validate configurations in your Kubernetes cluster, ensuring compliance with your organization's standards and best practices.

For more information and advanced usage of OPA Gatekeeper, refer to the official documentation and examples on the [Gatekeeper GitHub repository](https://github.com/open-policy-agent/gatekeeper).