# Part 1 - Jenkins & Docker

In this example we will run Jenkins from a container, which will in turn spin up containers on the host computer.


## Create a directory that will contain our image's files and the dockerfile

```
cd ~
mkdir jenkinsdocker
cd jenkinsdocker
```


## Create a new dockerfile

```
nano dockerfile
```


## Start with the official Jenkins container

The official Jenkins container can be found with a quick search of Docker hub. We can use the official Jenkins image as our starting point. We will specify a version but you could just as easily specify or default to the latest version. We use the dockerfile FROM directive to specify our base image.

```
FROM jenkins:2.19.3
```


## Place the jenkins user in the sudoers file

We want Jenkins to have root priviledges because it will be doing many system-level things (including running docker as part of our pipeline).

This would of course be considered a security vulnerability in a production setting, but we're keeping our examples deliberately simple to concentrate on learning Docker.

We use the dockerfile USER directive to specify that we want the following commands to be executed by the root user. We then add the sudo command by using the RUN directive to run apt-get.

Finally we add the jenkins user to the list of sudoers by using the RUN directive to echo out the appropriate line to the appropriate file.

```
USER root
RUN apt-get update \
      && apt-get install -y sudo \
      && rm -rf /var/lib/apt/lists/*
RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers
```
Press ctrl-x to exit nano, press y to save, and press return to accept the original "dockerfile" file name.


## Build our docker image

Building a docker image using the dockerfile we just wrote is straightforward. This will result in a newly built local docker image. The following command assumes you are in the same directory as the dockerfile.

```
docker build -t jenkinsdocker .
```


## Run the docker image

Once the docker run command is run, with luck we will have a local jenkins server that can
spawn local docker containers. This server will be available at http://localhost:8080/ or <ip>:8080 if you are running this from a cloud VM.

```
docker run -d -v /var/run/docker.sock:/var/run/docker.sock \
              -v $(which docker):/usr/bin/docker -p 8080:8080 jenkinsdocker
```


## Navigate to Jenkins and setup the instance

Once you go to the Jenkins homepage available at http://localhost:8080/, you will be prompted for a administrators password. This can be found in the logs of the Jenkins container while it was setting up. Furthermore, examining the logs is a great exercise as many future docker issues can be resolved this way.

First retrieve the container id with docker container ls

```
docker container ls
```

Next, pass in the container id to docker logs and scan for the creation of an admin password.

```
docker logs <container id>
```

This password will enable you to proceed with setting up Jenkins.


## Run a docker command from the Jenkins console

1. Open the Jenkins home page in a browser and click the “create new jobs” link.
2. Enter the item name (e.g. “docker-test”), select “Freestyle project” and click OK.
3. On the configuration page, click “Add build step” then “Execute shell”.
4. In the command box enter “sudo docker run hello-world”
5. Click “Save”.
6. Click “Build Now”.

With luck, you have triggered a successful docker hello world. On your host
docker VM, run

```
docker container ls -a
```

To verify that the Docker hello world image was indeed run on the host computer.

Next, practice using creating a persistent volume to share between Jenkins instances in [part 2](https://github.com/PeterLamar/docker-workshop/tree/master/sharevolume)
