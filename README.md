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


### 5- Deploy the application with Helm 3
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

6- Install the hello-world app:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: load-balancer-example
  name: hello-world
spec:
  replicas: 5
  selector:
    matchLabels:
      app.kubernetes.io/name: load-balancer-example
  template:
    metadata:
      labels:
        app.kubernetes.io/name: load-balancer-example
    spec:
      containers:
      - image: gcr.io/google-samples/node-hello:1.0
        name: hello-world
        ports:
        - containerPort: 8080
```
```
kubectl apply -f https://k8s.io/examples/service/load-balancer-example.yaml
```
7- Display hello-world app deployment:
```
kubectl get deployments hello-world
kubectl describe deployments hello-world
```

8- Display hello-world ReplicaSet objects:
```
kubectl get replicasets
kubectl describe replicasets
```

### 7-  Use the IBM-provided domain for your cluster
Paid clusters come with an IBM-provided domain. This gives you a better option to expose applications with a proper URL and on standard HTTP/S ports.

Create a Service object that exposes the hello-world deployment:

![plot](https://cloud.ibm.com/docs-content/v1/content/d7719795b28ea8f7b7514e07e872e2cc3e8e9c6f/solution-tutorials/images/solution2/Ingress.png)

7.1 Identify your IBM-provided Ingress domain
```
ibmcloud ks cluster get --cluster $MYCLUSTER
```
Deploy the service:
```
kubectl expose deployment hello-world --type=LoadBalancer --name=my-service
```

7.2 Display Kubernetes services deployed in previous step:
```
kubectl get services my-service
```
It may take up to 5 min to update Loadbalancer in IBM Cloud with new service. As an example of output:
```
Name:                     my-service
Namespace:                default
Labels:                   app.kubernetes.io/name=load-balancer-example
Annotations:              <none>
Selector:                 app.kubernetes.io/name=load-balancer-example
Type:                     LoadBalancer
IP Families:              <none>
IP:                       172.21.72.4
IPs:                      172.21.72.4
LoadBalancer Ingress:     f17a4a30-us-south.lb.appdomain.cloud
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  32691/TCP
Endpoints:                172.17.32.206:8080,172.17.32.207:8080,172.17.40.205:8080 + 2 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason                           Age   From                Message
  ----    ------                           ----  ----                -------
  Normal  EnsuringLoadBalancer             15m   service-controller  Ensuring load balancer
  Normal  EnsuredLoadBalancer              15m   service-controller  Ensured load balancer
  Normal  CloudVPCLoadBalancerNormalEvent  10m   ibm-cloud-provider  Event on cloud load balancer my-service for service default/my-service with UID d3026d55-587d-434d-b318-fecd19e32193: The VPC load balancer that routes requests to this Kubernetes LoadBalancer service is currently online/active.
  
```
7.5 Use the loadbalancer in "LoadBalancer Ingress:" field and HTTP port in "TargetPort:". Open your application in a browser at https://"LoadBalancer Ingress:"TargetPort:". Output display should be:
```
Hello Kubernetes!
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
	
