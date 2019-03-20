# Introduction to EKS

In June of 2018, Amazon announced the general availability of their [Elastic Container Service for Kubernetes](https://aws.amazon.com/blogs/aws/amazon-eks-now-generally-available/). Given that at Anchore we deliver our products as Docker container images, it came as no surprise to us that our users and customers would begin deploying our software on EKS. Since Kubernetes, Kubernetes on AWS, and Anchore on EKS adoption have increased, I thought it best to give EKS a test run to gain some knowledge. 

### Getting started

For the scope of learning purposes, I thought I'd test out creating a EKS cluster, and launch a simple application. If you aren't completely familiar with Kubernetes I highly recommend checking out the [tutorials](https://kubernetes.io/docs/tutorials/kubernetes-basics/) section of the website just so some of the concepts and verbage I use make a little more sense. I also recommend reading about [**kubectl**](https://kubernetes.io/docs/reference/kubectl/overview/) which is the command line interface for running actions against Kubernetes clusters. 

In addition to the above reading, you should complete the following:

- [Install kubectl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)
- [Install aws-iam-authenticator for Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)
- [Download and Install the Latest AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)

## Creating a cluster

There are a couple of ways to create an EKS cluster, with the console or with the AWS CLI.

**Create cluster using AWS console**

To begin, navigate here: https://us-east-2.console.aws.amazon.com/eks/home#/clusters and select *create cluster*.

You'll see several pieces of information you'll need to provide AWS for it to create your cluster successfully. 

- Cluster name (should be a unique name for your cluster)
- Role name
- VPC and Subnets
- Security groups

#### Role name

Here you will need to select the IAM role that will allow Amazon EKS and the Kubernetes control plane to manage AWS resources on your behalf. If you have not already, you should create an EKS service role in the IAM console. 

#### VPC and Subnets

Select a VPC and choose the subnets in the selected VPC where the worker nodes will run. If you have not created a VPC, you will need to create one in the VPC console, and create subnets as well. AWS has a great tutorial on VPC and Subnet creation [here](https://docs.aws.amazon.com/eks/latest/userguide/create-public-private-vpc.html).

**Note:** Subnets specified must be in at least two different availability zones. 

#### Security groups

Choose security groups to apply to network interfaces that are created in your subnets to allow the EKS control plane to communicate with your worker nodes.

Once all the necessary requirements have been fulfilled, you create the cluster. 

**Create cluster using AWS CLI**

You can also create a cluster via the AWS CLI by running the following:

`aws eks --region region create-cluster --name devel --role-arn arn:aws:iam::111122223333:role/eks-service-role-AWSServiceRoleForAmazonEKS-EXAMPLEBKZRQR --resources-vpc-config subnetIds=subnet-a9189fe2,subnet-50432629,securityGroupIds=sg-f5c54184`

Simply input your `--role-arn`, `subnetIds`, and `securityGroupIds` into the above command.

Once your cluster has been created the console should look like the following:

![alt-text](images/eks-active-cluster.png)