# Swarm Mode - Simple App

So far you've worked with containers running on a single host. As applications are usually more complex, and a single-server model will not coordinate tens (hundreds) of containers, and ensure high availability or horizontal  scaling without effort, we would need to integrate an orchestration tool to manage containers.

Let's take a look at Docker's own solution: **Swarm Mode**, which tells Docker that you will be running multiple Docker engines and want to coordinate operations across all of them. Swarm mode combines the ability to not only define the application architecture, like Compose, but to define and maintain high availability levels, scaling, load balancing, and more.

## Initialize Your Swarm

We could Swarm a dozen applications, but a good example to use would be the Docker Voting App. It is often used for demo purposes during meetups and conferences. It's architecture contains 5 containers:

* A Python webapp which lets you vote between two options
* A Redis queue which collects new votes
* A .NET worker which consumes votes and stores them in…
* A Postgres database backed by a Docker volume
* A Node.js webapp which shows the results of the voting in real time

This application is available on Github and updated very frequently when new features are developed. In most cases, first thing you need to do is tell the Docker hosts you want to use Docker in Swarm Mode. Because you're using Docker for Windows, the Swarm is already configured. So this is mostly for your information.

Swarms *can* be just a single node, but that is unusual as you would have no high availability capabilities and would severely limit your scalability. Most production swarms have at least three *manager* nodes in them and many *worker* nodes. 

> Note:
> Manager nodes can run your container tasks the same as a worker node, but this functionality can also be separated so that managers only perform management tasks.

To initialize Swarm Mode you can run the following command:

`	$	docker swarm init	`

 The output for the command above should be something like this:

```
Swarm initialized: current node (bxrwaon0coi8ff9y684lvc9hv) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-33akpkw6zvdkg0b52mpur4y3cyhagfli5ltpt1o37dw0qmwh8y-an85fq1f1rspdnpt3f2m19emm 192.168.65.3:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

In the output of your swarm init, you are given a command in the middle that looks like *docker swarm join -token SWMTKN-X-abcdef.....* which you would use to join workers nodes to the swarm. You are also given a second command `docker swarm join-token manager` for adding additional managers. From the first terminal window, check the number of nodes in the swarm (running this command from the second terminal worker node will fail as swarm related commands need to be issued against a swarm manager).

`	$ docker node ls	`

The above command should output at least 1 node, a manager. Please note that the manager node is also the "Leader". This is because you only have one manager node. The Leader is just what it sounds like: the main control node for all the managers. If your Leader node goes down for some reason, the other manager nodes will elect a new leader.

Here is a view of managers and workers in Docker Swarm Mode. In our exercise, we have just one manager and three workers, but you can see how multiple managers and workers interact in the diagram:
<p align="center">
  <img src="https://bitbucket.endava.com/projects/ENDAVA/repos/endava-docker-training/raw/swarm-mode/images/docker-swarm-infrastructure.svg?at=refs%2Fheads%2Fmaster" alt="Docker Swarm Architecture"/>
</p>
Now to do something interesting we will use the voting app in the bitbucket repo to deploy a stack.

## Deploy a Stack

A ***stack*** is a group of ***services*** that are deployed together (multiple containerized components of an application that run in separate instances). Each individual *service* can actually be made up of one or more containers, called ***tasks*** and then all the tasks & services together make up a *stack*.

As with Dockerfiles and the Compose files, the file that defines a stack is a plain text file that is easy to edit and track. In our exercise, there is a file called *docker-stack.yml* in the swarm-mode folder which will be used to deploy the voting app as a stack. You can use notepad++ or your text-editor of choice to inspect the file.

- Near the top of the file you will see the line "services:". These are the individual application components.

Ensure you are in the `manager1` manager terminal and do the following:

`	$ docker stack deploy --compose-file=docker-stack.yml voting_stack	`

You can see if the stack deployed from the [node1] manager terminal

`	$ docker stack ls	`

The output should be the following. It indicates the 6 services of the voting app's stack (named voting_stack) have been deployed.

```batch
NAME          SERVICES
voting_stack  6
```

For details on each service within the stack, use the command:

`	$ docker stack services voting_stack	`

The output should be similar to the following:

```batch
ID            		NAME                     	MODE        	REPLICAS  				IMAGE
10rt1wczotze  		voting_stack_visualizer  					replicated  1/1       	dockersamples/visualizer:stable
8lqj31k3q5ek  		voting_stack_redis       					replicated  1/1       	redis:alpine
nhb4igkkyg4y  		voting_stack_result      					replicated  1/1       	dockersamples/examplevotingapp_result:before
nv8d2z2qhlx4  		voting_stack_db          					replicated  1/1       	postgres:9.4
ou47zdyf6cd0  		voting_stack_vote        					replicated  2/2       	dockersamples/examplevotingapp_vote:before
rpnxwmoipagq  		voting_stack_worker      					replicated  1/1       	dockersamples/examplevotingapp_worker:latest
```

If you see that there are 0 replicas just wait a few seconds and enter the command again. The Swarm will eventually get all the replicas running for you. Just like our `docker-stack` file specified, there are two replicas of the *voting_stack_vote* service and one of each of the others.

Let's list the tasks of the vote service.

`	$ docker service ps voting_stack_vote	`

You should get an output like the following one where the 2 tasks (replicas) of the service are listed.

```batch
ID            		NAME                 IMAGE											NODE   			DESIRED STATE  			CURRENT STATE           ERROR  			PORTS
my7jqgze7pgg  		voting_stack_vote.1  dockersamples/examplevotingapp_vote:before  	node1  			Running        			Running 56 seconds ago
3jzgk39dyr6d  		voting_stack_vote.2  dockersamples/examplevotingapp_vote:before  	node2  			Running        			Running 58 seconds ago
```

From the NODE column, we can see one task is running on each node. This app happens to have a built-in [SWARM VISUALIZER](localhost:8080) to show you how the app is setup and running. You can also access the [front-end web UI](localhost:5000) of the app to cast your vote for dogs or cats, and track how the votes are going on the [result](localhost:8081) page. Try opening the front-end several times so you can cast multiple votes. You should see that the "container ID" listed at the bottom of the voting page changes since we have two replicas running.

The [SWARM VISUALIZER](localhost:8080) gives you the physical layout of the stack, but here is a logical interpretation of how stacks, services and tasks are inter-related:
![Stack, services and tasks](https://bitbucket.endava.com/projects/ENDAVA/repos/endava-docker-training/browse/swarm-mode/images/vote-app-infrastructure.svg)

## Scaling An Application

Let us pretend that our cats vs. dogs vote has gone viral and our two front-end web servers are no longer able to handle the load. How can we tell our app to add more replicas of our *vote* service? In production you might automate it through Docker's APIs but for now we will do it manually. You could also edit the `docker-stack.yml` file and change the specs if you wanted to make the scale size more permanent. Type the following at the [node1] terminal:

`	$ docker service scale voting-stack_vote=5	`

Now enter your `docker stack services voting_stack` command again. You should see the number of replicas for the vote service increase to 5 and in a few seconds Swarm will have all of them running. Go back to your [front-end voting UI](/){:data-term=""}{:data-port="5000"} - refresh the page a few times. You should see the *container ID* listed at the bottom cycle through all 5 of your containers. If you go back and refresh your [SWARM VISUALIZER](/){:data-term=""}{:data-port="8080"} you should see your updated architecture there as well.

Here's our new architecture after scaling:
![Swarm scaling](/images/ops-swarm-scale.svg)

## Raising a load-balancer

If at any point you will be interested in routing external traffic into the cluster, manage load balancing across all replicas and DNS discovery, then you will need a little bit of extra work outside of Docker Swarm. To leverage all of these features and use them all (to some extent), we will be using HaProxy. There are several other options out there, including nGinx and Traeffik. However, for the purposes of what we are working on, HaProxy is able to clearly evidence precisely what we are working on.

The first thing we will first have to do is remove the previous docker swarm stack.

`	$ docker stack rm vote	`

This should remove all services, containers and networks related to the vote-stack (as defined by the .yml file). Now before you create the stack again, you will need to make an external overlay network to attach services to.

`	$ docker network create --attachable --driver overlay frontend	`

The overlay type network is one that can relay traffic across multiple hosts. As we only have one node, there are no special requirements for the network to be usable. However, if we were scattered across multiple hosts, several requirements would have had to be taken into account:

 - TCP port 2377 for cluster management communications
 - TCP and UDP port 7946 for communication among nodes
 - UDP port 4789 for overlay network traffic

> Note!
> Before creating an overlay network, the docker node must either be initialized as swarm manager with the command `docker swarm init` or connect to an existing swarm using `docker swarm join`.

Next, we will need to spin up an instance that will act as the service we want to load balance. We can use the same voting app, but not in the same format. Instead, we will be using the yml found in *swarm-mode/load-balancer* folder.

`	$ docker stack deploy --compose-file=docker-stack.yml vote	`

This stack is slightly different than the previous one. It connects to the external overlay network and has several services that no longer have published ports. Instead in the yml, the argument *endpoint_mode dnsrr* in the yml file handles DNS Round Robin, which means that when we query Docker's Internal DNS server, we get the IP addressess of every container running the service.

In order to create the HaProxy service in Swarm, we will need a configuration file. Luckily we have one in *swarm-mode/load-balancer*. The _global_ section at the top contains several daemon specific lines, but also several docker-specific settings, such as: *fd@2 local2*, which provides the logs to Docker, and allow us to process them.

The second sections is _resolvers_. The Docker Swarm DNS service is always available at 127.0.0.11. So, this section configures HAProxy to direct DNS queries to there. It is essential for DNS-based service discovery.

Next, we have the _defaults_ section so that there are some sensible timeouts and other settings.

Lastly we have some frontend and backend sections. The configuration is very basic, but it essentially forwards request received on the front-end to the configured back-ends. Let's take a closer look at the backend section. 

- **server-template** - argument generates server lines based on the information it gets through DNS;
- **vote_result:80** - argument is the Swarm service name used for service discovery;
- **resolvers docker** - argument indicates to HAProxy which resolvers section to use;
- **init-addr libc,none** - argument tells HAProxy to perform service discovery at startup, but start even if there aren’t any running containers.

The following command will create a single HAProxy service:

```bash
$ docker service create \
--mode replicated --replicas 1 \
--name haproxy-service \
--network frontend \
--publish published=82,target=82,protocol=tcp,mode=ingress \
--publish published=83,target=83,protocol=tcp,mode=ingress \
--mount type=bind,src=/c/Repos/endava-docker-training/swarm-mode/load-balancer/,dst=/etc/haproxy/,ro=true \
--dns=127.0.0.11 haproxytech/haproxy-debian:2.0
```

The **publish** argument states that we want to use the native ingress routing mesh, specified with mode=ingress, which is the default mode of operation. The ingress routing mesh receives client requests on any node and relays them, transparently, to the correct node and container anywhere within the cluster.

The **--dns** argument is needed so that Docker sends DNS queries originating from this container to its own internal DNS server only. Without it, Docker will also forward queries to nameservers specified in the node’s resolv.conf file and to Google’s DNS nameservers.

Wait a few seconds, and then check to see if the service is running.

After creating your HAProxy service, take a look at the HAProxy logs with the following command:

`	$ docker service logs --tail 20 haproxy-service	`
