# Deploying Containerized Nodejs, MongoExpress, MongoDb Application 

In this demo application, we are going to deploy a **containerized** 3-Tier **Nodejs** web application with **MongoDB** and **Mongo-Express** on Kubernetes.

Application credit to [Nana Janashia](https://gitlab.com/nanuchi/developing-with-docker).

This application contains following tiers: 
1. `index.html` with pure js and css styles
2. `nodejs` backend with express module
3. `mongodb` for data storage

# Application Deployment Steps
Following steps are being followed to containerd the applicaiton and deployed on Minikube/Kubernetes. 

## Step 01: Create Own Namespace 
As application has 3-tiers, so each tier will be deployed as a separate Pod. And Pods work under some namespaces. We are going to create our own namespace, i.e. `nodejs-namespace` , to deploy all our Pods. 

To create the namespace, we are going to use following contents for manifest file, i.e. `nodejs-namespace.yaml`. 

```
apiVersion: v1
kind: Namespace
metadata:
  name: nodejs-namespace
```

Now run the following command to have it in the K8s environment. 

`kubectl apply -f nodejs-namespace.yaml`

<img src="screenshots/namespace.png" width="50%" />
<!-- ![Namespace Image](screenshots/namespace.png) -->


