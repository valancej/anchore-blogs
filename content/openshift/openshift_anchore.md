# Running Anchore Engine on Openshift

In this post, I will run through an installation of Anchore on OpenShift. I'll also discuss in brief how we can use Anchore to scan images, and work with policies. 

## Getting started

My environment and tooling consists of the following: 

- CentOS 7 on AWS
- RedHat [OKD](https://www.okd.io/) version 3.11 in a single node
- [Helm](https://helm.io)
- PostgreSQL on RDS (For Anchore external DB)

For the purposes of this post, I will assume a successful installation of OKD and Helm. For more information on installing Helm on OpenShift see [here](https://blog.openshift.com/getting-started-helm-openshift/).

To verify that Helm has been installed and configured successfully, running the command below should yield the following output: 

```
[centos@ip-172-31-7-54 ~]$ helm version
\Client: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
```