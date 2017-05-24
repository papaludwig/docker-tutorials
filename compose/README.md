# Part 4 - Summarize and cleanup your Docker/Jenkins implementation

Now that Jenkins is containerized and working, it would be an ideal time to
optimize a few details.

# Shorten startup command

The startup command for docker is either allot of typing and a long string
prone to errors when using control-paste. We can shorten the arguments passed
to Docker with a docker-compose yml.

Current Docker run command

```  
docker run -d -v /var/run/docker.sock:/var/run/docker.sock \
              -v $(which docker):/usr/bin/docker -p 8080:8080 -u root \
              jenkinsdocker
```

We can summarize the volumes, port and user with a docker compose yml. We can
start with the docker compose template and quickly develop the following:

```
version: '2'
services:
  jenkinsc:
    build: .
    user: "root"
    ports:
      - 8080:8080
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker
```

I did have to convert the $(which docker) cmd to the path on my Ubuntu VM
(/usr/bin/docker).

# Bypass Jenkins setup

If you would like to fully bypass the admin password, you can add this flag as
well so that the Jenkins setup is fully automated. Note that without further
work to add credentials the Jenkins container is wide open.

```
environment:
  JAVA_OPTS: "-Djenkins.install.runSetupWizard=false"
```  

# Run the Jenkins instance

Now we can create our Jenkins container with a call to docker-compose.

```
docker-compose up -d
```

We still have to pass in the -d flag if we wish it to run a daemon.

# Test

Make sure the Jenkins container performs as you expect. If everything is working
as expected, congratulations on standing up a working continuous integration
proof of concept!
