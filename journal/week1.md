# Week 1 — App Containerization

### **Docker Components**
![Docker Component](https://docs.docker.com/engine/images/architecture.svg)

- Client is basically is installed your docker locally (build, pull, run features)
- Server is the location where is running the container

Registry is a location of the images available on internet (an exampple is docker hub). you could have a private registry inside of your organisation.

#### **Security Best Practice**
- Keep Host & Docker Updated to latest security patches.
- Docker Deamon & containers should run in non root user mode
- Image Vulnerability Scanning
- Trust a Private vs Public Image Registry
- No Sensitive Data in Docker Files or Images
- Use Secret Management Services to share secrets.
- Read only file system and volume for dockers
- Separate databases for long term storage
- Use DevSecOps pratices while building application security
- Ensure all code is tested for vulnerabilities before production use


#### **Docker Compose** 
It is a tool for defining and running multi container Docker Applications (It uses yml file).

### Tool to indefity vulnerability on your Docker Compose
Snyk OpenSource Security

### Tools to Store and Manage Secrets
- Aws Secret Manager
- Hashicorp Vault

### Tools to scan Image Vulnerability
- AWS Inspector
- Clair
- Snyk COntainer Security

### Running Containers in AWS
Problem with docker compose and Docker Containers: If you need to change, you need to stop the machine update the file and restart.

For the Managed Containers you can use the following AWS service
- AWS ECS
- AWS EKS
- AWS Fargate

### Cruddur Task

To Run the containerize backend 

### Run Python

```sh
cd backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
python3 -m flask run --host=0.0.0.0 --port=4567
cd ..
```

- make sure to unlock the port on the port tab
- open the link for 4567 in your browser
- append to the url to `/api/activities/home`

https://4567-fathytek-awsbootcampcru-<GENERATED_URL_GITPOD>.gitpod.io/api/activities/home

image.png

### Build Container

under the root directory, I build my docker file image dedicated for the backend

```sh
docker build -t  backend-flask:latest ./backend-flask

docker run --rm -p 4567:4567 -it backend-flask
# equivalent command

docker build -t  backend-flask ./backend-flask

```

 



### Add Dockerfiler

under  the repository backend-flask, create Dockerfile with the following content 

```dockerfile
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```
### Run Container

Run 
```sh
docker run --rm -p 4567:4567 -it backend-flask
FRONTEND_URL="*" BACKEND_URL="*" docker run --rm -p 4567:4567 -it backend-flask
```
These two commands will not run the containers correctly, you can check the environment variables by clicking on the backend-flask container and select Attach Shell, It will run the following command in the terminal
```sh
docker exec -it 4c524ee49f4fdc2b3df3978fcd4ef3bad7309f5b9eca2510b7533b637697ce74 bash 

```
to fix this problem we need to run our container as follow:

```sh
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask

```
### Get  Running Container Ids and Images

you can run  docker ps to get the running containers in you terminal

to check the docker images present in your docker host use docker images


```
docker ps
docker images
```
you can run also 

```
docker ps -a 
```



### Running docker in background

Run in background
```sh
docker container run --rm -p 4567:4567 -d backend-flask
```

Return the container id into an Env Vat
```sh
CONTAINER_ID=$(docker run --rm -p 4567:4567 -d backend-flask)
```

### Send Curl to Test Server

To test your server, you can use the curl command

```
curl -X GET http://localhost:4567/api/activities/home -H "Accept: application/json" -H "Content-Type: application/json"
```


## Containerize Frontend

## Run NPM Install

Befor building our frontend container we have to install npm so please follow these instructions :

```
cd frontend-react-js
npm i
```

### Create Docker File

Create a file here: `frontend-react-js/Dockerfile`

```dockerfile
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```

### Build Container

```sh
docker build -t frontend-react-js ./frontend-react-js
```

### Run Container

```sh
docker run -p 3000:3000 -d frontend-react-js
```

## Multiple Containers

### Create a docker-compose file

Create `docker-compose.yml` at the root of your project.

```yaml
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```
## Adding DynamoDB Local and Postgres

We need to integrate the  Postgres and DynamoDb into our existing docker compose filenas follow:

### Postgres

```yaml
services:
  db:
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data
volumes:
  db:
    driver: local
```

To install the postgres client into Gitpod

```sh
  - name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev
```

### DynamoDB Local

```yaml
services:
  dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
```

### Challenge Dynamodb Local

The following section is taken from Andrew repo 

Dynamodb Local emulates a Dynamodb database in your local envirmoment for rapid developement and table design interation

## Create a table

```
aws dynamodb create-table \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema AttributeName=Artist,KeyType=HASH AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --table-class STANDARD
```
## Create an Item

```
aws dynamodb put-item \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --item \
        '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}}' \
    --return-consumed-capacity TOTAL  
```
## List Tables

```
aws dynamodb list-tables --endpoint-url http://localhost:8000
```
## Get Records

```
aws dynamodb scan --table-name cruddur_cruds --query "Items" --endpoint-url http://localhost:8000
```

Please refer to 
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.CLI.html

## Volumes

directory volume mapping

```
volumes: 
- "./docker/dynamodb:/home/dynamodblocal/data"
```

```
volumes: 
  - db:/var/lib/postgresql/data

volumes:
  db:
    driver: local
```


