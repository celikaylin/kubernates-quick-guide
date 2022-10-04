# Dive into Kubernetes
## Windows Setup

You need two thing to setup.

- `kubectl` is a communication device for talking clusters. Here is the [offical doc](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)to install and use it.

- `Minikube` is a tool to run Kubernetes locally. Here is the [doc](https://minikube.sigs.k8s.io/docs/start/) to install and use it.


If you get an error while starting it please check [here](https://minikube.sigs.k8s.io/docs/drivers/) for using driver.

## Some of Kubernetes Objects

**Pod** 
- It is the smallest unit and reponsible for running container/containers. 
- It hosts one or more containers and their resources (but best practise is one container per pod). 
- It is created and managed by `Kubernetes`
- It has cluster-internal Ip by default.
- Containers which are inside a pod can communicate via localhost.

**Deployment**
- It controls pods.
- Deployments can be paused, deleted, rollback.
- It can be scaled dynamically and automatically.

**Service**
- It exposes ports to the cluster or externally.
- It provides reaching pods from outside world.

## Key Kubernetes Commands

#### ⚡️ minikube

```bash
minikube start
```
or 

```bash
minikube start --driver=DRIVER_NAME
```

To check minikube statu
```bash
minikube status
```

To delete existing one
```bash
minikube delete
```

To get a visual dashboard
```bash
minikube dashboard
```

 To get a URL to connect to a service
```bash
minikube service SERVICE_NAME
```

#### ⚡️ kubectl

To create a deployment object
```bash
kubectl create deployment DEPLOYMENT_NAME --image=REMOTE_IMAGE_NAME
```

To delete the deployment object
```bash
kubectl delete deployment DEPLOYMENT_NAME
```

To list all deployments
```bash
kubectl get deployments
```

To list all deployments
```bash
kubectl get pods
```

To expose the deployment as a service
```bash
kubectl expose deployment DEPLOYMENT_NAME --type=TYPE  --port=PORT
```
#### 🚀 --type option
It is type for the service. 

`ClusterIP` means it is reachable only from inside of cluster. 

`LoadBalancer` gives a unique address and you can reach via that from outside.


To list services
```bash
kubectl get services
```

To set a new size for a deployment
```bash
 kubectl scale TYPE/DEPLOYMENT_NAME --replicas=REPLICA_NUMBER
```

Rollback
```bash
 kubectl rollout undo TYPE/DEPLOYMENT_NAME
 ```

To list rollback history
 ```bash
 kubectl rollout history TYPE/DEPLOYMENT_NAME
 ```

 To list rollback history in detail
```bash
kubectl rollout history TYPE/DEPLOYMENT_NAME --revision=REVISION_IN HISTORY_LIST
```

 Rollback to the specific older version
```bash
kubectl rollout undo TYPE/DEPLOYMENT_NAME --to-revision=REVISION
```

To delete service
```bash
kubectl delete service SERVICE_NAME
```

To delete deployment
```bash
kubectl delete deployment DEPLOYMENT_NAME
```

To delete resources via name
```bash
kubectl delete -f FILE_NAME
```

To apply configurations to a resource
```bash
kubectl apply -f FILE_NAME
```


- To update the deployment
```bash
kubectl set image TYPE/DEPLOYMENT_NAME EXISTED_IMAGE=NEW_IMAGE:TAG
```

#### 🚀 TAG is important
When you want to update the deployment firstly it looks tag is changed or not if you dont give a tag your image, deployment will not be updated so `TAG` is important and you must give a tag to your image when building it.


## Example of Deployment

There are two option which are imperative config it means you have to use the commandline for individual commads in deployment process and declarative config it means you use a file to whole deploy your app.

### Imperative Configuration

- Firstly build your image
```bash
docker build -t kub-first-app .
```


- Push your image to docker hub via [that doc](https://github.com/celikaylin/docker-quick-guide/blob/main/images-and-containers.md#pushing-image-to-docker-hub). 


- Now start minikube and create deployment object
```bash
minikube start
kubectl create deployment first-app --image=aylincelik/kub-first-app
```

- If you get 1/1 at `READY` when you list deployments and pod, you did it.


#### 🚀 Dont use local image directly
In cluster your local image can not be found so when ypu create a deployment object via local image it will be created but not ready to use. When you list your pods and deployments, their `READY` state must be same with target state (`READY` looks currentstate/targetstate).

- Open minikube dashboard and see your created cluster.
```bash
minikube dashboard
```

- Expose a port to reach out from outside from cluster
```bash
kubectl expose deployment first-app --type=LoadBalancer  --port=8080
```

To get externall address of the app (this case minikube specific normally, when not using minikube, you will get external address when list the services at EXTERNAL-IP column)  
```bash
minikube service first-app
```

#### Updating The Deployment

- Build your image with a tag 

- Push image to the `Docker Hub`

- Update the deployment
```bash
kubectl set image deployment/first-app kub-first-app=aylincelik/kub-first-app:2
```

### Declarative Configuration

Here is the `deployment.yaml` file.

```bash
apiVersion: apps/v1

# To define a Deployment object. 
kind: Deployment

#To indetify your deployment
metadata:
  name: dep-with-declarative

# Specification/configuration of the deployment
spec:
  # Default replica is 1 (if you dont set it)
  replicas: 1
  # Tell which pods will be mananaged by Deployment
  selector:
    matchLabels:
      app: app-with-dec
  # For adding a new pod
  template:
    # We use metadata again because pod is an object as Deployment.
    metadata:
      labels:
        # This key:value naming is up to you
        app: app-with-dec
    # Specification/configuration of the pod
    spec:
      containers:
        - name: node-with-dec
          image: aylincelik/kub-first-app
```

#### 🚀 Pod object without `kind` label 
I said `Pod` is an object bu I dont use `kind` label for Pod object. Because template of `Deployment` object has to be a pod so you dont need to add `kind:Pod`

Apply the deployment configuration
```bash
kubectl apply -f deployment.yaml
```

Here is the `service.yaml` file.
```bash
apiVersion: v1

# To define a Service object. 
kind: Service

# To indetify your deployment
metadata:
  name: service-dec

# Specification/configuration of the service
spec:
  selector:
    # Add your key:value pair here directly (it refers to the ports into the Deployment configuration)
     app: app-with-dec
  ports:
    - protocol: 'TCP'
      # Port is you want to expose it (To reach out from outside world).
      port: 80
      # Target port is a port inside container
      targetPort: 8080
  type: LoadBalancer

```

Apply the service configuration
```bash
kubectl apply -f service.yaml
```

#### 🚀 I dont want to create seperate `.yaml` files.
So you can, but the only thing you have to use `---` to seperate configurations in the same file.

Here is the example `my-deployment.yaml`

```bash

apiVersion: v1
kind: Service
metadata:
  name: service-dec
spec:
  selector:
     app: app-with-dec
  ports:
    - protocol: 'TCP'
      port: 80
      targetPort: 8080
  type: LoadBalancer

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: dep-with-declarative
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-with-dec
  template:
    metadata:
      labels:
        app: app-with-dec
    spec:
      containers:
        - name: node-with-dec
          image: aylincelik/kub-first-app
```
Apply all configurations at once
```bash
kubectl apply -f my-deployment.yaml
```

#### Updating The Deployment

To update your deployment just changed the `.yaml` file and run the `kubectl apply ..` command. That's all.


