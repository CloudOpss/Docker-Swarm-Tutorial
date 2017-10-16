# Cluster Management of Docker Containers using Docker Swarm  

### What is a swarm?  
A swarm is a cluster of Docker engines, or nodes, where you deploy services.

[For more information about swarm and its key concepts visit this link](https://docs.docker.com/engine/swarm/key-concepts/)  

### Highlights of Swarm Mode  
	Cluster management integrated with Docker Engine
	Decentralized design
	Declarative service model
	Scaling
	Desired state reconciliation
	Multi-host networking
	Service discovery
	Load balancing
	Secure by default
	Rolling updates

[For more information on highlights visit this link](https://docs.docker.com/engine/swarm/#feature-highlights)  

### Swarm Mode CLI Commands  
	swarm init
	swarm join
	service create
	service inspect
	service ls
	service rm
	service scale
	service ps
	service update

### Getting started with swarm mode  

###### Set up

Three Linux hosts which can communicate over a network, with Docker installed.

For this tutorial i am considering 3 ubuntu based Amazonn Ec2 instances.

One of these machines will be a manager (called Manager) and two of them will be workers (Worker1 and Worker2).

After You are done creating 3 hosts

Install Docker Engine on all 3 of them.

Here I am showing Docker CE(Community Editon) installation steps for ubuntu since our hosts are ubuntu machines.

	SET UP THE REPOSITORY
		
		Update the apt package index:
			```
			sudo apt-get update
			```
		Install packages to allow apt to use a repository over HTTPS:
			Jessie or Stretch:
				```
				sudo apt-get install \
     				apt-transport-https \
     				ca-certificates \
     				curl \
     				gnupg2 \
     				software-properties-common
     			```
     		Wheezy:
     			```
     				sudo apt-get install \
     				apt-transport-https \
     				ca-certificates \
     				curl \
     				python-software-properties
     			```
     	Add Docker’s official GPG key:
     		```
     			curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo apt-key add -
     		```
     	Use the following command to set up the stable repository. 
     		x86_64:
     		```
     			sudo add-apt-repository \
     				"deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \ 
     				$(lsb_release -cs) \ 
     				stable"
     		```
    INSTALL DOCKER CE
    	Update the apt package index.
    		```
    			sudo apt-get update
    		```
    	Install the latest version of Docker CE, or go to the next step to install a specific version.   
    	Any existing installation of Docker is replaced.
    		```
    			sudo apt-get install docker-ce
    		```
    	Verify that Docker CE is installed correctly by running the hello-world image.
    		```
    			sudo docker run hello-world
    		```
    Manage Docker as a non-root user
    	Create the docker group.
    		```
    			sudo groupadd docker
    		```
    	Add your user to the docker group.
    		```
    			sudo usermod -aG docker $USER
    		```
    	Log out and log back in so that your group membership is re-evaluated.
    	Verify that you can run docker commands without sudo.
    		```
    			docker run hello-world
    		```
    Configure Docker to start on boot
    	```
    		sudo systemctl enable docker
    	```
    To disable this behavior, use disable instead.
    	```
    		sudo systemctl disable docker
    	```

###### Open protocols and ports between the hosts  

The following ports must be allowed on security group for the instances

	TCP port 2377 for cluster management communications
	TCP and UDP port 7946 for communication among nodes
	UDP port 4789 for overlay network traffic


###### Create a swarm

1. Open a terminal and ssh into the machine Manager

2. Run the following command to create a new swarm:
		```
			docker swarm init --advertise-addr <MANAGER-IP>
		```
		MANAGER-IP -> IF ADDRESS OF Manager1 machine

3. Run docker info to view the current state of the swarm:
		```
			docker info
		```

4. Run the docker node ls command to view information about nodes:
		```
			docker node ls
		```

###### Add nodes to the swarm

1. Open a terminal and ssh into the machine worker1
2. Run the command produced by the docker swarm init
	If you don’t have the command available, you can run the following command on a manager node to retrieve the join command for a worker:	
	```
		docker swarm join-token worker
	```
3. repeat the steps 1 & 2 for Worker2 node
4. Open a terminal and ssh into the machine Manager
	```
		docker node ls
	```

###### Deploy a service to the swarm

1. Open a terminal and ssh into the machine worker1
2. Run the following command:
	```
		docker service create --replicas 1 --name helloworld alpine ping docker.com
	```
	The docker service create command creates the service.
	The --name flag names the service helloworld.
	The --replicas flag specifies the desired state of 1 running instance.
	The arguments alpine ping docker.com define the service as an Alpine Linux container that executes the command ping docker.com
3. Run docker service ls to see the list of running services:
	```
		docker service ls
	```

###### Inspect a service on the swarm

1. Open a terminal and ssh into the machine worker1
2. Run docker service inspect --pretty <SERVICE-ID> to display the details about a service in an easily readable format.  
	```
		docker service inspect --pretty helloworld
	```
3. Run docker service ps <SERVICE-ID> to see which nodes are running the service:
	```
		docker service ps helloworld
	```
4. Run docker ps on the node where the task is running to see details about the container for the task.  
	```
		docker ps
	```

###### Scale the service in the swarm

1. Open a terminal and ssh into the machine worker1
2. Run the following command to change the desired state of the service running in the swarm:  
	```
		docker service scale <SERVICE-ID>=<NUMBER-OF-TASKS>
	```
	For example:
	```
	docker service scale helloworld=5
	```
	helloworld scaled to 5

3. Run docker service ps <SERVICE-ID> to see the updated task list:
	```
		docker service ps helloworld
	```
4. Run docker ps to see the containers running on the node where you’re connected. 
	```
		docker ps
	```

###### Delete the service running on the swarm

1. Open a terminal and ssh into the machine worker1
2. Run docker service rm helloworld to remove the helloworld service.
	```
		docker service rm helloworld
	```
3. Even though the service no longer exists, the task containers take a few seconds to clean up.   
	You can use docker ps on the nodes to verify when the tasks have been removed.















