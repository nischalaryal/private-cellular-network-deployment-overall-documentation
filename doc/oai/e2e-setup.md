# LTE Testbed setup Using OpenAirInterface and Magma Core
In this repository, we have described the setup procedure we followed to setup an E2E LTE cellular network testbed using COTS UE, programmable SIM card, OpenAirInterface (for RAN and Core functionality). Configuration files have also been added to this repository for reference purposes.

This setup procedure is divided into 4 parts:
1. **UE Setup** where we program the SIM card with appropriate values.
2. **RAN setup** where we use OpenAirInterface and USRP (with UHD) to prepare EnodeB
3. **Core setup** where we use OpenAirInterface core network to setup LTE core.
4. **Routing** where we define routes to communicate between two systems (RAN system and Core system).

## 1. UE SETUP
Following devices and softwares were used for setting up a Commercially Off-The Shelf (COTS) User equipment.
- Hardware
  - Samsung Galaxy S4
  - Gemalto SIM card reader
  - Sysmocom Programmable SIM card
- Software
  - [pySim](https://github.com/osmocom/pysim)
  
 ### Steps
 1. Install [pySim](https://github.com/osmocom/pysim) and all it's dependencies. This repo also has a [documentation](https://osmocom.org/projects/pysim/wiki) regarding all the functionalities that can be achieved from the program. After downlading the program, access the folder. 
 
 **Note:** At the time of writing, executing the following code was sufficient. In future, you might need to adjust it according to the updates made in the program.
 
```
sudo apt install pcscd pcsc-tools libccid libpcsclite-dev python-pyscard
```
```
git clone git://git.osmocom.org/pysim.git 
```
``` 
cd pysim
```

2. To program the simcard, add the card into the card reader and attach it to the system. To verify if the card is recognized by the system or not, use the following command.
```
pcsc_scan
```
If the card reader is identified, the output of the command could be something like this.
```
PC/SC device scanner
V 1.5.2 (c) 2001-2017, Ludovic Rousseau <ludovic.rousseau@free.fr>
Using reader plug'n play mechanism
Scanning present readers...
Waiting for the first reader...found one
Scanning present readers...
0: Gemalto USB Shell Token V2 (64438828) 00 00

...
...
...

Possibly identified card (using /usr/share/pcsc/smartcard_list.txt):
3B 9F 96 80 1F C7 80 31 A0 73 BE 21 13 67 43 20 07 18 00 00 01 A5
	sysmoUSIM-SJS1 (Telecommunication)
	http://www.sysmocom.de/products/sysmousim-sjs1-sim-usim

```

3. Next, we need following information to program into the sim card. 
**Note:** All values used here does not necessarily match your requirement. Especially ADM key.

| Variable | Description | Value Used |
| --- | --- | --- |
| `pin-adm` | Administrative key for SIM card. Provided during purchase. VERY IMPORTANT. | 2611488 |
| `mcc` | Mobile Country Code | 208 |
| `mnc` | Mobile Network Code | 93 |
| `pcsc-device` | Which PC/SC reader number for SIM access. | 0 or 1 |
| `imsi` | International Mobile Subscriber Identity | 208930000000008 |
| `ki` | Authentication Key | 8baf473f2f8fd09487cccbd7097c6862 |
| `opc` | Operator Authentication Key | 2117cd3f66beb0811e6c72bc9ba82b3a |
| `acc` | Access Control Code | 0010 |
| `dry-run` | Perform a 'dry run', don't actually program the card | - |

**Note: The MCC and MNC values need to exactly match in the eNodeB configuration file and the core network. Ki and OPC values need to match exactly with the values stored in core network for successful attachment of UE.**

4. Once all the values are ready, we program the SIM card using pysim.
```
./pySim-prog.py --pcsc-device=0 \
                --type="sysmoUSIM-SJS1" \ 
                --mcc=208 \
                --mnc=93 \
                --imsi=208930000000008 \
                --opc=2117cd3f66beb0811e6c72bc9ba82b3a \
                --ki=8baf473f2f8fd09487cccbd7097c6862 \
                --iccid=8988211000000285844 \
                --pin-adm=73497301 \
                --acc=0010 \
                --dry-run
```

Upon success, you will get output like this. Once you are satisfied with the values, remove ```dry-run``` to actually program the card.
```
Using PC/SC reader interface
Generated card parameters :
 > Name     : Magic
 > SMSP     : e1ffffffffffffffffffffffff0581005155f5ffffffffffff000000
 > ICCID    : 8988211000000285844
 > MCC/MNC  : 208/93
 > IMSI     : 208930000000008
 > Ki       : 2117cd3f66beb0811e6c72bc9ba82b3a
 > OPC      : 8e27b6af0e692e750f32667a3b14605d
 > ACC      : 0010
 > ADM1(hex): 3733343937333031
 > OPMODE   : None
Dry Run: NOT PROGRAMMING!
Programming successful: Remove card from reader
```

5. Finally, we setup APN value which will be used for setting up internet access. Go to your mobile data option in settings. Search for Access Point Names (APN) option and add a new apn setting with ```Name``` as ```oai``` and ```APN``` value ```oai.ipv4```. The name can be according to your choice as long as it matches with the value stored in core network.



## 2. RAN SETUP
**Note:** For setting up RAN, **[system](https://gitlab.eurecom.fr/oai/openairinterface5g/-/wikis/OpenAirSystemRequirements) and [kernel](https://gitlab.eurecom.fr/oai/openairinterface5g/-/wikis/OpenAirKernelMainSetup) requirements** are very important. So it is important to properly go through the requirements from the [OAI webpage](https://gitlab.eurecom.fr/oai/openairinterface5g/-/wikis/OpenAirSoftwareSupport).

### Hardware
The hardware requirements are given in the [OAI git page](https://gitlab.eurecom.fr/oai/openairinterface5g/-/wikis/OpenAirSystemRequirements).

**NOTE:** The processor needs to support _avx2_ operations in order to run 5G related user equipment (nrUE) and base station (gNB).

### Operating System and Kernal
#### OS
1. Following command was used to view the OS information
```
cat /etc/os-release
```
OUTPUT:
```
NAME="Ubuntu"
VERSION="18.04.6 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.6 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
```

#### Kernel Setup
1. Run the following command to check your “kernel release” information
```
uname -r
```
Output:
``` 5.4.0-120-generic ```

2. Pay attention to the kernal version in output and use it to install the low latency kernel like in the following command. You need to restart after executing the command to see changes.
```
sudo apt-get install linux-image-5.4.0-120-lowlatency linux-headers-5.4.0-120-lowlatency
```
##### Disable CPU Frequency Scaling
1. Install “cpufrequtils” using following command
```
sudo apt install cpufrequtils
```
2. The fresh install of the OS will not have the following file. So, it is okay to create one. The following code will either create a new file or open the existing one. **Note:** gedit text editor is being used in the following terminal. You can use your own prefered editor.
```
sudo gedit /etc/default/cpufrequtils
```
3. Add the following line to the file and save it.
```
GOVERNOR="performance"
```
4. Finally, disable ondemand daemon to avoid losing your changes after reboot
```
sudo systemctl disable ondemand
```
##### Disable p-state and c-state
1. Open the following file. You need sudo access.
```
sudo gedit /etc/default/grub
```

2. Search for the line with _GRUB_CMDLINE_LINUX_DEFAULT_ variable and append the following words inside the string value of the variable. The initial line looks something like this _GRUB_CMDLINE_LINUX_DEFAULT= "quiet splash"_. The final line should look like the following.
```
GRUB_CMDLINE_LINUX_DEFAULT= "quiet intel_pstate=disable processor.max_cstate=1 intel_idle.max_cstate=0 idle=poll splash"
```
3. Finally, upgrade the grub
``` sudo update-grub ```

##### Blacklist intel_powerclamp module
1. Open the following file. You will need sudo access.
``` sudo gedit /etc/modprobe.d/blacklist.conf ```

2. Add the following line and save it.
``` blacklist intel_powerclamp ```

##### Disable power management from BIOS
This step might be different according to the system hardware. We need to disable hyperthreading, cpu frequency control, c-state, p-state, and any other power management from BIOS as well. All the options may not be present in a particular system. So disable the options which are present.

##### Verification
1. To verify proper installation, a software called i7z can be utilized. We need to first install the software.
``` sudo apt install i7z ```

2. To run the software.
``` sudo i7z ```
### Software
#### UHD 
We need to setup [UHD](https://kb.ettus.com/Building_and_Installing_the_USRP_Open-Source_Toolchain_(UHD_and_GNU_Radio)_on_Linux) for setting up USRP for radio communication.

1. For Ubuntu 18.04, the following dependencies are installed. **Note**: It is different according to the OS and version used. Visit the site above to get the dependencies according to your requirement. **Restart the system after completion.**
```
sudo apt-get -y install git swig cmake doxygen build-essential libboost-all-dev libtool libusb-1.0-0 libusb-1.0-0-dev libudev-dev libncurses5-dev libfftw3-bin libfftw3-dev libfftw3-doc libcppunit-1.14-0 libcppunit-dev libcppunit-doc ncurses-bin cpufrequtils python-numpy python-numpy-doc python-numpy-dbg python-scipy python-docutils qt4-bin-dbg qt4-default qt4-doc libqt4-dev libqt4-dev-bin python-qt4 python-qt4-dbg python-qt4-dev python-qt4-doc python-qt4-doc libqwt6abi1 libfftw3-bin libfftw3-dev libfftw3-doc ncurses-bin libncurses5 libncurses5-dev libncurses5-dbg libfontconfig1-dev libxrender-dev libpulse-dev swig g++ automake autoconf libtool python-dev libfftw3-dev libcppunit-dev libboost-all-dev libusb-dev libusb-1.0-0-dev fort77 libsdl1.2-dev python-wxgtk3.0 git libqt4-dev python-numpy ccache python-opengl libgsl-dev python-cheetah python-mako python-lxml doxygen qt4-default qt4-dev-tools libusb-1.0-0-dev libqwtplot3d-qt5-dev pyqt4-dev-tools python-qwt5-qt4 cmake git wget libxi-dev gtk2-engines-pixbuf r-base-dev python-tk liborc-0.4-0 liborc-0.4-dev libasound2-dev python-gtk2 libzmq3-dev libzmq5 python-requests python-sphinx libcomedi-dev python-zmq libqwt-dev libqwt6abi1 python-six libgps-dev libgps23 gpsd gpsd-clients python-gps python-setuptools
```

2. Clone the UHD repository using the following code
```
git clone https://github.com/EttusResearch/uhd.git
```

3. Go inside the cloned folder and checkout the branch according to your requirement. For our purpose, _v4.1.0.0_ is being used.
```
cd uhd
git checkout -b v4.1.0.0
```

4. Go into the _host_ folder and create a _build_ directory. Go inside the created build folder and invoke cmake command.
```
cd host
mkdir build
cd build
cmake ../
```
**Note:** Most of the issues faced during the cmake command is due to the required dependencies not being installed. Look at the errors carefully to identify the dependency you need to install. You can install the dependency via apt or pip.

5. After successfully completing the cmake command, build UHD.
``` make ```

6. After completion of the build process, verification can be done to check if it was successful.
``` make test ```

7. Install uhd using sudo privileges. The files will be copied to _/usr/local/lib_
``` sudo make install ```

8. After installation, update the systems library cache
``` sudo ldconfig ```

9. Add the following line to the _.bashrc_ file. Save the file and exit the terminal.
``` 
sudo gedit ~/.bashrc

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
```

10. Open a new terminal and run the following command to check if the drivers are successfully installed.
``` uhd_find_devices ```

OUTPUT could be something like this.
``` No UHD Devices Found. ```

11. Download the UHD FPGA Images. **Note**: Sometimes, it fails to download mentioning connection issues. Run the code again if you face such errors.
```
sudo uhd_images_downloader
```

12. Configure USB by adding udev rules so that non-root users may access the device. It is necessary for devices that use USB to connect to the host computer. _The following code assumes that the uhd folder was cloned in the home directory._
 ```
cd ~/uhd/host/utils
sudo cp uhd-usrp.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
sudo udevadm trigger
```
13. Connect the USRP via the USB port and run following commands to check.

USRP should be listed after following code execution.
```
lsusb 
```
Also try running:
```
uhd_find_devices
uhd_usrp_probe 
```
#### GNU
1. Install and test GNU
```
sudo apt install gnuradio
```
```
gnuradio-config-info --version
gnuradio-config-info --prefix
gnuradio-config-info --enabled-components
```

### OpenAirInterface EnodeB
Verify that the kernel and performance of the OS is properly set up. Otherwise, we might experience some overflow issues while running enb. Most of the issues we faced during the build setup was due to the hardware capabilities of our system. So it is important to verify that the hardware requirements are properly met.

1. Install git and clone OpenAirInterface inside _oai_ folder.
```
sudo apt install git
```
```
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git oai
```

2. Access the oai folder (oai will refer to as the source code folder). We checkout the _v1.1.0_ branch. **Note:** we can checkout any branch according to our requirement.
```
cd oai
```
```
git checkout v1.1.0
source oaienv
```

3. We now need to change the enb configuration file to change the _mcc_, _mnc_,_ mme ip_, _enb interface name_, and _enb ip_.

Assuming we are inside _oai_ folder.
```
gedit ci-scripts/conf_files/enb.band7.tm1.50PRB.usrp210.conf
```
Search for following code snippets and change the mcc and mnc values
```
plmn_list = ( { mcc = 208; mnc = 93; mnc_length = 2;} );
```

_192.168.61.149_ is the ip address of the system where our **core network** was installed. **Note:** Since our OAI core will be deployed in docker, the IP address used here will be of the core network host system. But for magma core, access gateway will be deployed in virtual machine, so the IP will be of the virtual machine (Most probably, _192.168.61.149_).
```
mme_ip_address      = ( { ipv4       = "192.168.61.149";
                              ipv6       = "192:168:30::17";
                              active     = "yes";
                              preference = "ipv4";
                            }
                          );
```
_192.168.1.215_ is the ip address of the system where our **RAN** was set up (i.e., the same system where you will be opening this file). **_eno1_** is the interface name of the same system. You can see these values by running the _ifconfig_ command.
```
NETWORK_INTERFACES :
    {
        ENB_INTERFACE_NAME_FOR_S1_MME            = "eno1";
        ENB_IPV4_ADDRESS_FOR_S1_MME              = "192.168.1.215";
        ENB_INTERFACE_NAME_FOR_S1U               = "eno1";
        ENB_IPV4_ADDRESS_FOR_S1U                 = "192.168.1.215";
        ENB_PORT_FOR_S1U                         = 2152; # Spec 2152
        ENB_IPV4_ADDRESS_FOR_X2C                 = "192.168.1.215";
        ENB_PORT_FOR_X2C                         = 36422; # Spec 36422
    };
```

4. Build and Run eNB

Assuming we are inside the oai folder
```
cd cmake_targets
```
Build OAI
```
./build_oai -c -C

./build_oai -I --install-optional-packages

./build_oai --eNB -w USRP
```
After successful build, connect the USRP and run the enb. **Note**: Based on software version, the binary gets built in different folder. We need to pay attention to the folder where the binary is being built. For v1.1.0, it had he following path (i.e., _/cmake_targets/lte_build_oai/build/_)

```
sudo ./cmake_targets/lte_build_oai/build/lte-softmodem -O ~/enb2/ci-scripts/conf_files/enb.band7.tm1.50PRB.usrpb210.conf
```
**Note:** After running enb, if you face overflow issues with a lot of L and U, check the software requirements again. If you still have this issue, then your hardware requirement is not met. You need the latest hardware. Check the hardware section.






















## 3. Core Setup
Here we will discuss about implementing 2 different core network setup.
- OpenAirInterface Core Setup
- Magma Core Setup

### 3.1 OpenAirInterface Core Setup

For implementing OAI core network, we utilized the OAI evolved packet core [repository](https://github.com/OPENAIRINTERFACE/openair-epc-fed). The documentation provided there is easy to follow. However, here we list the process we did for successful installation and build.

1. We installed the [pre-requisites](https://github.com/OPENAIRINTERFACE/openair-epc-fed/blob/master/docs/DEPLOY_PRE_REQUESITES_MAGMA.md)
	- **Ubuntu 18.04** was used as the Operating System.
	- **Docker** (version ```5:20.10.2~3-0~ubuntu-bionic```) and **docker-compose** (version ```1.27```) were installed from their main website and following code was executed to add docker to sudo group.
	 ```
	 sudo usermod -a -G docker <<username of system where Core is being deployed>>
	 ```
	- We then need to pull 3 base images from ```docker hub```: ```ubuntu:bionic```, ```cassandra:2.1``` and ```redis:6.0.5```.
	  ```
	  docker login
	  Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
	  Username: 
	  Password: 
	  ```
	  ```
	  docker pull ubuntu:bionic
	  docker pull cassandra:2.1
	  docker pull redis:6.0.5
	  ```
	  ```
	  docker logout
	  ```

2. Clone the required [versions](https://github.com/OPENAIRINTERFACE/openair-epc-fed/blob/master/docs/BUILD_IMAGES_MAGMA_MME.md). For our setup, we opted for the stable version (i.e. ```master``` branch).
```
# Clone directly on the latest release tag
git clone --branch v1.2.0 https://github.com/OPENAIRINTERFACE/openair-epc-fed.git
cd openair-epc-fed

# If you forgot to clone directly to the latest release tag
git checkout -f v1.2.0

# Synchronize all git submodules
./scripts/syncComponents.sh
---------------------------------------------------------
OAI-HSS    component branch : master
OAI-SPGW-C component branch : master
OAI-SPGW-U component branch : master
```


3. Build the modules.
 - Build HSS
```
docker build --target oai-hss --tag oai-hss:production \
               --file component/oai-hss/docker/Dockerfile.ubuntu18.04 \
               component/oai-hss
```
```
docker image prune --force
```
```
docker image ls
#OUTPUT
oai-hss                 production             f478bafd7a06        1 minute ago          341MB
...
```

 - Build SPGW-C
```
docker build --target oai-spgwc --tag oai-spgwc:production --file component/oai-spgwc/docker/Dockerfile.ubuntu18.04 component/oai-spgwc
```
```
docker image prune --force
```
```
docker image ls
#OUTPUT
oai-spgwc               production             b1ba7dd16bc5        1 minute ago          218MB
...
```

 - Build SPGW-U
```
docker build --target oai-spgwu-tiny --tag oai-spgwu-tiny:production --file component/oai-spgwu-tiny/docker/Dockerfile.ubuntu18.04 component/oai-spgwu-tiny
```
```
docker image prune --force
```
```
docker image ls
#OUTPUT
oai-spgwu-tiny          production             588e14481f2b        1 minute ago          220MB
...
```
 - Build Magma-MME
 ```
 cd ~
 git clone https://github.com/magma/magma.git
 cd magma
 ```
 ```
 docker build --target magma-mme --tag magma-mme:master --file lte/gateway/docker/mme/Dockerfile.ubuntu18.04 .
 docker image ls
 # OUTPUT
 magma-mme               nsa-support            b6fb01eb0d07        1 minute ago          492MB
 ...
 ```

4. Run the [containers](https://github.com/OPENAIRINTERFACE/openair-epc-fed/blob/master/docker-compose/magma-mme-demo/README.md).
 - Initialize Cassandra DB
 ```
 cd ~/openair-epc-fed/docker-compose/magma-mme-demo
 docker-compose up -d db_init

 # OUTPUT:
 Creating network "demo-oai-private-net" with the default driver
 Creating network "demo-oai-public-net" with the default driver
 Creating demo-cassandra ... done
 Creating demo-db-init   ... done
 ```
 ```
 docker logs demo-db-init --follow
 # OUTPUT:
 Connection error: ('Unable to connect to any servers', {'192.168.68.130': error(111, "Tried connecting to [('192.168.68.130', 9042)]. Last error: Connection refused")})
 Connection error: ('Unable to connect to any servers', {'192.168.68.130': error(111, "Tried connecting to [('192.168.68.130', 9042)]. Last error: Connection refused")})
 Connection error: ('Unable to connect to any servers', {'192.168.68.130': error(111, "Tried connecting to [('192.168.68.130', 9042)]. Last error: Connection refused")})
 Connection error: ('Unable to connect to any servers', {'192.168.68.130': error(111, "Tried connecting to [('192.168.68.130', 9042)]. Last error: Connection refused")})
 OK
 ```
 - Check cassandra log
 ```
 docker logs demo-cassandra
 ```
 Output should be have the following line at last. If no, execute ```docker-compose down``` and start again from initializing cassandra.
 ```
 ....
 INFO  20:19:40 Initializing vhss.extid_imsi_xref
 ```
 - Deploy rest of EPC
 ```
 docker-compose up -d oai_spgwu
 ```
 Output:
 ```
 demo-cassandra is up-to-date
 Creating demo-redis   ... done
 Creating demo-oai-hss ... done
 Creating demo-magma-mme ... done
 Creating demo-oai-spgwc ... done
 Creating demo-oai-spgwu-tiny ... done
 ```

5. Finally, we need to set the ```MCC, MNC, TAC``` values in the ```mme.conf``` file in order to successfully connect to EnodeB. The values must match the ones stored in the EnodeB configuration file (mentioned above in RAN section). This [link](https://github.com/OPENAIRINTERFACE/openair-epc-fed/blob/master/docker-compose/magma-mme-demo/README.md#4-how-to-edit-the-docker-compose-file) provides more information for modifying the required parameters. 


## 4. Routing
Establishing a proper route will be very important when implementing RAN and core network in separate system. The two systems need to identify eachother in order to exchange data.

### 4.1 Route for RAN and OAI Core
In this scenario, RAN (with IP = _192.168.2.215_) and OAI core (with IP = _192.168.2.171_) are on the same network. So, we setup a route in the RAN system using the following command.
```
sudo ip route add 192.168.61.128/26 via 192.168.2.171 dev eno1
```
where
- 192.168.2.171 is the **IP address** of the **system where OAI core** is deployed.
- eno1 is the **Network Interface Controller(NIC)** of the system where RAN is deployed.

To understand this code, it is similar to saying **_In order to access the IP range 192.168.61.128/26, contact IP address 192.168.2.171 using interface eno1_**.


# Video Demonstration

[Video Link](https://www.youtube.com/watch?v=jAqJEjkI3eI)

# Publication
This work was presented as a DEMO paper titled _"Private Cellular Network Deployment: Comparison of OpenAirInterface with Magma Core"_ in CNSM 2022, Thessaloniki, Greece. Paper can be accessed from [this link](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9964713&casa_token=evz-cpsK6xcAAAAA:nbavt4HH-9bHzuoQtepsRbZFy-TSjh8iXKfRoDVXifARMzOlMC-bbe37V5zuMCYuAsOs07WVXL6u) and [this link.](https://www.researchgate.net/publication/365705123_Private_Cellular_Network_Deployment_Comparison_of_OpenAirInterface_with_Magma_Core).

# References
- [OAI RAN](https://gitlab.eurecom.fr/oai/openairinterface5g/-/wikis/OpenAirSoftwareSupport)
- [Kernal Help](https://daobook.github.io/apollo/docs/howto/how_to_install_apollo_kernel.html)
- [Kernal Help](https://super-unix.com/ubuntu/ubuntu-disable-ondemand-cpu-scaling-daemon/)
- [UHD](https://kb.ettus.com/Building_and_Installing_the_USRP_Open-Source_Toolchain_(UHD_and_GNU_Radio)_on_Linux)
- [OAI Core](https://github.com/OPENAIRINTERFACE/openair-epc-fed/blob/master/docs/DEPLOY_HOME_MAGMA_MME.md)
