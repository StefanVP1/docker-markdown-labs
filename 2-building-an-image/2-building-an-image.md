# Constructing a docker image

Docker has plenty of resources to run ready-made containers, however if you want to use docker for more personalized use-cases, chances are you will need to construct an image on your own. For that we would need to first build a template, known as a ***Dockerfile***. 

## Dockerfile commands summary

The Dockerfile [commands](https://docs.docker.com/engine/reference/builder/) you write are *almost* identical to their equivalent Linux commands (with some small exceptions we will cover). This means you don't have to learn any new syntax to create Dockerfiles.

Here's a quick summary of the most basic commands we will use in writing our own Dockerfile.

> As a rule of thumb, all commands in CAPITAL LETTERS are intended for the docker engine.

- FROM
- RUN
- ADD and COPY
- CMD
- EXPOSE
- ENTRYPOINT

* `FROM` is always the first functional in the Dockerfile. It is a requirement that the Dockerfile starts with the `FROM` command. Since Images are built in layers, you can use another image as the base layer for your own. The `FROM` command defines said base layer.

* `RUN` is used to build up the image you're creating. For each `RUN` command, Docker will run the command then create a new layer of the image. This way you can roll back your image to previous states easily. The syntax for a `RUN` instruction is to place the full text of the shell command after the `RUN` (e.g., `RUN mkdir /user/local/foo`). This will automatically run in a `/bin/sh` shell.

* `COPY` copies local files into the container. The files need to be in the same folder (or a sub folder) as the Dockerfile itself. An example is copying the requirements for a python app into the container: `COPY requirements.txt /usr/src/app/`.

* `CMD` defines the default commands that will be executed when you run the container without specifying a command. If the container runs with a command, then the CMD will be ignored. If Dockerfile has more than one CMD instruction, all but last CMD instructions are ignored. An example of `CMD` commands would be:

```
  CMD ["python", "./app.py"]
  CMD ["/bin/bash", "echo", "Hello World"]
```

* `EXPOSE` instruction will actually expose the port internally, but it will not publish it to the host / network. You can view exposed ports by using the command: `$ docker inspect <container-id> (docker container inspect <container-id>)`.

> **Note:** The `EXPOSE` command does not make any ports accessible to the host! Instead, this requires publishing ports by means of the `-p` or `-P` flag when using `$ docker run (docker container run)`.

* `ENTRYPOINT` configures a command that will run no matter what the user specifies at runtime. These are not ignored when the Docker container runs with command line parameters. Syntax wise, it is very similar to the CMD variable. An example of the ENTRYPOINT instruction may look identical to the CMD:

```
ENTRYPOINT ["/bin/bash", "echo", "Hello World"]
```

Entrypoint can also be used in conjunction with CMD to set additional commands and parameters that are more likely to be changed. ENNTRYPOINT arguments are always used, but the CMD ones can be overwritten via command-line. For example:

```
ENTRYPOINT ["/bin/echo", "Hello"]
CMD ["world!"]
```

when the container is run with `docker run -it <image>`, it will produce the standard *Hello world!* output. However, if we run `docker run -it <image> Timmy!`, the result is *Hello Timmy!*.

## Write a Dockerfile

We are about to create a Docker Image with a small flask Python app. As mentioned above, all user images are based on a _base image_. Since our application is written in Python, we will build our own Python image based on [Ubuntu](https://hub.docker.com/_/ubuntu/). We'll do that using a **Dockerfile**.

> **Note:** If you want to learn more about Dockerfiles, check out the [Best practices for writing Dockerfiles](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

1. Create a blank file called **Dockerfile**, and add content to it as described below. We have already added a small file and app for you in the [/building-an-image](./building-an-image/) folder, so head on over there.

We'll start by specifying our base image, using the `FROM` keyword:

```
FROM ubuntu:latest
```

2. The next step is usually to write the commands of copying the files and installing the dependencies. We'll first install the Python pip package, and other dependencies, including the python interpreter. 

```
RUN apt-get update -y
RUN apt-get install -y python-pip python-dev build-essential
```

3. Let's add the files that make up the Flask Application.

Install all Python requirements for our app to run. This will be accomplished by adding the lines:

```
COPY requirements.txt /usr/src/app/
RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt
```

Copy the application app.py into our image by using the [COPY](https://docs.docker.com/engine/reference/builder/#copy) command.

```
COPY app.py /usr/src/app/
```

4. Specify the port number which needs to be exposed. Since our flask app is running on `5000` that's what we'll expose.

```
EXPOSE 5000
```

NOTE:

> The `EXPOSE` instruction does not actually publish the port. Instead it just exposes the port inside the container. You need the `-p`/`-P` command to actually publish the port to the host / network.

5. The last step is the command for running the application which is to simply use a CMD with python as an executable:

```
CMD ["python", "/usr/src/app/app.py"]
```

6. Verify your Dockerfile.

Your `Dockerfile` is now ready. It should look like the one below:

```
# The base image
FROM ubuntu:latest

# Install python and pip
RUN apt-get update -y
RUN apt-get install -y python-pip python-dev build-essential

# Install Python modules needed by the Python app
COPY requirements.txt /usr/src/app/
RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt

# Copy files required for the app to run
COPY app.py /usr/src/app/

# Declare the port number the container should expose
EXPOSE 5000

# Run the application
CMD ["python", "/usr/src/app/app.py"]
```

## Build the image

Now that you have your `Dockerfile`, you can build your image. The `docker build` command does the heavy-lifting of creating a docker image from a `Dockerfile`.

The `docker build` command is quite simple - it takes an optional tag name with the `-t` flag, and the location of the directory containing the `Dockerfile` - the `.` indicates the current directory:

`	$ docker build -t my-python-app .	`

```bash
Sending build context to Docker daemon   5.12kB
Step 1/8 : FROM ubuntu:latest
latest: Pulling from library/ubuntu
b6f892c0043b: Pull complete
55010f332b04: Pull complete
2955fb827c94: Pull complete
3deef3fcbd30: Pull complete
cf9722e506aa: Pull complete
Digest: sha256:382452f82a8bbd34443b2c727650af46aced0f94a44463c62a9848133ecb1aa8
Status: Downloaded newer image for ubuntu:latest
 ---> ebcd9d4fca80
Step 2/8 : RUN apt-get update -y
 ---> Running in 42d5752a0faf
Get:1 http://archive.ubuntu.com/ubuntu xenial InRelease [247 kB]
Get:2 http://security.ubuntu.com/ubuntu xenial-security InRelease [102 kB]
...
Get:20 http://archive.ubuntu.com/ubuntu xenial-backports/main amd64 Packages [4927 B]
Get:21 http://archive.ubuntu.com/ubuntu xenial-backports/universe amd64 Packages [6022 B]
Fetched 24.0 MB in 6s (3911 kB/s)
Reading package lists...
 ---> 07205bd484c9
Removing intermediate container 42d5752a0faf
Step 3/8 : RUN apt-get install -y python-pip python-dev build-essential
 ---> Running in 43e6e25b8c6b
Reading package lists...
Building dependency tree...
Reading state information...
The following additional packages will be installed:
  binutils bzip2 ca-certificates cpp cpp-5 dpkg-dev fakeroot file g++ g++-5
...
  python2.7-minimal rename xz-utils
0 upgraded, 87 newly installed, 0 to remove and 3 not upgraded.
Need to get 85.4 MB of archives.
After this operation, 268 MB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu xenial/main amd64 libatm1 amd64 1:2.5.1-1.5 [24.2 kB]
Get:2 http://archive.ubuntu.com/ubuntu xenial/main amd64 libmnl0 amd64 1.0.3-5 [12.0 kB]
Get:3 http://archive.ubuntu.com/ubuntu xenial/main amd64 libgdbm3 amd64 1.8.3-13.1 [16.9 kB]
...
Get:86 http://archive.ubuntu.com/ubuntu xenial/universe amd64 python-wheel all 0.29.0-1 [48.0 kB]
Get:87 http://archive.ubuntu.com/ubuntu xenial/main amd64 rename all 0.20-4 [12.0 kB]
debconf: delaying package configuration, since apt-utils is not installed
Fetched 85.4 MB in 24s (3514 kB/s)
Selecting previously unselected package libatm1:amd64.
(Reading database ... 4764 files and directories currently installed.)
Preparing to unpack .../libatm1_1%3a2.5.1-1.5_amd64.deb ...
Unpacking libatm1:amd64 (1:2.5.1-1.5) ...
...
Selecting previously unselected package python-wheel.
Preparing to unpack .../python-wheel_0.29.0-1_all.deb ...
Unpacking python-wheel (0.29.0-1) ...
Selecting previously unselected package rename.
Preparing to unpack .../archives/rename_0.20-4_all.deb ...
Unpacking rename (0.20-4) ...
...
Setting up python-setuptools (20.7.0-1) ...
Setting up python-wheel (0.29.0-1) ...
Setting up rename (0.20-4) ...
update-alternatives: using /usr/bin/file-rename to provide /usr/bin/rename (rename) in auto mode
Processing triggers for libc-bin (2.23-0ubuntu7) ...
Processing triggers for systemd (229-4ubuntu17) ...
Processing triggers for ca-certificates (20160104ubuntu1) ...
Updating certificates in /etc/ssl/certs...
173 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```

If everything went well, your image should be ready! Run `docker images (docker image ls)` and see if your image (`my-python-app`) is present.

## Run your image

The next step in this section is to run the image and see if it actually works.

```bash
$ docker run -p 8888:5000 --name myfirstapp my-python-app (docker container run -p 8888:5000 --name myfirstapp my-python-app)
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

Head over to `http://localhost:8888` or your server's URL and your app should be live.

When dealing with docker images, a layer, or image layer is a change on an image, or an intermediate image. Every command you specify (FROM, RUN, COPY, etc.) in your Dockerfile causes the previous image to change, thus creating a new layer.

Consider the following Dockerfile:

```
FROM ubuntu:latest
RUN apt-get update -y
RUN apt-get install -y python-pip python-dev build-essential
COPY requirements.txt /usr/src/app/
RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt
COPY app.py /usr/src/app/
EXPOSE 5000
CMD ["python", "/usr/src/app/app.py"]
```

First, we choose the base image: `ubuntu:latest`, which in turn has its own set of layers. We add another layer on top, running an apt-get update on the system. After that another one for installing the python ecosystem. Then, we tell docker to copy the requirements to the container. That's an additional layer.

The concept of layers comes in handy at the time of building images. Because layers are intermediate images, if you make a change to your Dockerfile, docker will build only the layer that was changed and the ones after that. This is called layer caching.

Each layer is build on top of it's parent layer, meaning if the parent layer changes, the next layer does as well.

If you want to concatenate two layers (e.g. the update and install (https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#run)), then do them in the same RUN command:

```
FROM ubuntu:latest
RUN apt-get update && apt-get install -y \
 python-pip \
 python-dev \
 build-essential
COPY requirements.txt /usr/src/app/
RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt
COPY app.py /usr/src/app/
EXPOSE 5000
CMD ["python", "/usr/src/app/app.py"]
```

If you want to be able to use any cached layers from last time, they need to be run _before the update command_.

> NOTE:
> Once we build the layers, Docker will reuse them for new builds. This makes the builds much faster. This is great for continuous integration, where we want to build an image at the end of each successful build (e.g. in Jenkins). But the build is not only faster, the new image layers are also smaller, since intermediate images are shared between images.

Try to move the two `COPY` commands before for the `RUN` and build again to see it taking the cached layers instead of making new ones.

## Image layer structure

As stated above, all FROM, RUN, ADD, COPY, CMD and EXPOSE will create a new layer in your image, and therefore also be an image of their own.

Take a look again at some of the output from building the image above:

```bash
 ---> c1f2dc732c7c
Removing intermediate container f92f9c719287
Step 6/8 : COPY app.py /usr/src/app/
 ---> 6ed47d3c544a
Removing intermediate container 61a68a949d68
Step 7/8 : EXPOSE 5000
 ---> Running in 1f939928b7d5
 ---> 6c14a93b72f2
```

So what docker actually does is

- Taking the layer created just before;
- make a container based of it;
- run the command given;
- save the layer.

in a loop untill all the commands have been made.
Try to create a container from your `COPY app.py /usr/src/app/` command.
The id of the layer will likely be different than the example above.

`docker run -it -p 5000:5000 6ed47d3c544a bash`.

You are now in a container run from _that_ layer in the build script. You can't make the `EXPOSE` command, but you can look around, and run the last python app:

```bash
root@cc5490748b2a:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

root@cc5490748b2a:/# ls /usr/src/app/
app.py  requirements.txt

root@cc5490748b2a:/# python /usr/src/app/app.py
* Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

And just like the image you builded above, you can browse the website now.

## Delete your image

If you make a `docker ps -a (docker container ls -a)` command, you can now see a container with the name *myfirstapp* from the image named *my-python-app*.

```bash
sofus@Praq-Sof:/4$ docker container ls -a
CONTAINER ID        IMAGE                     COMMAND                  CREATED              STATUS                      PORTS                                                          NAMES
fcfba2dfb8ee        myfirstapp                "python /usr/src/a..."   About a minute ago   Exited (0) 28 seconds ago                                                                  my-python-app
```

Make a `docker images (docker image ls)` command to see that you have a docker image with the name `my-python-app`

Try now to first:

- remove the container (`docker rm`)
- remove the image file as well with the ` docker rmi (docker image rm)` [command](https://docs.docker.com/engine/reference/commandline/image_rm/).
- run `docker images (docker image ls)` again to see that it's gone.
