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

Create a new project via `oc new-project anchore-engine`

Run the following command to install Anchore: 

```
helm install --name <release_name> -f anchore-values.yaml stable/anchore-engine
```

An initial install will take several minutes to complete. Additionally, it will also take some time to perform its initial data feed sync. 

You can run `oc get pods` to see how things are doing.

```
[centos@ip-172-31-7-54 ~]$ oc get pods
NAME                                                         READY     STATUS    RESTARTS   AGE
anchore-engine-anchore-engine-analyzer-7d5fc7fb4c-phkt8      1/1       Running   0          1h
anchore-engine-anchore-engine-api-55b785794-tk6qt            1/1       Running   0          1h
anchore-engine-anchore-engine-catalog-65bbfdd7c7-7ldzj       1/1       Running   0          1h
anchore-engine-anchore-engine-policy-8cb4787ff-sdw7v         1/1       Running   0          1h
anchore-engine-anchore-engine-simplequeue-5f7b7f866b-2hn2n   1/1       Running   0          1h
```

In addition, you can check on the installation via the OpenShift UI. 

![screenshot](images/pod-overview.png)

We can also install the [Anchore CLI](https://github.com/anchore/anchore-cli) to interact with our running Anchore Engine service. There is also a [CLI container](https://hub.docker.com/r/anchore/engine-cli/).

Configure you Anchore CLI environment variables to communitate with the anchore engine API service. Now we can check on the status of the Anchore services by running `anchore-cli system status`.

```
[centos@ip-172-31-7-54 ~]$ anchore-cli system status
Service apiext (anchore-engine-anchore-engine-api-55b785794-5qn79, http://anchore-engine-anchore-engine-api:8228): up
Service simplequeue (anchore-engine-anchore-engine-simplequeue-5f7b7f866b-2hn2n, http://anchore-engine-anchore-engine-simplequeue:8083): up
Service policy_engine (anchore-engine-anchore-engine-policy-8cb4787ff-p8tpf, http://anchore-engine-anchore-engine-policy:8087): up
Service analyzer (anchore-engine-anchore-engine-analyzer-7d5fc7fb4c-2z85z, http://anchore-engine-anchore-engine-analyzer:8084): up
Service catalog (anchore-engine-anchore-engine-catalog-65bbfdd7c7-7ldzj, http://anchore-engine-anchore-engine-catalog:8082): up
```