
Amazon Login

Account ID: 709992105530
Username: student
Password: Ca1_MuxgUey8
Region: US West 2

Logging into AWS Console
------------------------

-> Open Console
https://709992105530.signin.aws.amazon.com/console?region=us-west-2

-> Enter the following credentials created just for your lab session, and click Sign in:
Account ID or alias: 709992105530
IAM user name: student
Password: Ca1_MuxgUey8

-> Select the US West (Oregon) us-west-2 region using the upper-right drop-down menu on the AWS Management Console

Connecting to Virtual Machine using EC2 instance
------------------------------------------------

-> In the AWS Management Console search bar, enter EC2, and click the EC2 result under Services
-> To see available instances, click Instances in the left-hand menu
-> Right-click the cloudacademylabs instance, and click Connect
-> In the form, ensure the EC2 Instance Connect tab is selected
-> In the User name textbox, enter ec2-user
-> To open a browser-based shell, click Connect

Installing Docker on Amazon Linux
---------------------------------

1) Enter the following to install Docker:
> sudo yum -y install docker

2) Enter the command below to start Docker as a service
> sudo systemctl start docker

3) Verify Docker is running by entering
> sudo docker info

Using Docker without Root permission on Linux
---------------------------------------------

1) Try entering the same command without using root permission:
> docker info
You will receive a permission denied error message

2) Verify the docker group exists by searching for it in the groups file
> grep docker /etc/group

If you don't see a line beginning with "docker:", you will need to add the group yourself by entering
> sudo groupadd docker

3) Add your user to the docker group
> sudo gpasswd -a $YSER docker

The groups of the currently logged in user is cached, you can verify this by entering groups

4) You can login again to have your groups updated by entering:
> newgrp docker

Now you will have docker in your list of groups if you enter groups
Note: It is convenient to not have to terminate your current ssh session by using newgrp, but terminating the ssh session and logging in again will work just as well.

5) Verify that your user can successfully issue Docker commands by entering:
> docker info

Getting Docker Help from Command Line
-------------------------------------

The commands are organized into common commands and a more exhaustive list grouped around the management of a specific component of Docker. You use each with a different syntax. For a common command, the usage is:

> docker command-name [options] 

and the usage for a management group command is:

> docker management-group command-name [options]

1)  To see a list of the commands in Docker, simply enter:
> docker --help

2) Enter the following management command:
> docker system info

3) To see all of the commands grouped under system, enter:
> docker system --help

4) To view the commands grouped with images, enter:
> docker image --help

Images are read-only snapshots that containers can be created from. Images are built up in layers, one image built on top of another. Because each layer is read-only, they can be identified with cryptographic hash values computed from the bytes of the data. Layers can be shared between images as another benefit of being read-only. You can, and will later, build your own images. The build command accomplishes that. When you build your own image, you will select a base image to build on top of with your custom application. A pull can be used to pull or download an image to your server from an image registry, while push can upload an image to a registry. Read through the other command descriptions so you are aware of what else is available.

5) To view the commands grouped with containers, enter:
> docker container --help

A container is another core concept in Docker. Containers run applications or services, almost always just one per container. Containers run on top of an image. In terms of storage, a container is like another layer on top of the image, but the layer is writable instead of read-only. You can have many containers using the same image. Each will use the same read-only image and have its own writable layer on top. Containers can be used to create images using the commit command, essentially converting the writable layer to a read-only layer in an image. 

The relationship between images and containers aside, run is used to run a command on top of an image. A container can be stopped and started again. ls is used to list containers. It is aliased to ps and list as well. Read through the other commands in the list to see what else is available for working with containers. 

Running your first docker container
-----------------------------------

1) Enter the following to see how easy it is to get a container running:
> docker run hello-world

There are two sections to the output: output from Docker as it prepares to run the container and the output from the command running in the container.

Take a look at the Docker output first:

In the first line, Docker is telling you that it couldn't find the image you specified, hello-world, on the Docker Daemon's local host. The latest portion after the colon (:) is a tag. The tag identifies which version of the image to use. By default, it looks for the latest version.
In the next line, it notifies you that it automatically pulled the image. You could manually perform that task using the command docker pull hello-world. The library/hello-world is the repository it's pulling from inside the Docker Hub registry. library is the account name for official Docker images. In general, images will come from repositories identified using the pattern account/repository.
The last three lines confirm the pull completed and the image has been downloaded.

2) Re-run the same command:
> docker run hello-world

Notice this time the Docker output is not included. That is because the specific version of the image was found locally and there is no need to pull the image again

3) Try running a more complex container with some options specified:
> docker run --name web-server -d -p 8080:80 nginx:1.12

This runs the nginx web server in a container using the official nginx image

This time you specified the tag 1.12 indicating you want version 1.12 of nginx instead of the default latest version. There are three Pull complete messages this time, indicating the image has three layers. The last line is the id of the running container. The meanings of the command options are:

--name container_name: Label the container container_name. In the command above, the container is labeled web-server. This is much more manageable than the id, 31f2b6715... in the output above.

-d: Detach the container by running it in the background and print its container id. Without this, the shell would be attached to the running container command and you wouldn't have the shell returned to you to enter more commands.

-p host_port:container_port: Publish the container's port number container_port to the host's port number host_port. This connects the host's port 8080 to the container port 80 (http) in the nginx command.

You again used the default command in the image, which runs the web server in this case.

4) Verify the web server is running and accessible on the host port of 8080:
> curl localhost:8080

5)  To list all running containers, enter:
> docker ps

6) Enter the following to see a list of all running and stopped containers:
> docker ps -a

This time you can see the two hello-world containers from the start of the Lab Step. They simply wrote a message and then stopped when the command finished, whereas the nginx server is always listening for requests until you stop it. Notice that Docker automatically assigned random friendly names for the hello-world containers, musing_volard, and jovial_snyder in the image above. These names are useful if you need to reference a container that you didn't assign a name to by yourself.

7) To stop the nginx server, enter:
> docker stop web-server

8) Verify the server is no longer running by running:
> docker ps

9) To start running the command in the web-server container again, enter:
> docker start web-server

This is different from re-running the original docker run command, which would make a second container running the same command instead of using the stopped container.

10) To see the container's output messages, enter:
> docker logs web-server

11) You can run other commands in a running container. For example, to get a bash shell in the container enter:
> docker exec -it web-server /bin/bash

This indicates you are at a shell prompt in the container using the root container user. The -it options tell Docker to handle your keyboard events in the container. Enter some commands to inspect the container environment, such as ls and cat /etc/nginx/nginx.conf. When finished, enter exit to return to the VM ssh shell. Your shell prompt should change to confirm you are no longer in the container bash shell.

You were able to connect to a bash shell because the nginx image has a Debian Linux layer which includes bash. Not all images will include bash, but exec can be used to run any supported command in the container.

12) To list the files in the container's /etc/nginx directory, enter:
> docker exec web-server ls /etc/nginx

This runs the ls command and returns to the ssh shell prompt without using a container shell to execute the command. What commands are supported depends on the layers in the container's image. Up until now you have used two images but don't know how to find more.

13) Stop the nginx container:
> docker stop web-server

14) Search for an image that you don't know the exact name of, say an image for Microsoft .NET Core, by entering:
> docker search "Microsoft .NET Core"

This can be useful for recalling the name of an image but not very useful for images that have multiple configuration options. For that you should use the web version of Docker Hub.

15) Navigate to the following link. You will be viewing the web page of the .NET repository: 
	https://hub.docker.com/_/microsoft-dotnet

You will find all the supported tags along with helpful documentation for using the images. You will usually find useful documentation and examples for images on Docker Hub. You will notice a Dockerfile beside each group of tags on the dotnet page. A Dockerfile is a set of instructions for creating an image and it is what you will dive into in the next Lab Step.

Creating Your First Docker Image
--------------------------------

Docker containers run on top of images. You have seen how to use images on Docker's public registry, the Docker Hub. There are many different images available. It is worth trying to find existing images when you can. Inevitably, you will need to create your own images when making your own applications. In that case, you will still want to invest time into finding the right base layer to add your own layer(s) on top of.

There are a couple of ways to make images. You can use docker commit to create an image from a container's changes. The changes may come from using exec to open a shell in the container like in the previous Lab Step. The other method is using a Dockerfile. A Dockerfile is easier to maintain, easier to repeatedly create images from, and distributions easier. You will create a Dockerfile in this Lab Step. Just know that it is possible to create equivalent images using commits.

Dockerfiles specify a sequence of instructions. Instructions can install software, expose network ports, set the default command for running a container using the image, and other tasks. Instructions can really handle anything required to configure the application. Many of the instructions add layers. It is usually a good idea to keep the number of layers to a reasonable number. There is overhead with each layer, and the total number of layers in an image is limited. When the Dockerfile is ready, you can create the image using the docker build command.

1) Install Git:
> sudo yum -y install git

You will clone a code repository with the Flask app using Git.

2) Clone the code repository to your virtual machine:
> git clone https://github.com/cloudacademy/flask-content-advisor.git

3) Change to the apps directory:
> cd flask-content-advisor

4) Create and start editing a Dockerfile using the vi text editor:
> vi Dockerfile

5) Enter following in the file

	# Python v3 base layer
	FROM python:3

	# Set the working directory in the image's file system
	WORKDIR /usr/src/app

	# Copy everything in the host working directory to the container's directory
	COPY . .

	# Install code dependencies in requirements.txt
	RUN pip install --no-cache-dir -r requirements.txt

	# Indicate that the server will be listening on port 5000
	EXPOSE 5000

	# Set the default command to run the app
	CMD [ "python", "./src/app.py" ]

The lines beginning with # are comments and explain what each instruction is doing. Make sure you read the comments. Some highlights are:

FROM sets the base layer image
COPY . . copies all of the files in the code repository into the container's /usr/src/app directory
RUN executes a command in a new layer at the top of the image
EXPOSE only indicates what port the container will be listening on, it doesn't automatically open the port on the container
CMD sets the default command to run when a container is made from the image

https://docs.docker.com/engine/reference/builder/

6) Once you have all of the instructions in the Dockerfile, press the esc key to first exit Insert mode, and then enter :wq to write (save) the file and quit vi.

7) Build the image from the Dockerfile:
> docker build -t flask-content-advisor:latest .

The -t tells Docker to tag the image with the name flask-content-advisor and tag latest. The . at the end tells Docker to look for a Dockerfile in the current directory. Docker will report what it's doing to build the image. Each instruction has its own step. Steps one and four take longer than the others. Step one needs to pull several layers for the Python 3 base layer image and Step four downloads code dependencies for the Flask web application framework. Notice that each Step ends with a notice that an intermediate container was removed. Because layers are read-only, Docker needs to create a container for each instruction. When the instruction is complete, Docker commits it to a layer in the image and discards the container.

8) Record your VM's public IP address:
> curl ipecho.net/plain; echo

You will need your IP to test that the web app is available with your browser.  The echo is only to put the shell prompt on a new line.

9) pen a new browser tab and navigate to the public IP address you just recorded. The browser will fail to load anything since no server is running yet. Keep the tab open for later.

10) Now you can run a container using the image you just built:
> docker run --name advisor -p 80:5000 flask-content-advisor

This runs a container named advisor and maps the container's port 5000 to the host's port 80 (http). This time you didn't include -d to run in detached mode.  That is why you see output and you don't have the shell prompt returned to you. If you did run with -d, you could get the same information from docker logs.

11) Return to your browser tab with the public IP and refresh the page

12) Return to the shell and notice that some web requests will have been logged corresponding to your browser requests:

There are two requests because the browser automatically requests a favicon in addition to the page content.