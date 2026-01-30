## Docker Project 1

    - Run mongodb and mongo-express containers, use the application to add entry to mongodb and review the changes using mongo express.
    - Create mongo.yaml file that will be used for `docker compose`
    - Create `docker file` that will be used to build the image

### With Docker

#### To start the application

Step 1: Create docker network

    docker network create mongo-network 

Step 2: start mongodb
```
docker run -d \ 
-p 27017:27017 \ 
-e MONGO_INITDB_ROOT_USERNAME="" \ 
-e MONGO_INITDB_ROOT_PASSWORD="" \ 
--net mongo-network \ 
--name mongodb \ 
mongo
```
_NOTE: -d (detached mode), -p (port binding), -e (set environment variable)_

Step 3: start mongo-express
```
docker run -d \
-p 8081:8081 \
-e ME_CONFIG_MONGODB_ADMINUSERNAME="" \ 
-e ME_CONFIG_MONGODB_ADMINPASSWORD="" \ 
-e ME_CONFIG_BASICAUTH_USERNAME="" \ 
-e ME_CONFIG_BASICAUTH_PASSWORD="" \ 
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
Step 1 : Create a yaml file that will be used by `docker compose`. In our case it's called `mongo.yaml`

```yaml
version: '3'
services:
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
      - ME_CONFIG_MONGODB_SERVER=mongodb
      - ME_CONFIG_MONGODB_URL=mongodb://mongodb:27017
```

Step 2: start mongodb and mongo-express

    docker-compose -f mongo.yaml up

_NOTE : "up" option to start, "down" to stop.You can access the mongo-express under localhost:8080 from your browser_

Step 2: in mongo-express UI - create a new database "user-account"

Step 3: in mongo-express UI - create a new collection "users" in the database "user-account"

Step 4: start node server

    cd app
    npm install
    node server.js

Step 5: access the nodejs application from browser

    http://localhost:3000

#### To build a docker image from the application
Step 1 : Create a `docker file`
```dockerfile
FROM node:20-alpine

ENV MONGO_DB_USERNAME="" \
    MONGO_DB_PWD=""

RUN mkdir -p /home/app

COPY ./app /home/app

# set default dir so that next commands executes in /home/app dir
WORKDIR /home/app

# will execute npm install in /home/app because of WORKDIR
RUN npm install

# no need for /home/app/server.js because of WORKDIR
CMD ["node", "server.js"]
```
Step 2 : Build the app
    docker build -t my-app:1.0 .       

The dot "." at the end of the command denotes location of the Dockerfile.
"my-app:1.0" will be the image we will use to `docker run my-app:1.0`.