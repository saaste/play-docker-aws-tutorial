# Play / Docker / Amazon Lightsail (AWS) Tutorial for Noobs

Tutorial will explains how to:

- Install necessary tools (for Mac OS X)
- Create a new play project
- Create a new docker image
- Run docker image locally
- Push image to Docker Hub
- Run docker image on Amazon Lightsail instance

This tutorial is a step-by-step guide for total newbies. It is **NOT** a guide how to create and deploy a proper
production app! Learn your stuff before you start deploying real apps, mmmkay?


## Step 0: Prerequisities 

Create [Amazon Lightsail][lightsail] account.
It is free and running the smallest instance is free for the first month. After that it costs $5/month.

Create [Docker Hub][dockerhub] account.
Free account can have one private repository and unlimited number of public repositories.


## Step 1: Install necessary tools

Install [SBT][sbt] using Brew:

```bash
$ brew install sbt@1
```

Install [Docker for Mac][docker-mac] using the installer. 


## Step 2: Create a new play project

Download [minimal-play2 template][min-play] and unzip it.

```
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
```
play.filters.disabled += play.filters.hosts.AllowedHostsFilter
```

Make sure you are in the project root and start the app. First time might take a while.
```
$ sbt run
```

When you see the following result, you know your app is running.
```
[info] p.c.s.AkkaHttpServer - Listening for HTTP on /0:0:0:0:0:0:0:0:9000

(Server started, use Enter to stop and go back to the console...)
```

Go to `http://localhost:9000` using your web browser. After a while you should
see a white page with text "OK!"

Your app is now created! That was easy!


## Step 3: Create a new docker image

Create a file called `Dockerfile` in your project root. Docker image will be created using this file.
```dockerfile
# image is based on OpenJDK image version 8-jdk-alpine
FROM openjdk:8-jdk-alpine

# install and update bash
RUN apk add --update bash

# copy target/universal/dist directory to /app directory in your image
COPY ./target/universal/dist /app

# make port 8080 visible
EXPOSE 8080

# run /app/bin/my-app executable in port 8080
CMD bash /app/bin/my-app -Dhttp.port=8080
```

Next create a file called `build-docker-image.sh` and make it executable.
```
$ touch build-docker-image.sh
$ chmod 755 build-docker-image.sh
```

This will be our helper script that builds the app, make a distribution package and prepares it for the docker image.
Finally it builds the image it self. Content of the file should be the following: 
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
Script takes four parameters: name of the application, version of the image and your [Docker Hub][dockerhub] username.

Now we just need to run it. Make sure to replace `YOUR_DOCKER_HUB_USERNAME` with your username! 
```
$ ./build-docker-image.sh my-app 0.0.1 YOUR_DOCKER_HUB_USERNAME
```

So what is **image** and **container**?

**Image** is a blueprint. Basically it tells what to do when new container is started.

**Container** is created when you run an image. You can have multiple containers running the same image.

You can think that image is the recipe and container is the cake. You can make many many cakes with the same recipe.

If you want to check that the image was created, run the following command:

```
$ docker images
```

You should see something like this:
```
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
YOUR_ACCOUNT/my-app     latest              89e9d439ef8d        8 minutes ago       142MB
```


## Step 4: Run docker image locally

Run the image and make a new container.

```
$ docker run --name my-app-container -p 8080:8080 -d YOUR_DOCKER_HUB_USERNAME/my-app:0.0.1
```

This will create a new container called `my-app-container`.
It serves our app in port `8080`.
`-d` argument makes the Docker container run in the background.

Open your browser and go to `http://localhost:8080` and you should see that familiar "OK!" text.

Now we know our image is good to go and you can stop and remove the container.
```
$ docker stop my-app-container && docker rm my-app-container
```


## Step 5: Push image to Docker Hub

Go to [Docker Hub](https://hub.docker.com/) and `Create Repository`.

Name of the repository has to match to our image so make sure the name is `my-app` and visibility is `public`.

Next you need to log in to docker using your Docker Hub username and password.
```
$ docker login
```

If login was successful, output should be something like this:
```
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username (YOUR_USERNAME):
Password: 
Login Succeeded
```

Now you are ready to push your image to Docker Hub.
```
$ docker push YOUR_DOCKER_HUB_USERNAME/my-app:0.0.1
```

You'll probably see plenty of text, but it should look like this:
```
The push refers to a repository [docker.io/YOUR_DOCKER_HUB_USERNAME/my-app]
da7a39376352: Layer already exists 
4f401f7e5f4c: Layer already exists 
5bef08742407: Layer already exists 
0.0.1: digest: sha256:cd69001c87f71c02e893799ad40e74adda58e34ff634fe110cf4bf9d12416006 size: 1370
```

If you go to [Docker Hub][dockerhub] repo and open `Tags` tab, you should see one tag with name 0.0.1.
That is the image we just pushed. Tag name is the version we defined when we ran `./build-docker-image.sh`.

## Step 6: Run docker image on Amazon Lightsail instance

Log in to [Amazon Lightsail Console][lightsail-console] and click `Create instance`.
Select `Linux/Unix` platform and `OS Only > Amazon Linux` blueprint.

Click `Add launch script` and enter following script that will install docker:
```
yum update -y
yum install -y docker
service docker start
```

Finally select **$5/month** instance plan, give your instance a name and hit `Create`!

Wait until your instance status switches from `Pending` to `Ready`. It can take a minute or few.

Click `Create static IP`, attach it to your new instance, give it a cute name and click `Create`.
Now your instance has a static IP!

Go back to [home][lightsail-console] and click that small orange terminal icon next to your instance.
It will open a new browser window which works as a terminal.

All you need to do is to start the container. Just like you did on your local machine except this time
you have to run it using `sudo` command.
```
$ sudo docker run --name my-app-container -p 80:8080 -d YOUR_DOCKER_HUB_USERNAME/my-app:0.0.1
```

Output should look something like this:
```
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

You are done! Your application is now running on Amazon Lightsail!

[lightsail]: https://amazonlightsail.com
[lightsail-console]: https://lightsail.aws.amazon.com
[dockerhub]: https://hub.docker.com
[sbt]: http://www.scala-sbt.org/release/docs/Installing-sbt-on-Mac.html
[docker-mac]: https://docs.docker.com/docker-for-mac/install/
[min-play]: https://github.com/futurice/minimal-play2