Creating containers in a simple overlay-network
-----------------------------------------------

Disclaimer:
We'll touch some parts that we didn't talk about in detail in this hands-on, namely docker swarm and docker-stacks (docker service command). Don't be afraid, the commands should be straight forward and this should just show you how simple you can create a virtual overlay-network with two hosts that run docker :).

Step 1
------
* make sure you have two instances in play-with-docker, we'll call them node1 and node2
* you can see the IP addresses of the machines in the prompt and on the top of the page (above terminal)
* on node1, run ´docker swarm init --advertise-addr <ipAddressOfNode1>´
	* you'll see some output which shows that you've created a new docker swarm
	* the current node1 is a swarm master, other "worker nodes" can join the swarm
	* a little figure is now displayed next to your node, showing that this is a node master
	* notice that a secure token was created automatically for mutual authentication (secured by TLS by default)
* copy the complete `docker swarm join` shown in the output to node2 and execute it there
* tadaa! we have created a swarm with two nodes! 

Step 2
------
Now we can create a new overlay network (logical/virtual network) which we will afterwards span across both nodes.
* run on node1: 
	
	docker network create -d overlay ueber-net
	
* you can list the new network with ´docker network ls´
* since we're in "swarm mode" on our nodes now, docker will brint the new network automatically to the other node if the network is needed by a container

Step 3
------
* We will now use docker stack to start two containers in our swarm. We'll define that ueber-net should be used for both created containers. 

	docker service create --name t --network ueber-net --replicas 2 ubuntu sleep infinity
	
* notice that the default strategy is to distribute the containers in an even manner, so we can be sure that two containers run on different nodes. (side note: Normally it's recommended to create an odd number of instances to still have distinct majorities if a node disappears, but we will ignore that for this lab exercise =)
* as you can see, the containers are just idling, we just want to keep them running as an example ("sleep infinity" command)

Step 4
------
* verify that both nodes have 1 ubuntu container running now
* both containers use the ueber-net (=> docker container inspect)
* node2 automatically got the ueber-net network as well!
* the great thing is that this would also work if both hosts are running in completely different (physical) networks

Step 5 For quick people
------------------------
Jump into the container (with 'docker exec') and install ping command:
	apt-get update
	apt-get install iputils-ping
=> you can ping the other container by name or its internal docker IP (docker inspect shows this IP).
