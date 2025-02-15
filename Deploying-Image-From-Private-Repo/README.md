## Demo app - Deploying Images in Kubernetes from private Docker repository

This demo app shows a simple user profile app set up using 
- index.html with pure js and css styles
- nodejs backend with express module
- mongodb for data storage

All components are docker-based

### With Docker

#### To start the application

Step 1: Create docker network

    docker network create mongo-network 

Step 2: start mongodb 

    docker run -d -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=password --name mongodb --net mongo-network mongo    

Step 3: start mongo-express
    
    docker run -d -p 8081:8081 -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin -e ME_CONFIG_MONGODB_ADMINPASSWORD=password --net mongo-network --name mongo-express -e ME_CONFIG_MONGODB_SERVER=mongodb mongo-express   

_NOTE: creating docker-network in optional. You can start both containers in a default network. In this case, just emit `--net` flag in `docker run` command_

Step 4: open mongo-express from browser

    http://localhost:8081

Step 5: create `user-account` _db_ and `users` _collection_ in mongo-express

Step 6: Start your nodejs application locally - go to `app` directory of project 

    cd app
    npm install 
    node server.js
    
Step 7: Access you nodejs application UI from browser

    http://localhost:3000

### With Docker Compose

#### To start the application

Step 1: start mongodb and mongo-express

    docker-compose -f docker-compose.yaml up
    
_You can access the mongo-express under localhost:8080 from your browser_
    
Step 2: in mongo-express UI - create a new database "user-account"

Step 3: in mongo-express UI - create a new collection "users" in the database "user-account"       
    
Step 4: start node server 

    cd app
    npm install
    node server.js
    
Step 5: access the nodejs application from browser 

    http://localhost:3000

## Build and Push a docker image to ECR from the application

- Before login I need to configure as a admin user that has permission to execute this command `aws configure`

Step 1: Login to ECR Private repo `aws ecr get-login-password --region us-west-1 | docker login --username AWS --password-stdin 565393037799.dkr.ecr.us-west-1.amazonaws.com`    

    -- When login command executed . In the background it will automatically generate the .docker/config.js . This file hold aws token and CRED store to authenticate to private repo

Step 2 : Build Docker Image with Repo tag : `docker build -t 565393037799.dkr.ecr.us-west-1.amazonaws.com/js-app:1.0 .`

     -- The dot "." at the end of the command denotes location of the Dockerfile.   

Step 3 : Push Docker Image : `docker push 565393037799.dkr.ecr.us-west-1.amazonaws.com/js-app:1.0`

## Minikube 

Step 1: Start Minikube : `minikube start`
    
## Deploy Images in K8s from Private Docker Repo 

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
```

2. Pull Docker Image from Private Repo : `imagePullPolicy: always`
3. Configure Deployment with Secret : In the Pod's Specification same level as container
  ```
    imagePullSecrets :
    - name: my-registry-key

    # This 2 lines will configure deployment with access to the secret that contains docker registry access
  ```

  !!! NOTE: Secret has to be in the same namespace as Deployment 














