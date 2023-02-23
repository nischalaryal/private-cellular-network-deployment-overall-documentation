# Pre-requisite Software
Here, we discuss about all the necessary applications we need to setup before working with magma.

## Docker
- Remove old files
```
sudo apt-get remove docker docker-engine docker.io containerd runc
```
- Remove docker engine
```
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```
- APT update and pkg install
```
sudo apt-get update
```
```
sudo apt-get install ca-certificates curl gnupg lsb-release
```
- Add docker's official gpg key
```
sudo mkdir -m 0755 -p /etc/apt/keyrings
```
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
- Setup repository
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
 
- Update and install
```
sudo apt-get update
```
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- Managing Docker as non-root user
```
sudo groupadd docker
```
```
sudo usermod -aG docker $USER
```
```
newgrp docker
```

- After successful setup. Verify
```
sudo docker run hello-world
```

## Docker Compose V2
- Install
```
 sudo apt-get install docker-compose-plugin
```
- Verify
```
docker compose version
```

## Virtual Box

- Add gpg key
```
wget -O- https://www.virtualbox.org/download/oracle_vbox_2016.asc | gpg --dearmor | sudo tee /usr/share/keyrings/virtualbox.gpg > /dev/null 2>&1
```
- Setup repository
```
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/virtualbox.gpg] https://download.virtualbox.org/virtualbox/debian $(lsb_release -cs) contrib" | sudo tee /etc/apt/sources.list.d/virtualbox.list > /dev/null

```

- Install Vbox
```
sudo apt update  
```
```
sudo apt install virtualbox-7.0  
```

## Vagrant

- Get GPG key
```
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null 2>&1
```
- Add repository
```
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list > /dev/null
```
- Install vagrant
```
sudo apt update && sudo apt install vagrant
```
- Install necessary plugins
```
vagrant plugin install vagrant-vbguest vagrant-disksize vagrant-reload
```
- Add to .bashrc profile to make vbox default provider for vagrant
```
echo 'export VAGRANT_DEFAULT_PROVIDER="virtualbox"' >> ~/.bashrc && exec "$SHELL"
```
