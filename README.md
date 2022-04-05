# 1- Hands-on lab training http-webapp-iks-lab

Deploy a HTTP Web App on IKS cluster Workshop 

The purpose of this structured, no-cost, hands-on, instructor-led learning 
experience is to demonstrate how to combine different technologies such 
as Container Orchestration Platform, Container Registry, and cloud 
computing environment to kickstart cloud native applications. Specifically, 
the attendees will learn how to: 

     Deploy a web HTTP application to the Kubernetes cluster.
     Bind a custom subdomain.
     Monitor the logs and health of the cluster.
     Scale Kubernetes pods.

Tasks to complete:
  - [x] A developer downloads or clones a starter web application.
  - [x] Optionally build the application to produce a container image.
  - [x] Optionally the image is pushed to a namespace in the IBM Cloud Container Registry.
  - [x] The application is deployed to a Kubernetes cluster.
  - [x] Users access the application.

https://cloud.ibm.com/docs/solution-tutorials?topic=solution-tutorials-scalable-webapp-kubernetes

## 2- Lab preparation
Follow these instructions for the setup of the initial Lab environment.

1- Create an access group in your IBM Cloud account: "Demo LABs"
https://cloud.ibm.com/iam/groups
Create
Enter "Demo LABs" as name of access group
Enter a short description
Add the following access polices:
	
1.1 VPC Infrastructure Services service
1.2 Cloud Object Storage service
1.3 Security Advisor service
1.4 All resource group
    resourceType string equals resource-group
1.5 Toolchain service
1.6 Certificate Manager service
1.7 Schematics service
1.8 All resources in account (including future IAM enabled services)

2- Create a resource group "labs"
https://cloud.ibm.com/account/resource-groups

3- Invite users to "Demo LABs" access group via:
https://cloud.ibm.com/iam/users/invite_users
enter user's email address
select "Demo LABs"

Users need to go to their notification section to accept the invite:
https://cloud.ibm.com/notifications
First time users of ibm cloud need to create a new ibm cloud id. Consult with your ibm contact for first time ID creation ( https://cloud.ibm.com/registration ) 

4- Work with your IKS team to deploy an IKS cluster required for this lab.

## 3- Objectives

Deploy a web application to the Kubernetes cluster.
Monitor the logs and health of the cluster.
Scale Kubernetes pods.


![plot](https://github.com/bkoohi/webapp-iks-lab/blob/main/images/Screen%20Shot%202022-04-01%20at%2011.49.40%20AM.png)

3.1- A developer downloads or clones a starter web application

3.2- Optionally build the application to produce a container image

3.3- Optionally the image is pushed to a namespace in the IBM Cloud Container Registry

3.4- The application is deployed to a Kubernetes cluster

3.5- Users access the application


## 4- Start a new IBM Cloud Shell
4.1 From the IBM Cloud console in your browser, select the account where you have been invited.

4.2 Click the button in the upper right corner to create a new Cloud Shell ( https://cloud.ibm.com/shell )


## 5- Clone a starter application

In this section, you will clone a GitHub repo with a simple Helm-based NodeJS starter application with a landing page and two endpoints to get started. You can always extend the starter application based on your requirement.

1- On a terminal, run the below command to clone the GitHub repository to your machine:

```
git clone https://github.com/IBM-Cloud/kubernetes-node-app
```
2- Change to the application directory,
```
cd kubernetes-node-app
```
This starter application code contains all the necessary configuration files for local development and deployment to Kubernetes.


### 6- Deploy the application with Helm 3
The container image for the application as already been built and pushed to a public Container Registry. In this section you will deploy the starter application using Helm. Helm helps you manage Kubernetes applications through Helm Charts, which helps define, install, and upgrade even the most complex Kubernetes application.

1- Define an environment variable named MYPROJECT and set the name of the application by replacing the placeholder with your initials:
```
export MYPROJECT=<your-initials>kubenodeapp
```
2- Identify your cluster:
```
ibmcloud ks cluster ls
```

3- Initialize the variable with the cluster name

```
export MYCLUSTER=<CLUSTER_NAME>
```

4- Initialize the kubectl cli environment
```
ibmcloud ks cluster config --cluster $MYCLUSTER
```

5- You can either use the default Kubernetes namespace or create a new namespace for this application.

If you wish to use the default Kubernetes namespace, run the below command to set an environment variable
```
export KUBERNETES_NAMESPACE=default
```

6- Change to the chart directory under your starter application directory:
```
cd chart/kubernetesnodeapp
```

7- Install the Helm chart:
```
helm3 install $MYPROJECT --namespace $KUBERNETES_NAMESPACE . --set image.repository=icr.io/solution-tutorials/tutorial-scalable-webapp-kubernetes
```
### 7-  Use the IBM-provided domain for your cluster
Paid clusters come with an IBM-provided domain. This gives you a better option to expose applications with a proper URL and on standard HTTP/S ports.

Use Ingress to set up the cluster inbound connection to the service.

![plot](https://cloud.ibm.com/docs-content/v1/content/d7719795b28ea8f7b7514e07e872e2cc3e8e9c6f/solution-tutorials/images/solution2/Ingress.png)

7.1 Identify your IBM-provided Ingress domain
```
ibmcloud ks cluster get --cluster $MYCLUSTER
```
to find

Ingress subdomain: mycluster.us-south.containers.appdomain.cloud
Ingress secret:    mycluster

7.2 Define environment variables INGRESS_SUBDOMAIN and INGRESS_SECRET to hold the values
```
export INGRESS_SUBDOMAIN=<INGRESS_SUBDOMAIN_FROM_ABOVE_STEP>
```
```
export INGRESS_SECRET=<INGRESS_SECRET>
```
7.3 Change to your starter application directory and run the below bash command to create an Ingress file ingress-ibmdomain.yaml pointing to the IBM-provided domain with support for HTTP and HTTPS.
```
cd  ~/kubernetes-node-app
```
```
./ingress.sh ibmdomain_https
```
7.4 The file is generated from a template file ingress-ibmdomain-template.yaml under yaml-templates folder by replacing all the values wrapped in the placeholders ($) with the appropriate values from the environment variables.

Deploy the Ingress
```
kubectl apply -f ingress-ibmdomain.yaml
```
7.5 Open your application in a browser at https://<nameofproject>.<ingress-sub-domain>/ or run the below command to see the HTTP output
```
curl -I https://$MYPROJECT.$INGRESS_SUBDOMAIN
```
An exmaple output:
	
```
HTTP/2 200
	
date: Mon, 04 Apr 2022 13:47:39 GMT
	
content-type: text/html; charset=UTF-8
	
content-length: 58215
	
x-powered-by: Express
	
accept-ranges: bytes
	
cache-control: public, max-age=0
	
last-modified: Tue, 01 Feb 2022 15:43:18 GMT
	
etag: W/"e367-17eb5f45df0"
	
strict-transport-security: max-age=15724800; includeSubDomains
	
```
### 8-  Clean up and remove application
	
8.1 List application deployment
```
kubectl get deployments
```
	
8.2 Identify kubernetesnodeapp-deployment application in the list and delete
```
kubectl delete -n default deployment kubernetesnodeapp-deployment
```
	
8.3 List installed ingress controller 
```
kubectl get ingress
```
	
8.4 Delete ngress-for-ibmdomain-http-and-https Ingress controller 
```
kubectl delete -n default ingress ingress-for-ibmdomain-http-and-https
```
	
