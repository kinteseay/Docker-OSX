# Docker-OSX

![Running mac osx in a docker container](/running-mac-inside-docker-qemu.png?raw=true "OSX KVM DOCKER")

Run Mac in a Docker container! Run near native OSX-KVM in Docker! X11 Forwarding!

Author: Sick.Codes https://sick.codes/

Credits: OSX-KVM project among many others: https://github.com/kholia/OSX-KVM/blob/master/CREDITS.md

Docker Hub: https://hub.docker.com/r/sickcodes/docker-osx

Pull requests, suggestions very welcome!

```

docker pull kinteseay/docker-osx

docker run --privileged -e "DISPLAY=${DISPLAY:-:0.0}" -v /tmp/.X11-unix:/tmp/.X11-unix kinteseay/docker-osx

# press ctrl G if your mouse gets stuck

# scroll down to troubleshooting if you have problems

```

# Requirements: KVM on the host
Need to turn on hardware virtualization in your BIOS, very easy to do.

Then have QEMU on the host if you haven't already:
```
# ARCH
sudo pacman -S qemu libvirt dnsmasq virt-manager bridge-utils flex bison ebtables edk2-ovmf

# UBUNTU DEBIAN
sudo apt install qemu qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virt-manager

# CENTOS RHEL FEDORA
sudo yum install libvirt qemu-kvm -y

# then run
sudo systemctl enable libvirtd.service
sudo systemctl enable virtlogd.service
sudo modprobe kvm

# reboot

```

# Start the same container later (persistent disk)

This is for when you want to run your system later.

If you don't run this you will have a new image every time.

```
# look at your recent containers and copy the CONTAINER ID
docker ps --all

# docker start the container ID
docker start abc123xyz567

# if you have many containers, you can try automate it with filters like this
# docker ps --all --filter "ancestor=kinteseay/docker-osx"

```

# Additional Boot Instructions

```

# Boot the macOS Base System

# Click Disk Utility

# Erase the biggest disk

# Partition that disk and subtract 1GB and press Apply

# Click Reinstall macOS

```

# Troubleshooting

libgtk permissions denied error, thanks @raoulh + @arsham
```
echo $DISPLAY

# ARCH
sudo pacman -S xorg-xhost

# UBUNTU DEBIAN
sudo apt install x11-xserver-utils

# CENTOS RHEL FEDORA
sudo yum install xorg-x11-server-utils

# then run
xhost +

docker run --privileged -e "DISPLAY=${DISPLAY:-:0.0}" -v /tmp/.X11-unix:/tmp/.X11-unix kinteseay/docker-osx ./OpenCore-Boot.sh
```

Alternative run, thanks @roryrjb
```docker run --privileged --net host --cap-add=ALL -v /tmp/.X11-unix:/tmp/.X11-unix -v /dev:/dev -v /lib/modules:/lib/modules kinteseay/docker-osx```

Check if your hardware virt is on
```egrep -c '(svm|vmx)' /proc/cpuinfo```

Try adding yourself to the docker group
```sudo usermod -aG docker $USER```

Turn on docker daemon
```sudo nohup dockerd &```

Check /dev/kvm permissions
```sudo chmod 666 /dev/kvm```


If you don't have Docker already

```
### Arch (pacman version isn't right at time of writing)

wget https://download.docker.com/linux/static/stable/x86_64/docker-19.03.5.tgz
tar -xzvf docker-*.tgz
sudo cp docker/* /usr/bin/
sudo dockerd &
sudo groupadd docker
sudo usermod -aG docker $USER
# run docker later
sudo nohup dockerd &

### Ubuntu

apt-get remove docker docker-engine docker.io containerd runc -y
apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg |  apt-key add -
apt-key fingerprint 0EBFCD88
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get update -y
apt-get install docker-ce docker-ce-cli containerd.io -y
sudo dockerd &
sudo groupadd docker
sudo usermod -aG docker $USER

```

# Backup the disk

your image will be stored in:

/var/lib/docker/overlay2/...../arch/OSX-KVM/home/arch/OSX-KVM/mac_hdd_ng.img
```
# find your container's root folder

docker inspect $(docker ps -q --all --filter "ancestor=docker-osx") | grep UpperDir

# In the folder from the above command, your image is inside ./home/arch/OSX-KVM/mac_hdd_ng.img

# then sudo cp it somewhere. Don't do it while the container is running tho, it bugs out.

```


# Wipe old images

```

# WARNING deletes all old images, but saves disk space if you make too many containers

docker system prune --all
docker image prune --all

```


# Instant OSX-KVM in a BOX!
This Dockerfile automates the installation of OSX-KVM inside a docker container.

It will build a 32GB Mojave Disk.

You can change the size and version using build arguments (see below).

This file builds on top of the work done by Dhiru Kholia and many others on the OSX-KVM project.


# Custom Build
```

docker build -t docker-osx:latest \
--build-arg VERSION=10.14.6 \
--build-arg SIZE=200G

docker run --privileged -v /tmp/.X11-unix:/tmp/.X11-unix docker-osx:latest

```

## Todo:
```
# persistent disk with least amount of pre-build errands.
```
