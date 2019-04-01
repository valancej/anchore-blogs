# Introduction to Kubernetes Security

## Background

Over the past couple of years the software community has seen the rise of [Kubernetes](https://kubernetes.io/). First developed by Google, Kubernetes is the most popular open-source container management tool which automates container deployment, container scaling, and container load balancing. A few of the major features and benefits of Kubernetes are: 

- Automatic Binpacking
- Service Discovery & Load Balancing
- Storage Orchestration
- Self Healing
- Horizonal Scaling

Kubernetes is also backed by a large community and hosted by the [Cloud Native Computing Foundation](https://www.cncf.io/). When organizations increase their use of containers some of the challenges they being to run into include: Automated scaling up and down of containers, container management and deployment, distributing load between containers, etc. To address these issues, it generally becomes necessary to implement a container orchestration platform to reduce operational burden. Kubernetes can be run on-premises or on any one of the major cloud providers (AWS, Azure, GCP, IBM Cloud).

Given that Kubernetes is already being used widely in production environments, securing these workloads should be a top priority. In this post, I will discuss a handful of common Kubernetes security basics and best practices to administer in order to avoid your clusters becoming compromised.  

### Staying up to date

As with any software component, updating to the latest version of software will greatly reduce the risk of your system being compromised. When running unpatched software components with known vulnerabilities, hackers are typically well-aware, and ready to exploit these weaknesses. One of the more recently well-known vulnerabilities discovered in Kubernetes was [CVE-2018-1002105](https://nvd.nist.gov/vuln/detail/CVE-2018-1002105). If you are running managed Kubernetes in a cloud provider, these managed service providers make it simple to upgrade to the latest version. In addition to running the latest version of Kubernetes, it is imperitive to stay up to date on the software components that make up the applications you are running. Providing your teams with the necessary tools for SAST for proprietary source code and container image scanning at the CI or container registry layer will help ensure that you are not running vulnerable software in Kubernetes environments.

### Role-based Access Control

Kubernetes RBAC allows users to exercise fine-grained control over how users access the API resources running on your cluster. Cloud providers will likely have RBAC enabled by default, but it is a good practice to check to verify that your Kubernetes deployment has enabled this feature. Generally speaking, it is good practice to apply the principle of least privilege to make sure users and services only have the access needed to do their jobs. You can create RBAC permission that apply to your entire cluster, or to specific namespaces within your cluster. For more information of RBAC in Kubernetes, I recommend reading the Kubernetes documentation on [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/). If you are using a cloud provider to manager Kubernetes I also recommend reading up on how each of these providers work with access and authentication as you may occasionally run into permission issues:

- [AWS Managing Cluster Authentication](https://docs.aws.amazon.com/eks/latest/userguide/managing-auth.html)
- [Making Sense of Kubernetes RBAC and IAM Roles on GKE](https://medium.com/uptime-99/making-sense-of-kubernetes-rbac-and-iam-roles-on-gke-914131b01922)
- [Access and identity options for Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/concepts-identity)

#### Role

In the Kubernetes RBAC API, a role contains rules that represent a set of permissions. Below is an example Role in the default namespace that can be used to grant read access to pods: 

```YAML
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

#### Principle of least privilege a step further

Above I mentioned applying the principle of least privilege to RBAC in Kubernetes. However, this same principle can be applied to your software components as well. By restricting access so components can only access the information and resources they need to operate correctly, the blast radius of attack is greatly reduced should one occur. 

### A change in security