
# Practical Container Orchestration 
---
# Docker Labs
---

![Docker Logo](./images/docker2.png)


## Table of Contents

  * [Prerequisites](#prerequisites)
  * [Lab1 - Working with Docker](#lab1---working-with-docker)
    + [1. Check Docker version](#1-check-docker-version)
    + [2. Run your first container](#2-run-your-first-container)
    + [3. Rerun "hello-world".](#3-rerun--hello-world-)
    + [4.  `docker images` will show us that image.](#4---docker-images--will-show-us-that-image)
    + [5. From where was the `hello-world` image pulled?](#5-from-where-was-the--hello-world--image-pulled-)
    + [6. Container](#6-container)
    + [7. Here's how you would see the running container.](#7-here-s-how-you-would-see-the-running-container)
    + [8. An image can be run multiple times.](#8-an-image-can-be-run-multiple-times)
    + [9. Now we have two containers running the couchdb database.](#9-now-we-have-two-containers-running-the-couchdb-database)
    + [10. The containers look similar, but they have unique names and unique ids.](#10-the-containers-look-similar--but-they-have-unique-names-and-unique-ids)
    + [11. Stop the other container and see what is running.](#11-stop-the-other-container-and-see-what-is-running)
    + [12. Notice the image still exists.](#12-notice-the-image-still-exists)
    + [13. Did you forget about the hello-world image?](#13-did-you-forget-about-the-hello-world-image-)
    + [14. Oops, we can't delete that image until we delete the "couchdb" container.](#14-oops--we-can-t-delete-that-image-until-we-delete-the--couchdb--container)
    + [15. Delete the couchdb container, delete the couchdb image, and make sure it is gone. You can leave hello-world.](#15-delete-the-couchdb-container--delete-the-couchdb-image--and-make-sure-it-is-gone-you-can-leave-hello-world)
  * [Lab 2: Building Docker Images](#lab-2--building-docker-images)
    + [1. Our First Dockerfile](#1-our-first-dockerfile)
    + [2. Building a web server](#2-building-a-web-server)
  * [Conclusion](#conclusion)
    

## Prerequisites
This set of instructions requires that docker is already installed and docker commands can be run from a bash shell or a command line. You can get more information at the [Docker website](https://www.docker.com/get-docker)

>***Note:*** This lab assumes that you are running this from a "clean" environment. Clean means that you have not used docker with the images in this lab. This is important for someone who hasn't seen docker so they can see the activity as images are downloaded.

## Lab1 - Working with Docker



### 1. Check Docker version

Launch a shell or a command line and confirm that docker is installed.

The version number isn't particularly important.
However, you can see both the client (CLI) and the server (engine).

`docker version`

![Docker Version](./images/docker version.png)

If you receive an error then go back to the "PrepareLab.md" document to install Docker on your system. 

### 2. Run your first container

As with all new computer things, it is obligatory that we start with "hello-world"

`docker run hello-world`

![Run helloworld ](./images/helloworld2.png)   

 > Notice the message `Unable to find image 'hello-world:latest' locally` First you see that the image was automatically downloaded without any additional commands. Second the version `:latest` was added to the name of the image. We did not specify a version for this image.

### 3. Rerun "hello-world". 

Notice that the image is not pulled down again. It already exists locally, so it is run.

`docker run hello-world`

![Run helloworld ](./images/helloworld3.png)   


### 4.  `docker images` will show us that image.

 `docker images hello-world`

![docker images](./images/dockerimages.png)  


### 5. From where was the `hello-world` image pulled? 

Go to `https://hub.docker.com/_/hello-world/` and you can read about this image. Docker-hub is a registry that holds docker images for use. Docker-hub is not the only registry, IBM Cloud Public can serve as a docker registry. You can also have (or define) private registries.

![dockerHub](./images/dockerhubhello.png) 


### 6. Container

This image is a typical image. When an image is run it usually continues to run. 

The running image is called a **container**. Let us run a more typical image; this image contains the noSQL database "couchDB".

`docker run -d couchdb`

![couchdb](./images/couchdb.png) 

 The output above was captured while the image was still downloading from docker-hub. When the download is down you don't see anything from the container, like with hello-world. Instead you see a long hex id like `2169c6b42e5c590229c5c86f5ed3596b1b56c2366378914b082e5b000752bd34`. **This is the id of the container**.

### 7. Here's how you would see the running container. 

Notice only the first part of that long hex id is displayed. Typically this is more than enough to uniquely identify that container. `docker ps` provides information about when the container was created, how long it has been running, then name of the image as well as the name of the container. Note that each container must have a unique name. You can specify a name for each container as long as it is unique.

`docker ps | grep couchdb`

![docker ps](./images/multipledb2.png)


### 8. An image can be run multiple times. 

Launch another container for the couchdb image.

`docker run -d couchdb`
​    
![Multiple CouchDB](./images/multipledb.png)    

### 9. Now we have two containers running the couchdb database. 

Did you notice how quickly the second instance started? There was no need to download the image this time. The id of the container is show after is has started.

`docker ps | grep couchdb`

![Multiple CouchDB](./images/ps3.png)     


### 10. The containers look similar, but they have unique names and unique ids. 

Stop the most recent container and then check to see what's running.

`docker stop cde802ef4a40`

`docker ps | grep couchdb`

![Stop a container](./images/ps4.png)    


### 11. Stop the other container and see what is running.

 `docker stop c3703c8648a6`

 `docker ps | grep couchdb`

![Stop another container](./images/ps5.png)    

### 12. Notice the image still exists.

  `docker images`

  ![image is still there](./images/images5.png)    

  

### 13. Did you forget about the hello-world image? 

Go ahead and delete the couchdb image and double check that it is gone.

` docker rmi couchdb`

 ![error when removing the image](./images/images6.png)   


### 14. Oops, we can't delete that image until we delete the "couchdb" container. 
> Note the `docker ps -a` will show us all the containers, not just the ones that are running but also the ones that stop.
You will noticed that all containers that you are listing have been stopped. 

`docker ps -a | grep couchdb`

 ![Listing past images](./images/rmi.png)  

 

### 15. Delete the couchdb container, delete the couchdb image, and make sure it is gone. You can leave hello-world.

`docker rm cde802ef4a40 c3703c8648a6 bceece7628dc 676fe6a8eb5f`
​    
![removing past images](./images/rm.png)  

 `docker rmi couchdb`

 ![removing the image](./images/rmi2.png)  


`docker ps -a | grep couchdb`

 ![Listing past images](./images/images7.png)

  >***Note:*** Docker images and containers can be referenced by **name** or by **id**. 



## Lab 2: Building Docker Images

A Dockerfile is a text file that has a series of instructions on how to build your image. It supports a simple set of commands that you need to use in your Dockerfile. There are several commands supported like FROM, CMD, ENTRYPOINT, VOLUME, ENV and more. We shall look at some of them.

Let us first start with the the overall flow, which goes something like this:

1. You create a Dockerfile with the required instructions.
2. Then you will use the docker build command to create a Docker image based on the Dockerfile that you created in step 1.

With this information, let us get going.



### 1. Our First Dockerfile

First let's create a directory:

`mkdir images`

`cd images`

`nano Dockerfile`

Now type the following 2 lines:

```
FROM busybox:latest
MAINTAINER yourname
```

Replace **yourname** with your name. 

Since, a Docker image is nothing but a series of layers built on top of each other, we start with a base image. The FROM command sets the base image for the rest of the instructions. The MAINTAINER command tells who is the author of the generated images. This is a good practice. You could have taken any other base image in the FROM instruction too, for e.g. ubuntu:latest or ubunt:14.04, etc.

Now, save the file and come back to the prompt (ctrl O, enter, ctrl X).

Execute the following in the /images folder as shown below (don't forget the dot at the end):

`docker build -t myimage:latest .`

Result: 
``` console
root:[images]: docker build -t myimage:latest .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM busybox:latest
 ---> 8ac48589692a
Step 2/2 : MAINTAINER Phil
 ---> Using cache
 ---> 992cc1bd4bda
Successfully built 992cc1bd4bda
Successfully tagged myimage:latest
```
This command is used to build a Docker image. The parameters that we have passed are:

- -t is the Docker image tag. You can give a name to your image and a tag.
- The second parameter (a ‘.’) specifies the location of the Dockerfile that we created. Since we created the Dockerfile in the same folder in which we are running the docker build, we specified the current directory.

Notice the various steps that the build process goes through to build out your image.

If you run a docker images command now, you will see the myimage image listed in the output as shown below:

`docker images myimage`

```console
root:[images]: docker images myimage
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
myimage             latest              992cc1bd4bda        6 hours ago         1.15MB
```

You can now launch a container, any time via the standard docker run command:

`docker run -it myimage`

```console
root:[images]: docker run -it myimage
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 sh
    8 root      0:00 ps
/ # exit
```

We are back into the myimage shell. And you can use any shell commands like ls or ps. Type *exit* to come back to the Ubuntu shell.

`nano Dockerfile`

```console
FROM busybox:latest
MAINTAINER yourname
CMD ["date"]
```

Then build and run the myimage container:

`docker run -it myimage`

```console
root:[images]: docker run -it myimage
Mon Apr 16 12:44:10 UTC 2018
```


The CMD instruction takes various forms and when it is used individually in the file without the ENTRYPOINT command (which we will see in a while), it takes the following format:

`CMD ["executable","param1","param2"]`

So in our case, we provided the date command as the executable and when we ran a container based on the myimage now, it printed out the data.

In fact, while launching the container, you can override the default CMD by providing it at the command line as shown below. In this example, we are saying to launch the shell , thereby overriding the default CMD instruction for the Docker Image. Notice that it will lead us into the shell.

Change your Dockerfile to the following:

```console
FROM busybox
MAINTAINER myname
ENTRYPOINT ["/bin/cat"]
CMD ["/etc/passwd"]
```

Save, **Build** and re-rerun myimage. 

`docker run -it myimage`

```console
root:[images]: docker run -it myimage
root:x:0:0:root:/root:/bin/sh
daemon:x:1:1:daemon:/usr/sbin:/bin/false
bin:x:2:2:bin:/bin:/bin/false
sys:x:3:3:sys:/dev:/bin/false
sync:x:4:100:sync:/bin:/bin/sync
mail:x:8:8:mail:/var/spool/mail:/bin/false
www-data:x:33:33:www-data:/var/www:/bin/false
operator:x:37:37:Operator:/var:/bin/false
nobody:x:65534:65534:nobody:/home:/bin/false
```




### 2. Building a web server


Now, let us look at another Dockerfile shown below:

```console
FROM ubuntu
MAINTAINER Philippe
RUN apt-get update
RUN apt-get install -y nginx
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"]
EXPOSE 80
```

Here, what we are building is an image that will run the nginx proxy server for us. Look at the set of instructions and it should be pretty clear. After the standard FROM and MAINTAINER instructions, we are executing a couple of RUN instructions. A RUN instruction is used to execute any commands during the **build** process. In this case we are running a package update and then installing nginx. The ENTRYPOINT is then running the nginx executable and we are using the EXPOSE command here to inform what port the container will be listening on. Remember in our earlier chapters, we saw that if we use the -P command, then the EXPOSE port will be used by default. However, you can always change the host port via the -p parameter as needed.

Here are the steps:

`cd`

`mkdir webapp`

`cd webapp`

`nano Dockerfile`

Now copy and paste the Dockerfile text (see above). Save the file.

`docker build . -t myimage:latest`

`docker run -d -p 8081:80 --name webserver myimage`

`curl http://localhost:8081`

Results :

```console
# curl http://localhost:8081
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Or you can also open a browser on your laptop and type :

http://localhost:8081/ 



![Launching the web server](./images/nginx2.png)  

To **troubleshoot** your application, you may want to go inside the container to look at some logs or to see the processes.

`docker exec -it webserver "/bin/bash"`

> -it : this concerns the interation with the container by using a bash shell
>
> A prompt with the container id will be returned to you : **root@cc88b7536a57:/#**
>
> You can then type any kind of linux commands

Output:

```console
# docker exec -it webserver "/bin/bash"
root@cc88b7536a57:/# 
root@cc88b7536a57:/# 
root@cc88b7536a57:/# ll
total 72
drwxr-xr-x   1 root root 4096 Nov 30 14:59 ./
drwxr-xr-x   1 root root 4096 Nov 30 14:59 ../
-rwxr-xr-x   1 root root    0 Nov 30 14:59 .dockerenv*
drwxr-xr-x   1 root root 4096 Nov 30 14:58 bin/
drwxr-xr-x   2 root root 4096 Apr 24  2018 boot/
drwxr-xr-x   5 root root  340 Nov 30 14:59 dev/
drwxr-xr-x   1 root root 4096 Nov 30 14:59 etc/
drwxr-xr-x   2 root root 4096 Apr 24  2018 home/
drwxr-xr-x   1 root root 4096 Nov 12 20:54 lib/
drwxr-xr-x   2 root root 4096 Nov 12 20:55 lib64/
drwxr-xr-x   2 root root 4096 Nov 12 20:54 media/
drwxr-xr-x   2 root root 4096 Nov 12 20:54 mnt/
drwxr-xr-x   2 root root 4096 Nov 12 20:54 opt/
dr-xr-xr-x 580 root root    0 Nov 30 14:59 proc/
drwx------   2 root root 4096 Nov 12 20:56 root/
drwxr-xr-x   1 root root 4096 Nov 30 14:59 run/
drwxr-xr-x   1 root root 4096 Nov 30 14:58 sbin/
drwxr-xr-x   2 root root 4096 Nov 12 20:54 srv/
dr-xr-xr-x  13 root root    0 Nov 30 14:59 sys/
drwxrwxrwt   1 root root 4096 Nov 30 14:58 tmp/
drwxr-xr-x   1 root root 4096 Nov 12 20:54 usr/
drwxr-xr-x   1 root root 4096 Nov 30 14:58 var/
root@cc88b7536a57:/# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 14:59 ?        00:00:00 nginx: master process /usr/sbin/
www-data     7     1  0 14:59 ?        00:00:00 nginx: worker process
www-data     8     1  0 14:59 ?        00:00:00 nginx: worker process
www-data     9     1  0 14:59 ?        00:00:00 nginx: worker process
www-data    10     1  0 14:59 ?        00:00:00 nginx: worker process
www-data    11     1  0 14:59 ?        00:00:00 nginx: worker process
www-data    12     1  0 14:59 ?        00:00:00 nginx: worker process
www-data    13     1  0 14:59 ?        00:00:00 nginx: worker process
www-data    14     1  0 14:59 ?        00:00:00 nginx: worker process
root        15     0  0 15:03 pts/0    00:00:00 /bin/bash
root        27    15  0 15:03 pts/0    00:00:00 ps -ef
```



Don't forget to **exit** from the container:

`# exit`



## Conclusion

Congratulations, you have successfully completed this Containers lab!.  You've just deployed your first Docker-based web app on IBM Cloud !  In this lab, you learned how to tag and  inspect images for security vulnerabilities, and run hosted multi-container applications.

---
# End of the lab
---
# Practical Container Orchestration 
---
