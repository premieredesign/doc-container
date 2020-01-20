### Containers

```
1. Why Containers ?
- Container gives us many of the security and resources management features of VMs but without the cost of having to run a whole other operating system.
```

```
chroot - this command allows you to set the root directory of a new process. In our container use case, we just set the root directory to be where-ever the new container's new root directory should be.

1. To start Ubuntu server:
    - docker run -it --name docker-host --rm --privileged ubuntu:bionic

2. To see where what version on Ubuntu you have:
    - cat /etc/issue

3. Make a new folder and copy bash inside of it:
    - mkdir my-new-root/bin
    - cd ..
    - cp bin/bash my-new-root/bin/

4. Next you need to add the directories and packages needed to run commands that are needed:
    - # ldd bin/bash
    - command -> mkdir my-new-root/lib{,64}
    - run ldd bin/bash command again and then run
    - cp /lib/x86_64-linux-gnu/libpcre.so.3 my-new-root/lib

for list commands:
    - ldd /bin/ls
    - cp packages into my-new-root
```

### Commands

```
ps - this will show running process
ps aux - will show the tail process command

1. To Start another instance of Bash:
    - docker exec -it docker-host bash

```

### Setting up a better server:

```
1. install debootstrap:
    - apt install debootstrap -y
2. deboostrap --variant=minbase bionic /better-root

3. Unshare environments:
    - unshare --mount --uts --ipc --net --pid --fork --user --map-root-user chroot /better-root bash

    Mount environments:
        - mount -t proc none /proc
        - mount -t sysfs none /sys
        - mount -t tmpfs none /tmp

4. install cgroup htop:
    - apt install -y cgroup-tools htop

5. Create a new control group
    - cgcreate -g cpu,memory,blkio,devices,freezer:/sandboxzßxza

6. in the parent container:
    - first get the PID of the process running -> ps aux
    - cgclassify -g cpu,memory,blkio,devices,freezer:sandbox PID
    - to see what process is in the control group -> cat /sys/fs/cgroup/cpu/sandbox/tasks
    - to limit -> cgset -r cpu.cfs_period_=100000 -r cpu.cfs_quota_us=$[ 5000 * $(getconf _NPROCESSORS_ONLN) ] sandbox
    - to limit memory -> cgset -r memory.limit_in_bytes=80M sandbox
```

### Docker:

```
1. To see what docker processes you have running -> docker ps
2. To run docker from your local docker -> docker run -ti -v /var/run/docker.sock:/var/run/docker.sock --privileged --rm --name docker-host docker:18.06.1-ce
3. Next create another docker container inside of the docker container ->  docker run --rm -dit --name my-alpine alpine:3.10 sh
4. Next dump out the container which is really what an image is -> docker export -o dockercontainer.tar my-alpine

Running Docker images w/ Docker:
    - docker run --interactive --tty alpine:3.10

Detached:
    - docker run -it --detach ubuntu:bionic

Remove:
    - docker run -it --name my-alpine --rm alpine:3.10

Build:
    - First create a docker file -> Dockerfile
    - Add your build commands
    - Then run docker build -t [name] .
    - Then run it docker run [image name] -> docker run --init [name]
    - docker run --init --rm --publish 3000:3000 [name]


Inside Dockerfile:
    FROM node:12-stretch

    USER node

    COPY --chown=USER:GROUP index.js index.js

    ADD index.js index.js <- the only difference from COPY is this will go out into the network and download what you need>

    CMD ["node", "index.js"]

```
