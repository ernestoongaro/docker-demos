#The Demo Script

#Docker Demo - from zero to scalable and redundant (build, ship, scale)
#Adapted from https://docs.docker.com/get-started/part5/

#a basic container with nginix - run + show in browser
docker run --name docker-nginx -p 999:80 -d nginx

#in a different world, I'd have to have an operating system, system libraries,  running a ton of stuff, in this case my only dependency is docker
# but running a web server isn't enough, modern architectures are a little bit more complex so let me show you more!

#Dockerfile 
FROM nginx
RUN echo '<h1>Hello TIBCO NOW!</h1>' > /usr/share/nginx/html/index.html

#Build and Run 
docker build -t docker-nginx .
docker run -p 9999:80 docker-nginx

#Push Image to Repo
docker push ernestoongaro/demos
#Show pull command from Docker Hub

#Now we want to run our image in a production like environment 

#Check this into Github, now anybody can set this up on a different machine (this is the control part!)
#Great, now we're cooking with fire, and this would be enough of an infrastructure to run a static (web 1.0 site), but in reality we have a more complex app that requires a database as well

#Docker Compose Services - set up a network with auto load balancing

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
  
#docker swarm init
#Deploy services to swarm

docker stack deploy -c docker-compose.yml docker-nginx-swarm
docker stack ps docker-ngnix-swarm
docker service ls

# Cleanup
docker stack rm docker-nginx-swarm

 
