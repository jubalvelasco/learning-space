# Docker

## Installation

See Docs For Updates : https://docs.docker.com/engine/install/ubuntu/

### 1. Add Docker's official GPG key:

```
sudo apt-get update
```

```
sudo apt-get install ca-certificates curl
```

```
sudo install -m 0755 -d /etc/apt/keyrings
```


```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```

```
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### 2. Add the repository to Apt sources:

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
sudo apt-get update
```

### 3. Install

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Setup After Installation

### 0. See If Docker Services are enabled

```
sudo systemctl status docker.service
sudo systemctl status containerd.service
```

### 1. Enable Docker Service If Not Enabled When Booting

```
sudo systemctl enable docker.service
sudo systemctl enable containerd.service

# to disable
sudo systemctl disable docker.service
sudo systemctl disable containerd.service
```

### 2. Add docker usergroup

The `docker` group is usually already created by the package installation; if it is, this command just reports that it already exists.

```
sudo groupadd docker
```

### 3. Add user to user group

```
sudo usermod -aG docker $USER
```

NOTE: members of the `docker` group effectively have root-level privileges on the machine, so only add trusted users.

Log out and log back in so that your group membership is re-evaluated

You can also run the following command to activate the changes to groups:

```
newgrp docker
```

### 4. Verify that you can run docker commands without sudo.

```
docker run hello-world
```
