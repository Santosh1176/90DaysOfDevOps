# Building Containers using Dockerfile

We can create containers from a Dockerfile. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. Creating a Dockerfile is as simple as creating a text file but varies in syntax and naming convention. the Dockerfile should be named as... you guessed it right, **Dockerfile** with a capital **D**. 

We shall build a simple Dockerfile from which we would create a container running an application called **Figlet**, which 

> FIGlet is a computer program that generates text banners, in a variety of typefaces, composed of letters made up of conglomerations of smaller ASCII characters.

## Dockerfile

I've created a Dockerfile in my `~/Desktop/figlet` directory:

```Dockerfile
# A Dockerfile must begin with a FROM instruction. It's the Parent Image from which we are building.
# Here we are instructing to build from the latest image of the Alpine flavor of Linux
FROM alpine:latest


# RUN is the command we would run in our parent image
RUN apk update && apk update

# In this line we run the command to install figlet
RUN apk add figlet  

ENTRYPOINT ["figlet"]

```

Once the Dockerfile is written and saved, we would proceed to build and *Docker Image* from that Dockerfile. We could do this as follows:

`docker build -t figlet .`

Here, `docker build` is the command used to Build an image from a Dockerfile, and `-t` flag is used to *tag* the new image with a name, which in this case will be *figlet*. and the `.` indicates that the Dockerfile is in the current directory (`~/Desktop/figlet`), along with the so-called “context” — that is, the rest of the files that may be in that location.

The above command would build an image with the output of every step displayed on our console, like so:

```bash
santosh@figlet*$:docker build -t figlet .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM alpine:latest
 ---> 9c6f07244728
Step 2/4 : RUN apk update && apk update
 ---> Using cache  
 ---> ead3803c7bba
Step 3/4 : RUN apk add figlet
 ---> Using cache
 ---> ad88d547ba3d
Step 4/4 : ENTRYPOINT ["figlet"]
 ---> Using cache
 ---> f9bb54f9c4cc
Successfully built f9bb54f9c4cc
Successfully tagged figlet:latest

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
```

To check for the image build, we can use:
```bash
 santosh@figlet*$:docker images
REPOSITORY                         TAG             IMAGE ID       CREATED          SIZE
figlet                            latest        f9bb54f9c4cc    11 minutes ago     8.68MB
```

> You might have noticed while the build process was on, there is an out named **Using cache**. This is one of the most crucial features of Docker, which enables building images should be fast, efficient, and reliable. Docker builds images using a Layered approach. Every command you execute results in a new layer that contains the changes compared to the previous layer. This means, all previously built layers are cached and can be reused. From the above example, you can see Running `apk update && apk upgrade` The Docker recognized that this layer is already present on my system and used the same and did not update operation system dependencies. This makes building images really fast and saves a lot of memory on our system. However, If you are in need of overriding this caching behavior and running these updates we can pass the `no-cache` flag. 


To run a container from our previously built image, we run the `docker run -it figlet:latest 90DaysOfDevOps` command, this would override the `ENTRYPOINT` command defined in our Dockerfile with `90DaysOfDevOps` by running the container in an *interactive mode* and telling Docker to allocate a *virtual terminal session* within the container. thats have been achieved by passing the `-it` flag. The result would be like this:

```bash
santosh@figlet*$:docker run -it figlet:latest 90DaysOfDevOps
  ___   ___  ____                   ___   __ ____              ___            
 / _ \ / _ \|  _ \  __ _ _   _ ___ / _ \ / _|  _ \  _____   __/ _ \ _ __  ___ 
| (_) | | | | | | |/ _` | | | / __| | | | |_| | | |/ _ \ \ / / | | | '_ \/ __|
 \__, | |_| | |_| | (_| | |_| \__ \ |_| |  _| |_| |  __/\ V /| |_| | |_) \__ \
   /_/ \___/|____/ \__,_|\__, |___/\___/|_| |____/ \___| \_/  \___/| .__/|___/
                         |___/                                     |_|        

```
Here is an official [Redis Docker image](https://github.com/docker-library/redis/blob/413d7277d124de16b28f21f6ce7a54e81b942c08/7.0/Dockerfile) for reference to checkout various commands used to build the Redis Image.

# Resources:
- [Docker Cache... by FreeCodeCamp](https://www.freecodecamp.org/news/docker-cache-tutorial/)
- [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Advanced Dockerfiles... by Docker](https://www.docker.com/blog/advanced-dockerfiles-faster-builds-and-smaller-images-using-buildkit-and-multistage-builds/)
