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

## Using the Anchore Helm chart

I will be installing Anchore via Helm and the chart located [here](https://github.com/helm/charts/tree/master/stable/anchore-engine).

For my installation, I've set up a PostgreSQl database in Amazon RDS that I will configure Anchore to use. Although there is a managed PostgreSQL service that can be installed with the chart, it is recommeded to use an external DB for production installations. 

### Configuring the external db

In order to configure the external db, create a new file named `anchore-values.yaml` and add the following: 

```YAML
## anchore-values.yaml

postgresql:
  # To use an external DB, uncomment & set 'enabled: false'
  # externalEndpoint, postgresUser, postgresPassword & postgresDatabase are required values for external postgres
  enabled: false
  postgresUser: db_username
  postgresPassword: db_password
  postgresDatabase: anchore_db

  # Specify an external (already existing) postgres deployment for use.
  # Set to the host and port. eg. mypostgres.myserver.io:5432
  externalEndpoint: anchore-db-instance.<123456>.us-east-2.rds.amazonaws.com:5432
```

For more details on using the Helm chart please consult the GitHub repo. 

## Installing Anchore 

Run the following command to install Anchore: 

```
helm install --name <release_name> -f anchore-values.yaml stable/anchore-engine
```