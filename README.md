# Kubernetes 

## Big Picture 

**What is Kubernetes ?**
  - Open source container tool developed by Google
  - Kubernetes help manage Applications that are that made up of hundreds or maybe thousands of Containers in the Differents Environment like ( Physical Machine, Virtual Machine, or Cloud Enviroment)

**What is problems Kubernetes solve ?**
  - Trends from Monolith to Microservices
  - The Rise of Microservices Technology increased usage of Container Technology bcs the Container offer perfect host for small independent applications, like microservices
  - And the Rise of Container resulted in Application they are now comprised 100 or 1000 of container Managing those container accross multiple Environment using script and self make tools can be really complex

**What features do Orchestration tools offer ?**
  - High Availability | No down time
  - Scalability | high performance (I can scale my App fast when I more load on it and more user trying to access it, scale down when user go down)
  - Recovery Disaster

## Main component

### Commnucation between Pods

**Node and Pod**
```
  - Node = Server, Virtual or Physic Machine
  - Pod : Basic component or smallest unit of Kubernetes
    - Pod is a abstraction over container
    - What Pod does is create running Environment or layer on top of the container . The reason is bcs Kubernetes want to abstract away the container runtime or container technologies so i can replace them. I don't have to work with docker or other container technologies
    - Only interact with Kubernets layer
  - 1 Application per Pod
  - Each Pod get its own IP address
  - Pod are ephemeral which mean they can die very easily
  - Bcs Pod easy to die and everytime it died the new Pod create with new IP . So inconvinient that when My app talk to DB via IP so I have Service and Ingress
```

**Service and Ingress**
```
  Service:
    - Service is Static IP address or permanent IP can be attach to each Pod
    - Lifecyle of Pod and Service not connected
    - Each Pod has 1 service
    - External Service :
      - Make a App to accessable from Browser
      - External service is service that open the commnunication from external sources
    - Internal Service:
      - For only insde connection like (DB or ...)

  Ingress:
    - Instead of services the request first go to Ingress and then service
    - DNS 
```

### External Configuration 

**ConigMap and Secrect**
```
  So Pod Communicate with each other using Service . So my App will have database endpoint let's say MongoDB service that uses to communicate with DB but where do I configure DB URL . Usually I would do it in Application properties files , or inside of built Image of Application . For example if the endpoint of the service name changed. The solution for that is ConfigMap .

  ConfigMap:
    - External configuration of the Application
      - Usually contain configuration data like URLs of database or some other services that you use
      - In Kubernetes, I just connect it to the pod so that pod acctually gets the data that config map contain . Now you change the name of the service and endpoint of the service I just need adjust the config map
      - Part of external Config can be username and password which may also change
      - But config map is non-confidental data only so I have Secrect

  Secret:
    - Secret just like config map but the different is used to store secret data in base 64 encoded format
    - But base 64 encoded format not secure so I use third party tools (tools from cloud provider or kubernetes)
    - All the data that i don't want to expose will go in Secret
    - Then i use this secrect to connect with my App . My app can get data from secrect as ENV 
```

### Data Persistence

**Volumes**
```
  Data Storage:
    - Need the way to persisted data when a database a gone or down
  How is it work ?
    - It attaches a physical storage on the hard drive to my Pod
    - And that storage ethier from the local machine meaning on the same server where the pod is running, or it could be on remote storage meaning outside of Kubernetes cluster ( could be cloud storage or my own premise storage )

  The Different between the Kubernetes cluster and all of its components and the storage
    - Regardless of whether it is a local or remote storage, think of the storage as a external hard drive plugged in into the Kubernetes cluster 
```

### Pod Blueprint | Replication Mechanism 

**Deployment and Stateful Set**
```
  - When everything work and Application can access through a browser but what if my Application Pod died ? That mean downtime happen and I don't want this happen . I take advantage of distributed system and container .
  - So instead of relying on just one application pod and one database etc ... I replicate everthing on multiple server with also connected to service

Deployment:
  - Deployment is Service also the load balancer which mean the service actually catch the request and forward it to whichever pod is least busy
  - In other to create this second replica of the My application pod i wouldn't create a second pod . but instead i would define a blueprint for my Application pod and specify how many replicas of that pod, you would like to run. So that blueprint call Deployment .
  - In Practice I would not working with Pod or I would not creating Pod . I would creating Deployment, bcs I can specify how many replicas I want, and I can scale up or scale down number of replica of pods that I need
  - Pod is abstraction on top of the Container . and Deployment is abstraction on top on Pod which make it more convinience replication and do other replication
  - However I can not replicate DB using the Deployment the reason is DB have states which is its data . That mean I have clone DB they would need to access the same shared data storage and there I would need some kind of mechanism that manages which pods are currently writing to that storage or which pods are reading that storage in order to avoid data incosistence and that mechanism, in addition replicating feature is offered by another component called Statefull Set

Statefull set:
  - This components meant specificiclly for Application like Database or any order Statefull Application should be created by using Statefull Set and Not Deployment
  - And Statefull set is just like Deployment would take care of replicating the pods and scaling them up or scaling them down, but make sure the database read and write are synchronized so that no database inconsistencies are offered 
  - However Deploy DB application using Statefull Set in Kubernetes cluster can be so hard . Common practice to host DB outside of the Kubenetes cluster and just have a deployment or Stateless Application that replicate and scale with no problem inside of Kubernetes cluster 
!!! Deployment = Stateless App | Statefull set = Statefull App or Database
```

**Daemon Set:** 
```
  - Daemon Set is Kubernetes Component, just like a deployment or stateful Set with the different that it automatically calculate how many replica of Apps it should deploy depending on how many Node .
  - It is also deploy just one Pod or 1 Replica on each Node in the cluster
  - So when I added new Node it will automatically Pod replica there and when I remove Node it will remove Replica pod
```

## Kubernetes Architecture 

### 2 types of Node ( machine ) that Kubernetes operate on 

#### How Kubernetes does, What it does, and how cluster is self-managed and self-healing and automated ? 

**Worker Node**
```
  Node Processes :
    - 1 Node can have multiple Pods with container running on that Node

  Three Processes must be installed on every Node that are used to schedule and manage those Pods:
    1. Container Runtime: Docker, Container D, Cri-o ... (container d is the most used container runtime in Kubernetes Cluster) 
    2. The processes that actually schedule those pods and the containers then underneath is Kubelet which is a process of Kubernetes itself that has interface both container run and and Node (machine)
      - Kublet start the pod with the container inside
      - Kublet is responsible for taking that configuration and actually running a pod or starting a pod with a container inside then assigning Resources from that Node to that Container like CPU, RAM and Storage
 
  !!! Usually Kubernetes cluster is made up of multiple nodes which also must have container runtime and Kublet services installed. And I can have hundred of those worker Node which will run other pods and container and replica of the existing pods and the way communication between them work is using Services ( static IP address )
  !!! Services which is sort of the load balancer basically catch the request directed to the Pod or application like DB and forward that to respective Pods

   3. KubeProxy is Responsible for forwarding the Request from Services to Pods
      - Kube Proxy has accutally intelligent forwarding logic inside that makes sure the communication also works in the performant way with low overhead
      - Example : If my Application, my-app replica making the request to DB instead of Service randomly forwarding the request to any Replica it will forward to a Replica that running on the same Node as the Pods that initiated the request. Thus this way avoiding low network overhead
```


**Control Plane** -> **How to interact with Cluster**
```
  How to : Schedule Pod, Monitor, Restart Pods , Join new Node ?

  Control Plane proccesses
    - Control plane server or Control Node have completely different processes running inside
    - That is 4 processes that run on every control plane node that control the Cluster state and the Worker Node as well

    1. API Server:
      - So me as the user want to deploy a new application in the Kubenetes Cluster , I interact with API server using some client which is UI, CLI like Kublets or Kubernetes API
      - API Server is like a cluster gateway which get a initials request of any update into the cluster or even query from cluster and also act as a gatekeeper for authentication
      - That mean when I want Schedule new Pod, Create new Services, Deploy new App I have to talk to a API Server on the Control Plane Node and API Server then Validate my Request if everything is fine then it will forward My request to other processes in order to schedule the Pods or create Component I requested

    2. Scheduler:
      - If I send an API server a Request to shedule a new Pod, API server after it validates my request will actually hand it over to the Scheduler in order to start Application Pod on one of the worker Nodes, and of course instead of randomly assign it to any Nodes , Scheduler has a intelligent where to put a Pod, the next Pod will be schedule or next component will be schedule
      - First it will look at my Request and see how much Resources the application that I want to schedule will need how much CPU, RAM and It will go through Worker Node and see available resources in which one of them and if it see one Node has the least busy or have the most resources avalable it will schedule on that Node
      - Note : Sheduler just decide on which Node the new Pod should be scheduled the Process that actually does the schedule mean that actually start the container is Kubelet so it get the request from Scheduler then get the request from that Node

    3. Manager Controller
      - What happen when pod died or crash . It has to detect that pod died and reschedule these pod as soon as possible
      - Manager Controller detect state changes like crashing pods . When pod died Manager Controller detect that and try to recover cluster state as soons as possible for that it make a request to scheduler to reshedule those dead pod and the same cycle happen when the sheduler decide based on the resources calculation which worker node should restart those pod again and make request to the corresding Kublet on those worker node to acctually restart the pod

    4. etcd
     - etcd is a key value store of cluster
     - Can think of it as a cluster's brain
     - Which mean every change of the cluster . For example when a new Pod get schedudled, when I pod died all of these changes get save or updated into this key value store
     - The reason why etcd store is cluster's brain is Bcs all of it mechanism with Scheduler, Controller Manager etc ... work bcs of its data
     - Example : How does sheduler know that resources are available on each worker node ? How does Controller Manager knows that cluster state changed in some way like Pod died, or that Kubelet restart new pod upon the request of the scheduler Or when I make a query request to API server about cluster heath or your application deployment state where API server get all this infomation ?
      - So all of these infomation store in etcd cluster
      - What is Not store in etcd cluster is the actual Application data . For example I have DB Applications running inside of cluster, the data will store somewhere else

  !!! In Practice a Kubernetes cluster is often made up of multiple control planes where each control plane node run its control plane processes Where API server is load balanced and the etcd store form distributed storage accross all control plane nodes
```

## Minikube and kubectl 

**Minikube**
```
  - When I am setting a production cluster .
  - I would have multiple Control Plane Nodes and Worker Node both have their own seprate Responsiblity
  - Separate Virtual or Physical Machine
  - If I want to test something on my local Environment or run something quick etc ... obviously setting a cluster this will be pretty difficult, or maybe even impossible if you don't have enough resources like memory and CPU ...etc

  - So Minikube is 1 Node cluster where Control Plane Processes and Worker Processes both run in 1 Node
  - And this Node will have Docker runtime preinstall so I will able to run a container or the pods with container of this node
```

**kubectl**
```
  Now I have this virtual Node I need the way to interact with that cluster
  The way to create and others Kubernetes componets on that Node
  kubectl help to do that

  - Kubectl is the Kubernetes CLI tools for Kubernetes cluster
   
  How is work ?
  - Minikube run both Control Plane and Worker Processes
  - One of Control Plane Proccesses called API Server is the main entry point to the Kubenetes cluster -> Create anything, configure anything first have to talk to API Server .
  - And the way to talk to API Server is : UI, API, CLI (Kubectl)
  - So when Kubectl command go through API Server to create, destroy pods or etc ... the worker proccesses on minikube node will actually make it happen they will be executing a command to create, destroy pod ..etc  
```

**How to install and run Minikube**
```
  To Install ( https://minikube.sigs.k8s.io/docs/start/ )

----
  - Minikube can run either the Container or Virtual Machine (A driver for Minikube)
  - Inside of Kubernetes cluster, We run Docker container !!! Important to note here that Minikube installation actually come with Docker already installed to run those container
  - But Docker as a driver for Minikube mean that We're hosting Minikube on our local machine as a Docker container itself so We have two layers of Docker
    1. Minikube run as a Docker Container
    2. Docker inside Minikube to run Applications Container
----

  1. Create and Start Minikube Cluster :  minikube start --driver docker
  2. To check Minikube status : minikube status

  To Interact with Cluster I use kubectl

  Minikube CLI to start or create a cluster
  kubectl to configure a cluster
```

## Kubernetes Namespaces 
**What is Namespaces ?**
```
  - In Kubernetes cluster I can organize resources in namespaces
  - I can have multiple Namespaces in Cluster 
  - I can think of namespaces as a Virtual cluster inside of a Kubernetes Cluster 

  ----4 Namespaces per Default----
  - kubectl get namespace : To get name space

  1. kube-system : 
    - Not mean for use 
    - The component that are deployed in a namespace are the system processes that are from the control plane managing processes or Kubectl etc ...

  2. kube-public : 
    - kubectl cluster-info
    - Publicly accessible data
    - Configmap which contains cluster infomation which is accessible even without authentication 
    
  3. kube-node-lease : 
    - Heartbeats of Nodes 
    - Each Node has associated lease object in namespace
    - Determines the availability of a Node  
  
  4. default : 
    - Is the one I will use to create the resources at the beginning if I haven't created a new namespace
    - I can add a new namespace : kubectl create namespace <namespace-name>
    - Using configuration file to create namespace : better way
```

**What need for Namespace ?**
```
```