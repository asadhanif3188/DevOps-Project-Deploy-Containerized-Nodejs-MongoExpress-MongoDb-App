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

### Run the command 
Now run the following command to have it in the K8s environment. 

`kubectl apply -f nodejs-namespace.yaml`

<img src="screenshots/namespace.png" width="60%" />

## Step 02: Create ConfigMap Manifest file
ConfigMap is used to maintain all the configuration details for different components. 

Since mongodb and mongo-express need some environment variables, we'll keep all those non-sensitive details in our configmap. 

Create a file `mongo-configmap.yaml` with following contents. 

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
  namespace: nodejs-namespace
data:
  database_url: mongodb-service
```

### Run the command 
Now run the following command to create the configmap under our namespace. 

`kubectl apply -f mongo-configmap.yaml -n nodejs-namespace`

<img src="./screenshots/configmap.png" width="60%" />

## Step 03: Create Secret Manifest file
Secret is used to maintain all the **secret** configuration details for different components. 

Since mongodb and mongo-express environment variable for password so we'll keep this sensitive info in secret. 

Create a file `mongo-secret.yaml` with following contents. 

```
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
  namespace: nodejs-namespace
type: Opaque 
data:
  mongo-root-username: YWRtaW4=
  mongo-root-password: cGFzc3dvcmQ=
```

**Note:** In the secret manifest file `username` and `password` are not placed as plaintext, rather these items are transformed into non-readable format using `base64` algorithm. 

### Run the command 
Now run the following command to create the secret under our namespace. 

`kubectl apply -f mongo-secret.yaml -n nodejs-namespace`

<img src="./screenshots/secret.png" width="60%" />


## Step 04: Create Deployment Manifest file for MongoDB
In this step we are going to create a deployment manifest file to execute MongoDB container at port `27017`. Official docker image of [mongodb](https://hub.docker.com/_/mongo) is being used with `latest` tag.  

Create a `mongo-deployment.yaml` file with following contents. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  namespace: nodejs-namespace
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:latest
        
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        ports:
        - containerPort: 27017
```

#### Environment Variables for MongoDB
When we start the mongo image, we can adjust the initialization of the MongoDB instance by passing one or more environment variables in the deployment file. 

Following are the two most important environment variables for the container:
- `MONGO_INITDB_ROOT_USERNAME`
- `MONGO_INITDB_ROOT_PASSWORD`

**Note:** These variables, used in conjunction, create a new user and set that user's password. This user is created in the admin authentication database and given the role of root, which is a "superuser" role. 

### Run the command 
Now run the following command to create the secret under our namespace. 

`kubectl apply -f mongo-deployment.yaml -n nodejs-namespace`

<img src="./screenshots/mongodb-deployment.png" width="60%" />

