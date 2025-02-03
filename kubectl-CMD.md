## kubectl Command

```
  - Get status of Node : kubectl get nodes

  - Check Pod : kubectl get pod
    -> NAME of pod: name-<podId>-<replicasetId>

  - Check Services : kubectl get services

  - Create Pod : kubectl create deployment NAME --image=image 
    -> The way it work is Pod is smallest unit but usally in practice I am not creating pod or not working with pod directly there is Abstraction layer over pod is Deployment
    -> NAME : NAme of the pod
    -> --image=image : The pod need to create base on Image

  - Get deployment : kubectl get deployment
    -> The way it work is that When I create deployment . The Deployment has all the information or the Blueprint for creating Pods
    -> Between the pod and deployment there is another layer call automatically manage by Kubernetes called Replicaset

  - kubectl get replicaset :
    -> Replica set is managing Replica of the Pod
    -> In Practice I do not have to create or delete replica set . I will working with Deployment directly

  Layer Abstraction
    - Deployment manage a -> Replicaset manage a -> Pod is abtractions of Container
    - Everythin Below Deployment should be manage by Kubernetes

  Make change from Deployment
    - edit in the Deployment : kubectl edit deployment [name]
      -> I will get auto generated file configuration


    -----Debugging Pods------

  - kubectl logs [pods name]
  - kubectl describe pod [pods name]

  - kubectl exec -it [pod name] -- bin/bash
    -> With this command I can get into terminal of Mongodb Application

    -----Delete and Apply Configuration File------

  -  kubectl delete deployment [deployment name]

  When I am creating Kubernetes component likes deployment, using kubectl create deployment I have to provide all the option on the CLI
  -> Better practice Kubernetes Configuration files

  - kubectl apply -f [filename] : take the configuration files as a parameter and does whatever in the file
```


