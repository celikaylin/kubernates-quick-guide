# Kubernates

It helps us with container orchestration.

It helps us with;

⚡️ Automatic deployment

⚡️ Scaling and Load Balancing

⚡️ Management

#### 🚀 Dont confuse with them
- It is not a cloud provider, we can say collection of software and concepts can be used with a cloud provider.

- It does not create machines or virtual instances, install softwares. It just keeps your app up and running.

## Concept and Architecture

Here is the Kubernates Architecture

![Kubernates Architecture](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

⚡️ `Pod` is reponsible for running container/containers. It hosts one or more containers and their resources. It is created and managed by `Kubernates`

⚡️`Worder Node` is the thing you can think as your machine/virtual machine. It is managed by `Master Node`. It can has more than one pod on it and also some softwares/services. 

`Docker` is for create and run the containers.

`kubelet` is for providing the communication between `Worker Node` and `Master Node`. So `Master Node` can manage the pods on the `Worker Node`.

`kube-proxy` is responsible for handling incoming and outgoing traffic.

⚡️`Master Node/Control Panel` cotrols all worker nodes. It has some softwares/services.

`API Server` is for the `kubelets` to communicate.

`Scheduler` is for watching and controlling `Worker Nodes`.

⚡️ All these are connected in the `Cluster`.