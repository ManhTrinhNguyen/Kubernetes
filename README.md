# Kubernetes 

## Big Picture 

**What is Kubernetes ?**
  - Open source container tool developed by Google
  - Kubernetes help manage Applications that are that made up of hundreds or maybe thousands of Containers in the Differents Environment like ( Physical Machine, Virtual Machine, or Cloud Enviroment)

**What is probles Kubernetes solve ?**
  - Trends from Monolith to Microservices
  - The Rise of Microservices Technology increased usage of Container Technology bcs the Container offer perfect host for small independent applications, like microservices
  - And the Rise of Container resulted in Application they are now comprised 100 or 1000 of container Managing those container accross multiple Environment using script and self make tools can be really complex

**What features do Orchestration tools offer ?**
  - High Availability | No down time
  - Scalability | high performance (I can scale my App fast when I more load on it and more user trying to access it, scale down when user go down)
  - Recovery Disaster

## Main component

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
    - Service is Static IP address or permannet IP can be attach to each Pod
    - Lifecyle of Pod and Service not connected
    - External Service :
      - Make a App to accessable from Browser
      - External service is service that open the commnunication from external sources
    - Internal Service:
      - For only insde connection like (DB or ...)

  Ingress:
    - Instead of services the request first go to Ingress and then service 
```

**ConigMap and Secrect**
```
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



