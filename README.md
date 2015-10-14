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

Go to [AWS Web Page](https://aws.amazon.com) and click `Sign In to the Console` button. From there you can create a new Amazon account or log in if you already have an account.

After logging in, select `EC2 (Virtual Servers in the Cloud)` from the service list. Amazon EC2 instance is a virtual server which will eventually host your Docker container.

Next, click `Launch Instance` button. This will start a wizard.

**Step 1: Choose an Amazon Machine Image (AMI)**

From the list select the first option which should be Amazon Linux AMI. Notice, that this AMI is included in the free tier which means you can try and run it for free.

**Step 2: Choose an Instance Type**

By default t2.micro type is selected. If it is not, select it. This *free* option is good testing and playing around.

Click `Next: Configure Instance Details` to continue.

**Step 3: Configure Instance Details**

Make sure the number of instances is `1`.

Click `Next: Add Storage` to continue.

**Step 4: Add Storage**

In here you can accept the default values. You should have one Root volume and nothing more.

Click `Next: Tag Instance`

**Step 5: Tag Instance**

You don't have to do anything here.

Click `Next: Configure Security Group`

**Step 6: Configure Security Group**

You have to create a new security group which basically defines what kind of inbound trafic is allowed to your instance. By default SSH is enabled from *anywhere*.

Because our plan is to run a web app, we need to open port for it. Click `Add Rule` and select `HTTP type`. That is all you have to do.

Click `Review and Launch` to continue.

**Step 7: Review Instance Launch**

Amazon will nag you about the open SSH but this is just a testing instance, you don't have to worry about it. You can review all the things you did and finally click `Launch`

You will probably get a popup asking about key pair. We assume you don't have it yet so select `Create a new key pair`. Then give a name for your key pair. In this tutorial we assume the key is called `test-aws-key`. Click `Download Key Pair`

You'll download a key file. Save it to a secure location and don't loose it! You will need it later!

Finally click `Launch Instances` and your instance will be initiated.

![image](images/instances-launching.png)

Click to instance id to open the details of your brand new instance.

![image](images/instance-list.png)

Here you can see all the instances. When know the instance is up and running when Instance State has a green circle with the text `running`.










## Step 5: Publish container to EC3 instance
**TODO**
