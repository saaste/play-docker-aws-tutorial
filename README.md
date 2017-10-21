# Play / Docker / Amazon Lightsail (AWS) Tutorial for Noobs

This tutorial is a step-by-step guide for total newbies. It is **NOT** a guide how to create and deploy a proper
production app! Learn your stuff before you start deploying real apps, mmmkay?

Tutorial will explains how to:

- Install necessary tools (for Mac OS X)
- Create a new Play project
- Create a new Docker image
- Run Docker image locally
- Push image to Docker Hub
- Run docker image in Lightsail
 

Before you start, make sure you have [Lightsail](https://lightsail.aws.amazon.com)
and [Docker Hub](https://hub.docker.com/) accounts ready. Registration does not cost anything and first Lightsail
month is free!

## Step 1: Install necessary tools

### Mac OS X

Install [SBT](http://www.scala-sbt.org/release/docs/Installing-sbt-on-Mac.html) using Brew:

```bash
$ brew install sbt@1
```

Download [Docker for Mac](https://docs.docker.com/docker-for-mac/install/) and install it using the installer.


## Step 2: Create a new Play project

Download [minimal-play2 template](https://github.com/futurice/minimal-play2) and unzip it.
```bash
$ wget https://github.com/futurice/minimal-play2/archive/master.zip -O minimal-play2.zip
$ unzip minimal-play2.zip -d my-app
$ cd my-app
```

Edit `build.sbt` and change `name` parameter from `minimal-play2` to `my-app`.
```scala
lazy val root = (project in file("."))
  .enablePlugins(PlayScala)
  .settings(
    organization := "com.futurice",
    name := "my-app",                       // Change this
    version := "1.3.0",
    scalaVersion := "2.12.3",
    libraryDependencies += guice,
  )
```

Edit `conf/application.conf` and add a new line.
```scala
play.filters.disabled += play.filters.hosts.AllowedHostsFilter
```

Make sure you are in the project root and start the app. First time might take a while.
```bash
$ sbt run
```

You should get the following result:
```bash
[info] p.c.s.AkkaHttpServer - Listening for HTTP on /0:0:0:0:0:0:0:0:9000

(Server started, use Enter to stop and go back to the console...)
```

Go to `http://localhost:9000` using a web browser. After a while you should
see a white page with text "OK!"

Your Play app is now created! That was easy!


## Step 3: Create a new Docker image

Create a file called `Dockerfile` in your project root:
```dockerfile
FROM openjdk:8-jdk-alpine

RUN apk add --update bash

COPY ./target/universal/dist /app

EXPOSE 8080

CMD bash /app/bin/my-app -Dhttp.port=8080

```

Docker image is created using this file. Basically does the following things:
- Image is based on [openjdk image](https://hub.docker.com/_/openjdk/) version 8-jdk-alpine
- Install and update `bash`
- Copy `target/universal/dist` from your computer to `/app` directory in the image
- Expose port `8080`
- Run `bash` and then `app/bin/my-app` program with port `8080` 

Next make a file called `build-docker-image.sh`:
```bash
#!/bin/bash

APP_NAME=$1                                                     # Name of the app
VERSION=$2                                                      # App version
USERNAME=$3                                                     # Docker Hub username

sbt playUpdateSecret dist                                       # Update app secret and build Play app

cd ./target/universal                                           # Go to directory where application zip is located
rm -rf ./tmp ./dist                                             # Deleted existing directories if those exist

unzip ./${APP_NAME}*.zip -d ./tmp                               # Extract application zip
mv ./tmp/${APP_NAME}* ./dist                                    # Move application files into dist folder
rm -rf ./tmp                                                    # Remove temp directory
cd ../..                                                        # Go back to application root

docker build -t ${USERNAME}/${APP_NAME}:${VERSION} .            # Build docker image

```
I know, it looks a bit scary but this file makes our life a bit easier so we don't have to run
all these commands manually.

Now you need to make that file executable.
```bash
$ chmod 755 build-docker-image.sh
```

One more step and our docker image is ready. Make sure to replace `YOUR_DOCKER_HUB_USERNAME`!
```bash
$ ./build-docker-image.sh my-app 0.0.1 YOUR_DOCKER_HUB_USERNAME
```

Every time you run this script, it will build your app and make a new docker image. Just use different version
number to separate your app versions.

So what are **image** and **container**?

**Image** is a blueprint. Basically it tells what to do when new container is started.
**Container** is created when you run an image. You can have multiple containers running the same image.

You can think that image is the recipe and container is the cake. You can make many many cakes with the same recipe.

Run the following command and verify the new image is on your computer.

```bash
$ docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
YOUR_ACCOUNT/my-app     latest              89e9d439ef8d        8 minutes ago       142MB
```


## Step 4: Run docker image locally

Run the image using the following command:

```bash
$ docker run --name my-app-container -p 8080:8080 -d YOUR_DOCKER_HUB_USERNAME/my-app:0.0.1
40d76d61e28a305c20c7646873e4e0a3aa0c621a96b57b817a2cae40cfa9415f
```

This will create a new container by name `my-app-container` and it will serve our app in port `8080`. `-d` argument makes the Docker container run in the background.

Open your browser and got to `http://localhost:8080` and you should see that familiar "OK!" text.

If everything is OK, you can stop and remove the container:
```bash
$ docker stop my-app-container && docker rm my-app-container
```


## Step 5: Push image to Docker Hub

Go to [Docker Hub](https://hub.docker.com/) and `Create Repository`.

Name of the repository has to match our image so make sure the name is `my-app` and visibility is `public`.

Next you need to log in to docker using your Docker Hub username and password.
```bash
$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username (******):
Password: 
Login Succeeded
```

Now you are ready to push your image to Docker Hub. Don't worry if you see plenty of text, that is normal.
```bash
$ docker push YOUR_DOCKER_HUB_USERNAME/my-app:0.0.1
The push refers to a repository [docker.io/YOUR_DOCKER_HUB_USERNAME/my-app]
da7a39376352: Layer already exists 
4f401f7e5f4c: Layer already exists 
5bef08742407: Layer already exists 
0.0.1: digest: sha256:cd69001c87f71c02e893799ad40e74adda58e34ff634fe110cf4bf9d12416006 size: 1370
```

If you go to Tags tab in your my-app repo, you should now see one tag with name 0.0.1 - that is the version we used.

## Step 6: Run docker image in Lightsail

Log in to [Amazon Lightsail](https://lightsail.aws.amazon.com) and click `Create instance` and select
`Linux/Unix` platform and `OS Only > Amazon Linux` platform.

Click `Add launch script` and enter following script that will install docker:
```bash
yum update -y
yum install -y docker
service docker start
```

Finally select $5 instance plan, give your instance a name and hit `Create`!

Wait until your instance status switches from `Pending` to `Ready`. It can take a minute or few.

Then hit `Create static IP`, attach it to the instance you just made, give it a cute name and click `Create` one more time.
Now your instance has a static IP. Sweet!

Go back to Lightsail home and click that small orange terminal icon next to your instance. It will open a new browser
window which works as a terminal - no usernames or passwords!

All you need to do is to start the container just like you did on your local machine.
```bash
$ sudo docker run --name my-app-container -p 80:8080 -d YOUR_DOCKER_HUB_USERNAME/my-app:0.0.1
Unable to find image 'YOUR_DOCKER_HUB_USERNAME/my-app:0.0.1' locally
0.0.1: Pulling from YOUR_DOCKER_HUB_USERNAME/my-app
88286f41530e: Pull complete 
720349d0916a: Pull complete 
42a4b3080d3c: Pull complete 
00555cc129c2: Pull complete 
c4b96aada161: Pull complete 
Digest: sha256:cd69001c87f71c02e893799ad40e74adda58e34ff634fe110cf4bf9d12416006
Status: Downloaded newer image for YOUR_DOCKER_HUB_USERNAME/my-app:0.0.1
81b33ae0dd3eb00f7d0c327202815b30388c7ea3d4944e5db9238c5c29c3f65d
```

Enter your static IP address to your browser and you should see our old friend "OK!".

You are done! Your Play application is now running on Lightsail (AWS)!