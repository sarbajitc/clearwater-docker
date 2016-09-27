# Clearwater Docker deployment on Swarm cluster

At this point in time the compose file that deployes Clearwater containers does not support Swarm cluster deployment. This fork is to address that problem.

## Steps to deploy Clearwater

Prerequisite:
- Docker Swarm cluster is running and Docker version on nodes are 1.10+ with docker compose version 1.8+.
- A docker registry is setup locally to host the Clearwater images


Deployment:
- On a independant docker host (with internet access), build the Clearwater images by cloning this repo and running the following commands
```
# Build the base Clearwater docker image.
cd clearwater-docker
for i in base bono ellis homer homestead ralf sprout ; do sudo docker build -t clearwater/$i $i ; done
```
- Now tag the images according to the local registry (`docker tag clearwaterdocker_bono docker_host:5000/clearwaterdocker_bono` where **docker-host:5000** is the private registry URL) that you have setup and push them (`docker push docker_host:5000/clearwaterdocker_bono`).
- Once the images are in local registry, clone or copy the *docker-compose.yml* file in the machine where you have compose tool installed. Make sure to source the environment for Swarm cluster (I use docker-machine to deploy my Swarm so, I can set the environment by using `eval "$(docker-machine env swarm-master)"` where **swarm-master** is the swarm manager machine name).
- Edit the *docker-compose.yml* file to update the image property for each Clearwater container to point to your local registry.
- Now you can run `docker-compose up -d` on the same directory where the *docker-compose.yml* file is kept. It will deploy the Clearwater containers in a multi-node swarm cluster.


Few explanations:
- This compose file uses docker overlay network to communicate between containers. Docker overlay network also provides name resolution (DNS) for containers. So, link property is not required to be set.
- At present bono service only listens on internal interface for 5060 and 3478 port. So, [svendowideit/ambassador](https://hub.docker.com/r/svendowideit/ambassador/) is used here as a proxy to 5060 and 3478 ports. It just forwards the external client requests to bono on those ports.
- If you are running Clearwater livetests to verify the setup be sure to set PROXY element to the bono_ambassador_5060 container's host IP and ELLIS element to ellis container's host IP.


Limitations:
- Currently the ambassador containers only listen on TCP ports so, SIP calls over UDP will not work.
- Even if bono service is scaled out (increase no. of bono containers) the ambassadors always forward packets to the same bono container. So, there is no load balancing.


For more details refer to [Project Clearwater](http://www.projectclearwater.org) and the original repo of clearwater-docker.
