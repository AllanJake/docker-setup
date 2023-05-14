Update system

`sudo apt update`

`sudo apt upgrade -y`

Prerequisets

`sudo apt install -y ca-certificates curl gnupg`

Install Docker

`sudo install -m 0755 -d /etc/apt/keyrings`

`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg`

`sudo chmod a+r /etc/apt/keyrings/docker.gpg`


`echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`

`sudo apt update`

`sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`


Set up user group so docker can be executed without sudo

`sudo usermod -aG docker jake`

`newgrp docker`

Set up nvidia docker

`distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list`

`sudo apt update`

`sudo apt-get install -y nvidia-container-toolkit`

`sudo nvidia-ctk runtime configure --runtime=docker`

`sudo systemctl restart docker`

`docker run --rm --runtime=nvidia --gpus all nvidia/cuda:11.6.2-base-ubuntu20.04 nvidia-smi`
