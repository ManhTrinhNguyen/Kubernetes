## 3 Parts of a K8s Configuration File 

**The First 2 lines declare what I want to deploy**
```
  apiVersion: App name 
  kind: Deployment Or Service 
```

**First part is Metatdata**

**Second part is Specification**
```
  - Specification is where I basically put every kind of configuration I want to apply for that component
  - The Attribute will be specific to the Kind of the component that I am creating
  - So Deployment will have its own attributes that only apply for deployment and Service will have its own attribute that only apply for Server
```

**Third part is Status**
```
  - Automatically generated and edited by Kubernetes
  - Kubernetes alway compare what is a Desire State and what is Actual State !! If Actual State and Desire State do not match then Kubernetes know something to be fixed there it will try to fix it . This is a basis of self-healing feature that Kubernetes provided

  - K8's get data from etcd .
  - etcd as part of cluster's brain one of the control plane processes actual store the data 
```

**Configure format is YAML** 
  - If I have a long yaml I can go to validate yaml to validate it
  - Store Config file with my code

**Blueprint for Pods (Template)**
```
  - Under Specification part config.yaml file I have template
  - Template also have its own metadata and specification
  - Template configuration apply to a Pod 
```

**Connecting Component ( Labels & Selectors & Ports )**
```
  - The way the connection is established is using Labels and Selectors
  - Metadatas part contain the Label and the Specification part contain Selector
    
    ----Connecting Deployment to Pod----
  - In metadata, I give components like Deployment or Pods, a key value pair and it could be any key value pair that I can think of like 'app:nginx'
  - Pod get lables through template blueprint
  - And we tell the deployment to connect or to match all the labels with 'app:nginx' to create connection

    ----Connecting Services to Deployment----
  - Now Deployment has its own label 'app:nginx' and these two label are used by Serviced Selector
  - In Specification of Service , We define a selector which basically make a connection between a Service and Deployments or its Pod

    ----Configure Port for Services and Pod----
  - Service :
    - protocal : TCP
    - port : 80
    - targetPort: 8080
  - Pod :
    - port : 8080

  !!! Service has a Port where the Service itself accessible at ( port : 80 ) . And the Service need to know to which Pod it should forward a request and also which port is that Pod is listening ( targetPort: 8080 ) is where Pod is listening ( port : 8080 )
```

















