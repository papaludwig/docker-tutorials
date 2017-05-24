# Part 2 - Persistent Docker Volumes shared between Jenkins instances

In this example we will clarify how to share a persistent volume between two or
more instances of Jenkins. Walking though this helps clarify syntax around
Docker Volume management.

# Create a Named Volume

The first step is to create a named volume for use by the future Jenkins
containers.

```
docker create -v /var/jenkins_home --name jenkinstore library/jenkins
```

This creates a directory volume in /var/jenkins_home with the name jenkinstore
and shares with the /library/jenkins image so that container layers are common
which will save disk space.

# Reference the Name Volume from another container instance of Jenkins

The volumes-from flag enables the jenkins data from /var/jenkins_home to be
referenced in another Jenkins container

```
docker run --volumes-from jenkinstore --name jenkin1 -p 8080:8080 -p
           \ 50000:50000  library/jenkins
```

Walk through setting up the Jenkins instance by setting up a new user account
and the typical plugins until Jenkins the Jenkins launch wizards are completed
and you are at the home screen.

# Verify state in shared volume

Shut down the Jenkins container and start a new Jenkins container from the
shared volume to verify the state is preserved between the two containers.

1. Shut down the original running Jenkins container with Ctl-C or a docker stop
command if the instance is running as a daemon.

2. Create a second Docker container referencing the persistent volume.

```
docker run --volumes-from jenkinstore --name jenkin2 -p 8080:8080 -p 50000:50000
           \ library/jenkins
```

Once this container is running, you should observe that Jenkins is already setup
with the plugins you just installed with the other container and has the login
user from the prior container as well. 

Next, practice creating a running continous intergration example in [part 3](https://github.com/PeterLamar/docker-workshop/tree/master/ciexample)
