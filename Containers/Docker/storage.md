# Container Storage:

- Storage Drivers
    - **FileSystem**: Docker stores all the Volumes at when **Local Storage Driver is used**:
    
    ```jsx
    **/var/lib/docker**
          ├── aufs
          ├── containers
          ├── image
          **├── volumes**
    ```
    

With Docker’s Layered architecture. There are mainly two layers when we the containers:

**Image Layer:**

This layer is further a layer of all the images and commands we’ve used to build the image. One image is placed on top of another layer. thus, whenever we create an different image containing a similar base image, Docker uses the same image used while building  an erlier image. 

**This Layer is also known as IMAGE LAYER and its a READ ONLY Layer. Its created when we run `docker build` command**

**Container Layer:**

This is a layer which is created when we run the built image and provides an **READ/WRITE**  Layer. We can exec into the layer and store some files inside the container for ex: logs, metrics, etc. This layer also provides a feature known as COPY on WRITE, i.e,  it gets the Copy of the image created during the **IMAGE LAYERS** so ****as we can make modification to the files created in the IMAGE Layer. But when this container is killed or gets down due to some reason. the files stored into the container are also lost.

**This Layer is also known as CONTAINER LAYER and its READ/WRITE Layer. Its formed when we run `docker run` command.**

- **Volume in Docker**
    - **Volume Mounting**

We can create an Persistent Volume in Docker, so as the storage in the containers are persisted in case of containers getting killed. To create an persistent volume run `docker volume create <volume name>` This creates another directory in the Volume folder as **described above. T**hen we can mount the newly created volume while running a container by running **`docker run -v <volume name>:<mount path> <image name>.`** *mount path* is a location within the container we wish to store the contents.

- **Bind Mounting**

Bind mounting is a concept used when we have an storage outside the container on the Docker host, which we want to mount on the container. suppose its in `/data` in such cases we can use Bind Mounting. In this scenario we will run the container by `docker run --mount type=bind,source=/data<Full Source path>,targer=<Target path within the container>`