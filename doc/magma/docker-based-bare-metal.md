# Installing Docker-based Magma AGW on Ubuntu Server and Connecting with Orchestrator

In this repository, we will go over each step of deploying docker-based Magma AGW on Ubuntu Server. I will explain the hardware, software, and network configuration utilized during this set up. We will also see how we can connect the AGW with Magma Orchestrator.

## A. Hardware Used
- AGW System 
  - Intel(R) Core(TM) i7-2720QM CPU @ 2.20GHz
  - 8Gb RAM
  - 120 Gb HDD space
 
## B. Software Used
- **Operating System** - Ubuntu Server 20.04
- **Magma Version** - v1.8.0

## C. System Setup

In this section, we'll go over how to set up the operating system and install AGW.

### C.1. Creating a bootable USB stick for Ubuntu Server

- First, I downloaded [Ubuntu Server 20.04 LTS](https://ubuntu.com/download/server) from the Ubuntu website. It is preferable to use the most recent server version. However, we must determine whether the OS version is compatible with the Magma software because we may encounter issues during installation.
- Next, I followed the [guidelines](https://ubuntu.com/tutorials/create-a-usb-stick-on-ubuntu#1-overview) provided by Ubuntu to create a bootable Ubuntu stick. You can do it your way if you know how to make a bootable device. There is no hard and fast rule here.

### C.2. Installing Ubuntu server on the system

- First, I changed my system's boot priority to make booting via USB the top priority.
- Next, I followed the [guidelines](https://ubuntu.com/tutorials/install-ubuntu-server#8-configure-storage) to install the ubuntu server on the system. 
> **Note** 
> During this phase, you can also configure your network. But, I have not yet tried this approach yet so I cannot guarentee this will work.

After the installation is finished, we can begin configuring Magma AGW.

### C.3. Setting appropriate network confiugrations

AGW requires **2 ethernet ports**. These ports are named eth0 and eth1 by the script. eth0 is used to make connections to the internet, and eth1 is used to communicate with the Radio Access Network. It is also possible to attach an externel etherent port, if your system has only 1 built-in port.

- Once, two ethernet ports have been attached to the system, we manually change the configuration of these ports.
```
sudo /etc/netplan
```
- This folder contains the configuration files for wifi and ethernet. We need to modify the ethernet configuration file. The file may be named differently based on Ubuntu server installation method. For me, the name was ```00-installer-config.yaml```. For you, it may be ```50-cloud-init.yaml```. Following were the contents in the file.
```
# This network config written by 'subiquity'
network:
  ethernets:
    eno1:
      dhcp4: true
  version: 2

```

And I added following information.
```
# This network config written by 'subiquity'
network:
  ethernets:
    eth0:
      dhcp4: true
    eth1:
      dhcp4: false
      addresses: [10.0.2.2/24]
  version: 2

```
> **Note**
> DHCP is true for eth0 and false for eth1. I have also added a static IP to eth1. One of the interface in RAN system should also be in the same network range. You can use your own valid IP range. A successfull connection can be verified by pinging each system.

- Save the above file and try the settings.
```
sudo netplan try
```
- If every setting is fine, the command will prompt you to press enter to apply the settings. Verify the settings by checking ifconfig. 
```
ifconfig -a
```

- You should see the new names and static IP in eth1.

### C.4. Installing Docker-based AGW on Ubunter Server

- After the system has been configured and the relevant network configurations have been made, we create a folder to store the permission file that will be used to connect with the orchestrator.
```
sudo mkdir -p /var/opt/magma/certs
```
- We then create the file named ```rootCA.pem``` and copy the contents of rootCA.pem from the orchestrator. 
> **Note**
> To find the permission file, if you have cloned magma in home directory with folder titled magma, the ```rootCA.pem``` file is present inside ```~/magma/.cache/test_certs``` folder.
```
sudo nano /var/opt/magma/certs/rootCA.pem
```

- Then, staying in the home directory (```cd ~```) and download a script called ```agw_install_docker.sh``` from Magma github.
```
sudo wget https://github.com/magma/magma/raw/v1.8/lte/gateway/deploy/agw_install_docker.sh
```

- Then we execute the script.
```
sudo bash agw_install_docker.sh
```

- The installation will take some time. After successful installation, you can check the docker container list where all the functionalities of AGW are running.
```
sudo docker ps
```

### C.5. Connecting AGW system to the Orchestrator system

- First, verify that the AGW system is running, and you have setup the orchestrator in another system.

- In AGW system,
  - Verify that you have successfully copied ```rootCA.pem``` file to the mentioned directory.
  - Open file ```/etc/hosts```
  ```
  sudo nano /etc/hosts
  ```
  - And add the following lines. Note that **IP address should be the IP address of the Orchestrator machine. So, replace <Orc8r_IP> with your IP.**
  ```
  <Orc8r_IP> controller.magma.test
  <Orc8r_IP> bootstrapper-controller.magma.test
  <Orc8r_IP> fluentd.magma.test
  ```
  - Go to ```/var/opt/magma/docker``` and print the **Hardware ID** and **Challenge Key** for AGW. These values will be used to register the AGW in Orchestrator.
  ```
  cd /var/opt/magma/docker
  ```
  - Print the values
  ```
  sudo docker-compose exec magmad show_gateway_info.py
  ```
  - The output should come something like this.
  ```
  Hardware ID
  -----------
  01235b62-6d55-ec47-ee10-02300074****

  Challenge key
  -------------
  MFkwEwYHKoZIzj0CAQYIKoZqdfgtsDAQ********************************IH6l8fYxpEJ5xCWk3tniaryamvQ8tKoqdtg3lvmGyMzhUHwg==

  Build info
  -------------
   Commit Branch: unknown
   Commit Tag: unknown
   Commit Hash: unknown
   Commit Date: unknown

  Notes
  -----
  - Hardware ID is this gateway's unique identifier
  - Challenge key is this gateway's long-term keypair used for
    bootstrapping a secure connection to the cloud
  - Build info shows git commit information of this build
  ```
  - Use these keys to add the AGW to Orchestrator as shown in this [guideline](https://magma.github.io/magma/docs/nms/equipment#adding-a-new-gateway).
  
  - Finally restart AGW and monitor the Orchestrator NMS to get a good signal.
  ```
  cd /var/opt/magma/docker && sudo docker-compose restart
  ```

## References
- [Magma Documentation](https://magma.github.io/magma/docs/basics/introduction)
- [Dockerized Magma AGW Configuration Guide by RAKwireless Team](https://docs.rakwireless.com/Knowledge-Hub/Learn/Magma-Orchestrator-and-NMS/)
