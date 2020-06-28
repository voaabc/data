---
layout: post
title: Docker Foundations 03 Building Container Images 
categories: Docker
tags: Docker
---

* content
{:toc}







#### Step 1 - Base Images

All Docker images start from a base image. A base image is the 
same images from the Docker Registry which are used to start containers.
 Along with the image name, we can also include the image tag to 
indicate which particular version we want, by default, this is latest.

These base images are used as the foundation for your additional 
changes to run your application. For example, in this scenario, we 
require NGINX to be configured and running on the system before we can 
deploy our static HTML files. As such we want to use NGINX as our base 
image.

Dockerfile's are simple text files with a command on each line. To define a base image we use the instruction *FROM <image-name>:<tag>*

#### Task: Creating a Dockerfile

The first line of the Dockerfile should be *FROM nginx:1.11-alpine*

Make the change in the Dockerfile editor. Within the environment, a 
new Dockerfile will be created with the contents of the editor.

#### Caution

It's tempting to use the tag *:latest* however this can result
 in you building your image against a version which you were not 
expecting. We recommend that you always use a particular version number 
as your tag and manage the updating yourself.

    Copy to Editor: 
    FROM nginx:1.11-alpine


        
        
        

#### Step 2 - Running Commands

With the base image defined, we need to run various commands to
 configure our image. There are many commands to help with this, the 
main commands two are *COPY* and *RUN*.

*RUN <command>* allows you to execute any command as you
 would at a command prompt, for example installing different application
 packages or running a build command. The results of the RUN are 
persisted to the image so it's important not to leave any unnecessary or
 temporary files on the disk as these will be included in the image.

*COPY <src> <dest>* allows you to copy files from 
the directory containing the Dockerfile to the container's image. This 
is extremely useful for source code and assets that you want to be 
deployed inside your container.

#### Task

A new *index.html* file has been created for you which we want
 to serve from our container. On the next line after the FROM command, 
use the COPY command to copy index.html into a directory called 
/usr/share/nginx/html

#### Protip

If you're copying a file into a directory then you need to specify the filename as part of the destination.

    Copy to Editor:
    COPY index.html /usr/share/nginx/html/index.html


        
        
        

#### Step 3 - Exposing Ports

With our files copied into our image and any dependencies 
downloaded, you need to define which port application needs to be 
accessible on.

Using the *EXPOSE <port>* command you tell Docker which 
ports should be open and can be bound to. You can define multiple ports 
on the single command, for example, *EXPOSE 80 433* or *EXPOSE 7000-8000*

#### Task

We want our web server to be accessible via port 80, add the relevant *EXPOSE* line to the Dockerfile.

    Copy to Editor:
    EXPOSE 80


        
        
        

#### Step 4 - Default Commands

With the Docker image configured and having defined which ports
 we want accessible, we now need to define the command that launches the
 application.

The *CMD* line in a Dockerfile defines the default command to 
run when a container is launched. If the command requires arguments then
 it's recommended to use an array, for example ["cmd", "-a", "arga 
value", "-b", "argb-value"], which will be combined together and the 
command `cmd -a "arga value" -b argb-value` would be run.

#### Task

The command to run NGINX is `nginx -g daemon off;`.  Set this as the default command in the Dockerfile.

#### Protip

An alternative approach to *CMD* is *ENTRYPOINT*. While a CMD can be overridden when the container starts, a *ENTRYPOINT* defines a command which can have arguments passed to it when the container launches. 

In this example, NGINX would be the entrypoint with -g daemon off; the default command.

    Copy to Editor:
    CMD ["nginx", "-g", "daemon off;"]


        
        
        

#### Step 5 - Building Containers

After writing your Dockerfile you need to use `docker build` to turn it into an image. The *build*
 command takes in a directory containing the Dockerfile, executes the 
steps and stores the image in your local Docker Engine. If one fails 
because of an error then the build stops.

#### Task

Using the `docker build` command to build the image. You can give the image a friendly name by using the *-t <name>* option.

#### Protip

You can use `docker images` to see a list of the images on your local machine.

`docker build -t my-nginx-image:latest .`


        
        
        

#### Step 6 - Launching New Image

With the image successfully created, you can now launch the container in the same way we described in the first scenario.

#### Task

Launch an instance of your newly built image using either the ID 
result from the build command or the friendly name you assigned it.

NGINX is designed to run as a background service so you should include the option *-d*.  To make the web server accessible, bind it to port 80 using *p 80:80*

For example:

`docker run -d -p 80:80 <image-id|friendly-tag-name>`

You can access the launched web server via the hostname *docker*. After launching the container, the command `curl -i http://docker` will return our index file via NGINX and the image we built.

#### Protip

You can check the container is running using `docker ps`

`docker run -d -p 80:80 my-nginx-image:latest`


      