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

<img width="749" alt="Screenshot 2025-02-07 at 09 59 08" src="https://github.com/user-attachments/assets/36dc3565-2569-4dac-8311-590978d46ded" />

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
    - Ingress is a API object manage external access to Service within Cluster . Provide routing rule to manage how request redirect to which service within Cluster
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
      - In Kubernetes, I just connect it to the pod so that pod acctually gets the data that config map contain . Now I change the name of the service and endpoint of the service I just need adjust the config map
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
  1. Frist use case is 'Following' 
    - If I create all my resources in that default namespace with my complex Application that have multiple deployment replicas of many pods and I have resources etc ... Very soon my namespace will fill with different component and it will very difficult to have an overview of what is in there 
    - Better way is to group resources into namespace 
    - For example : I have DB namespace where I deploy all my DB resources or Mornitoring namepsace where I deploy Mirnitoring resources I need 

  2. Second use case is 'Multiple team conflict': 
    - Use namespace to avoid conflict so each team can work in their own namespace 

  3. Resource Sharing: Staging and Development 
    - If I have 1 cluster and I want to host staging and Development . I can deploy in 1 cluster and use it for both environment. That way I don't have to deploy common resources twice . So now Staging can use both resources as well as Development

    - Blue/Green deployment : Mean in the same cluster I want to have 2 different version production . One is in production now and other one will be the next production the verison of those 2 applications will be different but those 2 still need to use the same resources with that they can use both resources without set up another cluster 

  4. Access and Resources Limit on Namespace 
    - When I have 2 team working on the same cluster each one of them have their own namespace . So I can give a team access to only their namepsace so they can CRUD resources on their own namespace 
    - Limit the resources each team consume . Limit CPU, RAM, Storage per Namespace

  ----Wrap up----
  1. Structure Component 
  2. Avoid Confict between team
  3. Share services between differnt Environment
  4. Access and Resources Limits on Namespaces Level 

  !!! NOTE : I Can't access most of resources from another namespace like: ConfigMap, Secret 
  !!! NOTE : Resources can share is Service
  !!! NOTE : Live globally in cluster and can't isolate them : Persistence volumne and node 

    - I can list components there is not bound to a namespace using : kubectl api-resources --namespaced=false and vice versa kubectl api-resources --namespaced=true
```

**2 ways to create namepsace**
```
   - kubectl apply -f <yaml file> --namepsace=my-name-space

   - In the configuration file : 
      - Under metadata: 
        - namespace: my-namespace

   - Get configuration from namespace is : kubectl get configmap -n <my-namespace> 
```

**Change Active namespace**
```
  - To change default namepsace to whatever namepsace i choose 

  - kubectl config set-context --current --namepsace=my-namepsace

  - There also a tool call kubens but I have to install separtely

  - brew install kubectx 

  https://github.com/ahmetb/kubectx
```

## Services 

**What is Services and Why do we need it ?**

```
  - Each Pod has its own IP address. But Pod easy to die and everytime it regenerate it will create new IP address which is so inconvinience bcs I have to input that new IP address to connect to other pod
  - Stable IP address : 
    - With Service We have solution of stable or static IP address. That stay even when the pod died
    - In front of each pod I set a Service which represent a persistent stable IP address access this Pod
  - Load Balancer :
    - Service also provide Load Balancing Bcs when I have Pod Replica (let's say 3 Repilica) the Service basically get each request target to those 3 Repilica Pod and forward to one of those Pod so client can call 1 single IP address instead of calling each pod individually
  - Loose Coupling:
    - Service are good abstraction ofr loose coupling communication within a cluster . So within cluster, components or pods inside the cluster also from external services like if I have browser request coming to a cluster or if you talking to external DB 
```

**ClusterIP Services**

<img width="600" alt="Screenshot 2025-02-07 at 11 04 08" src="https://github.com/user-attachments/assets/2a88c861-5fe9-44a3-82ea-8b96a1025da7" />

```
  - Default type of Service

  ----How it is work and where it is used ?----

  - When I have Microservice deployed on the cluster -> A Pod with Micro container running inside -> Beside that Micro container I have side-car container that collects logs of Micro and then send that to DB -> Micro container run at Port 3000 and Loggin container run at Port 9000 -> Pod also get an IP address from range that is assigned to Node . If I have 3 worker nodes (3 servers) in my Kubernetes cluster each worker node will get a range of IP address which are internal in the cluster -> If Replica count to 2 we gonna have another Pod which identical to the first one, which will open to the same port and it will get different IP address -> If Micro accessible from Browser so I have Ingress configured and the Request coming in from Browser to the Micro service will be handle by Ingress -> The incoming request get forward from Ingress to the Pod through Service (ClusterIP or Internal Service)

  - A Services in Kubernetes is a component just like a pod but it is not process . It is a abtraction layer that basically represent an IP address . So Service will get an IP address that it is accessible at also be accessible at certion port (Let's say at 3200) . So Ingress will talk to a Service or hand over the request to the service at this IP address at this Port

  - So the way it work is I define Ingress rule that forward the request based on the request address to certain services and we define the services by its name and the DNS resolution then map Service name to IP address that this service actually got assigned  So this is how Ingress knows how to talk to a Service

  - Once the request hand over Service at this address and then Service will know to forwards this request to one of those pods that register at service endpoint
```

**How does Service know which Pods to forward the request to and which Ports to forward it to ?**

<img width="600" alt="Screenshot 2025-02-07 at 11 27 25" src="https://github.com/user-attachments/assets/d75e6041-c6e5-43d0-80aa-726af23b0948" />

```
- Service know which Pods bcs of Seletor :
  - Pod identified via Selector
  - Key-Value pair defined in Selector (Lable of the Pod)

- If a Pod has multiple Ports. How do Service know which port to forward to
  - Defined in "targetPort" atrribute

  ----Service Endpoint----
  - When I create Service K8 create endpoints object that has the same name as Service itself . K8 will use this to keep track of which Pods are members of Services or Which Port are endpoint of the Service 
```

**Service Communication Example**

  <img width="600" alt="Screenshot 2025-02-07 at 11 32 03" src="https://github.com/user-attachments/assets/46dd9ec2-dda7-4f91-8503-5ba18a364086" />

```
  Let's say Microservice Applications using MongoDB

  - I have 2 Repilica of MongDB in the cluster which also have Service Endpoint (ClusterIp) and the Service has its own IP address 
  - Now the microservice application inside the pod can talk to the MongoDB also using Service endpoint so the request will come from one of the Pod that get request from the Service to the MongoDB service at Service of MongDB IP address and the Port the Service has open then Service will again select one of those Pod Replica and forward those request to the Selected Pods at the Port 
```

**Multi Port Service**

<img width="600" alt="Screenshot 2025-02-07 at 12 16 54" src="https://github.com/user-attachments/assets/5f7582c8-4d9f-4bcb-a9b8-8a0089ec3334" />

```
  - Inside the MongoDB Pods, there is another container running that select the monitoring metrics for Prometheus that would be MongoDB exporter
  - And that container run at Port 9216
  - In the cluster , We have Prometheus Application that scrapes the metric endpoints from this MongoDB exporter container from this endpoint. That mean that service has to handle two different endpoints request which also mean that service has two of its own port open for handling these two different request -> 1 from client that request to MongoDB and 1 from the client like Prometheus that want to talk to the MongoDB exporter Application . So that is Multi-Port Service

  - So when I have Multi-Port in 1 Service . I have to Name those Port 
```

**Headless Service**

<img width="600" alt="Screenshot 2025-02-07 at 12 27 03" src="https://github.com/user-attachments/assets/7c07a5cc-e0a7-4379-b61b-d4eda7152476" />

```
  - Client  want to communicate with 1 Specific Pod directly
  - Or Pod want to talk directly with specific Pod
  - Not randomly selected

  ----Use case----
  - Deploy Statefull Application like DB -> Pod replica are not indentical -> Each one has individual state and characteristic

  - Client need to figure out IP address of each Pod
    - Option 1 : API call to K8s API Server ?
    - Option 2 : DNS lookup
      - DNS lookup Service - return single IP address (ClusterIP)
      - However if I set ClusterIP : none it will return the Pod IP address instead of the service

  !!! So the way to defined Headless Service is to set ClusterIP : none

  !!! When we deploy Stateful Application in the Cluster like MongoDB . We have ClusterIP Service (Do loadbalancing stuff, ...) and along side with Headless Service (Client need to communicate to one of those Pod directly)
```

**Node Port Services**

<img width="600" alt="Screenshot 2025-02-07 at 13 46 24" src="https://github.com/user-attachments/assets/d33c50a1-a608-4eef-b4a2-610cf7343fd8" />

```
  - When we define Service Configuration , we can specify the type of the Service
  - 3 Type Attribute : ClusterIP, NodePort, Load Balancer

  - Type Node Port Services is create a service that is accessible on a static port on each worker node in the cluster

  ----To compare Cluster IP and Node Port----
  - The ClusterIP service is only accessible within the cluster itself so no external traffic can directly address Cluster IP
  - The NodePort Service makes the external Service accessible on static or fixed port on each worker Node
    - So instead of Ingress Browser will request directly to the Worker Node at the Port that the Service Specification define
    - And the Port that node Port service type expose is defined in the Node Port attributed
    - Nodeport predefine value has range 30000 - 32767
```

**Load Balancer Services**

  <img width="600" alt="Screenshot 2025-02-07 at 13 56 55" src="https://github.com/user-attachments/assets/c85cc99e-f80b-4f06-87e7-5a33de956440" />

```
  - Service become accessible externally through a cloud provider load balancer functionallity
  - Each cloud provider has its own native load balancer implementation and that is created and used whenever we created a load balancer service type
  - Whenever we create Load Balancer service, Nodeport and clusterIP service are created automatically by Kubernetes to which the external load balancer of the cloud platform will route the traffic to

  ----How to define Load Balancer?----
  - Type: Load Balancer
  - Port of the Service (Cluster IP Service)
  - nodePort: Port open on Worker Node but it is not directly accessible externally but only through the Load Balancer itself
  - So entry point become the Load Balancer first and it can then direct the traffic to NodePort and ClusterIP
```

**Wrap Up**
```
  NodePort Service not for external connection

  I would have Ingress for external connection OR

  I would have Load Balancer for external connection
```

## Ingress 

**External Service and Ingress**
```
  - User Ingress instead of external Service I would instead have internal Service
  - I would not open to the Browser through a IP address and Port 
```

**Yaml File syntax**
```
  apiVersion: networking.k8.io/v1
  kind: Ingress
  metadata:
    name: myapp-ingress
  # Routing rules
  spec:
    rules:
      # The main address or all the requests to this host must be foward to a internal Service
      # This should be a valid domain address
      # Should map domain name to IP address which is the entrypoint 
      - host: myapp.com
        # This http is Incoming request get forward to internal-service 
        http:
          paths:
              # URL path . Everything after the domain name 
            - path: /
              pathType: Prefix
              # Backend is a target where the request, the incoming request will be redirected and the service name should corespond the internal-service and the port should be internal service port 
              backend:
                service:
                  name: my-app-internal-service
                  port:
                    number: 8080
```

**How to configure Ingress in the Cluster**
```
  - Need a Implementation for Ingress call Ingress Controller
  - Step 1 : Install an Ingress Controller : https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
    - Which is basically another Pod or another set of Pod that run on My Node and does evaluation and processing of Ingress rule
    - Ingress Controller is to evaluate all the rules that I have define in my Cluster, and this way to manage all the redirections -> This will be a Entrypoint for the Cluster request to that domain or subdomain rules that I have configured . And this evaluate all the rule 
```

**Consider Environment on which your cluster run**

<img width="600" alt="Screenshot 2025-02-09 at 13 57 53" src="https://github.com/user-attachments/assets/018d0cef-802a-43c1-ae55-57c6c59d0194" />

```
  ---Things need to understand for setting up whole cluster to be able to recive external request---

  1. Consider The Eviroment where K8 is running
    - Cloud Provider: Have their own K8's solution and their own virtualization load balancer
      - I would have Cloud load Balancer that is specificlly implemented by cloud provider and external request will first hit  a load balancer and the balancer will redirect the request to ingress controller
    - In Bare Metal Environment : I would have to do that part myself . I need to configure some kind of entrypoint myself
      - Use Proxy Server that will take a role to of that load balancer
      - I have seperate Server, Give it Puplic IP address, open port in order for the request to be accepted, and this proxy server will act as a entry point to my cluster

  ----Wrap up----
  - External Request -> proxy server or cloud provider load balancer -> Ingress controller -> Decide which Ingress rule to that specific request 
```

**Ingress Controller in Minikube**
```
  Step 1. Install Ingress controller on Minikube : minikube addons enable ingress
  Step 2: Create Ingress rule that the controll can evaluate
    - Enabling the Minikube dashboard: minikube dashboard -> this will set up dashboard in my environtment and open up in a new browser window for us to access internally when it ready
  Step 3: kubectl get all -n kubernetes-dashboard ---- Show me all the components I have in kubernetes-dashboard
  Step 4: Create Ingress rule -> Then apply ingress rule : kubectl apply -f dashboard-ingress.yaml 
  Step 5: kubectl get ingress -n kubenertes-dashboard -> to see my Ingress 
      -- The Address in my ingress would be the address that would map to the host dashboard.com -> But minikube work different . For demo I will use the localhost address of 127.0.0.1 as a replacment to complete the ingress step instead of the Address
      -- Go to /etc/hosts -> 127.0.0.1  dashboard.com
      -- Which mean that the Request will come to Minikube cluster will be hand over to Ingress controller and Ingress controller will go and evaluate the rule that we defined and finally foward that rerquest to Service
  Step 6 : run minikube tunnel
      -- Tunnel specific to Minikube
      -- Minikube tunnel is a process will map this local address to a Service for Ingress 
      -- If I were set up in the Traditional Kubernetes Environment on the cloud platform I just need to put the assign IP address IP address to our host file 
```

**Ingress Default Backend**

<img width="961" alt="Screenshot 2025-02-10 at 11 48 51" src="https://github.com/user-attachments/assets/cf5e73ce-a0f8-4d93-b4dc-2e607123f44b" />

```
  - kubectl describe ingress dashboard-ingress -n kubernetes-dashboard
  - The attribute default backend that map to attribute default backend port 80 -> What is mean is that whenever the request come in to Kubernetes cluster that is not mapped to any backend so no rule for mapping that request to a service then this default backend is used to handle that request . If I don't have this service created or defined in my cluster, Kubernetes will try to forward it to Service, it won't find i will get default error

  ----Good Use----
  - To define custom error messages when a page isn't find, when request coming in i can handle

  - So I have to create a Internal Service, Default backend and port number, and also the pod or application that sends that error custom response 
```

**Multiple path for the same host**

<img width="600" alt="Screenshot 2025-02-10 at 11 54 54" src="https://github.com/user-attachments/assets/b61a6e4f-fe1d-49a0-b0d5-8299e29bcf05" />

**Multiple Subdomain or Domain**

<img width="600" alt="Screenshot 2025-02-10 at 11 57 04" src="https://github.com/user-attachments/assets/7da581a4-7a90-4d22-a9cf-fcf6de42d07c" />

**Configuring TLS Cerificate - https://**

<img width="600" alt="Screenshot 2025-02-10 at 12 00 06" src="https://github.com/user-attachments/assets/14ecbf54-d063-4ade-8de0-ae5c648ad34b" />

<img width="600" alt="Screenshot 2025-02-10 at 12 01 18" src="https://github.com/user-attachments/assets/ba42cc09-33cb-4e7f-b8e3-d84871a42447" />

## Kubernetes Volumes 

**3 Components of Kubernetes Storage : Persisten Volume, Persisten Volume Claim, Storage Class**

**The needs of Volumes**
```
  - If the Pod died and restart . All the data in that pod will be gone . So I need volumes

  ----Storage Requirment----
  - I need storage that doesn't depend on the Pod lifecycle to store data
  - Storage must be available on all Node in the cluster
  - Storage need to survive even whole cluster crash

  ----Another Use Case is a Directory----
  - Maybe I have Apps that writes and read files from pre-configured directory . This could be session file for Apps or configuration file etc.... 
```

**Persist Volumne**

<img width="600" alt="Screenshot 2025-02-10 at 12 42 38" src="https://github.com/user-attachments/assets/ddf100b0-6f6b-4566-b2d7-571e0fa91f60" />

```
  - Peristent Volume is like cluster Resource like RAM, CPU to store data
  - Create Persistent Volumne by using Kubernetes Yaml file
  - Persitent Volume is just an abstract component . It need physical Storage like : local hardrive, nfs server, or cloud storage ....

  ----Where does this storage come from and who makes it available to  cluster?----
  - K8's doesn't care about what actual physical store is
  - I have to decide what type of storage my cluster services or application would needs and create and manage them myself .
  - Managing meaning do backup, and make sure they don't get corrupt ...etc

  - So storage in Kubernetes is an external plugin to my cluster . Whether it local storage (on actual node where cluster running) or remote storage

  - I can have multiple storages configured for my cluster .

  - By creating persistent volumne I can use this actual physical storage

  - In the Peristent Volumne Spec section I can define which storage backend I want to use to create storage abstraction or storage resources for application

  - Depend on storage type, Spec attribute will be different

  - Peristence Volume are not namespace -> Their accessible to the whole cluster
```
<img width="400" alt="Screenshot 2025-02-10 at 13 00 19" src="https://github.com/user-attachments/assets/b20ff559-a96a-478b-b9e4-eb0eb3861086" /> <img width="400" alt="Screenshot 2025-02-10 at 13 00 51" src="https://github.com/user-attachments/assets/05bd1a28-1041-4344-823a-138af6d1602e" /> <img width="400" alt="Screenshot 2025-02-10 at 13 07 31" src="https://github.com/user-attachments/assets/23053794-eb78-4a7d-801f-2f31c65091a3" />

**Local and Remote volumes Type**
```
  - Local volume type is violate 2/ 3/ requirment for data persistence:
    -- Being tied to 1 specificed Node
    -- Not survive when cluster crashed
    -- Should alway use remote storage 
```

**Who create peristen volume and when?**
```
  - Persistence volumne are Resources need to be there BEFORE then the Pod depend on it created

  ----K8s Administator and K8s User----
  - K8s Administator will set up cluster and maintain it . Make sure cluster has enough resources
    - K8s Admin will configure actual storage
    - Create the PV components from these storage backends

  - K8s User deploy K8s applications directly or through CI/CD pipelines
    - K8s User will configure Applications base on the Storage K8 admin provioned
    - In other words Applications have to claim that Persistence Volume . To do that I use PVC (Persistent Volume Claim)
```

**Peristence Volume Claim**

<img width="600" alt="Screenshot 2025-02-10 at 13 50 00" src="https://github.com/user-attachments/assets/26632c0f-1c63-4cd4-9dfe-a697519bd945" />

```
  - Created by Yaml configuration
  - PVC claim a volume with certain storage size or capacity which is define in the persistent volume claim . and some additional characteristic like access type like Readonly ReadWrite etc... whatever PV match this criteria or sastisfy this claim will be used for the Application then I have to use that claim in Pod configuration . So in the Pod specification here, I have the volume attribute that references the persistent volume claim with its name so now the pod and all the container inside the pod will have access to Peristent volumne
```

**Level of Volume Abstractions**

<img width="600" alt="Screenshot 2025-02-10 at 14 22 49" src="https://github.com/user-attachments/assets/de0ff827-906c-4a4f-b619-02f9a464489e" />

```
  - Pod request Volume through PVC (Persistent Volume Claim) -> PVC will try to find a PV (Persistence Volumn) in the cluster that sactify the PVC -> The PC that has the backend Storage that it will create Storage Resources from

  !!! NOTE : PVC must be in the same namespace as the Pod using PVC

  - Once the Pod finds matching persistence Volume through PVC -> The Volume is then mounted into the Pod -> then Volume can be mounted into the Container inside the Pod -> Now the container can read and write to the Storage and when the pod die new one get created it will have access to the same storage and see all the change the previous Pod or previous container made

  ----Why so many Abstraction?----
  - The benefits is as a K8 user or developer when I create cluster I don't care where the actual volume is
  - Don't want to setup actual storage 
```

<img width="600" alt="Screenshot 2025-02-10 at 14 51 26" src="https://github.com/user-attachments/assets/c41a3d09-6bf0-4aec-964f-5aecea823fe1" />

**Wrap up**

<img width="600" alt="Screenshot 2025-02-10 at 15 03 44" src="https://github.com/user-attachments/assets/5af710fe-b7c8-4ce2-9771-d10717d084c7" />

```
  - In the volumes : Specify what volume to provide
  - In the container : Where to mount those in the container
      - mouthPath : The path to mount the volume into inside the container

  !!! Note : Pod can use multiple Volumne of different Type  
```

**Different Volunme in 1 Container**
<img width="600" alt="Screenshot 2025-02-10 at 15 09 34" src="https://github.com/user-attachments/assets/c0405b4b-3006-41b0-ba2e-af81d239a808" />

**Storage Class**

<img width="500" alt="Screenshot 2025-02-11 at 11 52 31" src="https://github.com/user-attachments/assets/7e95a769-1ebe-474c-b070-deb7d8ae66fd" /> <img width="600" alt="Screenshot 2025-02-11 at 11 58 13" src="https://github.com/user-attachments/assets/4adacde4-99da-412d-8320-b1cb95fc65dc" />

```
  - To persist data in Kubernetes, admin need to configure storage for the cluster , created PV so Developer can use PVC to claim that storage
  - But consider cluster where 100 of applications where things get deployed daily and storage is needed for these application so Developer ask Admin to create Volumne they need before deploying them and Admin may have to manually request storage from cloud or storage provider and create 100 of PV for the Apps manually and that can be time consuming, tediuous,
so to make it more efficent the solution is Storage Class

  - Storage Class create Peristent Volumne dynamically when PVC claim it
  - Created by using Yaml
  - Storage Backend is defined in the Storage Class component via Provisioner attr
  - Provisioner tell Kubernetes which Provisioner to be use for specific storage platform or cloud provider to create PV out of it
  - Each storage backend has own provisioner
  - Internal Provisioner - "Kubernetes.io"
  - other type is External Provisioner
  - Parameters : Configure for storage we want to request for our Persistent Volumn

  ----Storage Class is another Abstraction Level----
  - Abstract undelying storage provider
  - Parameter for storage

  ----Storage Class usage----
  - Request or claim by PVC 
```

## ConfigMap & Secret Volumn Type 
**When to use Configmap and Secret Volumn ?**
```
  - Think of Applications that take Configuration as Parameters when they start like Promethus, elastic search, mosquitto ... or Application that have password or sensitive data need to secure

  ----How to pass these config files to Kubernetes Pods?----
  - ConfigMap and Secret are used for external configuration of individual value
  - Individual value are used as value for the environment variable

  ----Create files of Configmap and Secrect then mount to the pod and the container----
```

**Example of Create files of ConfigMap and Secret**

<img width="600" alt="Screenshot 2025-02-11 at 12 17 04" src="https://github.com/user-attachments/assets/5231f362-d997-424b-8456-e54a3ee245ed" />

```
  - ConfigMap and Secret must be created and exist before Pod start in the cluster 
```

<img width="600" alt="Screenshot 2025-02-11 at 12 51 33" src="https://github.com/user-attachments/assets/4f23ae5c-d4fb-444f-a8e3-f65840253f48" />


## Stateful set 

**Stateful Applications**
```
  - State Apps like data basis : MongoDb, elasticsearch, MySql ect ... or any applications that store data to keep track of its state .
  - In other words is the Application that track state by saving that infomation in some storage
```

**Stateless Applications**
```
  - Don't keep record of previous interaction
  - Each request is completely new | isolated interaction based entirely on the information come with it
  - Sometime Stateless Apps connect to Statefull Apps to forward those request like 
```

**Example**

<img width="600" alt="Screenshot 2025-02-11 at 13 17 08" src="https://github.com/user-attachments/assets/803839f5-c6f2-4907-9dc3-b3e731a7dbea" />

```
  - Nodejs Apps (Stateless )connect to MongoDB database (Statefull)
  - When request come in Nodejs Apps it doesn't depend on any previous data to handle incoming request
  - It can handle on the payload of the request itself
  - Now typical such request will additionally need to update some data in the database or query the data that's where MongoDB comes in
  - When Nodejs forward the request to mongoDB, mongoDB will update the data based on its previous state or query the data from its storage, for each request it need to handle data and alway depend on data or state to be available
  - Nodejs is just pass through for data query/update 
```

**Deploy of Stateful and Stateless applications**

<img width="600" alt="Screenshot 2025-02-11 at 13 30 18" src="https://github.com/user-attachments/assets/46e08802-e740-4a39-99af-7622218a9ce4" />

```
  - Stateless Apps deploy using Deployment

  - Statefull Apps deploy using Statefullset Component
    - Just like Deployment . StatefulSet relicate apps pods or to run multiple replica of it

  - They both manage Pods base on the container Specification
  - Configure Storage the same way

  ----What is the different of Deployment and StatefulSet ?----
  - Replicate Stateful Apps is more difficult
  - Statefull Apps has additional requirment that Stateless Apps do not have
    - Can't be create/delete at the same time
    - Can't be randomly address
    - Bcs Replica Pods are not Identical
    - Pods have their own identity on top of the blueprint of the Pod that they are created from
    - And giving each pod its own required individual indentity is what StatefulSet does
```

**Pod identity**

<img width="600" alt="Screenshot 2025-02-11 at 13 38 06" src="https://github.com/user-attachments/assets/f1bc29b3-8c9b-4019-a9b3-afebbb053454" />

```
  - Sticky identity for each pod
  - Create from same Specication but they Not interchangeable
  - Peristent Identifier accross any re-sheduling : Mean when pod died it get replaced by new Pod with the same Indentity

  !!! Note : Next Pod is created if previous is up and running !
  !!! Note: Delete StatefulSet or scale down to 1 Replica

  ----Why is it need Identity ?----
  - Every Pod has its own Identifer
  - Unlike Deployemnt where Pods get random-hash at the end StatefulSet Pods get fixed ordered name
  - Made up of StatefulSet name : $(stateful-set name)-$(oridinal) . Exp: mysql-0 , mysql-1 .... Frist one is Main , Next one is Replica 
  
    ----Scaling DB applications----
      - When I start with Single Mysql Pod I can use for Read/Write data
      - But then I add the second one it can not act the same way bcs if I let 2 instances of Mysql Write or Change the same data I will end up with data Inconsistence
      - Instead there is mechanism that decide only 1 pod is allowed to Change or Write the data which is Shared Reading at the same time by multiple Pod .
      - And the Pod that is allow to Update or Change data is called the main other is call replica
      - And there are different between those Replicas Pod in term of Storage

    ----Next point of differnt is Storage----
      - They do not use the same physical storage eventhough they use the same data they are not using the same physical storage data
      - They each have their own replica of storage that each one of them have access for itself
      - This mean each Pod replicas at any time must have the same data as the other ones -> To achive that they have to continously syncchronization of the data
      - Since the Main is the only one can Change data and Replicas need to take care of their own data storage . Replicas must know about each such change so they can update their own data storage to be up to date for the next query request

      ----What happen when new Pod join that existing setup ?----
      - New Pods also need to create its own Storage and take care of synchronzing it
      - It first clone the data from previous pod . Once it has up to date data from previous pod . It start continous synchornize as well
      - It also mean I can have temporary storage theoetically for Statefull Application and not persist the data at all . Since the data get replica between the pod . It just rely on data replication between the pod -> But data will lost when pod die
      - Therefore it still best practice is using PV (Persistence Volume)
      - When Pod died or StatefulSet get comepletely wiped out the PV still remain the data Bcs PV lifecycle isn't tied to other component's lifecycle

   !!! Note: All this mechanism in place to Protect the Data and its State 
```

**Pod State**
```
 -----So I have to configuring PV for my StatefulSet----
    - Since each pods has its own Storage meaning it's the own persistent volume that is then backed up by its own physical Storage which include the synchoronized data or replica database but also the State of the Pod
    - So each Pod has its own State which has infomation about whether it a Main Pod or Replica or other individual characteristics and all of this get stored in the Pods own storage -> That mean when a Pod died and get replace the Persistence Pod Identifier make sure that the storage volume get reattched to the replacement Pod

    ----For reattachment to work----
    - Use Remote Storage to available on other Nodes can not do that using local volumne Storage 
```

**2 Pod Endpoint**

<img width="600" alt="Screenshot 2025-02-11 at 14 56 27" src="https://github.com/user-attachments/assets/dd07039a-d22b-4c98-a122-c2fd687e119e" /> <img width="400" alt="Screenshot 2025-02-11 at 14 57 21" src="https://github.com/user-attachments/assets/bb90e5cf-e697-43f3-8c1e-3ca9d75070e2" />

```
  - To these fixed name: Each pod in the StatefulSet get its own DNS endpoint from a Service
  - There is Service Name for Stateful Apps that will address any Replicas Pod . And in addition to that there is individual DNS name for each Pod which deployment Pods do not have
  - Individual DSN name made up of Pod name
```

**2 Characteristic**

<img width="400" alt="Screenshot 2025-02-11 at 15 00 03" src="https://github.com/user-attachments/assets/a07c2f7c-cbee-4e68-bc5e-f75aca6b57e3" /><img width="400" alt="Screenshot 2025-02-11 at 15 00 59" src="https://github.com/user-attachments/assets/254a4ba9-d904-454d-b4ba-247121fe08c4" />

```
  1. Having Fixed name
  2. Fixed DNS name

  - When Pod restarted the IP address will change and the name and endpoint will stay the same
  - Sticky identity make sure each Replicas Pod can retain its State and Role
```

**Important**
```
  - Relicate StatefullSet is complex
  - I need to do :
    - Configure Clonging and Data synchornize
    - Setup Remote Storage

  Bcs Statefull App not perfect for containerized Environment 
```

## Manage K8 Service 

**Build a Use Case Web App**
```
  ----Set Up----
    - Web app with DB
    - Available from a Browser https with my domain name
    - Security Cluster
    - Data Peristent for DB
    - Have Dev and PRO Environment so I can test new features before release
    - Set up as effecicent as possible 
```

**Managed and UnManaged K8s CLuster**
```
  ----Where will my K8s Cluster will be deploy?----
    - Consider when I want to set up K8s Cluster on cloud platform like Linode . I have 2 options : 
      1 . Set up from sratch : Spin up 6 Server Instances . Set up Control Plane and Node Worker . I need to manage completely myself
      2 . Cloud Provider provide Manage Kubernetes Service
        - I don't have to create Cluster from Sratch
        - Done Automatically by Cloud Platform
        - I need to choose how many Worker Node -> Everthing pre-installed included container runtime
        - Control Plane Nodes created and managed by Cloud Provider
        - Save time and effort and cost 
```

**Process of Managed K8 CLuster**
```
  1. Spin up my K8s cluster on cloud
    - I created Cluster with 3 Worker Node to Deploy my Application 
    - Choose Work Node and their resources
    - Select Region/Data center where Worker Node will run
    - Connect using Kubectl

  2. Data Peristence for my Cluster
    - If I need 3 replicas DB PV I need Storage for all 3
    - I need to create physical Storage and make it available for cluster like Database
    - I need to create Peristence Volume with storage backend
    - Then I need to attach Volume to DB pod
    - If I Use Linode Block Storage I don't need to Create physical storage, create PV ...
    - I just use Linode's Storage Class and Linode will create the persistent volume with a respective physical storage in the background automatically
    - And When I deploy my DB Applications the storage will get attach to DB pod

  3. Load balancing K8 cluster
    - So my Apps.js is running with MongoDB and storaged configured
    - Now we need Services access from Browser through Ingress
      - Ingress manage routing of incoming request
      - Need to install and run Ingress controller in K8 cluster
      - And configure Ingress routing rules

    ----How does Ingress get incoming request?----
      - It happen through Linode load balancer which is entrypoint of my cluster

      ----How Load Balancer work?----
      - Linode load balancer is a load balancer for Worker Node where Nodes are the Linode Server Instances
      - When I have 2 server Nodes 1 run App.js 1 run MongoDB . and I expose web server using IP address -> So Browser can send request directly to the server -> But it can't scale when I have a lot of traffic to come into App -> Also my app will become unavailable when I need to fix or reboot or reconfigure it
      - Instead I add the NodeBalancer in front that take incoming request and direct it to the web server -> So NodeBalancer will get public IP address and Web server will be hidden away and accessible only at private IP from the load balancer itself -> Now I can add multiple Balancer Under that NodeBlanacer. Balancer can forward the request to .
      - So I can scale up and down without user notice anything
      - Setup only once

    ----Session Stickiness with LoadBalancer----
    - Which mean  If user authenticated on 1 Server that keep sessions in the memory meaning only that web server instance has session information and knows the clients You can configure the Balancer to forward the next request from that client to the same web server instead of randomly picky and passing the request to any of server

    ----Secure connection with SSL certificate----
    - I can configure the NodeBalancer with SSL certificate
    - In Linode I can do it with Cert Manager plugin which help managing TLS, SSL certificate
    - Then I can get this certificate and store it into Secret and use it to secure the connection to my cluster

  4. Data center for K8s Cluster
    - Can Move Application closer to my users by using Availablity Zone of Cloud Platform

  5. Move App to othet Cloud Platform
    - Migrate part to Private Cloud
    - Migrate part of my App to another Cloud Platform
    - When I using a cloud provider, I use services specific to that cloud provider
    - My App will get closely tied up to that cloud provider and I can't migrate easily
    - I may need to re-program or re-configure . That's call Vendor-lockin

  6. Automate Task
    - Setup grow inside .
    - My Infrastructure get more complex
    - Automate creating, Automate configuring
    - Automate Deploy App and Service
    - And I can do that using Automation tools: Teraform 
```

## Helm (Package Manager for K8)

**What is Helm**
```
  - Helm is Package Manager for K8
  - To Package YAML Files and distribute them in public and private Repo

  ----Helm Chart----
  - Bundle of Yaml File
  - Using Helm to create Helm Chart
  - And push to Helm Repository
  - Or Download and use Existing one

  ----Commonly Use Deployment Repo----
  - Database Apps: MongoDB, Elastic Search, MySQL, etc ....
  - Mornitoring Apps 
```

**Example**
```
  - I have deployed my App in K8 cluster and I want to deploy Elastic Search additionally -> That my Cluster use to collect its logs
  - To Deploy Elastic Search stack in my K8 Cluster I would need couple of K8 Component like :
    - Statefull Set for Statefull Application
    - ConfigMap with External Configuration
    - Secret for Credential or Secrect Data
    - K8s User with Permissions
    - and Services
  - So it take some time to Configure all of that . But Elastic Search deployment is standard accross all Cluster, other people will properly have to go through the same .
  - So If someone created files once and packaged them up and make it available somewhere who that other people who also use the same kind of deployment could use them in K8 Cluster
  - And that bundle of Yaml File called Helm Chart

  - Now I have a Cluster I need to deploy third Party Application -> I can look it up
    - Using CLI: helm search <keyword>
    - OR Heml'own Public Repo at ArtifactHub
```

**Public and Private Repo Helm**

```
  - Public Repo at ArtifactHub

  - Private Repo Helm : Shared  in Origanization . Not Public 
```

**Second Feature of Helm**

<img width="400" alt="Screenshot 2025-02-12 at 12 25 24" src="https://github.com/user-attachments/assets/1f9ff2c7-d195-4dd8-9f83-9b19f0ff99f0" /> <img width="400" alt="Screenshot 2025-02-12 at 12 26 21" src="https://github.com/user-attachments/assets/338d1c01-06bb-435d-ac86-7cdc91078765" />

```
  - Templating Engine
    - Imagine If I have an Applications that made up of multiple MicroServices in my K8 Cluster
    - And Deployment and Service of each of those microservice are pretty much the same and the different is Application name and verison or docker image name and verison tag
    - Since most value are the same I can use Helm :
      1. To define common blueprint for all MicroServices
      2. Dynamic Value are replaced by placeholders and that would be Template File
    - Practical for CI/CD : In Build I can replace Value on the Fly before deployment
```

**Another Use Case**

<img width="600" alt="Screenshot 2025-02-12 at 12 57 12" src="https://github.com/user-attachments/assets/69034734-a811-4e42-8cd5-00c425e9a3b8" />

```
  - When I deploy the same Applications accross different Kubernetes Cluster
  - Consider use Case when I have my MicroService Applications that I want to Deploy on Development, Staging, and Production Cluster
  - Intead of Deploy the individual yaml files separately in each cluster, you can package them up to make your own application chart, that will have all the necessary yaml files that particular deployment needs. And then to can use them to redeploy the same environment in Kubernetes Cluster 
```

## Helm Chart Structure 

<img width="600" alt="Screenshot 2025-02-12 at 13 07 42" src="https://github.com/user-attachments/assets/071e9d29-170c-4ad5-b3e2-66a84d9c81e3" />

```
  - Chart is made up of Dir Structure
  - Top level: name of the chart
  - chart.yaml: metadata infomation
  - values.yaml: Values for template files -> default value that I can overwrite
  - chart folder: Chart dependencies inside
    - If this chart depend on other charts, then those chart dependencies will be stored here .
  - Template folder: Acutal template files

  - Install command : helm install <chartname>
    - When command executed Helm install command to actually deploy those yaml
    - Template file will be filled with the values from values.yaml, producing Kubernetes manifests that can be deploy to Kubernetes 
```

**Values Injection**

<img width="600" alt="Screenshot 2025-02-12 at 14 06 02" src="https://github.com/user-attachments/assets/b6f4f80a-5bac-4137-936e-6fb190761002" />

<img width="600" alt="Screenshot 2025-02-12 at 14 14 55" src="https://github.com/user-attachments/assets/9e1b38a2-4738-48fc-a573-d9fd299106a2" />

```
  ----Default value can be override----
    - Can override when execute : helm install --values=my-values.yaml<chartname>
      --values : with this flag I can provide the alternative value
    - OR command line : helm install --set verions=2.0.0
      --set : Can also define value with this flag

  ----Release Managment----
  - Manage by Helm Binary 
  - The way it works is by using the Helm upgrade and rollback command
    - If an update to Helm Chart is Release or if need to change Configuration of my Deployment I can run Helm upgrade
    - When helm upgrade <chartname> executed : Any Change since installation of the last Release are going to be applied . Simply Remove it and Create new one
  - If Upgrade go wrong I can rollback with command : helm rollback <chartname>
```


## Helm Demo Project Managed K8 Cluster 

**Install Statefull App Using Helm**

**Technology Use**
  - Linode : Cloud platform
  - Helm K8's cluster

**Overview**

<img width="600" alt="Screenshot 2025-02-12 at 14 57 36" src="https://github.com/user-attachments/assets/e79a5741-9612-42f1-8597-d855e41c6c8b" />

```
  - I will deploy a replicated DB and configure its PV and make it accessible from UI client Browser using Ingress
  - I will use Helm to make process more efficent

  1. Deploy MongoDB in Linode Cluster using Helm 
  2. Create replicated MongoDB using Statefulset . Also Configure data Persistence for DB using Linode Clould Storage
  3. Deploy UI client Mongo Express in order to access it from Browser
  4. For this Client I will configure Niginx Ingress -> Deploy Ingress controller in the cluster and configure Ingress rule in order to demonstrate handling browser request in the cluster

  !!! Almost 100% of this whole setup is what I properly will always need to do when I set up my Kubernetes Cluster 
```

**Create K8s Cluster on LKE Linode**
  
  1. Sign Up for Linode
  2. After Sign Up -> In the UI Choose Kubernetes
     1. Choose Label
     2. Region
     3. Kubernetes Version
     4. Choose Node : Worker Node (Linode Already has taken care of Control Plane . I don't need to set up Control Plane)
       1. Choose Share CPU
       2. Choose Capacity
   3. While Nodes are running :
      1. I need access credentials to access from my local machine so I download Kubeconfig credentials that Linode provided
      2. Change Permission to only my user can read from the file: `chmod 400 Kubeconfig`
      3. Set Kubeconfig.yaml as ENV `export KUBECONFIG=kubeconfig.yaml` -> Then my local machine connected to Nodes

**Deploy MongoDB Statefulset in the cluster**

  ----2 ways to deploy StatefulSet----
  1. Create all Configuration file myself
  2. Use Bundle of those Configuration files | Helm Chart

  ----Using Helm Chart----
  1. Install helm on mac : `brew install helm`
  2. Bitnami maintain MongoDB Helm Chart . I will install the Bitnami repo : `helm repo add bitnami https://charts.bitnami.com/bitnami` 
  3. To search bitnami repo : `helm search repo bitnami` -- Search for mongodb helm chart in bitnami : `helm search repo bitnami/mongodb`
  !!! Note : When I execute helm command It is going to execute it against the Cluster that I am connected to

  ----Installing the Chart----
  - When I am installing the chart there might be some value that I want to overide -> Chart provide some default value and I want to check what Parameter or Value I can override:

  - Example Yaml file that I want to overwrite some value
  <img width="300" alt="Screenshot 2025-02-13 at 13 11 24" src="https://github.com/user-attachments/assets/adc9494b-1866-4751-a71f-9ccd50be8ecb" />

  1. Define to run Statefulset with : `architecture`
  2. Set root password : `auth.rootPassword`
  3. Configure Volume -> Helm Chart should use the StorageClass of Linode Cloud Storage
  4. Install the MongoDB chart : `helm install mongodb --values helm-mongodb.yaml bitnami/mongodb`
  <img width="300" alt="Screenshot 2025-02-13 at 13 17 37" src="https://github.com/user-attachments/assets/9e724a72-6991-4999-b393-7cc37845543f" />
  
  - When I execute this . It will install that mongoDB chart in my cluster

  ----Check my Cluster----
  - Check pod: `kubectl get pod`
  - Check all: `kubectl get all`
  - Check secret: `kubectl get secret`
    - This contain root password I provided in the yaml file
    - Persistence Storage :
      - The configuration Linode block storage defined as storage class
      - What happen is that when Statefulset was created for each pod one, a physical storage was created for each of three pods
      - And for each physical storage , a persistence volume was created and that is now attached to the Node where the Pod is running 

**Deploy Mongo Express - UI for MongoDB**

  - I will create my own Configuration file for 1 Pod and 1 Service for Mongo Express

  1. Create helm-mongo-express.yaml

  ```
  apiVersion: apps/v1
  kind: Deployment 
  metadata:
    name: mongo-express
    labels:
      app: mongo-express
  spec:
    replicas: 1
    selector: # This is the selector that will be used to match the labels of the pods
      matchLabels:
        app: mongo-express
    template: # This is the pod template
      metadata: # This is the metadata of the pod
        labels:
          app: mongo-express
      spec:  # This is the specification of the pod
        containers: # This is the container that will be running in the pod
        - name: mongo-express # This is the name of the container
          image: mongo-express # This is the image that will be used to create the container
          ports: # This is the port that will be exposed by the container
          - containerPort: 8081
  
          # This is Configuration use to connect to the MongoDB database
          env: # This is the environment variables that will be used by the container
          - name: ME_CONFIG_MONGODB_ADMINUSERNAME # This is the name of MongoDB root username
            value: root
          - name: ME_CONFIG_MONGODB_SERVER # This 
            value: mongodb-0.mongodb-headless # This is a value of MongoDB server
            # This is a endpoint where Mongo Express will connect to the MongoDB Pod . 
            # mongdb-0 is the name of the MongoDB Pod and mongodb-headless is the name of the service that expose the MongoDB Pod
          - name: ME_CONFIG_MONGODB_ADMINPASSWORD # This is the name of MongoDB root password and It is stored in the secret yaml file
            # I use command kubectl get secret mongodb -o yaml to get name of the secret and key of the password
            valueFrom:
              secretKeyRef:
                name: mongodb
                key: mongo-root-password
---
  # This is a Internal Service . Only accessible within the cluster
  apiVersion: v1
  kind: Service
  metadata:
    name: mongo-express-service
  spec:
    selector: # This is the selector that will be used to match the labels of the pods
      app: mongo-express
    ports: # This is the port that will be exposed by the service
      - protocal: TCP
        port: 8081 # This is the port that will be exposed by the service
        targetPort: 8081 # This is the port that will be forwarded to the pods
  ```

  2. Once Yaml file Created I will create it in the Cluster : `kubectl apply -f [yamlfile]`
  3. I use to check that Mongo express run correctly : `kubectl logs [pod name]`


**Deploy Ingress Controller & Create Ingress rule**

**Deploy Ingress Controller**
  1. User Helm Chart for Ingress Controller (Nginx controller)
  2. Add NGINX repo : `helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx`
  3. Installing the Helm Chart : `helm install [RELEASE_NAME] ingress-nginx/ingress-nginx --set controller.publishService.enabled=true`
     - `--set controller.publishService.enabled=true` : This attribute to make sure that we are automatically allocated the Public IP for my Ingress Address to use with Nginx
  4. `kubectl get pods`: To check Ingress Controller running or not

**Create Ingress rule**
  - Ingress Controller uses some cloud native load balancer in the background
  - Linode'own Node Balancer or Worker Node Balancer, that was dynamically created and privisioned as I created the Ingress Controller
  - NodeBalancer is an Entry Point of K8 Cluster -> This NodeBalancer give me External IP, Ports -> and NodeBlancer will forward the request coming in into CLuster to the Ingress Controller and to the Internal Services base on the Ingress Rule I create

  ```
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    annotations: # this is the annotation that will be used by the Ingress Controller
      kubernetes.io/ingress.class: nginx
    name: mongo-express
  spec: 
    rules:
    - host: 23-239-6-26.ip.linodeusercontent.com # Hostname that will be used to access the application
      http: # Define the HTTP forwarding of request coming from host
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: mongo-express-service
                port:
                  number: 8081
  ```

  - Browser : hostname configure in Ingress Rule
  - Hostname get resolved to extetnal IP of NodeBalancer
  - Ingress Controller resolved the rule and forwared request to internal MongoExpress Service 

  ----Test UI : MongoExpress connected to MongoDB----
  - I added some data to the MongoDB
  - Then I delete the pod and restart them : `kubectl scale --replicas=0 statefulset/mongodb` - But the data still be there bcs I have configured PV - Then I scale back to 3
  - To see my charts : `helm ls`
  - If I done with the chart or reinstall or update I can use : `helm uninstall mongodb`

## Deploy Images in K8s from Prive Docker Repo 

**Common Workflow**

<img width="600" alt="Screenshot 2025-02-14 at 13 19 02" src="https://github.com/user-attachments/assets/c30e746d-0a44-4106-bbd9-dbcc91704ab9" />

- Commit my code to Git --> Trigger CI build (Jenkins) . Package My app with its Environment configuration into Docker Image --> Then Docker Images get push to Docker Registry (Docker Hub, Nexus, AWS container...) .
- Now When I have Docker Image in Repo . How to get Docker Image on K8's Cluster
- From Private Repo : I need explicit access

**Steps to pull Image from Private registry**
  1. Create Secret Component : contain access token or credentials to my docker registry
  2. Configure Deployment/Pod : To use that Secret using Attribute called imagePullSecrets

**Setup**
  1. Docker Image in ECR (Nodejs Application)
  2. Inside the Repo I have 3 Images with 3 different version tag
  3. Locally I have Minikube cluster setup (It's empty)

**Create Secret Component**
  - This Secret Component need to have credentials to Private Repository (ECR) which will allow Docker to pull that Image

  ----First thing I need to Login to Docker Repo----
  - Create config.json file for Secret
  - Login into Docker Private Repo : `aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin .....`
  - After Login Succeeded -> In the background it automatically created `config.json` (in .docker/config.json) file that hold the authentication to my Private Repo
  - There is 2 ways that this Config.json will store Authentication
    1. Is in .docker/config.json
    2. Or in externally "credsStore": "desktop" -> More secure bcs access token will store in Credentials Store
  - Now Whenever Docker tries to pull the Images from my Private Repo. It will use those credentials in config.json for this Private Registry to pull that Image to authenticate itself and pull that Image
  - However I am running cluster in Minikube and Minikube doesn't have access to CRED Store bcs it is running in Virtual Box . When the Docker which is packaged in the Minikube . So Minikube has its own Docker so when Docker inside Minikube tries to pull that Image from this private Repo, It will see the CRED Store and it won't be able to access that
    
    ----Diffent method to authenticate to my Private Repo via Minikube----
    
    <img width="400" alt="Screenshot 2025-02-14 at 14 30 42" src="https://github.com/user-attachments/assets/6ed34749-e58b-4db1-8fee-6540417c9e1d" />

    1. I execute this command `aws ecr get-login-password` to get Login password (A token use to login into AWS)
    2. Then I ssh to Minikube `minikube ssh`
    3. In /home/docker I execute `docker login --username AWS --p <AWS token> <URL to my AWS Repo>`
    4. Inside minikube I will use .docker/config.json to create Secrect Component

    ----Create Secret component----

    <img width="400" alt="Screenshot 2025-02-14 at 14 43 34" src="https://github.com/user-attachments/assets/2328e9de-2e7a-4379-b575-d1cb854891bd" />

    1. Copy config.json file inside minikube to my Local Machine : `minikube cp minikube:/home/docker/.docker/config.json /users/trinhnguyen/.docker/config.json`
    2. Turn config.sjon file into base 64 then put it into .dockerconfigjson : `cat .docker/config.json | base64`
    3. I also can configure secret using kubectl : `kubectl create secret generic my-generic-key --from-file-.dockerconfigjson=.docker/config.json --type=kubernetes.io/dockerconfigjson`
    4. Use `kubectl get secret` to check secret is running
       
   <img width="400" alt="Screenshot 2025-02-14 at 14 52 05" src="https://github.com/user-attachments/assets/80486ecc-509d-4ac3-b29f-1d547f8b3d08" />

    - Another the way to create Secret with 1 step `kubectl create secret docker-registry my-registry-key-two --docker-server=<aws-server-endpoint> --docker-username=AWS --docker-password=<AWS access Token>`
      
    !!! NOTE: Directly having authentication inside the config.json file is not secure as having CRED Store 

**Create Deployment Component**

1. Create Deployment Configuration
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: js-app
    labels:
      app: js-app
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: js-app
    template:
      metadata:
        labels:
          app: js-app
      spec:
        imagePullSecrets: # This is the secret that I created in the previous step
        - name: my-registry-key
        containers:
        - name: js-app
          image: 565393037799.dkr.ecr.us-west-1.amazonaws.com/js-app:1.0 # This is the image from the private repository
          imagePullPolicy: Always
          ports:
          - containerPort: 3000
  ```

2. Pull Docker Image from Private Repo : `imagePullPolicy: always`
3. Configure Deployment with Secret : In the Pod's Specification same level as container
  ```
    imagePullSecrets :
    - name: my-registry-key

    # This 2 lines will configure deployment with access to the secret that contains docker registry access
  ```

  !!! NOTE: Secret has to be in the same namespace as Deployment 

## K8s Operator 

**What is K8s Operator | Why Operators used ? | When to use Operators ?**

**Overview**
```
  - Operators use mainly for Stateful Application 
```

**Stateless Apps in K8**

<img width="600" alt="Screenshot 2025-02-15 at 10 56 15" src="https://github.com/user-attachments/assets/4456efeb-a6d5-4940-ad64-e52472c8f75d" />

```
  - Let's say I deploy Web Apps in K8s
  - So I will create Deployment, ConfigMap, Service and Application start
  - I will scale My App to 3 Replicas . If 1 died K8s built in Control Loop Mechanism will regenerate it for me | If I release new version I adjust a deployment configuration and replicas get restart with new version 
```

**Stateful Application without Operator**

```
  - Stateful App like Database .... I need data persistence
  - K8s can not do automation with Stateful Apps
  - Bcs When I create Replicas for Stateful Apps . All 3 Replicas is different and have different Identity and State | They need to Update and Destroy in certain order
  - There must be constant communication between these replica and synchronization so that data stay consistent

  That's why Statefull Apps required manually intervention

  Stateful Apps need in K8s : Promethues Monitoring, etcd store

  So I use Operator to Manage Stateful Apps ( https://operatorhub.io/ )
```

**Stateful Application with Operator**

<img width="600" alt="Screenshot 2025-02-15 at 11 21 50" src="https://github.com/user-attachments/assets/d6b2072f-6b2e-4aa3-9afb-7b80ed41a1b7" />

```
  - Operator basically replaces this human operator with a software operator

  - All the manually tasks that Devops team person would do to operate stateful Application now packed into program that has knowlege and intelligent about how to deploy that specific Application

    - Like How to create cluster of replicas | or How to recover if 1 failed

    - Task automate and reuseable

    - If I have 2 K8s Cluster with the same set up I don't have to manually configure and then maintain these Apps on both Environments . But rather you have 1 standard automated tool
```

**How does Operator work?**

```
  - As it core . It has the same Control Loop mechanism

  - What is Control Loop does
    - Watch for change Applcation State 
    - If Relica died then it created new one
    - If configuration change then it will updated
    - If Image version get updated it will restart with new version

  - Make use of CRD (Custom Resources Definition) :
    - It is a custom K8s Component (extends K8s API)
    - By default I have Deployment, Stateful set, Config map, etc ...
    - So Operator take the basics resources and its controller concept as a foundation to build upon
    - And on top of that it included the domain or application specific knowledge to automate the entire lifecycle of the application it manage or operate 
```

## Secure Cluster - Authentication with RBAC 

**Overview**
```
  How Authenication and Authorization work in K8 ?
  
  How to configure users, groups, their permission ?
  
  Authentication with Role Based Access Control (RBAC) ?
  
  Which K8s resources to use to define permissions in the Cluster ?
````

**Why Do I need to manage permission in K8s Cluster in the first place?**

<img width="600" alt="Screenshot 2025-02-15 at 13 54 47" src="https://github.com/user-attachments/assets/deb25806-b8dd-46c0-91db-e0bb985e664d" />

```
  - I have K8s Cluster that Admin and User use to Administer and Deploy into

  ----How to manage permission for a cluster ?----
    - Admin and Developer have different access
    - Developer should be limited what they can do in the Cluster so they don't accidently break stuff . Admin should administer the Cluster
    - In Security best practice we have the Least Privilege rule which mean we give user only the Permission they need 
```

**Developers : Role and Role Binding**

<img width="600" alt="Screenshot 2025-02-15 at 14 04 10" src="https://github.com/user-attachments/assets/c9673960-e79e-4438-b0d5-1d0c65757fbb" />

```
  - Let's say I have multiple Namespaces in the Cluster and each Developer team deploy Apps in the different Namepsaces

  ---How to restrict access to only that Namespaces ?---
    - For that K8s has RBAC = Role Based Access Control
      -- With RBAC I can define permission to each Namespace by using Role Component 
      -- Role bound to specific Namespace
      -- Define what resources in that Namespace I could access like : Pod, Deployment, Secret etc...
      -- What action I can do with these Resources like : list, delete, update, get etc...
      -- Role only Define resources and access Permission | No infomation WHO get access

  ---How to attach Role defined into Developer Team ?---
    - For that K8s has RoleBinding Component
    - Link ("Bind") a Role to a User or Group
    - All members of Group get permissions defined in the Role 
```

**Administrator : Cluster Role and Cluster Role Binding**

<img width="600" alt="Screenshot 2025-02-16 at 09 31 40" src="https://github.com/user-attachments/assets/3cfd6f0b-d594-4e08-af23-acbb655589aa" />

```
  - Managing all the Namespaces

  - Configuring Volume for the Developer team are available Cluster-wide Volumes

  - They are doing Cluster Wide operation

  ---How to define Admin Permission in K8s ?---
  - For that K8s has ClusterRole
  - Define Resources and Permission Cluster Wide
  - For Admin I would define a ClusterRole and Admin Group and then attach the cluster role to the Admin group using a ClusterRoleBinding 
```

**User and Groups in Kubernetes**

<img width="400" alt="Screenshot 2025-02-16 at 09 47 17" src="https://github.com/user-attachments/assets/e68e3331-3e48-4a90-baf0-680efb7e656e" />
<img width="400" alt="Screenshot 2025-02-16 at 09 50 02" src="https://github.com/user-attachments/assets/69e4afaa-9ab3-4d87-bdaf-5b0b30314fd3" />

```
  - Kubernetes doesn't manage Users natively

  - Admins can choose from different authentication strategies

  - No Kubernetes Objects exist for representing normal user account

  - It relies on External Sources for Authentication

  ---External Source---
  - Could be a Static Token file : username, uid , token ....
  - Could also be certificate design by Kubernetes itself
  - Or third party identity service, like LDEP

  - So Admin configurs external source so Kubernetes has access to the information and then API server which is one of the control plane components of Kubernetes and which is the one handling authentication of all the requests coming in K8s Cluster.

  - K8s Cluster will try to authenticate any user trying to connect to a Cluster using configure resoruces provided

  - I can pass these file to API Server so that API Server knows where to look up to Authenticate Users, when they try to connect to the Cluster

  - If I want to define Group that User belong to , I can simply add the Group Column in this users file, in the Static file

  - For Certificate, As a Admin will have to manually creare Certificate for different User

  - OR Configure LDAP as a authentication source for API server

  - K8s allow me to configure those external sources but I have to manage them myself 
```

**Service Account**

<img width="600" alt="Screenshot 2025-02-16 at 10 21 53" src="https://github.com/user-attachments/assets/f28a1217-62c8-4fa5-b350-1b970e590437" />

```
  - Service Account is Applications User

  - These Applications can be internal inside the cluster or outside the cluster

  - For example : Inside the cluster I might have a Mornitoring Application, like Promesthus that need access to all other Apps to collect metrics from them And We also have microservice applications that only need resources and access to resources within their specific namespace

  - These Applications are running inside the cluster and are talking to other Applications or using resources in the cluster .

  - And We also have External Applications like Jenkins that deploy Application to a Cluster | OR Terreform that configure the Cluster itself

  - With the Least Privilege Rule We also want Applications have the only need permission , not more .

  ---How to define Applications User---
  - K8s component that represent an Application User call ServiceAccount 

  - Instead of User or Group for human user , I create a Service Component for Application User : kubectl create serviceaccount sa1

  - So my Applications like Metric Server (Prometheus) or external CI/CD (Jenkins) will get Service Account Component inside the cluster as a representation of that application user

  - As the same way for human user I can link service account to Role or ClusterRole with RoleBinding or ClusterRoleBinding . And with binding service account or the application that is behind that service account will get permission that are defined in the Role or Cluster Role 
```

**Example Configuration File**

<img width="400" alt="Screenshot 2025-02-16 at 10 33 13" src="https://github.com/user-attachments/assets/d867866c-183b-4c87-80f1-412c9e6bcda7" />
<img width="400" alt="Screenshot 2025-02-16 at 10 35 23" src="https://github.com/user-attachments/assets/30b59b56-5a97-4d82-abf3-7d9832a33ec9" />
<img width="400" alt="Screenshot 2025-02-16 at 11 29 28" src="https://github.com/user-attachments/assets/961edbf0-223c-4b73-a36f-6056efbc512f" />

```
  - Define what resources have what kind of access ?

  - In each Role we have 3 Section:
    - apiGroup : indicate the core API group
    - The Default one is the core API group
    - When using the core API group I can leave it empty 
    - For other, I have to specify

  - Within the API group we have Resources :
    - K8 component like Pods, Deployment, etc....

  - Then third part access Verbs :
    - Action on the resources

  - If I want to give access to the defined resources for Namespace like 'My-app', I will add namespace: my-app in metadata

  - To set more grangular access
    - Instead giving access to all the Pod in that Namespace, I can define to only certain Pod in that Namespace using resources name Attribute
    - For example If in the same NameSpace I have my own application and maybe a database and I want to basiccally define seperate role in the same NameSpace, but for my own Application and for Database, basically seperately at different role I can target these specific Pods and Applications using the resources name attribute and then define different privellege on resources 
```

**And then I have RoleBinding**

<img width="600" alt="Screenshot 2025-02-16 at 11 31 49" src="https://github.com/user-attachments/assets/6f411d8f-d4d5-458a-bd7b-6b17a7c3a59f" />

```
  - RoleBinding to attach it to a User, Group or Service Account 
```

**ClusterRole**

<img width="400" alt="Screenshot 2025-02-16 at 11 35 04" src="https://github.com/user-attachments/assets/8beb7050-d0ad-4ebe-9514-c5878b6117f6" />
<img width="400" alt="Screenshot 2025-02-16 at 11 37 21" src="https://github.com/user-attachments/assets/a356d0f6-d1b2-465d-820e-9fd87ad6dc8c" />
<img width="400" alt="Screenshot 2025-02-16 at 11 42 07" src="https://github.com/user-attachments/assets/ad25dda3-fbdf-4785-895a-039a95e781de" />

```
  - ClusterRole for Nodes : What priviledges they have on that Worker Node

  - Or I can define Cluster role for namespace management Since NameSpace is Also a cluster Wide resources

    - But In a Addition to defining permission for cluster wide resources for cluster role
    -I can also defined cluster role for namespace like Pods, Services etc ... Just like in the role

  - And if I define a cluster role with Pod access, it will mean access to Pod in all Namespaces, not just one specific one as in the role

  - And in Cluster Role binding I will then link that Cluster Role again to a user group, or service account 
```

**ClusterRole Binding**

<img width="500" alt="Screenshot 2025-02-16 at 11 47 05" src="https://github.com/user-attachments/assets/67079c84-a067-4e3d-9f3c-cde70714c1d3" />

**Create And Viewing RBAC Resources**

<img width="500" alt="Screenshot 2025-02-16 at 11 50 28" src="https://github.com/user-attachments/assets/f71cc370-c069-4c5a-924e-fe149cdba4d7" />

**Check API access**

<img width="500" alt="Screenshot 2025-02-16 at 11 52 20" src="https://github.com/user-attachments/assets/71c78985-9604-4c78-8c1f-68dcecd84b5f" />

```
  - I can check access , priviledges, and permission of current user

  - To see what Permission that user has in any Namespace
```

**Wrap up**

<img width="600" alt="Screenshot 2025-02-17 at 12 40 06" src="https://github.com/user-attachments/assets/b7f2e7c9-f279-4225-a4c2-3f73cc801c86" />

```
  ----Layer of Security----

  - For example, Jenkins send a request to API server , Request to Create new Service in default namespace

    -- First Level API will authenticate the Application User  or Human User in the Cluster to see Application user is allow to connect to the cluster

    -- I can enable multiple authentication at once

    -- If authenticated as a second step K8s will check Authorization which is RBAC . Check Role or ClusterRole that user has and based on the permission defined there does User has permission to perform specific task that it is trying to perform 
```

## MicroServices 

**Overiew**
```
  - Main Benefit of Microservices Application

  - What I need to know to depkoy MicroServices Application in K8 ?
```

**Introduction**

<img width="500" alt="Screenshot 2025-02-17 at 12 53 41" src="https://github.com/user-attachments/assets/7e4adf7d-4a31-47e8-8a85-855e640fa74b" />

```
  - Microservices emerged as a Platform for Microservices Applications

  - From Monolith (1 Big Applications) to Microservice (Many Small App)

  - Can be developed, deployed, and managed independently

  - For example Big Company like Linked I might have 1 Microservices for User Account, 1 Microservices for Message, ect...

    -- Each business functionality escapsulated into own Microservices
    -- Smaller independent applications that developer manage
    -- Each Microservices can be develope, packaged and released independently
    -- 1 Changes of Microservices without effect to other 
```

**How do Microservice communicate?**

<img width="400" alt="Screenshot 2025-02-17 at 13 24 45" src="https://github.com/user-attachments/assets/fd32bb92-47c3-404e-be8a-988c88cefcc4" />
<img width="400" alt="Screenshot 2025-02-17 at 13 07 30" src="https://github.com/user-attachments/assets/0c25986d-41fb-4454-9712-b5acf134c82a" />

```
  ----Through API----

    -- Microservice communicate through Interfaces or API is basically a code with a bunch of function inside the Application itself . Which is responsible for handling communication with other service which is responsible for handling commnunication with other service.
  
    -- The logic of communication inside of Micro itself

  ----Through Message Broker Apps----

    -- Instead of each Service sending request to all the others, all Micro talk to Message Broker

    -- Less code complexity inside Microservices

  ----Service Mesh----

    -- Instead of 1 big Central Broker handle all communication . Each Microservices have its own helper program that handle the communication for that specific microservice 
```

**What do I need to know as Devops?**

<img width="400" alt="Screenshot 2025-02-17 at 13 26 04" src="https://github.com/user-attachments/assets/fb109f4d-429a-4509-9e46-25739e958102" />
<img width="400" alt="Screenshot 2025-02-17 at 13 22 39" src="https://github.com/user-attachments/assets/ec49b947-6c9e-405e-94b6-d358a8bef6dc" />

```
  - Task : To Deploy existing Microservice Application in the K8s Cluster . So developer has already created and developed these microservices and asking Devops to take these Microservices and deploy it into Cluster

  - At this point what infomation do I need to know from Developer about these Micro to be able to deploy them in the cluster

  1. What are Mircos I need to deploy ?

  2. Which Micro talking to which Micro ?

  3. How are they communicate ? API | Message Broker | Service Mesh ?

  4. Which Database they are using ? 3rd party Services

  5. On which Port does it Microservice run ?

  - Using these Infomation now I can go and Prepare the K8 Environment

    -- Deploy any 3rd Party apps

    -- Or creating Secret and Configmap for Microservices

  - Then Once Environment is Prepare

    -- Create Deployment and Service for each Microservice

    -- And I also can define Micro run on the same Namepsace or each Micro has its own Namespace 
```

## Deploy Microservices Apps : Demo Project 

**Overview**

<img width="600" alt="Screenshot 2025-02-17 at 14 19 15" src="https://github.com/user-attachments/assets/4bdebd17-9ce1-4328-8b6d-fc01d5548943" />
<img width="600" alt="Screenshot 2025-02-17 at 14 36 17" src="https://github.com/user-attachments/assets/672c5bd8-d481-4bde-aa8b-65fe41943d8d" />

```
  - Need couple of key Infomation

    -- What Micro Services I am deploy ?

    -- How Micro Services Connect to each other ?

    -- Micro Service need any 3rd party Services or Database ?

    -- Which Service is accessible from outside the cluster ? Which one get entrypoint request from a Browser ?

    -- Using those Information about how Microservices talk to each other, what other Services they are depending on and which is the entry point microservice and I will create the connection grapse and visualize all of this

  ----How Application connect to each other?----
    -- Images Name for each Microservices 
    -- What ENV each Mircroservices expected ?
    -- Which Port Service start ? Bcs we have to define that in the configuration file for Deployment but also for Service to be able to connect infomation inside the Pod 
    -- All Microservices will deploy into the same Namespace

  ----If The Developer also tell this is a Load Generator----
    -- Optional Development
    -- It only serve to test the Load . It just send a bunch of request to a Front end


  - Now Create Deployment Config and Service Config for each Micro Services

 !!!  Redis is a Message Broker and also the In-Memory Database
```

**Create Development and Service Config**

<img width="500" alt="Screenshot 2025-02-18 at 09 43 19" src="https://github.com/user-attachments/assets/9a0655e4-f0e9-459c-89af-a9d70a246c2a" />

Step 1 : Minimum Require Configuration File for Development and Service . Since I have 11 Microservices I will have 11 Development and Service in that File 

```
apiVersion: v1
kind: Deployment
metadata:
  name: xxxx
spec
  selector:
    matchLabels:
      app: xxxx
  template:
    metadata:
      labels:
        app: xxxx
    spec: 
      container:
      - name: xxxx
        image: xxxx
        ports:
        - containerPort: xxxx
---
apiVersion: v1
kind: Service
metadata:
  name: xxxx
spec: 
  type: ClusterIP
  selector:
    app: xxxx
  ports:
  - protocol: TCP
    port: xxxx
    targetPort: xxxx
```

Step 2 : Configure Development and Service 
  
  1. Email Service :
  ```
  - Container Port: 8080
  - name : emailservice
  - Image : xxx
  - Service Port : 5000 (Can be the same Target Port or can be different)
  - Environment Var : env :
     - name: PORT 
     - value : "8080"
  ```

  2. Recommendation Service :
  ```
  - Recommendation Service talk to Catalog Service : I need to tell the Recommendtation Service the endpoint or the service name of the Product Catalog service, where it can connect to it
  - The way it work is Recommendation microservice take the enpoint or the service name of the Product Catalog microservice so Recommendation microservice can connect to Product Catalog microservice (I defined the Product Catalog service enpoint in the Cofiguration yaml file)
  - My Recommendation Service communicate with my Product Catalog Service through the K8 DNS-based service discovery mechanism

  - Container Port: 8080
  - name : recommendationservice
  - Image : xxx
  - Service Port : 5000 (Can be the same Target Port or can be different)
  - Environment Var :
    - env :
      - name: PORT 
     value : "8080"
      - name: PRODUCT_CATALOG_SERVICE_ADDR
      value : "productcatalogservice:3550"
  ```

  3. Product Catalog Service :
  ```
  - Container Port: 3550
  - name : productcatalogservice
  - Image : xxx
  - Service Port : 3550 (Can be the same Target Port or can be different)
  - Environment Var :
    - env :
      - name: PORT 
     value : "3550"
  ```

4. Payment Service :
  ```
  - Container Port: 50051
  - name : paymentservice
  - Image : xxx
  - Service Port : 50051 (Can be the same Target Port or can be different)
  - Environment Var :
    - env :
      - name: PORT 
     value : "3550"
  ```

5. Cart Service
  ```
  - Container Port: 7070
  - name : paymentservice
  - Image : xxx
  - Service Port : 7070 (Can be the same Target Port or can be different)
  - Environment Var :
    - env :
      - name: PORT 
      value : "3550"
      - name: REDIS_ADDR
      value : "redis-cart:6739"
  ```

6. Redis
  ```
  - Container Port: 6379
  - name : redis-cart
  - Image : redis:alpine
  - Service Port : 7070 (Can be the same Target Port or can be different)

  - I need to provide Volume for Redis -> Bcs Redis persist data
    - volumes:
      - name : redis-data
        emptyDir: {}

    - volumeMounts:
      - name : redis-data
        mountPath: /data -> This is where redis will store Temporary data 

  - Volume type:

    -- Persistence Type : Exist beyond lifetime of Pod

    -- Ephemeral Volume Type : K8s destroy when pod ceases to exist -> emptyDir

      --- Empty Dir start as empty Dir  .
      --- Empty Dir gets created when a pod that reference that empty directory volume start on a Node
      --- Exist as long as that the Pod is running on that specific Node
      --- If the Pod died and restart EmptyDir will be deleted permantly and start new EmptyDir with another Pod just restart
      --- If the container inside the Pod crashed but not itself . It doesn't acctually remove the emptyDir content or data
  ```

7. Checkout Service

<img width="600" alt="Screenshot 2025-02-18 at 10 24 05" src="https://github.com/user-attachments/assets/e80631bf-0d55-468f-9641-d5bede431b13" />
<img width="561" alt="Screenshot 2025-02-18 at 10 25 17" src="https://github.com/user-attachments/assets/487d0b8b-53d6-47aa-b3b3-ac73b77e709b" />


!!! NOTE : 2 Of services are trying to initiate a profiler , when they start up. And by setting these ENV we tell the Application to ignore the Profiler Service, as we don't have them in the cluster
      --- In PaymentService: Add new ENV -> DISABLE_PROFILER = 1
      --- In Currency Service: Add new ENV -> DISABLE_PROFILER = 1

8. The Frontend

<img width="499" alt="Screenshot 2025-02-18 at 10 50 18" src="https://github.com/user-attachments/assets/a911c4fd-dde5-4d2f-9dc6-498f967c421f" />

  ```
    - Front end Micro talk to all these other microservices to distribute the request that its get to whoever responsible for the request. So I need to set the endpoint of all those Microservices

    - Front end Micro has to be available or accessible externally from a Browser 
  ```

**Deploy Micros to a Cluster**

  Step 1: Create Linode cluster 
  
  Step 2: Add 3 small Node running on the shared CPU 
  
  Step 3 : Download Kubeconfig file So we can connect to a Cluster -> Before I use kubeconfig I have to set perrmission to more stricter permission bcs this file contain credentials to my Kubernetes cluster . Also I need to securely store this file : `chmod 400 kubeconfigcredentials.yaml`
  
  Step 4 : `export KUBECONFIG=<absolute path to my project>`

  Step 5 : Instead of set up everything on the Default Namespace . I will create Namespace : `kubectl create ns microservices`

  Step 6 : Deploy my micro service into my NameSpace : `kubectl -f config.yaml -n microservices`

  Step 7 : Get Pods : `kubectl get pods -n microservices`

**Security and Production Best Practice**

  Best Practice 1: Pinned (Tag) Version for each Container Image 
  
  <img width="500" alt="Screenshot 2025-02-20 at 09 26 49" src="https://github.com/user-attachments/assets/8fd0bb9c-5c49-4845-b293-63f2a3b2832e" />
    
  ```
    - Define specific version for Image that I use inside the Pod

    - If I don't specify the version tag . K8s automatically pulled the latest version . That mean the new Image get built and push to the Repo the Pod will restart and pull the latest version . It might cause break current App, unpridictable, Don't know what version running on my Cluster 
  ```

  Best Practice 2: Liveness Probe for each container 

  <img width="500" alt="Screenshot 2025-02-20 at 09 34 09" src="https://github.com/user-attachments/assets/d8795879-ec0b-4689-aa03-7c3b200377c0" />

  ```
    - K8s has intelligent managing its resources

    - Restart Pod When it Crash

    - But what if the Application inside the Pod (container) ? Pod is running K8 think everything fine but in reality the container crashed or stuck in the loop then the Apps is not accessible . How to let K8s know wheather the Applications inside the Pod is running ?

    ----Perform Health check with Liveness Probe----
    - With Liveness Probe K8s will restart the Pod if the Application crash, or has issue

    ----How to define a liveness Probe in the Container ?----
    - The program that ping the Application endpoint every 5 or 10 second to check Application Response
    - In this Application the developer has added small program can be check wheather Application healthy or accessible . So I can use that in the Liveness Probe

    - Inisde the container attribute same level as name, image etc ...:
      -- I defined the attribute depending on what protocol the Application use it. Could be : TCP, HTTP, gRPC ...
      -- Once I defined Liveness Probe protocal I should tell what port should send that grpc request to
      -- Next I need to define how often the Application should be check for its health : periodSeconds: 5
  ```

  Best Practice 3 : Readness Probe for each container 

  <img width="400" alt="Screenshot 2025-02-20 at 09 34 09" src="https://github.com/user-attachments/assets/e2bf168e-fea1-4385-8783-0da4a8ce7219" />
  <img width="400" alt="Screenshot 2025-02-20 at 10 26 00" src="https://github.com/user-attachments/assets/57711691-4b6b-44eb-89ec-cbcf76290e28" />
  <img width="400" alt="Screenshot 2025-02-20 at 10 41 34" src="https://github.com/user-attachments/assets/cbc5f93e-f77e-44e2-94e6-f2ca0afe0fc6" />
  <img width="300" alt="Screenshot 2025-02-20 at 10 48 15" src="https://github.com/user-attachments/assets/7621f4a3-4810-425c-9b01-ca5a06ba2a5e" />

  ```
    - K8s know the Pod state but it doesn't know the Application State inside the Pod . And I solved this problem with LivenessProbe

    - But LivenessProbe help K8s see that Application is running successfully only After the Application started

    - What about the starting up itself Application ?

    ----How does K8s know the Apps is fully intiated and started and ready to receive the request ?----
    - Peform Health Check with Readiness Probe
    - Let's K8 know if Application is ready to recevice traffic
    - Without Readiness Probe, K8s assume the app is ready to receive traffic as soon as container start

    ----How to configure Rediness Probe----
    - Similar to LivenessProbe

  !!! Note: These mircoservices Application are using grpc protocol and that's why I have use grpc attribute but other Application may use other protocol -> There are 2 alternative to this :
      1. TCPsocket: With this protocol The kubelet will attempt to open a socket to my container on that container Port . If it succeeds to establish a connection on this Port the container is considered to be heathy
      2. HTTPs: If application has endpoint inside Application that expose bacsically the health status of Application itself, we could hit that endpoint to check whether application is healthy or not . This configuration will tell Kubelet there is an HTTP endpoint on the Application on this Port and this URL you can check wheather the URL is healthy or not

  !!! Note: Generally for readiness and liveness probes, no matter if it's a command execution or TCP socker, we can actually execute initial delay seconds in case we know that Application take longer to start 
  ```












