# Docker

## Overall
- Each container has its own Libraries and dependencies
- Each application can be deployed on its own container
- No conflicting / incompatible Libraries (Libs) or Dependencies (Deps)
- Docker works with any system that uses the Linux Kernel. In other words, any flavour of linux can run Docker.
- Windows OS cannot support docker as it runs on Windows Kernel. The way around it is to create a Linux VM inside a Windows OS to host Docker.

## Benefits
Containers:
- Quick and easy to provision 
- Less resource requirements
- Only runs on linux distribution system

Virtual Machines:
- Better isolation from each service (not sharing OS)
- Can run multiple different OS in one machine
- Is resource hungry
- Containers + virtual machines
- Can run multiple OS in one machine
- Can quickly provision and scale applications using containers

## Containers vs. Image
- Image: is a package of the service. 
- Container: a running instance of the image (system package) you require

Dockerfile
- This is the guide which used to be given to the Ops team to provision application
- It’s a guide informing Docker how to run an application. 
- The guide alongside the application is packaged inside a container/docker Image.

# Docker run
Create a container
```
docker run redis
docker run --name redis-container redis
docker run --name redis-container redis:5.0.5
docker run (--name=<container-name>) <image-name>:(<image-version>)
```

Port mapping
```
Open docker-host (localhost) port and point it to docker container port.
docker run -p 8080:5000 nginx
docker run -p <docker-host-port>:<docker-container-port> <image-name>
```

Volume mapping
```
Mount container directory to localhost docker-host container
docker run -v /Users/salim/container-data:/var/lib/mysql msql
docker run -v <docker-host-directory>:<docker-container-directory> <image-name>
```
There is now a new way of mounting volumes using (--mount)
Type can be:
- volume: mounts docker container path to default docker hosts location: /var/lib/docker/volume
- bind: any other location than the default volume mount
- docker run --mount type=bind,source=/Users/Salim/container-data,target=/var/lib/mysql msql msql

Investigate container
```
docker inspect <container-name>
docker logs <container-name>
```

Jenkins
- Find Jenkins web server’s IP given by docker host using “docker inspect <container-name>” then use web browser to view Jenkin e.g http://172.17.0.2:8080
- Run using port-mapping and volume-mapping/mounting. This ensures config data is backed up at Users/salim/testing
- docker run -p 8080:8080 -v /Users/salim/testing:/var/jenkins_home jenkins/jenkin


## Docker images

Dockerfile
```
# creating my own image

# Base image
FROM UBUNTU

# Update and install dependencies
RUN apt-get update
RUN apt-get install python

# install python dependencies
Run pip install flask
Run pip install flask-mysql

# Copy source code to container
COPY ./  /opt/source-code

# Start flask
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

Once you’ve created your dockerfile, you can build it into an image by running the following

Building image
```
docker build Dockerfile -t nabeel-docker/my-custom-app
docker build Dockerfile -t <docker-account-id>/<docker-image-name>
```

Push your image to docker registry (repo)
```
docker push nabeelaccount/my-custom-app
Docker push <docker-account-id>/<docker-image-name>
```

Below we can see how a docker image is written. The registry can be docker.io (docker), gcr.io (GCP), ecr.io (AWS)
Insert-image


## Docker Compose

- Used for composing multiple applications
- Runs only on one single Docker Host.
- Docker compose allows us to link multiple docker run applications together. It does the networking.
- We can manually link docker run app’s by using the “--link <container-name>:<target-container-name>”
- e.g. docker run --name=vote -p 5000:80 --link redis:redis voting-app 
- Multiple links e.g. docker run --name=worker --link redis:redis --link db:db worker
- note: redis:redis = redis

Docker-compose.yaml (version 1)
```
vote:
	image: voting-app
	ports:
	  - 5000:80
	links:
	  -  redis
redis:
image: redis
links:
	  -  worker
worker:
image: worker
	links:
	  - db
db:
image: db
	links:
	  -  result
result:
Image: result-app
	ports:
	  - 5001:80
```

Docker-compose.yaml (version 3)
```
version: "3"
services:
    vote:
    	image: voting-app
    	ports:
    	  - 5000:80
    	links:
    	  -  redis
    redis:
    image: redis
    links:
    	  -  worker
    worker:
    image: worker
    	links:
    	  - db
    db:
    image: db
    	links:
    	  -  result
    result:
    Image: result-app
    	ports:
    	  - 5001:80
```
Start docker-compose with `docker-compose up`

## Network

There are three types
Bridge: default network used by docker and it’s private within the docker network. The network type is called bridge
host: uses the hosts network. Meaning a port 5000 opened on the container will use docker hosts 5000 port
none: container does not connect to any network (isolated)

Embedded DNS
Docker has an inbuilt DNS that will resolve to container names. 
You do not need to provide the IP address of the container you want to connect to. You can use its name and the inbuilt DNS server will resolve the name to the IP of the container you would like to connect to.

Separation of Containers
Docker uses namespaces to separate the containers. The isolated namespaces are then provided with virtual network interfaces/ports for them to have the capacity to connect to other containers. This assumes you chose a bridge or host network type.

Network commands

```
docker network ls
docker network inspect <network-name>
docker inspect <container-name>  and look for network to find more detailed info
docker run --network=none/bridge/host 

# create a new network
docker network create --help
```
