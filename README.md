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

	1.SET UP THE REPOSITORY
		
		a. Update the apt package index:
			```
			sudo apt-get update
			```
		b. Install packages to allow apt to use a repository over HTTPS:
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
     		c. Add Docker’s official GPG key:
     		```
     			curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo apt-key add -
     		```
     		d. Use the following command to set up the stable repository. 
     		x86_64:
     		```
     			sudo add-apt-repository \
     				"deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \ 
     				$(lsb_release -cs) \ 
     				stable"
     		```
    2. INSTALL DOCKER CE
    	a. Update the apt package index.
    		```
    			sudo apt-get update
    		```
    	b. Install the latest version of Docker CE, or go to the next step to install a specific version.   
    	Any existing installation of Docker is replaced.
    		```
    			sudo apt-get install docker-ce
    		```
    	c. Verify that Docker CE is installed correctly by running the hello-world image.
    		```
    			sudo docker run hello-world
    		```
    3. Manage Docker as a non-root user
    	a. Create the docker group.
    		```
    			sudo groupadd docker
    		```
    	b. Add your user to the docker group.
    		```
    			sudo usermod -aG docker $USER
    		```
    	c. Log out and log back in so that your group membership is re-evaluated.
    	d. Verify that you can run docker commands without sudo.
    		```
    			docker run hello-world
    		```
    4. Configure Docker to start on boot
    	```
    		sudo systemctl enable docker
    	```
    5. To disable this behavior, use disable instead.
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

###### Apply rolling updates to a service

1. Open a terminal and ssh into the machine worker1
2. Deploy Redis 3.0.6 to the swarm and configure the swarm with a 10 second update delay:
	```
		docker service create \
  			--replicas 3 \
  			--name redis \
  			--update-delay 10s \
  			redis:3.0.6
	```
	a. You configure the rolling update policy at service deployment time.
	b. The --update-delay flag configures the time delay between updates to a service task or sets of tasks. You can   		describe the time T as a combination of the number of seconds Ts, minutes Tm, or hours Th. So 10m30s indicates 		    a 10 minute 30 second delay.
	c. By default the scheduler updates 1 task at a time. Use --update-parallelism flag to change this behaviour.
	d. If, at any time during an update a task returns FAILED, the scheduler pauses the update. 
	   --update-failure-action flag to control this behaviour.
3. Inspect the redis service:
	```
		docker service inspect --pretty redis
	```
4. Now you can update the container image for redis. The swarm manager applies the update to nodes according to the 	     	UpdateConfig policy:
	```
		docker service update --image redis:3.0.7 redis
	```
5. Run docker service inspect --pretty redis to see the new image in the desired state:
	```
		docker service inspect --pretty redis
	```
	a. The output of service inspect shows if your update paused due to failure:
	b. To restart a paused update run docker service update <SERVICE-ID>
		```
			docker service update redis
		```
6. Run docker service ps <SERVICE-ID> to watch the rolling update:
	```
		docker service ps redis
	```
	* Before Swarm updates all of the tasks, you can see that some are running redis:3.0.6 while others are running 	  redis:3.0.7 *

###### Drain a node on the swarm

1. Open a terminal and ssh into the machine worker1
2. Verify that all your nodes are actively available.
	```
		docker node ls
	```
3. Run docker service ps redis to see how the swarm manager assigned the tasks to different nodes:
	```
		docker service ps redis
	```
4. Run docker node update --availability drain <NODE-ID> to drain a node that had a task assigned to it:
	```
		docker node update --availability drain worker1
	```
5. Inspect the node to check its availability:
	```
		docker node inspect --pretty worker1
	```
	* The drained node shows Drain for AVAILABILITY.
6. Run docker service ps redis to see how the swarm manager updated the task assignments for the redis service:
7. Run docker node update --availability active <NODE-ID> to return the drained node to an active state:
	```
		docker node update --availability active worker1
	```
8. Inspect the node to see the updated state:
	```
		docker node inspect --pretty worker1
	```











