# Demo Notes

*Docker Demo - from zero to scalable and redundant (build, ship, scale)*
*Adapted from (https://docs.docker.com/get-started/part5/)*

## Part 1: a basic container with nginx
docker run --name docker-nginx -p 999:80 -d nginx

*Now we should have the Nginx server running here [http://localhost:999](http://localhost:999)*

*in a different world, I'd have to have an operating system, system libraries,  running a ton of stuff, in this case my only dependency is docker, but running a web server isn't enough, modern architectures are a little bit more complex so let me show you more!*

## Part 2: Build a Dockerfile to Customize our Container

*We'll build a Dockerfile so that we can custimize our container with a little message*

```
#Dockerfile 
FROM nginx
RUN echo '<h1>Hello TIBCO NOW!</h1>' > /usr/share/nginx/html/index.html
```

### Build and Run the Container 
`docker build -t docker-nginx .`
`docker run -p 9999:80 docker-nginx`

*Now we should have the Nginx server running here [http://localhost:9999](http://localhost:9999)*


## Part 3: Push Image to Repo
*We want others to be able to see our image, in my case I pushed the image to Docker's public hub, could be private or you can run your own hub internally*
`docker push ernestoongaro/demos`
*You will now see it at [https://hub.docker.com/r/ernestoongaro/demos/](https://hub.docker.com/r/ernestoongaro/demos/)*

##Part 4: Complexity!

*Great, now we're cooking with fire, and this would be enough of an infrastructure to run a static (web 1.0 site), but in reality we have a more complex app that requires more than one service*
*We'll create a docker Compose file - it will set up a network with auto load balancing and manage multiple images*

```
#docker-compose.yml
version: "3"
services:
  nginx:
    image: nginx
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "99:80"
    networks:
      - webnet
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8888:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
networks:
  webnet:

```

*Check this into Github, now anybody can set this up on a different machine (this is the control part!)*
*Finally, we can run our service*

`docker-compose up`

## Part 5: Scale

*We now want to scale our service and run it here or on the cloud*   
*First we initalize our "swarm" which is Docker's way to run an entire cluster*
`docker swarm init`

Second, we deploy our services to the swarm

`docker stack deploy -c docker-compose-complex.yml docker-nginx-swarm`

checks status

`docker stack ps docker-ngnix-swarm`

`docker service ls`

*[http://localhost:8888](http://localhost:8888)*



### Scattered Notes

`docker stack rm docker-nginx-swarm`

 
