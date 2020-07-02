---
layout: post
title: Docker Foundations 04 Dockerizing Node.js applications 
categories: Docker
tags: Docker
---

* content
{:toc}


This scenario continues to explore how to build and deploy your applications as a Docker container. The previous scenario covered deploying a static HTML website. This scenario explores how to deploy a Node.js application within a container.

The environment is configured with access to a personal Docker instance, and the code for a default Expressjs application is in the working directory. To view the code use ls and cat <filename> or use the editor.

The machine name Docker is running on is called docker. If you want to access any of the services then use docker instead of localhost or 0.0.0.0.

### Step 1 - Base Image

As we described in the previous scenario, all images started 
with a base image, ideally as close to your desired configuration as 
possible. Node.js has pre-built images available with tags for each 
released version.

The image for Node 10.0 is node:10-alpine. This is an Alpine-based 
build which is smaller and more streamlined than the official image.

Alongside the base image, we also need to create the base directories of where the application runs from. Using the *RUN <command>* we can execute commands as if they're running from a command shell, by using `mkdir` we can create the directories where the application will execute from. In this case, an ideal directory would be */src/app* as the environment user has read/write access to this directory.

We can define a working directory using *WORKDIR <directory>* to ensure that all future commands are executed from the directory relative to our application.

#### Task: Define Base Environment

Set the *FROM <image>:<tag>*, *RUN <command>* and *WORKDIR <directory>* on separate lines to configure the base environment for deploying your application.

    Copy to Editor:
    
    FROM node:10-alpine
    RUN mkdir -p /src/app
    WORKDIR /src/app
    


        

### Step 2 - NPM Install

In the previous set, we configured the foundation of our 
configuration and how we want the application to be deployed. The next 
stage is to install the dependencies required to run the application. 
For Node.js this means running NPM install.

To keep build times to a minimum, Docker caches the results of 
executing a line in the Dockerfile for use in a future build. If 
something has changed, then Docker will invalidate the current and all 
following lines to ensure everything is up-to-date.

With NPM we only want to re-run `npm install` if something within our *package.json* file has changed. If nothing has changed then we can use the cache version to speed up deployment. By using *COPY package.json <dest>* we can cause the *RUN npm install* command to be invalidated if the *package.json*
 file has changed. If the file has not changed, then the cache will not 
be invalidated, and the cached results of the npm install command will 
be used.

#### Task: Add Dockerfile Lines

The following two lines are required in order Dockerfile to run npm install.

    Copy to Editor:
    
    COPY package.json /src/app/package.json
    RUN npm install
    

Copy the lines to the Dockerfile now so they can be used in the build later.

#### Protip

If you don't want to use the cache as part of the build then set the option *--no-cache=true* as part of the `docker build` command.

    Copy to Editor:
    
    COPY package.json /src/app/package.json
    RUN npm install
    


### Step 3 - Configuring Application

After we've installed our dependencies, we want to copy over 
the rest of our application's source code. Splitting the installation of
 the dependencies and copying out source code enables us to use the 
cache when required.

If we copied our code before running *npm install* then it 
would run every time as our code would have changed. By copying just 
package.json we can be sure that the cache is invalidated only when our 
package contents have changed.

#### Task: Deploy Application

Create the desired steps in the Dockerfile to finish the deployment of the application.

We can copy the entire directory where our Dockerfile is using *COPY . <dest dir>*.

Once the source code has been copied, the ports the application requires to be accessed is defined using *EXPOSE <port>*.

Finally, the application needs to be started. One neat trick when using Node.js is to use the *npm start* command. This looks in the *package.json* file to know how to launch the application saving duplication of commands.

In the next step, we'll build and launch the image.

    Copy to Editor:
    
    COPY . /src/app
    EXPOSE3000CMD [ "npm", "start" ]
    

Show Solution
        
          Continue
        

### Step 4 - Building & Launching Container

To launch your application inside the container you first need to build an image.

#### Example: Build & Launch

The command to build the image is `docker build -t my-nodejs-app .`

The command to launch the built image is `docker run -d --name my-running-app -p 3000:3000 my-nodejs-app`

#### Testing Container

You can test the container is accessible using curl. If the 
application responds then you know that everything has correctly 
started.

`curl http://docker:3000`


        

### Step 5 - Environment Variables

Docker images should be designed that they can be transferred 
from one environment to the other without making any changes or 
requiring to be rebuilt. By following this pattern you can be confident 
that if it works in one environment, such as staging, then it will work 
in another, such as production.

With Docker, environment variables can be defined when you launch the
 container. For example with Node.js applications, you should define an 
environment variable for *NODE_ENV* when running in production.  

Using *-e* option, you can set the name and value as *-e NODE_ENV=production*

#### Example

`docker run -d --name my-production-running-app -e NODE_ENV=production -p 3000:3000 my-nodejs-app`

