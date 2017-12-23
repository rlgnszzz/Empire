---
date: 2016-03-09T00:11:02+01:00
title: Getting started
weight: 10
---
## Supported Platforms

We only support a few base platforms at this time:

* Debian / Kali
* Ubuntu LTS
* Fedora

## Initial Setup

Run the **./setup/install.sh** script. This will install the few dependencies and run the **./setup/setup_database.py** script. The setup_database.py file contains various setting that you can manually modify, and then initializes the ./data/empire.db backend database. No additional configuration should be needed- hopefully everything works out of the box.

Running `./empire` will start Empire, and `./empire --debug` will generate a verbose debug log at .**/empire.debug**. Running `./empire --debug 2` will provide verbose output to empire console. The included **./setup/reset.sh** will reset/reinitialize the database and launch Empire in debug mode. There's also a detailed "Empire Tips and Tricks" [post up here](http://enigma0x3.net/2015/08/26/empire-tips-and-tricks/).

## Docker Setup

A major benefit to using docker is reducing the need to handle package management and allowing you to easily upgrade between releases of Empire. If you are unfamiliar with Docker I highly recommend the Quick start tutorial. There are two different ways of using Docker for Empire, the first being using Docker-Hub to pull images.

### Docker Pull Setup
Simply use the following command, this will pull the `latest` image/version from the hub. In total after all compiled libs and src it currently sits at 400MB compressed:
```
docker pull empireproject/empire
```
Once your image has been pulled down to the local host, we need setup a persistent storage volume to prevent the loss of data between stoping and starting. One of the powerful parts of Docker is now you have a base/clean image to always work off and all you need to do is pull the newest image and run with our mounted volume. To do this we must create the volume:
```
docker create -v /opt/Empire --name data empireproject/empire
```
Now to run the container we simply run in interactive mode, this may change in the future but seems to be the simplest method to ensure maximum usability.
```
docker run -ti --volumes-from data empireproject/empire bash
```
{{< note title="Note" >}}
This will enable only "Local" networking and wont allow you to bind to the host networking. This is great for local testing etc.. but NOT ideal for production of course.
{{< /note >}}

To fix this issue we simply expose a range of ports and map them 1:1:
```
docker run -ti --volumes-from data -p 80:80 empireproject/empire bash
```
or
```
docker run -ti --volumes-from data -p 80-443:80-443 empireproject/empire bash
```
Finally since we have a fresh new image all we need to do is simply start our new DB, and required PKI. We do this by simply running the `reset.sh` script:
```
root@e7059c42661b:/opt/Empire/setup# ./reset.sh
```
{{< warning title="Don't be dumb" >}}
DO NOT share or publish your empire-chain.pem, empire-priv.key, empire.db, by default these are cleaned within the Dockerfile for you at build.
{{< /warning >}}

### Dockerfile Build Instructions
So you don't trust us to build your image. Fair enough, I truly don't blame you! The following steps should be followed to build the image from scratch or even host your self.
```
git clone https://github.com/EmpireProject/Empire.git
```
Once you have the required source running the following commands to build the `Dockerfile` which is found in the root of Empire.
```
./build.sh
```
This will set the following ENV variables, and run the docker build process:
```
#!/usr/bin/env bash
set -ex
# SET THE FOLLOWING VARIABLES
# docker hub username
USERNAME=empireproject
# image name
IMAGE=empire
docker build -t $USERNAME/$IMAGE:latest .
```
If you don't want to pull the entire repo local this is also fine, just make sure to place this `Dockerfile` in the current build directory:
```
# NOTE: Only use this when you want to build image locally
#       else use `docker pull empireproject\empire:{VERSION}`
#       all image versions can be found at: https://hub.docker.com/r/empireproject/empire/

# -----BUILD COMMANDS----
# 1) build command: `docker build -t empireproject/empire .`
# 2) create volume storage: `docker create -v /opt/Empire --name data empireproject/empire`
# 3) run out container: `docker run -ti --volumes-from data empireproject/empire /bin/bash`

# -----RELEASE COMMANDS----
# 1) `USERNAME=empireproject`
# 2) `IMAGE=empire`
# 3) `git pull`
# 4) `export VERSION="$(curl -s https://raw.githubusercontent.com/EmpireProject/Empire/master/lib/common/empire.py | grep "VERSION =" | cut -d '"' -f2)"`
# 5) `docker tag $USERNAME/$IMAGE:latest $USERNAME/$IMAGE:$VERSION`
# 1) `docker push $USERNAME/$IMAGE:latest`
# 2) `docker push $USERNAME/$IMAGE:$VERSION`

# -----BUILD ENTRY-----

# image base
FROM ubuntu:16.04

# author
MAINTAINER Killswitch-GUI

# extra metadata
LABEL version="1.0"
LABEL description="Dockerfile base for Empire server."

# expose ports for Empire C2 listerners
# EXPOSE 80,443

# update repo sources
RUN apt-get clean
RUN apt-get update

# build depends
RUN apt-get install -qy apt-utils
RUN apt-get install -qy git
RUN apt-get install -qy wget
RUN apt-get install -qy curl
RUN apt-get install -qy sudo
RUN apt-get install -qy lsb-core
RUN apt-get install -qy python2.7
RUN apt-get install -qy python-pip

# cleanup image
RUN apt-get -qy autoremove

# build empire
RUN git clone https://github.com/EmpireProject/Empire.git /opt/Empire
ENV STAGING_KEY=RANDOM
RUN cd /opt/Empire/setup/ && ./install.sh

# -----END OF BUILD-----
```
If you are going to host your own private Docker repo, this is easily done with editing the `.release.sh` script. Simply replace the push location ENV variables:
```
#!/usr/bin/env bash
set -ex

# SET THE FOLLOWING VARIABLES
USERNAME=**CHANGME**
IMAGE=empire
VERSION="$(curl -s https://raw.githubusercontent.com/EmpireProject/Empire/master/lib/common/empire.py | grep "VERSION =" | cut -d '"' -f2)"

# UPDATE THE SOURCE CODE
git pull

# ALERT VERSION
echo "Building Version: $VERSION"

# START BUILD
./.build.sh

# DOCKER TAG/VERSIONING
docker tag $USERNAME/$IMAGE:latest $USERNAME/$IMAGE:$VERSION

# PUSH TO DOCKER HUB
docker push $USERNAME/$IMAGE:latest
echo "Docker image pushed: $USERNAME/$IMAGE:latest"
docker push $USERNAME/$IMAGE:$VERSION
echo "Docker image pushed: $USERNAME/$IMAGE:$VERSION"
```
