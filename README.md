## demo app - developing with Docker

This demo app shows a simple user profile app set up using
- index.html with pure js and css styles
- nodejs backend with express module
- mongodb for data storage
- Pre-requisites : mongo and mongo-express images (`docker pull`)

All components are docker-based

### With Nexus

#### Setup Nexus and Docker login

Step 1: Create repository (docker hosted). Tick the HTTP (since we use a non secure connection) option and specify a port (8083 for example) to allow `docker login`

_NOTE: You can use `netstat -lnpt` to verify that the port is open inside the droplet._

Step 2 : In Digital Ocean we also need to open the port we specified (8083)

Step 3 : In Nexus we need to configure Realms to allow Docker Bearer Token Realm

_Note : That token will be stored in `~/.docker/config.json`_

Step 4: Create and assign a Role to allow the user to access Docker repository (viewer privileges)

### With Docker
Step 1: Configure Docker client to allow access to the "not secure"(HTTP) registry
 - In Linux you need to edit `/etc/docker/daemon.json` and add :
```
{
    "insecure-registries" : ["yourregistryIP:port"]
}
```
 - If you are using Docker desktop you won't find this file so you need to edit Docker engine in Docker desktop settings and add :
```
"insecure-registries" : ["yourregistryIP:port"]
```

Step 2: Use docker login

    docker login youregristryIP:port

_Note : After the first login the `~/.docker/config.json` is updated with the authentication token so we won't need to use credentials every time we use docker login_ 

Step 3: Push the image to Nexus repository

- We already have an image called `"my-app:1.0"`, but docker push won't know where to push it if we don't specify the repository. To do that we need to use this command:
```
docker tag my-app:1.0 yourrepositoryIP:port/my-app:1.0
```
- We can then use this command :
```
docker push yourrepositoryIP:port/my-app1:0
```

Step 4: Using Nexus Api we can check if the images is uploaded :
```
curl -u username:password -X GET 'yourepositoryIP:port/service/rest/v1/components?repository=repositoryname'
```

### With Docker Compose

We now have our application in our private Nexus repository and mongodb + mongo express in the public docker hub repository. Using docker compose we are going to pull and run all the images.

#### To start the application

Step 1: List/configure all the container in a yaml (`mongo.yaml` in our case) that will be used by `docker compose`
```yaml
services:
  my-app:
    image: yourrepositoryIP:port/my-app:1.0
    ports:
     - 3000:3000
  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=""
      - MONGO_INITDB_ROOT_PASSWORD=""
  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=""
      - ME_CONFIG_MONGODB_ADMINPASSWORD=""
      - ME_CONFIG_BASICAUTH_USERNAME=""
      - ME_CONFIG_BASICAUTH_PASSWORD=""
      - ME_CONFIG_MONGODB_SERVER=""
      - ME_CONFIG_MONGODB_URL=mongodb://mongodb:27017
```
_Note : This mongo.yaml will need to be on the server that will run the application. In our case we just created the mongo.yaml outside the app directory and we will execute `docker compose` from there._

Step 2 : Update server.js code

In our application we were using localhost:port (variable : `mongoUrlLocal`) since we were running the app outside the docker network were the mongodb was located. But with docker-compose all three container will run in the same isolated docker network. So we will need to update the variable `mongoUrlDockerCompose` in server.js code so our application can access to the mongodb container.

- Replace all `mongoUrlLocal` by `mongoUrlDockerCompose`
- Here is the difference between the two variables
```js
// use when starting application locally with node command
let mongoUrlLocal = "mongodb://admin:password@localhost:27017";

// use when starting application as docker container, part of docker-compose
let mongoUrlDockerCompose = "mongodb://admin:password@mongodb";
```
Step 3 : Rebuild the app and push the app again into the private Nexus repository

Since we updated the code we need to rebuild and push the application

- Remove the previous images if it still exists :
```
docker rmi yourrpositoryIP:port/my-app:1.0
```
- Rebuild the app
```
docker build -t yourrpositoryIP:port/my-app:1.0 .
```
- Push the app
```
docker push 161.35.220.26:8083/my-app:1.0
```


Step 4 : Use docker login command (if we were on a different machine)
```
docker login youregristryIP:port
```

Step 5: Run all three container using `docker compose`
```
docker compose -f mongo.yaml up
```