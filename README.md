# play-docker-aws-tutorial
101 tutorial for publishing Play app to Amazon Web Services (AWS) using Docker. This tutorial explains how to:

- Install necessary tools
- Create a new Play project
- Create a new Docker container
- Run container locally
- Setup AWS EC3 instance
- Publish container to EC3 instance

## Step 1: Install necessary tools
This instructions are for Mac OS X:

First Typesafe Activator

```
$ brew install typesafe-activator
```

Then download [Docker Toolbox](https://www.docker.com/toolbox) and install it using the installer.

## Step 2: Create a new Play project

Create a new app using `Activator`

```
$ activator new example-app-1 play-scala
```
This command creates a new project using play-scala template. Template is fetched from Typesafe [template repository](http://typesafe.com/activator/templates). Check out the [actual template](https://typesafe.com/activator/template/play-scala).


Run the application so you can see it is working

```
$ cd example-app-1
$ activator run
```

You should get the following result:

```
--- (Running the application, auto-reloading is enabled) ---

[info] p.c.s.NettyServer - Listening for HTTP on /0:0:0:0:0:0:0:0:9000

(Server started, use Ctrl+D to stop and go back to the console...)
```

Open browser and go to `http://localhost:9000/` and you should see the following page.

![image](images/play-default-page.png)

Your Play app is now created! That was simple!

## Step 3: Create a new Docker container

Go to your application directory and run the following command:

```
$ activator docker:stage
```

It will create `target/docker/stage` directory which contains all the files for the Docker image.

So what are *image* and *container*? A *container* is a stripped-to-basics version of a Linux operation system. An *image* is software you load into a container.

In that directory there is a file called `Dockerfile`. Add the following line to the file after RUN line:

```
EXPOSE 9000
```

So your `Dockerfile` might look something like this:

```
FROM java:latest
WORKDIR /opt/docker
ADD opt /opt
RUN ["chown", "-R", "daemon:daemon", "."]

EXPOSE 9000

USER daemon
ENTRYPOINT ["bin/example-app-1"]
CMD []
```

Next open Docker Quickstart Terminal and run the following command in the `stage` directory:

```
$ docker build -t example-app-1 .
```

This command reads the Dockerfile in the current directory and builds an image called **example-app-1**.

You can run the following command and verify the new is on your computer.

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
example-app-1       latest              89e9d439ef8d        8 minutes ago       894.1 MB
```
## Step 4: Run docker image locally

The image is ready. Now we can run it locally.

```
$ docker run --name example-container-1 -p 80:9000 example-app-1
[info] - play.api.Play - Application started (Prod)
[info] - play.core.server.NettyServer - Listening for HTTP on /0:0:0:0:0:0:0:0:9000
```

This will create a new container by name `example-container-1` and it will serve our app in port 80.

But how do you open your app in browser? You need the IP address of the container. But how do you know what is the IP address?

First, hit `Ctrl+C` to stop the container. Then you can start it again using the following command:

```
$ docker start example-container-1
```

Next you have to find what is the external IP address of your docker container. One easy way is to use the following command:

```
$ docker-machine inspect default | grep IPAddress
        "IPAddress": "192.168.99.100",
```

So, now you can open your browser and go to `http://192.168.99.100/` and you should see a text saying "Your new application is ready". Your app is running!

Now you can stop your app:

```
$ docker stop example-container-1
```








## Step 4: Setup AWS EC3 instance
**TODO**

## Step 5: Publish container to EC3 instance
**TODO**
