# How to Deploy a Node-Express App on Amazon EKS

The purpose of this repository is to demonstrate how to deploy a simple web application built by Express - Node.js web application framework on Amazon EKS by using Amazon ECR (Elastic Container Registry)

## Requirements

 - [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) configured with a profile having sufficient access.

 - Your environment needs to have [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) and [Docker](https://docs.docker.com/get-docker/) installed. Dockerfile and the sample code are provided in this repository.

 - We'll push the image to [Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-authenticate-registry). If you're using the EC2 Instance for this, make sure its assigned role has necessary rights (such as AmazonEC2ContainerRegistryFullAccess )

 - [The eksctl command line utility](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html) ( a sample `yaml` file are  provided  in this repository) and [kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)


## Step-1: Build a Docker Image and Push to Amazon ECR

In this part we will build an image and then push it to a container registry - Amazon ECR in order to use it in an Kubernetes object. We'll proceed with a private repository in this demo, but public repositories and Docker Hub are also supported.

```bash
# install git
$ sudo yum install -y git
$ git clone https://github.com/aws-samples/amazon-eks-node-express-sample

# build docker image & verify 
$ cd amazon-eks-node-express-sample/sample-nodejs-app
$ docker build -t sample-nodejs-app .
$ docker images

# Authenticate to your default registry
# update the region and aws_account_id on the below command
$ aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com

# Once you receive 'Login Succeeded" , you can create your private repo on ECR
# update the region on the below command
$ aws ecr create-repository \
    --repository-name sample-nodejs-app \
    --image-scanning-configuration scanOnPush=true \
    --region <region>

# tag and push your image
# update the region and aws_account_id on the below command
$ docker tag sample-nodejs-app:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/sample-nodejs-app:latest
$ docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/sample-nodejs-app:latest
```
Congratulations! Now, you've pushed this image to a private repository of Amazon ECR. This should now be listed on your images under Amazon ECR repositories. Copy the Image URI as we'll need it in the next step.

## Step-2: Creating an Amazon EKS cluster 

There are two getting started guides available for creating a new Kubernetes cluster with nodes in Amazon EKS:

- [Getting started with Amazon EKS – eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html) – This getting started guide helps you to install all of the required resources to get started with Amazon EKS using eksctl, a simple command line utility for creating and managing Kubernetes clusters on Amazon EKS.

- [Getting started with Amazon EKS – AWS Management Console and AWS CLI](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html) – This getting started guide helps you to create all of the required resources to get started with Amazon EKS using the AWS Management Console and AWS CLI. At the end of the tutorial, you will have a running Amazon EKS cluster that you can deploy applications to.

In this sample we'll proceed with the 1st option and launch an EKS development environment with 1 node : 

```bash 
# Check that you've an updated version of eksctl installed
$ eksctl version
$ eksctl create cluster -f eks-sample-cluster.yaml
$ kubectl get nodes
```

## Step-3: Deploy Kubernetes Objects 
```bash
$ kubectl apply -f k8s/sample-k8s-deployment.yaml
$ kubectl apply -f k8s/sample-k8s-service.yaml
# Now that we have a running service that is type: LoadBalancer we need to find the ELB’s address. We can do this by using the get services operation of kubectl:
$ kubectl get service amazon-eks-node-express-sample -o wide
```

It will take several minutes for the ELB to become healthy and start passing traffic to the frontend pods.

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

