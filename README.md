# Overview
This project was created due to me having multiple issues getting the python/python3 module running inside my dockerized Freeradius Server. (rlm_python and rlm_python3)
Originally I was going to spend more time developing this for the community to serve as way to enable both python2 and python3, but since python2 is no longer in support, lets just do python3.

My version of this runs on a Ubuntu18.04 docker container. I have instructions below to rebuild using any container you would like, they should work for any OS that freeradius supports. Look below if you don't like Ubuntu. The entire image is a modified version of the Freeradius Github and should stay up to date with that Github as it pulls and builds from the master branch.

# Getting Started
There are two ways you can go about loading the Freeradius Python3 image. 
The first option is easier, but will only allow you to use Ubuntu18.04 as your image.
The second option is more work, but you should be able to use any OS flavor with your image.

## Easy Option
Use my image and call it a day. The docker hub Dockerfile I have uploaded here is monolithic and not fun to maintain. I will not be maintaining the Dockerfile directly. The Dockerfile takes an Ubuntu18.04 image, adds dependencies, and builds from source.

### 1.) To get the docker image.
```
docker pull vondy-games/freeradius-python3
```


### 2.) Collecting configurations.
I highly recommend taking this step. There does not seem to be an easy way to collect active configurations for Raddb without spinning up an instance yourself. The files from the freeradius Github do not correspond directly with the files from the Freeradius server. (I give my collection image privileged for simplicity, we will be deleting it shortly). Once you have these files, edit them to your leasure to enable any python modules you need. Here is a tutorial link to enable authorization, just replace python with python3 anywhere you see python.
```
docker run -t -d --name=radius-collect --privileged vondy-games/freeradius-python3
docker cp <container>:/opt/freeradius/etc/raddb <Where-ever you want to store dockerfile contexts>
```

### 3.) Making your own dockerfile.
If you need an example, please look at this github for Dockerfile.example.
Your dockerfile will be copying your raddb configuration to your docker raddb directory and installing any python packages you need.
In your dockerfile, add the folling lines:
```
COPY /raddb /opt/freeradius/etc/raddb
RUN pip3 install <enter your packages here>
```

From here, if you have never worked with Freeradius before, I recommend following the freeradius tutorial for docker.

## Difficult Option
Essentially, I will walk you through what I did to get this working.
1. Navigate to the [FreeRADIUS Github](https://github.com/FreeRADIUS/freeradius-server).
2. In the [/scripts/docker Directory](https://github.com/FreeRADIUS/freeradius-server/tree/master/scripts/docker) find the OS flavor you like.
3. Copy the Dockerfile.deps and Dockerfile files to your docker host.
4. Modify the Dockefile.deps (See Dockerfile.deps in this directory for an example).
    1. Add the python3-pip and python3-dev packages to you base image. This may take some googling to find the name for your OS, Ubuntu is below.
    ```
    apt-get install -y python-dev python3-dev python-pip python3-pip
    ```
    2. Add the configuration option that enables the python3 module to be included duruing make. Below is what your configuration command should look like.
    ```
    RUN CC=${cc} ./configure --prefix=/opt/freeradius --with-rlm-python3-config-bin=python3-config
    ```
5. Modify the Dockerfile (See Dockerfile in this directory for an example). You only have to point the from to the dependency image, and link the python module to enable it.
```
FROM freeradius/deps
RUN ln -s /opt/freeradius/etc/raddb/mods-available/python3 /opt/freeradius/etc/raddb/mods-enabled/python3
```
6. Build the dependency dockerfile and build the freeradius image.
```
docker build -f Dockefile.deps -t freeradius/deps .
docker build -f Dockerfile.server -t freeradius/server .
```
6. Start for #2 of the "Easy Option" and follow the instructions from there.


# WIP - I have this running locally so I know this works. In the unlikely circumstance that someone sees this Git before I upload the project, I apologize, it should be up by the end of the week, just cleaning some things up.
