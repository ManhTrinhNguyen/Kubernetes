# Deploy MongoDB and Mongo Express

## Demo 
```
  1. Create MongoDB pod 
    - In order to talk to Pod I need service 
    - I will create Internal Service . Only the components inside cluster can request to it 
  2. Create Mongo Express 
    - Need URL of MongoDB so Mongo Express can connect to it 
    - Credentials : Need username and password to authenticate 
    - The way I can pass these Infomation is via Deployment configuration file through ENV (That is how application configure)
    - I will create a Config Map that containe DB URL and I will create Secret that container Credentials and I will reference both inside the Deployment 
    - Once I have that set up I will need Mongo Express accessiable through browser in order to do that will create external Service that will allow external request to talk to the Pod
```

**Create MongoDB Deployment**
```
  
```