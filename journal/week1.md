# Week 1 — App Containerization

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





