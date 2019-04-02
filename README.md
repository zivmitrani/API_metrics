# API_metrics
API server that Expose metrics to Prometheus
This project is an API server written by Node.js that expose 4 metrics from 2 metric types (Gauge and Count) on servername:8000/metrics.
Prometheus scarping those metrics, and Presented in Grafana dashboards.

## Requirements:
- insatll docker on your server.
- open those ports: ` 3000 - for grafana, 9090 - for prometheus, 8000 - for my API metrics exposer, 8080 - for cAdvisor. `

## Create an API server:
In my API server I used a library call `prom-client`- A prometheus client for node.js that supports histogram, summaries, gauges and counters.
that allows me to expose my own metrics and parsing the string text to a valid metrics.
Link:
https://github.com/siimon/prom-client
Example for how generate a number:
- curl --GET yourServer:8000/randomint //to generate an int number for the metrics.
- curl --GET yourServer:8000/randomfloat //to generate a float number for the metrics.
- curl --GET yourServer:8000/metrics // to see all total 4 metrics.

## Dockerfile for Node.js app:
I used this tutorial to write a Dockerfile to my project and then build an Image: https://nodejs.org/en/docs/guides/nodejs-docker-webapp/ 
```
 FROM node:latest
 MAINTAINER zivmitrani 
 WORKDIR /app 
 COPY package.json /app
 RUN npm install
 COPY . /app
 EXPOSE 8000
 CMD [ "npm","start" ]
```
after I had this dockerfile I was created an Image.
In the project directory on the server I used this commands: 
`docker build -t node_app .` -t = tag, name of the image
I wanted to know that everything is working so I checked it with:
`docker run -d --name node-app -p 8000:8000 node-api`
Later I used Docker-compose for run and orchestrate all my containers together. 

## Docker-compose.yml
Compose is a tool for defining and running multi-container Docker applications. 
tutorials I used for:
- how to create docker-compose file to nodejs project:
  https://medium.com/@francoisromain/getting-started-with-docker-for-local-node-js-development-192ceca18781
- https://takacsmark.com/docker-compose-tutorial-beginners-by-example-basics/
- https://medium.com/@salohyprivat/prometheus-and-grafana-d59f3b1ded8b

###### A brief description of each service:

```
version: '3'
services: 
```
Version it's the Compose file format, Compose refers to each container as a service.

```
  node_api:
   image: node-api
   container_name: node_app 
   ports:
     - 8000:8000
```
This service use the image I created of  the API server and create a new container named node-api, 
exposed on port 8000 outside and also in the container.

```
  prometheus:
   image: my-prom 
   container_name: prometheus
   ports:
     - 9090:9090
   depends_on:
     - node_api
```

  grafana:
   image: my-grafana
   container_name: grafana
   ports:
     - 3000:3000
   volumes: 
     - /home/ziv/project/grafana/provisioning/datasource/datasource.yml:/etc/grafana/provisioning/datasources/datasource.yml
     - /home/ziv/project/grafana/provisioning/dashboards/all.yml:/etc/grafana/provisioning/dashboards/all.yml
     - /home/ziv/project/grafana/dashboards:/var/lib/grafana/dashboards
   depends_on:
     - prometheus


  cadvisor:
    image: google/cadvisor:latest
    container_name: cadvisor
    ports:
      - 8080:8080
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    depends_on:
      - node_api


 `docker-compose up -d`



###### Prometheus
 promethus.yml & dockerfile explaintion
 
###### cAdvisor:
provides container users an understanding of the resource usage and performance characteristics of their running containers.
documentation I used for:
repo: https://github.com/google/cadvisor
toturial: https://prometheus.io/docs/guides/cadvisor/

###### Grafana:
- provisioning
- running as a container
- dashboards

_Run grafana as a container:_
first we need to build a docker image 
`docker run -d --name=grafana  -v /YourFolder/grafana/datasource.yml:/etc/grafana/provisioning/datasources/datasource.yml -p 3000:3000 my-grafana`





