
# Getting Started with Docker
This tutorial will explain the fundamentals of Docker and start you with some basic usage.

## What is Docker?

Docker is open source software to pack, ship and run any application as a lightweight container. Containers are completely hardware and platform independent so you don't have to worry about whether what you are creating will run everywhere.

In the past virtual machines have been used to accomplish many if these same goals. However, Docker containers are smaller and have far less overhead than VMs. VMs are not portable as different VM runtime environments are very different. Docker containers are extremely portable. Finally, VMs were not built with software developers in mind; they contain no concept of versioning, and logging/monitoring is very difficult. Docker images, on the other hand, are built from layers that can be version controlled. Docker has logging functionality readily available for use.

You might be wondering what could go into a "container". Well, anything! You can isolate pieces of your system into separate containers. You could potentially have a container for nginx, a container for MongoDB, and one for Redis. Containers are very easy to setup. Major projects like nginx, MongoDB, and Redis all offer free Docker images for you to use; you can install and run any of these containers with just one shell command. This is much easier than using a virtual machine (even with something like Vagrant).

## Installation

Installing Docker is very easy. Visit the [official Docker installation page][1] and follow the instructions tailored for your operating system. There are simple installers for both Mac OS X and Windows.

After you've installed Docker, open the terminal and type the following:
    
    
    $ docker info
    
    

If your installation worked, you will see a bunch of information about your Docker installation. If not, you will need to revisit the install docs.

## Creating Your First Docker Image

Every Docker container is an "instance" of a Docker image. There is a [massive library of pre-built Docker images][2]. However, in order to really understand Docker, you should create an image as an exercise.

Let's create a Docker image for running Redis. [Redis][3] is an easy to use in-memory key/value store. It is commonly used as an object cache for many different platforms across many different environments and programming languages.

Remember how I said Docker images are built from layers? Well, every Docker image has to start with a base layer. Common base layers are Ubuntu and CentOS. Let's use Ubuntu. (In production I would use Debian since it is much smaller.)

The following command will start a Docker container based on the `Ubuntu:latest` image. `:latest` is called the image tag and in this case refers to the latest version of Ubuntu. If you don't have the image locally, it will download it first. The container will be started in a bash terminal. Run the following:
    
    
    $ docker run --name my-redis -it ubuntu:latest bash
    
    

`-it` let's us interact with our container via the command line. `\--name` just gives us a convenient way to reference our container. You should now be inside your container in a bash terminal seeing something like this:
    
    
    $ root@ed35631e96f9
    
    

As you can see, you are logged in as root to the container so no need for `sudo`. The Ubuntu base image is very bare bones. An important stratey for creating Docker images is keeping them as light as possible. Therefore you have to install a lot of things you normally just have. First, let's install `wget`:
    
    
    $ apt-get update
    $ apt-get install wget
    
    

We need a few other things to build Redis from source and run it:
    
    
    $ apt-get install build-essential tcl8.5
    
    

Now let's install Redis:
    
    
    $ wget http://download.redis.io/releases/redis-stable.tar.gz
    $ tar xzf redis-stable.tar.gz
    $ cd redis-stable
    $ make
    $ make install
    $ ./utils/install_server.sh
    
    

This downloads the newest version of Redis, builds it from source, and runs the installer. You will need to answer some configuration questions. Just use all the defaults. Now start Redis by running the following (it might already be started):
    
    
    $ service redis_6379 start
    
    

You now have Redis started in a Docker container. The next step is saving your image. We want to be able to save the image as it is so we can distribute it and use it elsewhere.

_Note_: this container is an example, and is missing some things to make it truly usable such as port mapping. We will make a production ready image in the next section.

Exit your container by running:
    
    
    $ exit
    
    

Note that your container is now stopped since you exited bash. You can easily configure containers to run in the background though.

Run the following command:
    
    
    $ docker ps -a
    
    

This command shows us all of our docker containers, running or stopped. See the container tagged with `my-redis`. That's the one we created! Now let's commit our container as an image:
    
    
    $ docker commit -m "Added Redis" -a "Your Name" my-redis tlovett1/my-redis:latest
    
    

This command compiles our container's changes into an image. `-m` specifies a commit message, and `-a` let's us specify an author. `tlovett/my-redis:latest` is formatted author/name:version. Author refers to your username on [Docker Hub][4]. If you don't want to push your image to the Docker Hub, then this doesn't matter, and you can use anything you want. If you do, you will need to create an account and use `docker push` to push the image upstream.

Docker commit creates an image containing the _changes_ we made to the original Ubuntu image. This makes distributing Docker containers super fast since people won't have to re-download layers (such as Ubuntu:latest) that they already have. In a container, every time you run a command, add a file or directory, create an environmental variable, etc. a new _layer_ is created. Docker commit groups these layers into an image. When distributing Docker images, you should carefully optimize your layers to keep them as small as possible. This tutorial does not cover layer optimization.

You might be thinking that this is somewhat messy since your container is basically a black box. What if you want to redo your image? Would you have to write down the steps to reproduce the entire thing? What if you wanted to recreate your image from CentOS instead of Ubuntu? Your thinking would be correct. Creating Docker images in this way is not the best idea. Instead you should use Dockerfiles.

### Your First Dockerfile

A Dockerfile is a set of instructions written as a shell script for creating a Docker image. Let's create a Dockerfile that generates an image like the one we just created manually but with some important additions.

Create a file called `Dockerfile`. Paste the following into the new file:
    
    
    FROM ubuntu:latest
    RUN apt-get update
    RUN apt-get install -y wget
    RUN apt-get install -y build-essential tcl8.5
    RUN wget http://download.redis.io/releases/redis-stable.tar.gz
    RUN tar xzf redis-stable.tar.gz
    RUN cd redis-stable && make && make install
    RUN ./redis-stable/utils/install_server.sh
    EXPOSE 6379
    ENTRYPOINT  ["redis-server"]
    
    

There are some special things in this Dockerfile. `FROM` tells Docker which image to start from. As you can see, we are starting with Ubuntu. `RUN` simply runs a shell command. `EXPOSE` opens up a port to be publically accessible. 6379 is the standard Redis port. `ENTRYPOINT` designates the command or application to be run when a container is created. In this case whenever a container is created from our image, `redis-server` will be run.

Now that we've written our Dockerfile, let's build an image from it. Run the following command from within the folder of your Dockerfile:
    
    
    $ docker build -t redis .
    
    

This command will create an image tagged `redis` from your Dockerfile.

Finally, let's create a running container from our image. Run the following command:
    
    
    $ docker run -d -p 6379:6379 redis
    
    

That's it! Now you have Redis up-and-running on your machine.This container/image is production ready.

## Conclusion

Docker is a powerful tool for creating and running distributable, lightweight applications both locally and in production.

There are many tools and services available to be used in Docker. For example, [Dockunit][5] is a tool powered by Docker that lets you test your software across any environment. This tutorial has just scratched the surface of the Docker world.

[1]: https://docs.docker.com/installation/
[2]: https://hub.docker.com
[3]: http://redis.io/
[4]: https://hub.docker.com/
[5]: https://dockunit.io

  
