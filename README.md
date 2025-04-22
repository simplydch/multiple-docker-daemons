## Running multiple Docker Daemons in the same machine

#### Forked from https://github.com/antIggl/multiple-docker-daemons (Big Thanks) with minor adjustments. This allows me to run additional docker containers on same device as a home assistant supervised install without Home Assistant flagging the install and unsupported (e.g. "System is unsupported because additional software outside the Home Assistant ecosystem has been detected").

#### I added a second containerd daemon for the second docker daemon, this prevents (probably harmless) cross talk between the docker daemons (i.e. requests to restart containers running on the alternative daemon).  

-------------------------------------

We separate the number of Docker daemons in the machine
docker : the default installation
docker-apps : the isolated docker installation for applications

### Goal
The main goal is to have separate docker daemons running on the same machine isolated.
Our use case was the jenkins server. I used to run my jenkins server in a docker container and I wanted to isolate the docker that uses jenkins to launch containers.

### Procedure

First clone this repository and cd into it.

1. Create the service file for the daemon
Edit appropriately docker-apps.service file and place it in ```/etc/systemd/system/``` directory
``` bash
sudo cp ./docker-apps.service /etc/systemd/system/docker-apps.service
```
2. Create the socket file
``` bash
sudo cp ./docker-apps.socket /etc/systemd/system/docker-apps.socket
```
3. Create the new daemons data-root \& run directories
``` bash
sudo mkdir /var/lib/docker-apps /var/lib/containerd-dockers-apps /run/containerd-dockers-apps
```
4. Place the configuration file in ```/etc/docker/```

Note: The bridge ip address (bip) is set to 172.25.0.1/16, different from Docker's default (172.17.0.1/16), to avoid ip conflicts

```bash
sudo cp ./docker-daemon-apps.json /etc/docker/docker-apps.json
```
5. Create service file for containerd daemon
```bash
sudo cp ./containerd-docker-apps.service /etc/systemd/system/containerd-docker-apps.service
```
6. Create containerd configuration file
```bash
sudo cp ./config-docker-apps.toml /etc/containerd/config-docker-apps.toml
```
7. Enable new services
``` bash
sudo systemctl enable docker-apps.service
sudo systemctl enable docker-apps.socket
sudo systemctl enable containerd-docker-apps

```
8. Run new services
``` bash
sudo systemctl start docker-apps.socket
sudo systemctl start docker-apps.service
sudo systemctl start containerd-docker-apps
```

9. Create alias so "docker-apps" can be used just as "docker" would
``` bash
sudo echo alias docker-apps='docker -H unix:///var/run/docker-apps.sock' >> ~/.bashrc
source ~/.bashrc
```

### Additional Notes

1\)

I experienced issues with port mapping to containers on the second Docker daemon at system reboot which was only resolved by restarting the container.  This seems to be an issue with the container starting before port mapping is set-up completely.  I'm not sure if this is to do with the second daemon or not.  It can be resolved by either setting up the container to run as a service with appropriate after conditions.  Or adding a delay to the container starting via a Docker compose YAML file.
### Contributors
* Antonios Inglezakis (@antIggl)
