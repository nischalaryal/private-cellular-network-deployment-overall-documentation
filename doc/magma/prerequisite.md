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

## Go
- Download tar file
```
wget https://linuxfoundation.jfrog.io/artifactory/magma-blob/go1.18.3.linux-amd64.tar.gz
```
- Extract to /usr/local directory
```
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.18.3.linux-amd64.tar.gz
```
- Add to PATH env variable
```
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc && exec "$SHELL"
```
- Verify
```
go version
```

## pyenv
- Update and install dependencies
```
sudo apt update -y && apt install -y make build-essential libssl-dev zlib1g-dev libbz2-dev    libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev  libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python3-openssl git
```
> ❗ For Ubuntu version 22.04 use python3-openssl instead of python-openssl
- Clone pyenv repository
```
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
```
- Configure env variables
```
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
```
```
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
```
```
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n eval "$(pyenv init -) "\nfi' >> ~/.bashrc
```
```
exec "$SHELL"
```
- Install python and set to global
```
pyenv install 3.8.10
```
```
pyenv global 3.8.10
```
- Install pip
```
sudo apt install python3-pip
```
- Install dependencies through pip
```
pip3 install ansible fabric3 jsonpickle requests PyYAML
```
