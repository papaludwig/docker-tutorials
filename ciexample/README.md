# Part 3 - Setup CI with sample pipeline in Jenkins/Docker

Lets go ahead and setup a sample pipeline in Jenkins/Docker. This will give
us a good working starting point for more advanced activity. We will fork a
simple already built pipeline on Github and adapt it to work on our 
Jenkins Container implementation

# Clone sample project

We will start by forking 
https://github.com/russmckendrick/jenkins-docker-example. You should have a 
copy of the repo in your own github account.

# Add Docker-Pipeline to plugins

The first step is to add the Jenkins Docker-Pipeline plugin. We could add this 
every time we start Jenkins from the GUI, but automating the process saves us 
the manual effort. We will need the Docker-Pipeline if we are to build Docker
from Jenkins.

Please add to the end of the Dockerfile

```
/usr/local/bin/install-plugins.sh docker-workflow
```

# Install Docker Compose in Dockerfile

Since our example requires Docker-Compose, we will need to install Docker 
Compose within our Dockerfile. There are two common ways to install Docker 
Compose, the first is with a curl script and the second is through Pip. Since
doing so with a curl script in our Dockerfile tends to be interfered with by 
proxy's lets elect to use the pip method. 

We will add the following lines to our Dockerfile

```
RUN apt-get install -y sudo python-pip 
RUN pip install docker-compose
```

# Run as root

Finally, we need to pass in the root user as an additional parameter to our
Docker run command as it must call sudo docker when the docker pipeline plugins
runs commands. 

Call this new command from the terminal

```
docker run -d -v /var/run/docker.sock:/var/run/docker.sock \
              -v $(which docker):/usr/bin/docker -p 8080:8080 -u root \
              jenkinsdocker
```

# Create a new Jenkins Pipeline and test deployment

Once Jenkins is running, do the following:

1. Select the create new jobs button
2. Select the Pipeline type and name the job any desired name
3. Add the following in the pipeline script, changing the git url to match
the repo of your forked repository

```
node {  
  stage 'Checkout' 
  git url: 'https://github.com/PeterLamar/jenkins-docker-example.git'  

  stage 'Build' 
  docker.build('mobycounter')  

  stage 'Deploy'  
  sh './deploy.sh'
}
```

If the Checkout, Build, and Deploy steps all succeed then you can view the application running on the ip of the Jenkins Server at port 80. Enter the IP in your browser window and if you see a 'click to add icons' message then your application has succeeded. 


If everything is working please continue to [step 4](https://github.com/PeterLamar/docker-workshop/tree/master/compose)
