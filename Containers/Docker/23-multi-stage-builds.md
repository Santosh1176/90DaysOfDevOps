# Multi-stage builds in Docker

A few sessions ago we experienced Docker's Build kit and specifically `buildx` command. Around 2017, Docker announced support for multistage builds to build slimmer container images. earlier to the implementation of the feature, the size of an image was a blocker to many developers. As we still can see, if we build an image with simple `build` command, the image would be around 400Mb for a simple Go application with very few layers while building the image.
```bash
santosh@bookstore-api*$:docker build -t santoshdts/bookstore:0.1.0 .
Sending build context to Docker daemon   38.4kB
Step 1/7 : FROM golang:1.19-alpine
 ---> d2745094e3b1
Step 2/7 : WORKDIR /home
 ---> Running in d3eecce32e5b
Removing intermediate container d3eecce32e5b
 ---> 57e48afbb6f4
Step 3/7 : COPY ./pkg .
 ---> b359e8ec3572
Step 4/7 : RUN go mod download
 ---> Running in dbe63a7ca8c8
Removing intermediate container dbe63a7ca8c8
 ---> b78cb1da50ac
Step 5/7 : RUN  go build -o bookstore
 ---> Running in e95fc934f5af
Removing intermediate container e95fc934f5af
 ---> 71cceb713ac0
Step 6/7 : EXPOSE 8080
 ---> Running in 1ff3180af9cf
Removing intermediate container 1ff3180af9cf
 ---> 2a1c535abfaa
Step 7/7 : CMD ["./bookstore"]
 ---> Running in 0290e8abd0e2
Removing intermediate container 0290e8abd0e2
 ---> 3686f9a0ee0d
Successfully built 3686f9a0ee0d
Successfully tagged santoshdts/bookstore:0.1.0

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them

santosh@bookstore-api*$:docker images
REPOSITORY                               TAG               IMAGE ID       CREATED          SIZE
santoshdts/bookstore                     0.1.0             3686f9a0ee0d   15 seconds ago   366MB
```
As you can see the resulting image is 366MB. Now if I need to use this image to build a container and use it in Kubernetes Pods, that would be a burden on our resources. One of the main reasons is that each `RUN`, `COPY`, and `ADD` instruction in the Dockerfile adds a layer to the image.

This process of building images can be overcome with the use of Dockers *multi-stage builds*. By using the Multi-stage builds, we can add as many of these layers, which can be discarded in the next stage of the same build process by using a `FROM`  instruction. Each `FROM` instruction here can use a different base and starts a new stage of the build process. We can leverage this process by selectively copying required instructions and adding the binary which was built in our earlier stage. build a tiny image in this way and push the image to the registery.

Example:
```bash
FROM golang:1.19-alpine AS build
WORKDIR /home
COPY ./pkg .
RUN go mod download
EXPOSE 8080
RUN  go build -o bookstore

## Deploy
FROM alpine:latest 
WORKDIR /root
COPY pkg/templates/. /root
COPY --from=build /home/bookstore /root
COPY --from=build /home/templates/. /root/templates
ENTRYPOINT ["./bookstore"]
```
As you can see from the above example, I've used the first build process and aliased it as `build` where I create a `WORKDIR` and copy all the required files to it. in the next step, I download the dependencies,  expose port `8080` and build the binaries. These are the same steps when building with earlier conventional workflow. The interesting part comes next. on *line 54* I create a new build process with a new `alpine:latest` image and copy some required files and on *line 57* I copy from the previous build process the pre-built Go binary to the `/root` the `WORKDIR`. You might notice that I've used `--from=build`, this refers to the new process of copying the said files from the previous build process. While using multiple stages during our build process, we can also refer to external images as a *stage*. We can write the *Dockerfile* like, Using build arguments in `‐‐from`, using conditions using build arguments to make our images much leaner.

 This second build process strips all the unwanted files and builds toolchains that were bundled into the previous build and creates a new build with only the required files to run the application. The resulting size is dramatically reduced.

**Multi-stage Build Process**
```bash
santosh@bookstore-api*$:docker build -t santoshdts/bookstore:build-2 . -f Dockerfile.multistage 
Sending build context to Docker daemon   38.4kB
Step 1/11 : FROM golang:1.19-alpine AS build
 ---> d2745094e3b1
Step 2/11 : WORKDIR /home
 ---> Using cache
 ---> 57e48afbb6f4
Step 3/11 : COPY ./pkg .
 ---> Using cache
 ---> b359e8ec3572
Step 4/11 : RUN go mod download
 ---> Using cache
 ---> b78cb1da50ac
Step 5/11 : RUN  go build -o bookstore
 ---> Using cache
 ---> 71cceb713ac0
Step 6/11 : EXPOSE 8080
 ---> Using cache
 ---> 2a1c535abfaa
Step 7/11 : FROM alpine:latest
latest: Pulling from library/alpine
c158987b0551: Pull complete 
Digest: sha256:8914eb54f968791faf6a8638949e480fef81e697984fba772b3976835194c6d4
Status: Downloaded newer image for alpine:latest
 ---> 49176f190c7e
Step 8/11 : WORKDIR /root
 ---> Running in 9f6f87cd463b
Removing intermediate container 9f6f87cd463b
 ---> c79a03dad812
Step 9/11 : COPY pkg/templates/. /root
 ---> a4af30e6e1fe
Step 10/11 : COPY --from=build /home/bookstore /root
 ---> 2219bbf0c44c
Step 11/11 : ENTRYPOINT ["./bookstore"]
 ---> Running in 47d6d49d0517
Removing intermediate container 47d6d49d0517
 ---> c4cd60d05987
Successfully built c4cd60d05987
Successfully tagged santoshdts/bookstore:build-2

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
```

```bash
santosh@Docker*(main)$:docker images
REPOSITORY                               TAG               IMAGE ID       CREATED          SIZE
santoshdts/bookstore                     build-2           c4cd60d05987   13 minutes ago   17MB
```
As you can see the newly built image of the same Go application is now 17MB. This is because the Go SDK and any intermediate artifacts are left behind, and not saved in the final image. That's a considerable reduction in size, which would surely enhance the performance of our applications running on the cloud.

# Resources:
- [Multi-stage builds — Docker docs](https://docs.docker.com/build/building/multi-stage/)
- [Builder pattern vs. Multi-stage builds in Docker by Alex Ellis](https://blog.alexellis.io/mutli-stage-docker-builds/)
- [Using Docker Multi-Stage Builds by Viktor Farcic](https://youtu.be/zpkqNPwEzac)
- [Advanced Dockerfiles: Faster Builds and Smaller Images Using BuildKit and Multistage Builds](https://www.docker.com/blog/advanced-dockerfiles-faster-builds-and-smaller-images-using-buildkit-and-multistage-builds/)