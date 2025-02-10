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







