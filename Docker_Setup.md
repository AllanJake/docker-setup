Update system
----
```
sudo apt update
sudo apt upgrade -y
```

Prerequisets
----
```
sudo apt install -y ca-certificates curl gnupg
```

Install Docker
----
```
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```

```
sudo apt update
RUNLEVEL=1 sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```


Set up user group so docker can be executed without sudo
----
```
sudo usermod -aG docker jake
newgrp docker
```

Set up nvidia docker
----
```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

```
sudo apt update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
docker run --rm --runtime=nvidia --gpus all nvidia/cuda:11.6.2-base-ubuntu20.04 nvidia-smi
```

Configuring Docker to work with your GPU(s)
----

The first step is to identify the GPU(s) available on your system. 
Docker will expose these as 'resources' to the swarm. 
This allows other nodes to place services (swarm-managed container deployments) on your machine.  

_These steps are currently for NVidia GPUs._

Docker identifies your GPU by its Universally Unique IDentifier (UUID). 
Find the GPU UUID for the GPU(s) in your machine. 

```
nvidia-smi -a
```

A typical UUID looks like `GPU-45cbf7b3-f919-7228-7a26-b06628ebefa1`. 
Now, only take the first two dash-separated parts, e.g.: `GPU-45cbf7b3`.

Open up the Docker engine configuration file, typically at `/etc/docker/daemon.json`.


Add the GPU ID to the `node-generic-resources`. 
Make sure that the `nvidia` runtime is present and set the `default-runtime` to it.
Make sure to keep other configuration options in-place, if they are there. 
Take care of the JSON syntax, which is not forgiving of single quotes and lagging commas. 

```json
{
  "runtimes": {
    "nvidia": {
      "path": "/usr/bin/nvidia-container-runtime",
      "runtimeArgs": []
    }
  },
  "default-runtime": "nvidia",
  "node-generic-resources": [
    "NVIDIA-GPU=GPU-45cbf7b"
    ]
}
```

Now, make sure to enable GPU resource advertisting by adding or uncommenting the following in `/etc/nvidia-container-runtime/config.toml`

```
swarm-resource = "DOCKER_RESOURCE_GPU"
```

Restart the service. 

```
sudo systemctl restart docker.service
```

Initializing the Docker Swarm
----

Initialize a new swarm on a manager-to-be. 

```
docker swarm init
```

Add new nodes (slaves), or manager-nodes (shared masters). 
Run the following command on a node that is already part of the swarm: 

```
docker swarm join-token (worker|manager)
```

Then, run the resulting command on a member-to-be.

Show who's in the swarm: 
```
docker node ls
```
