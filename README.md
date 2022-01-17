# Provision a Kubernetes Cluster in AWS
# Prerequisites 

1. An AWS account with the IAM permissions listed on the EKS module documentation, 

2. A configured AWS CLI 

3. AWS IAM Authenticator 

4. kubectl 

5. wget (required for the eks module) 

# Set up and initialize your Terraform workspace 
1. $ terraform init 
2. $ terraform apply 

This terraform apply will provision a total of 53 resources (VPC, Security Groups, AutoScaling Groups, EKS Cluster, etc...). Confirm the apply with a yes. 

This process should take approximately 10 minutes. Upon successful application, your terminal prints the outputs defined in outputs.tf. 

# Configure kubectl 
    $ aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -	raw cluster_name) 

# Deploy Kubernetes Metrics Server 

The Kubernetes Metrics Server, used to gather metrics such as cluster CPU and memory usage over time, is not deployed by default in EKS clusters. 

Download and unzip the metrics server by running the following command. 
    $ wget -O v0.3.6.tar.gz https://codeload.github.com/kubernetes-sigs/metrics-server/tar.gz/v0.3.6 	&& tar -xzf v0.3.6.tar.gz 

# Deploy the metrics server to the cluster by running the following command. 
    $ kubectl apply -f metrics-server-0.3.6/deploy/1.8+/ 

Verify that the metrics server has been deployed. If successful, you should see something like this. 
    $ kubectl get deployment metrics-server -n kube-system 

# Deploy and access Kubernetes Dashboard 
    $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-b	eta8/aio/deploy/recommended.yaml 

Create a proxy server that will allow you to navigate to the dashboard from the browser on your local machine. This will continue running until you stop the process by pressing CTRL + C. 
    $ kubectl proxy 

# You should be able to access the Kubernetes dashboard here (http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/). 

# Authenticate the dashboard 
To use the Kubernetes dashboard, you need to create a ClusterRoleBinding and provide an authorization token. This gives the cluster-admin permission to access the kubernetes-dashboard. Authenticating using kubeconfig is not an option. You can read more about it in the Kubernetes documentation. 

In another terminal (do not close the kubectl proxy process), create the ClusterRoleBinding resource. 
    $ kubectl apply -f https://raw.githubusercontent.com/hashicorp/learn-terraform-provision-eks-cluster/main/kubernetes-dashboard-admin.rbac.yaml 

# Generate the authorization token. 
    kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep service-controller-token | awk '{print $1}') 

Select "Token" on the Dashboard UI then copy and paste the entire token you receive into the dashboard authentication screen to sign in. You are now signed in to the dashboard for your Kubernetes cluster. 

Navigate to the "Cluster" page by clicking on "Cluster" in the left navigation bar. You should see a list of nodes in your cluster. 

Congratulations, you have provisioned an EKS cluster, configured kubectl, and deployed the Kubernetes dashboard. 

Compiled by Shedrack...

Ref: This repo is a companion repo to the [Provision an EKS Cluster learn guide](https://learn.hashicorp.com/terraform/kubernetes/provision-eks-cluster), containing
Terraform configuration files to provision an EKS cluster on AWS.
