# Exercise 1: hello-world!

Try running the following command:

`	$ docker run hello-world (docker container run hello-world)	`

Your terminal output should look similar to this:

```bash
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
78445dd45222: Pull complete
Digest: sha256:c5515758d4c5e1e838e9cd307f6c6a0d620b5e07e6f927b07d05f6d12a1ac8d7
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
```

To generate the message, Docker followed the steps below:
 1. The Docker client contacted the Docker engine (daemon);
 2. The Docker engine (daemon) pulled the "hello-world" image from Docker Hub;
 3. The Docker engine (daemon) created a new container from the image, and ran the executable that produced the "Hello from Docker!" output;
 4. The Docker engine (daemon) streamed the output to the Docker client, which sent it to the terminal.
	
# Exercise 2: Working with Containers

Now that you have everything setup, it's time to try something a bit more complex. In this section, you're going to run a CentOS container and get a better understanding of the **docker run (docker container run)** command. To get started, let's first run the following command in our terminal:

` $ docker pull centos `

The **pull** command fetches the default CentOS **image** (centos:latest) from the **Docker Hub** registry and saves it on your system. To list all Docker Images on your system, you may use the following command:

` $ docker images (docker image ls) `

The output should look similarly to the one below:

```bash
  REPOSITORY              TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
  centos                  latest              c51f86c28340        4 weeks ago         487 MB
  hello-world             latest              690ed74de00f        5 months ago        960 B
```

## Running a container

Great! Now let's create a **container** based on the image. To do that you're going to use the **docker run (docker container run)** command.

` $ docker run centos ls -l (docker container run centos ls -l) `

```bash
total 56
-rw-r--r--   1 root root 12005 Aug  4 22:05 anaconda-post.log
lrwxrwxrwx   1 root root     7 Aug  4 22:04 bin -> usr/bin
drwxr-xr-x   5 root root   340 Sep  3 11:44 dev
drwxr-xr-x   1 root root  4096 Sep  3 11:44 etc
drwxr-xr-x   2 root root  4096 Apr 11 04:59 home
lrwxrwxrwx   1 root root     7 Aug  4 22:04 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Aug  4 22:04 lib64 -> usr/lib64
......
......
```

Let's try something with a different output:

`	$ docker run centos echo "Seriously!? Hello!" (docker container run centos echo "Seriously!? Hello!")	`

The result should obviously be:

```bash
  Seriously!? Hello!  
```

In this case, the Docker client ran the `echo` command in our CentOS container and then closed it. A Docker Container is up as long as its main process / command is running. Try another command:

`	$ docker run -it centos bash (docker container run -it centos bash)	`

When we use the -it arguments (-i comes from STDIN and -t stands for pseudo-tty, although its easier to consider that *-it* comes from interactive). Inside the container, you can run bash commands just like you would in a minimal CentOS. You can exit the container by using the `exit` command or the `CTRL+D` shortcut.

If we want to see all running containers, we may use the following command:

`	$ docker ps (docker container ls)	`

There should be no running containers, and that is because when we `exit` the container, the process was shut-down. 

```bash
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

To view all containers, running or otherwise, we may use:

`	$ docker ps -a (docker container ls -a)`

The output should be similar to the one below:

```bash
	CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
	36171a5da744        centos              "/bin/sh"                5 minutes ago       Exited (0) 2 minutes ago                        fervent_newton
	a6a9d46d0b2f        centos              "echo 'hello from "   	 6 minutes ago       Exited (0) 6 minutes ago                        lonely_kilby
	ff0a5c3750b9        centos              "ls -l"                  8 minutes ago       Exited (0) 8 minutes ago                        elated_ramanujan
	c317d0a9e3d2        hello-world         "/hello"                 34 seconds ago      Exited (0) 12 minutes ago                       stupefied_mcclintock
```

What you see above is a list of all containers that you ran. Notice that the `STATUS` column shows that these containers exited a few minutes ago. Try using the `run` command again with the `-it` flag, so it attaches you to an interactive tty in the container. (Remember, you can write `exit` when you want to quit.)

## Naming a container

Let's check output of `docker ps -a (docker container ls -a)` again:

```bash
	CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
	36171a5da744        centos              "/bin/sh"                5 minutes ago       Exited (0) 2 minutes ago                        fervent_newton
	a6a9d46d0b2f        centos              "echo 'hello from alp"   6 minutes ago       Exited (0) 6 minutes ago                        lonely_kilby
	ff0a5c3750b9        centos              "ls -l"                  8 minutes ago       Exited (0) 8 minutes ago                        elated_ramanujan
	c317d0a9e3d2        hello-world         "/hello"                 34 seconds ago      Exited (0) 12 minutes ago                       stupefied_mcclintock
```

All containers have an **ID** and a  randomly generated **name**. If you want to assign a specific name to a container then you can use the `--name` argument. That can make it easier for you to reference the container going forward. (e.g. **docker run --name container-name image-name** )

## Exercise 3: Exposed and Published ports

Running arbitrary Linux commands inside a Docker container might be fun, but let's try something a bit more useful. 

First, let's pull the `nginx` Docker image from Docker Hub. This Docker image uses the **Nginx** (http://nginx.org/) webserver to serve a website. Start a new container from the `nginx` image that exposes port 80 from the container and maps it to the published port 9000 on your host. You will need to use the `-p` flag with the docker container run command.

> NOTE:
> Mapping ports between your host machine and your containers can get confusing. Here is the syntax you will use:

`	$ docker container run -p 9000:80 nginx	`

> The trick is to remember that **the published host port always goes to the left**, and **the exposed container port always goes to the right.** Remember it as traffic coming **from** the host, **to** the container.

Open a web browser and go to port 9000 on your host.

* **Native Linux** - http://localhost:9000

If you see a webpage saying **"Welcome to nginx!"** then you're done!

If you look at the console output from docker, you see nginx producing a line of text for each time a browser hits the webpage:

`	$ docker run -p 9000:80 nginx (docker container run -p 9000:80 nginx)	`

```bash
	172.17.0.1 - - [31/May/2017:11:52:48 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:53.0) Gecko/20100101 Firefox/53.0" "-
```

Press **control + c** in your terminal window to exit your container.

If you intend to run a webserver in the background, thus freeing the terminal for other things, you can use the `-d` (detached) parameter.

` $ docker container run -p 9000:80 -d nginx  `

The output should only contain the container ID, then return to the CMD / Powershell / bash terminal.

```bash
	78c943461b49584ebdf841f36d113567540ae460387bbd7b2f885343e7ad7554
```

## Attach & execute to running containers

If you want to go into a container again to execute something you have two options:

- `attach`  Attach to the specified process running on the container. In our example this would be the nginx server.
- `exec` Executing another process inside the container. This could be a shell, or a script of some sort.

> NOTE:
> If the container was started using /bin/bash command, you can access it using attach, if not then you need to execute the command to create a bash instance inside the container
> When you attach to an already started container, you cannot exit normally without killing the container unless you issue the attach command like this:

`	$	docker attach --sig-proxy=false CONTAINER-ID (docker container attach --sig-proxy=false CONTAINER-ID)	` or by using the shortcut `Ctrl+p+Ctrl+q` inside the container

> If you issue the `exec` command, you can stop it by `Ctrl+d` (this will also turn off the container) or detach yourself by `Ctrl+p+Ctrl+q`

**Attach to a running process**

First fire up a new Nginx container (if you don't have it running already):

`	$ docker run -d -p 9000:80 nginx (docker container run -d -p 9000:80 nginx)	`

Then, try to attach to the container using the following command:

`	$ docker attach CONTAINER-ID	`

Exit it then browse the webpage again (http://localhost:9000) to acknowledge it is gone. The container should be stopped. Start it up again with `docker start nginx` command.

**Running a new process instance**

Step into the container by executing a bash inside the container:

`	$ docker exec -it CONTAINER-ID bash (docker container exec -it CONTAINER-ID bash)	`

Inside, we want to run a long running process inside the container, such as a self-ping for 500 times. Because containers only have the bare minimum installed, we first need to install ping, and then use it:

```bash
  $	apt-get update && apt-get install iputils-ping -y
  $	ping 127.0.0.1 -c 500 > /tmp/ping
```

Detach from the container with `Ctrl+p+Ctrl+q`. then execute into the container again (using the **docker exec -it** command we used prior) and run the following:

```bash
$	tail -f /tmp/ping
	64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.156 ms
	64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.183 ms
	64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.158 ms
```

Here you see that the ping process started in another shell is still running and producing this logfile. Just stop the process with `Ctrl+d`.



