# Docker Practice Notes
---

## Useful commands

`docker run <image-name>`
Here docker has started the container, run a command, and then immediately shut down the container. Good for when there is one main process that needs to be executed.

`docker run --rm -ti <image-name> sleep 5`
Running a docker container in this method runs a command against the  container with a specified image and then removes the container all in 1 command.

The `-it` or `-ti` option means it is an interactive container and loads in terminal drivers

`--rm -ti` makes it so that the file system is removed when the docker container is closed, good for rapid testing.

`docker run -ti <image-name> -c "sleep 3; echo all done"`
Spins up the container based off the passed in image and executes two commands one after the other, sleeps for 3 seconds and then echos a string.

`docker run -ti <image-name> bash`
This command takes a docker image and then spins up a container with that image as well as an interactive shell. In this case the shell is bash but other shells can be used.

`docker run -d -ti <image-name> bash`
Runs a container based off a specified image in a detached mode 

`docker attach <container-name>`
Allows you to reattach to a detached container

`docker exec -ti <container-name> bash`
Executes a new process on an existing container. In this case it is executing the bash process to get a new shell.

`docker commit <container-name> <new-image-name>`
`docker commit <container-name> <new-image-name>:<tag>`
Example: `docker commit 18f436781306 hello-server:v2.1`
Takes an existing docker container (`<container-name>`) and makes a new image out of it while tagging it with a new name that is passed in (`<new-image-name>`). If you then do `docker images` you will see the new image created. You can also assign a tag to the name of the image so when you look at the image the tags information will also be presented. It will default to latest if not specified.

`docker logs <container-name>`
Keeps output of containers. Very convenient but do not let logs get too massive.

`docker kill <container-name>`
Stops the container.
`docker rm <container-name>`
Removes files associated with the container

`docker pull`
This command gets run automatically when `docker run` is used but you can use this to pull images manually. 

`exit`
exits the shell if you are inside a docker container

`docker images`
Displays docker images on the system

`docker ps`
Shows running docker processes

`docker ps -l`
Shows information on the last container that exited.


---

# Resource Control

#### Memory limits

`docker run --memory <maximum-allowed-memory> <image-name> command`

#### CPU Limits

`docker run --cpu-shares` (Relative to other containers)
`docker run --cpu-quota` (To limit it in general)

#### Volumes

- Persistent Volumes
- Ephemeral Volumes

NOT PART OF IMAGES
Similar to shared folders with virtual machines.

`docker run -ti -v /path/to/share/folder:/share-folder ubuntu bash`
You can use this command to share an existing folder with a docker container
`docker run -ti -v /path/to/file:/share-file ubuntu bash`
Same as previous command but shares a file instead of a folder



---

# Exposing ports

`docker run --rm -ti -p 45678:45678 -p 45679:45679 --name echo-server ubuntu:14.04 bash`
This command gets a viable ubuntu instance, names it `echo server` and then exposes 2 ports. The first set of ports being exposed are `45678:45678`. This connects the port 45678 on the docker container to the PCs port 45678 so so that when the port is accessed it goes to the docker container. The same is done with port `45679:45679` because this example requires two ports open.

`nc -lp 45678 | nc -lp 45679`
`-l` (Listens for connections instead of using connect mode)
`-p` (Binds the Netcat source port to `port`)
This command listens to both ports, takes information entered into the system from one port and spits it out on the other port. 

Then on two different terminals you would run `nc localhost 45678` and `nc localhost 45679` and this would make the two terminals connected and information would be sent between them.

`host.docker.internal`
This is the keyword used when referencing the machine that is hosting the docker container. This would be the same exact thing as the previous example except:
- All terminals are in docker containers now
    - Previous example only the echo-server was in a container
- Instead of running localhost we have `nc host.docker.intenal {port}`

#### Dynamic Port Assignment

`docker run --rm -ti -p 45678 -p 45679 --name echo-server ubuntu:14.04 bash`
This command is the same as the static version except we are dynamicaly assigned a port on our local machine to be associated with the command specified docker container port.

`docker port echo-server`
This would show the ports on the `echo-server` and what they are connected to on the host.

#### UDP

Previous examples have been using TCP however UDP can also be used.

`docker run --rm -ti -p 45678/udp --name echo-server ubuntu:14.04 bash`
`nc -ulp 45678`

`docker port echo-server`
`nc -u localhost {dynamically assigned port}` (-u is for UDP)


# Networking

#### Virtual Networks
`docker network ls`
Shows the existing networks on the system for docker containers.
Defaults:
- Bridge: default network, bidged networking through host
- Host: Container is in place of host, no network isolation
- None: no network

`docker network create learning`
This command uses the built in docker function to create a new network with the name `learning` for example purposes

`docker run --rm -ti --net learning --name cat-server ubuntu:14.04 bash`
This will spin up a ubuntu container in the learning network with the container name `cat-server`

`docker run --rm -ti --net learning --name dog-server ubuntu:14.04 bash`

If these 2 instances are made then the two machines can picg eachother since they are on the same network.

`docker network connect catsonly cat-server`
This would connect an existing `cat-server` container to a `catsonly` network you created. Connect existing container to new network.

