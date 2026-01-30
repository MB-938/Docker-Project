## Docker Project - developing with Docker

This demo app shows a simple user profile app set up using 

    - index.html with pure js and css styles
    - nodejs backend with express module
    - mongodb for data storage
    - Nexus 
    - Pre-requisites : mongo and mongo express images

All components are docker-based

Check the feature branches for more details : 

- feature/Docker-Project-1 : 
    - Run mongodb and mongo-express containers, use the application to add entry to mongodb and review the changes using mongo express.
    - Create mongo.yaml file that will be used for `docker compose`
    - Create `docker file` that will be used to build the image


- feature/Docker-Project-2 (our own application image pushed on a private repository + two images `mongodb` and `mongo express` from docker hub):

    - Configure and create a private Docker repository in Nexus
    - Configure Docker to access the private Nexus repository and then push the application image
    - Add the application to the yaml file that will be used for `docker compose`
    - Update the `server.js` code so the application can access mongodb container in a `docker compose`scenario
    - Rebuild and push the app then run it using `docker compose` on another machine