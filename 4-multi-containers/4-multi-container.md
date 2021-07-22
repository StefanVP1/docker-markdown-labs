# Multi container setups

This scenario handles multiples containers working simultaneously on the same host. You will be working with a multi-service application, and show several ways in which you can get the containers to work together. For this scenario, you are going to build a Wordpress site using Docker.

> WordPress is a free and open source CMS based on PHP and MySQL, which usually runs on a youb hosting service.

You will need two containers:

- One container to serve the Wordpress page;
- One container to serve as a MySQL database;.

Both containers already exists on the dockerhub: [Wordpress](https://hub.docker.com/_/wordpress/) and [Mysql](https://hub.docker.com/_/mysql/).

## Individual containers

To start the mysql container, you can use the command:

`	$ docker run --name mysql-container --rm -p 3306:3306 -e MYSQL_ROOT_PASSWORD=wordpress -d mysql:5.7	`

MySQL is now exposed on port 3306 on the host, and everyone can attach to it.

You need to connect the wordpress container to the host's IP address. You can get the IP by checking the network status or using the  the `ipconfig` command in a CMD terminal.

After you have noted down the ip, spin up the wordpress container with the host ip as a variable:

`	$ docker run --name wordpress-container --rm -e WORDPRESS_DB_HOST=<YOURIP> -e WORDPRESS_DB_PASSWORD=wordpress -p 8090:80 -d wordpress	`

You can now browse to the localhost:8090 (or 127.0.0.1:8090) and have the own wordpress server running.

## Making a container network

Even though you managed to get the setup running in only two commands, there are some problems here you can fix:

- You need to know the host IP to get them to talk to each other.
- And you have exposed the database to the outside world.

In order to connect multiple docker containers without binding them to the hosts network interface you need to create a docker network.

The `docker network` command securely connect and provide a channel to transfer information from one container to another.

First off make a new network for the containers to communicate through:

`	$ docker network create test_wordpress	`

Docker will return the networkID for the newly created network. You can reference it by name as youll as the ID.

Now you need to connect the two containers to the network, by adding the `--network` option:

`	$ docker run --name mysql-container --rm --network test_wordpress -e MYSQL_ROOT_PASSWORD=wordpress -d mysql:5.7	`
as well as:
`	$ docker run --name wordpress-container --rm --network test_wordpress -e WORDPRESS_DB_HOST=mysql-container -e WORDPRESS_DB_PASSWORD=wordpress -p 8090:80 -d wordpress	`

Notice the `WORDPRESS_DB_HOST` env variable. When you make a container join a network, it automatically gets the container name as DNS name as well, making it easy to get containers to discover each other.

You have now deployed both containers into the network. Take a deeper look into the container network by issuing: `docker network inspect if_wordpress`.

```bash
docker network inspect test_wordpress
[
    {
        "Name": "test_wordpress",
        "Id": "04e073137ff8c71b9a040ba452c12517ebe5a520960a78bccb7b242b723b5a21",
        "Created": "2017-11-28T17:20:37.83042658+01:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "Containers": {
            "af38acac52301a7c9689d708e6c3255704cdffb1972bcc245d67b02840983a50": {
                "Name": "mysql-container",
                "EndpointID": "96b4befec46c788d1722d61664e68bfcbd29b5d484f1f004349163249d28a03d",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "fb4dad5cd82b5b40ee4f7f5f0249ff4b7b4116654bab760719261574b2478b52": {
                "Name": "wordpress-container",
                "EndpointID": "2389930f52893e03a15fdc28ce59316619cb061e716309aa11a2716ef09cde17",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

As, you have linked both the container now wordpress container can be accessed from browser using the address http://localhost:8080 and setup of wordpress can be done easily. MySQL is not accessible from the outside so security is much better than before.

Close both of the containers down by issuing the following command:

`	$	docker stop wordpress-container mysql-container(docker container stop wordpress-container mysql-container)`

## Using Docker compose

If you have started working with Docker and are building container images for ythe application services, you most likely have noticed that after a while you may end up writing long `docker run (docker container run)` commands.
These commands, while very intuitive, can become cumbersome to write, especially if you are developing a multi-container applications and spinning up containers quickly. To help ease this process, you may use:

[Docker Compose](https://docs.docker.com/compose/install/) is a “*tool for defining and running the multi-container Docker applications*”.

The applications can be defined in a YAML file where all the options that you used in `$ docker run (docker container run)` are defined.

Compose also allows you to manage the application as a single entity rather than dealing with individual containers. This file defines all of the containers and settings you need to launch the set of clusters. The properties map onto how you use the docker run commands, hoyouver, are now stored in sthece control and shared along with the code.

The docker cli is used when managing individual containers on a docker engine. It is the client command line to access the docker daemon api.

The docker-compose cli together with the yaml files can be used to manage a multi-container application.

So you want to take advantage of docker-compose to run the wordpress site.

In order to to this you need to:

1. Transform the setup into a docker-compose.yaml file
1. Invoke docker-compose and watch the magic happen!

Head over to this labs [/multi-container](./multi-container/) folder:

Open the file `docker.compose.yaml` with a text editor. You should see something like this:

```yaml
version: '3.1'

services:

#  wordpress_container:

  mysql_container:
    image: mysql:5.7
    ports:
      - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: wordpress
```

This is the template you are building the compose file upon so let's drill this one down:

- `version` indicate what version of the compose syntax you are using
- `services` is the section where you put the containers
  - `wordpress_container` is the section where you define the wordpress container
  - `mysql_container` is the ditto of MySQL.

> For more information on docker-compose yaml files, head over to the [documentation](https://docs.docker.com/compose/overview/).

The `services` part is equivalent to the `docker run` command. Likewise there is a `network` and `volumes` section for those as well corresponding to `docker network create` and `docker volume create`.

Let's look the mysql_container part together, making you able to create the other container theself. Look at the original command you made to spin up the container:

`	$ docker run --name mysql-container --rm -p 3306:3306 -e MYSQL_ROOT_PASSWORD=wordpress -d mysql `

The command gives out following information: a `name`, a `port` mapping, an `environment` variable and the `image` you want to run.

Now look at the docker-compose example again:

- `mysql_container` defines the name of the container
- `image:wordpress` describes what image the container spins up from.
- `ports` defines a list of port mappings from host to container
- `environment` describes the `-e` variable made before in a yaml list

Try to spin up the container in detached mode:

```bash
$ docker-compose up -d
	Creating network "multicontainer_default" with the default driver
	Creating multicontainer_mysql_container_1 ...
	Creating multicontainer_mysql_container_1 ... done
```

Looking at the output you can see that it made a `docker network` named `multicontainer_default` as well as the MySQL container named `multicontainer_mysql_container_1`. Issue a `docker ps (docker container ls)` as well as `docker network ls` to see that both the container and network are listed.

To shut down the container and network, issue a `docker-compose down`

> **note**: The command `docker-compose down` removes the containers and the default network.

## Creating the wordpress container

You now have all the pieces of information to make the Wordpress container. You copied the run command from before if you cant remember it by heart.

`	$	docker run --name wordpress-container --rm --network if_wordpress -e WORDPRESS_DB_HOST=mysql-container -e WORDPRESS_DB_PASSWORD=wordpress -p 8090:80 -d wordpress	`

You must

- un-comment the `wordpress_container` part of the services section
- map the pieces of information from the docker container run command to the yaml format.
- remove MySQL port mapping to close that from outside reach.

When you made that, run `docker-compose up -d` and access the wordpress site from http://localhost:8090

> **Hint**: If you are stuck, look at the file docker-compose_final.yaml in the same folder.
