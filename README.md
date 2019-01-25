# tibco-as-datagrid-k8s
Setup TIBCO ActiveSpaces DataGrid using Kubernetes

## Overview

An activespaces datagrid is setup on either Minikube or Kubernetes cluster on AWS. Following Tools are must be installed on the your machine. Alternatively, a ubuntu virtualbox can be setup using Vagrant. The [Vagrantfile](https://github.com/shivchelwa/tibco-as-datagrid-k8s/blob/master/Vagrantfile) uses[bootsrtap.sh](https://github.com/shivchelwa/tibco-as-datagrid-k8s/blob/master/bootstrap.sh) to provision these tools.

1. kops
2. kubectl
3. awscli
4. helm

## Prerequisites

Below TIBCO ActiveSpaces DataGrid component images must be available on your AWS Elastic Container Registry(ECR). In this exercise AS 3.5.1 and FTL 5.3.1 is used. If you are using a different version replace versions in yaml files.
as-tibdg
as-tibdgadmind
as-tibdgkeeper
as-tibdgmonitoring
as-tibdgnode
as-tibdgproxy	
as-operations
ftl-tibrealmserver

You should have a registed domain name. Either register a domain name in AWS or use one if you already have a registered one. You must create a hosted zone and add a sub-domain that will be used as your kubernetes cluster name.

Create a S3 bucket in the region where you intent to setup K8s cluset. This s3 bucket is used to store kops metadata.

## AS DataGrid on Minikube

Use the below command to create AS datagrid on Minikube. In minikube-values.yaml replace the image names with your repository names. The ActiveSpcaces docker images must be accessible with Minikube. The images can be made accessible in different ways.
1. Ssh into minikube and pull images to local repository
2. Ssh into minikube and login to your private repository.
3. Create a kubernetes secret using below command and set imagePullSecrets in the yaml file.

In this project, imagePullSecrets is used and need to be replaced once secret is created. Once the above step is complete. Use below command to deploy a AS datagrid.

`helm install --name tibdg-helm -f minikube-values.yaml --debug .`

## AS DataGrid on AWS

The awscli must be configured using secret key and password.

`aws configure`

Create a ssh key on the machine from where you are going to create the kops cluster. The ssh public key will be uploaded to the cluster nodes and the private ssh key is used when you login to master node or other cluster nodes. 

`ssh-keygen -f .ssh/id_rsa`

Create a kops cluster using below command. Be sure to replace S3 bucket name and kubernetes cluseter name which is same as the hosted sub-domain on AWS.

`kops create cluster --name=kubernetes.psgamericas-team-cep.be --state=s3://kubernetes-kops-2019 --zones=us-east-1a --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=kubernetes.psgamericas-team-cep.be`

This may take a few minutes. You can validate your cluster status using below command. Once cluster is ready, the command will show appropriate message on console.

`kops validate cluster --state=s3://kubernetes-kops-2019`

After the cluster is ready, use below to command to create a service account.

`kubectl create -f helm-rbac.yaml`

Initialize helm using this service account

`helm init --service-account tiller`

Create the AS datagrid using below command in the directory having Chart.yaml. In this exercise, gp2 (general purpose) storage class is used for reamlserver, keeper and as node data store. You can change this storage class with a different storage class in aws-values.yaml.

`helm install --name tibdg-helm -f aws-values.yaml --debug .`

Verify the AS datagrid status using below command

`kubectl run -it --rm --restart=Never --image=233043593649.dkr.ecr.us-east-1.amazonaws.com/as-tibdg:3.5.1 tibdg -- -r http://realmserver:30080 status`

To delete the datagrid

`helm delete --purge tibdg-helm`

To delete the kubernetes cluseter

`kops delete cluster --name=kubernetes.psgamericas-team-cep.be --state=s3://kubernetes-kops-2019 --yes`

