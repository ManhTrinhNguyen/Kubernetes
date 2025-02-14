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
