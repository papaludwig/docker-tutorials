# Part 1 - Jenkins & Docker

In this example we will run Jenkins from a container, and, because we will share a volume that includes access to docker.sock, our new container will be able to communicate with the host docker engine to spin up containers on the host.


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

1. Go to the Jenkins homepage for your container at http://FQDN:8080/ (where FQDN is the fully qualified domain name for your VM machine that your instructor has given you).
2. You will be prompted for a one-time administrators password. Jenkins writes this password to the system log while it is being installed. Learning how to examine a container's logs is a great exercise as many future docker issues can be resolved this way.
3. Retrieve the jenkins container id with docker container ls

```
docker container ls
```

4. Pass in the container id to docker logs and scan for the creation of an admin password.

```
docker logs <container id>
```

5. If you don't see the password, it might be that Jenkins is still installing and running for the first time in your container. Keep trying and you should see that the log is getting longer. The password will eventually show up and will be a long hexadecimal string.


## Set up a job in Jenkins that runs a new Docker container

1. Once Jenkins has complete it's internal setup with the one-time password, it will start the SetupWizard.
2. Click "Install Recommended Plugins" to allow the recommended plugins to be installed (including the pipeline plugin that we'll be using later). Remember that all this "action" is happening within the container.
3. Create your first admin user... use admin as the username and DockerStudentPW0 as the password. You can use any email address. Click "Save and Finish".
4. Once the Jenkins dashboard appears, click the “create new jobs” link.
5. Enter the item name (e.g. “docker-test”), select “Freestyle project” and click OK.
6. On the configuration page in the *Build* section, click “Add build step” then “Execute shell”.
7. In the command box enter “sudo docker run hello-world”
8. Click “Save”.
9. Click “Build Now” in the left-side nav.

With luck, you have triggered a successful docker hello world. On your host
docker VM, run

```
docker container ls -a
```

To verify that the Docker hello world image was indeed run on the host computer.

Next, practice using creating a persistent volume to share between Jenkins instances in [part 2](https://github.com/papaludwig/docker-tutorials/sharedvolume)
