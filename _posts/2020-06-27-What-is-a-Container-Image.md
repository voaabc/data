---
layout: post
title: What is a Container Image?
categories: Docker
tags: Docker Image
---

* content
{:toc}




## Container Image

A container image is a tar file containing tar files. Each of the tar file is a layer. Once all tar files have been extract into the same location then you have the container's filesystem.

This can be explored via Docker. Pull the layers onto your local system.

`docker pull redis:3.2.11-alpine`

Export the image into the raw tar format.

`docker save redis:3.2.11-alpine > redis.tar`

Extract to the disk

`tar -xvf redis.tar`

All of the layer tar files are now viewable.

`ls`

The image also includes metadata about the image, such as version information and tag names.

`cat repositories`

`cat manifest.json`

Extracting a layer will show you which files that layer provides.

`tar -xvf da2a73e79c2ccb87834d7ce3e43d274a750177fe6527ea3f8492d08d3bb0123c/layer.tar`


        

## Creating Empty Image

As an image is just a tar file, an empty image can be created using the command below.

`tar cv --files-from /dev/null | docker import - empty`

By importing the tar, the additional metadata will be created.

`docker images`

However, as the container doesn't contain anything, it can't start a process. 

        
        

## Creating Image without Dockerfile

The previous idea of importing a Tar file can be extended to create an entire image from scratch.

To start, we'll use BusyBox as the base. This will provide us with the foundation linux commands. This is defined as the *rootfs*. The *rootfs* is...

Docker provide a script to download the BusyBox rootfs at [https://github.com/moby/moby/blob/a575b0b1384b2ba89b79cbd7e770fbeb616758b3/contrib/mkimage/busybox-static](https://github.com/moby/moby/blob/a575b0b1384b2ba89b79cbd7e770fbeb616758b3/contrib/mkimage/busybox-static)

`curl -LO https://raw.githubusercontent.com/moby/moby/a575b0b1384b2ba89b79cbd7e770fbeb616758b3/contrib/mkimage/busybox-static && chmod +x busybox-static`

`./busybox-static busybox`

Running the script will download the rootfs and main binaries.

`ls -lha busybox`

The default Busybox rootfs doesn't include any version information so let's created a file.

`echo KatacodaPrivateBuild > busybox/release`

As before, the directory can be converted into a tar and automatically imported into Docker as an image.

`tar -C busybox -c . | docker import - busybox`

This can now be launched as a container.

`docker run busybox cat /release`


    