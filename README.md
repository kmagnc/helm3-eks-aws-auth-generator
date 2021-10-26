# helm3-eks-aws-auth-generator
Uses helm3 templating to dynamically generate an EKS aws-auth configmap

# The Problem
aws-auth is a configmap automatically created when an EKS cluster is provisioned. It is not owned by helm, so it cannot be overriden by a helm chart. There are a few dynamic solutions to updating it. This is just another one.

# The Solution
- create a [helm template](https://helm.sh/docs/helm/helm/) of the [aws-auth configmap](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html) (yamlish syntax)
- use [helm3 template command](https://helm.sh/docs/helm/helm_template/) to generate the configmap manifest
- kubectl patch the dynamically generated manifest

# Usage

## Input Values

| Name                        | Description                                     | Value                                      |
| --------------------------- | ----------------------------------------------- | ------------------------------------------ |
| `eksNodeIAMRole`            | ARN of the EKS cluster Node IAM Role            | `"arn:aws:iam::000000000000:/role/role1"`  |
| `eksAdminUsersIAMRole`      | ARN of your admin user's IAM Role               | `"arn:aws:iam::000000000000:/role/role2"`  |
| `eksNamespaceAUsersIAMRole` | ARN of Namespace-A-Only user's IAM Role         | `"arn:aws:iam::000000zero00:/role/role3"`  |

## Using the chart

```bash
helm template me . --debug --output-dir . 
```

```bash
install.go:178: [debug] Original chart version: ""
install.go:199: [debug] CHART PATH: ${pwd}/helm3-eks-aws-auth-generator

wrote ./aws-auth-generator/templates/configmap.yaml
```

## Patch the existing configmap

:warning: **This can break your EKS cluster** :warning: If the EKS Node IAM Role ARN isn't exactly right, the EKS apiserver can no longer communicate with the nodes, and they will all lose their `Ready` status. Test on a disposable cluster.

```bash
k patch cm -n kube-system aws-auth -p "$(cat aws-auth-generator/templates/configmap.yaml)"
```