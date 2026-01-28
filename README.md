## demo app - developing with Docker

This demo app shows a simple user profile app set up using
- index.html with pure js and css styles
- nodejs backend with express module
- mongodb for data storage
- Pre-requisites : mongo and mongo-express images (`docker pull`)

All components are docker-based

### With Docker

#### To start the application

Step 1: Create docker network

    docker network create mongo-network 

Step 2: start mongodb
```
docker run -d \ 
-p 27017:27017 \ 
-e MONGO_INITDB_ROOT_USERNAME=admin \ 
-e MONGO_INITDB_ROOT_PASSWORD=password \ 
--net mongo-network \ 
--name mongodb \ 
mongo
```
_NOTE: -d (detached mode), -p (port binding), -e (set environment variable)_

Step 3: start mongo-express
```
docker run -d \
-p 8081:8081 \
-e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \ 
-e ME_CONFIG_MONGODB_ADMINPASSWORD=password \ 
-e ME_CONFIG_BASICAUTH_USERNAME=user \ 
-e ME_CONFIG_BASICAUTH_PASSWORD=pass \ 
-e ME_CONFIG_MONGODB_SERVER=mongodb \ 
-e ME_CONFIG_MONGODB_URL=mongodb://mongodb:27017 \ 
--net mongo-network \ 
--name mongo-express \ 
mongo-express
```
_NOTE: The first two variables are used to access monggodb. `BASICAUTH` are used to access the web UI of mongoexpress. `MONGODB_SERVER` is used to tell mongo-express container which container it should look for. And the `MONGODB_URL` is an important one to set to avoid error saying that mongo-express can't locate mongodb. 
All those variables can be found in the mongo express documentation on github_

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

_ "up" option to start, "down" to stop.You can access the mongo-express under localhost:8080 from your browser_

Step 2: in mongo-express UI - create a new database "user-account"

Step 3: in mongo-express UI - create a new collection "users" in the database "user-account"

Step 4: start node server

    cd app
    npm install
    node server.js

Step 5: access the nodejs application from browser

    http://localhost:3000

#### To build a docker image from the application

    docker build -t my-app:1.0 .       

The dot "." at the end of the command denotes location of the Dockerfile.
Check Dockerfile to see the configuration and explanation