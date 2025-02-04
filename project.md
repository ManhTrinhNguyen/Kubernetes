# Deploy MongoDB and Mongo Express

## Demo 
```
  1. Create MongoDB pod 
    - In order to talk to Pod I need Service 
    - I will create Internal Service . Only the components inside cluster can request to it 
  2. Create Mongo Express 
    - Need URL of MongoDB so Mongo Express can connect to it 
    - Credentials : Need username and password to authenticate 
    - The way I can pass these Infomation is via Deployment configuration file through ENV (That is how application configure)
    - I will create a Config Map that containe DB URL and I will create Secret that container Credentials and I will reference both inside the Deployment 
    - Once I have that set up I will need Mongo Express accessiable through browser in order to do that will create external Service that will allow external request to talk to the Pod
```

**1. Create MongoDB Deployment Yaml**
```
```

**2. Create Secret**
```
  - From there I will reference value
  - Secrect live in K8s, not it the Repo

  - kubectl apply -f mongo-secret.yaml : Apply secret 
  - kubectl get secret : To check secret
```

**3. Create Internal Service**
```
  - Internal Service help other component or other pods can talk to MongoDB

  ----Service Configuration----
  - kind : Service 
  - metadata/name: Random Name
  - selector: to connect to Pod through Label 
  - ports : 
    - port : Service Port
    - targetPort: Container of Deployment 
```

**4. Create Mongo Express Deployment (External Configuration where will put DB in MongoDB)**
```
  - Use configmap inside the deployment is : configMapKeyRef
```

**5. Create Configmap**
```
  - External Configuration
  - Centralized
  - Other component can use it
  - So if I have 2 Application they are using MongoDB then I can just reference that external Configuration. And if i have to change it I will change it in 1 place . Nothing get updated

  kind: Configmap
  metadata/name: a random name
  data: The actual content in key-value pair
```

**Create external Service to access mongo express from browser**
```
  To expose the service externally 
  - type: LoadBalancer : Assign service an external IP address and so accepts external request
  - nodePort: Port for external IP address must be between 30000-32767
  - ClusterIP: same as internal service type, is default
    - ClusterIP will give the Service an internal IP
    - And LoadBalancer will also give Service an internal IP and also give Service external IP

  In Minikube :
    minikube service <service name> : Assign external service a public IP
    - When command executed Minikube create a tunnel and use localhost http://127.0.0.1 instead of real public IP
```