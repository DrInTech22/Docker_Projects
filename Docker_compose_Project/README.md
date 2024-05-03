
# Containerize and Deploy a 3-tier application with Docker-compose.

This project involves a front-end application written in simple html and a nodejs backend packaged together and deployed as a single application. MongoDB serves as the database, with Mongo-express providing a GUI for database interaction.

The application is a profile page that saves information about a user.


## Project Task
1. Containerize the application and push the image (tagged as `1.0`) to a central repository for easy distribution.
2. Utilize a MongoDB container as the database and ensure data persistence.
3. Connect the Mongo-express container to the MongoDB container to enable visual interaction with the database.
4. Enable deployment of the entire setup with a single command (docker-compose up) without requiring additional configuration.
5. Enable deletion of the setup with a single command (docker-compose down) without requiring additional commands to clean up resources such as networks or stopped containers.
## Solution 
### Containerizing the application
```yaml
FROM node:13-alpine

ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PWD=password

RUN mkdir -p /home/app

COPY ./app /home/app

# set default dir so that next commands executes in /home/app dir
WORKDIR /home/app

# will execute npm install in /home/app because of WORKDIR
RUN npm install

CMD ["node", "server.js"]
```
- The application is built using `node:13-alpine` base image (runtime).
- Database connection credentials are provided as environment variables (note: it's not best practice to provide credentials in plain text).
- All necessary files and dependencies are copied to the image with the `COPY` command.
- Created a new repository named `my-app` on my dockerhub.
- A docker image of the application is built with the specified tag using the command below
``` 
docker build -t my-app:1.0 
``` 
- The image is then tagged with the username/image_name:tag and pushed to docker hub.
```yaml
docker tag my-app:1.0 maestrops/my-app:1.0
docker push maestrops/my-app:1.0
```

### Let's deploy the application with Docker Compose.

```yaml
version: '3'

services:
  my-app:
    image: maestrops/my-app:1.0
    ports:
     - 3000:3000
  mongodb:
    image: mongo
    ports:
     - 27017:27017
    environment:
     - MONGO_INITDB_ROOT_USERNAME=admin
     - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
     - mongo-data:/data/db
  mongo-express:
    image: mongo-express
    restart: always
    ports:
     - 8081:8081
    environment:
     - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
     - ME_CONFIG_MONGODB_ADMINPASSWORD=password
     - ME_CONFIG_MONGODB_SERVER=mongodb
    depends_on:
     - "mongodb"

volumes:
  mongo-data:
    driver: local
```
The docker compose file above deploys the entire setup, let's see in bits how it works.

#### MY-APP (application)
```yaml
  my-app:
    image: maestrops/my-app:1.0
    ports:
     - 3000:3000
```
- `my-app` service represent the application, exposed on port `3000`.
- `image: maestrops/my-app:1.0` : the application will be built with the image we created.

#### VOLUME 
```yaml
volumes:
  mongo-data:
    driver: local
```
- This create a docker volume on the host.

#### MONGODB (database)
```yaml
mongodb:
    image: mongo
    ports:
     - 27017:27017
    environment:
     - MONGO_INITDB_ROOT_USERNAME=admin
     - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
     - mongo-data:/data/db
```

- This utilizes the official Mongo image and exposes the default port `27017`.
- For authentication, the username and password is set as environment variables (MONGO_INITDB_ROOT_USERNAME, MONGO_INITDB_ROOT_PASSWORD) in the container.
- The docker volume `mongo-data` on host is mapped to database storage volume `/data/db` on the container for data persistence.

#### MONGO-EXPRESS (GUI for MONGO)
```yaml
mongo-express:
    image: mongo-express
    restart: always
    ports:
     - 8081:8081
    environment:
     - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
     - ME_CONFIG_MONGODB_ADMINPASSWORD=password
     - ME_CONFIG_MONGODB_SERVER=mongodb
    depends_on:
     - "mongodb"
```
- The official mongo-express image is used.
- `restart:always` : restarts the mongo-express container paradventure it fails.
- the container is exposed on port `8081`
- To establish connection with the mongo database, the credentials of the mongo database are provided as environment variables.
- `depends_on` : makes sure the mongo container is up and running before the mongo-express starts.

### A Look into the application
How does the application connect to the database?
The nodejs code uses mongo client service to establish connection with the database using the URL in the format below
`protocol://username:password@ip_addr:port`
`mongodb://admin:password@mongodb`
- `@mongodb` reference the `mongodb` service in the docker-compose file, which holds the ip address and url of the mongodb database.

### Final deployment
- The simple command `docker-compose up` will pull the images, set up network and get our applications running.
- The command `docker-compose down` will destroy the entire setup and clean up resources.
