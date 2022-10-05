# Volumes
Volumes are pod specific and attached to pods.

Without volumes you would loose your data when container are crashed, deleted, restarted.

[Here](https://kubernetes.io/docs/concepts/storage/volumes/) there are types of volume that you can look deeper.

#### 🚀 `emptyDir: {}`
It is a type of volumes and when you set `{}` it measn you will use default configurations. At this point in the pod a directory will be created whenever port starts, kept alive and filled data in it as long as the pod is alive.

Here is the examples.

- Firstly we need `Dockerfile` file to build an image

```
FROM node:14-alpine

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 3000

CMD [ "node", "app.js" ]

```
- Run below command to build the image
```
docker build -t kub-app-volumes .
```

- Push image to the remote by [following these steps](https://github.com/celikaylin/docker-quick-guide/blob/main/images-and-containers.md#pushing-image-to-docker-hub).

- Here is the `deployment.yaml`
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: movie-deployment
spec: 
  replicas: 1
  selector:
    matchLabels:
      app: movie-pod
  template:
    metadata:
      labels:
        app: movie-pod
    spec:
      containers:
        - name: movie-con
          image: aylincelik/kub-dep-data:2
          volumeMounts:
            - mountPath: /app/movie
              name: movie-volume
      volumes:
        - name: movie-volume
          emptyDir: {}
```

- Here is the `service.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: movie-service
spec:
  selector: 
    app: movie-pod
  type: LoadBalancer
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 3000
```

- Run below command to apply configurations
```
kubectl apply -f deployment.yaml -f service.yaml
```

After all steps I have a deployment with volumes. My data will store even though it is crashed or restarted.


## Persistent Volumes
Some types of volumes are depend on pods or nodes. It means the volumes avaliable as long as pod or nodes are up. So when I scale up my pods or nodes I will loose my data, it does not make sense on some circumtances. At this point persistent volumes can be required.

Persistent volume is not attached to a `Pod` so it is a standalone Cluster resource. It is created standalone and it is claimed via `Persistent Volume Claim`


There must be `Persistent Volume` and `Persistent Volume Claim`

- Here is te `pv.yaml`

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  storageClassName: standart
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data
    type: DirectoryOrCreate

```

- Here is the `pvc.yaml`

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  volumeName: pv
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests: 
      storage: 1Gi

```

- Run the apply command
```
kubectl apply -f pv.yaml -f pvc.yaml
```

- Here is the `deployment.yaml`

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: movie-deployment
spec: 
  replicas: 1
  selector:
    matchLabels:
      app: movie-pod
  template:
    metadata:
      labels:
        app: movie-pod
    spec:
      containers:
        - name: movie-con
          image: aylincelik/kub-dep-data:2
          volumeMounts:
            - mountPath: /app/movie
              name: movie-volume
      volumes:
        - name: movie-volume
         persistentVolumeClaim: 
          claimName: pvc

```

- And run the apply command
```
kubectl apply -f deployment.yaml
```