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

#### To build and push a docker image to ECRfrom the application

- Before login I need to configure as a admin user that has permission to execute this command `aws configure`

Step 1: Login to ECR Private repo `aws ecr get-login-password --region us-west-1 | docker login --username AWS --password-stdin 565393037799.dkr.ecr.us-west-1.amazonaws.com`    

    -- When login command executed . In the background it will automatically generate the .docker/config.js . This file hold aws token and CRED store to authenticate to private repo

Step 2 : Build Docker Image with Repo tag : `docker build -t 565393037799.dkr.ecr.us-west-1.amazonaws.com/js-app:1.0 .`

     -- The dot "." at the end of the command denotes location of the Dockerfile.   

Step 3 : Push Docker Image : `docker push 565393037799.dkr.ecr.us-west-1.amazonaws.com/js-app:1.0`
    

