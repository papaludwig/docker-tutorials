# Part 2 - Persistent Docker Volumes shared between Jenkins instances

In this example we will clarify how to share a persistent volume between two or more containers -- which in this case will have individual instances of Jenkins. Walking though this helps clarify syntax around Docker Volume management, and shows how to attach volumes directly to local containers.


# Create Container / Shared Volume

This first step creates a container based on the library/jenkins image, calls it jenkinstore, and simultaneously creates (or attaches if it exists already) a shared volume (living locally at /var/jenkins_home). Creating the volume simultaneously with a container based on library/jenkins image will cause further containers based on the library/jenkins image to share layers (saving disk space).

```
docker create -v /var/jenkins_home --name jenkinstore library/jenkins
```
Notice that we are not sharing any ports from this container. This will not be a running container, it exists soley as a placeholder for the shared volume.


# Reference the Named Volume from Another Jenkins Container Instance

The volumes-from flag enables the shared volume (jenkinstore) which will contain jenkins data from the host's /var/jenkins_home directory to be referenced in this new container based on library/jenkins.

```
docker run --volumes-from jenkinstore --name jenkin1 -p 8080:8080 -p 50000:50000 library/jenkins
```

Once again, walk through setting up the Jenkins instance by grabbing password from the new container's log, setting up a new admin user account, and installing the recommended plugins and you are at the home screen.


# Verify State in Shared Volume

Shut down the Jenkins container and start a new Jenkins container from the shared volume to verify the state is preserved between the two containers.

1. Shut down the original running Jenkins container with Ctl-C or a docker stop command if the instance is running in detached mode.

2. Create a second Docker container referencing the persistent volume.

```
docker run --volumes-from jenkinstore --name jenkin2 -p 8081:8080 -p 50000:50000 library/jenkins
```

Once this new container is running, you should observe that Jenkins is already setup with the plugins you just installed with the other container and has the login user from the prior container as well. 

Next, we will practice creating a running continous intergration example in [part 3](https://github.com/PeterLamar/docker-workshop/tree/master/ciexample)
