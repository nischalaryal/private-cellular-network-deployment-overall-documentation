# LTE Testbed setup Using OpenAirInterface and Magma Core
In this repository, we have described the setup procedure we followed to setup an E2E LTE cellular network testbed using COTS UE, programmable SIM card, OpenAirInterface (for RAN), and Magma Core software. Configuration files have also been added to this repository for reference purposes.

This setup procedure is divided into 4 parts:
1. **UE Setup** where we program the SIM card with appropriate values.
2. **RAN setup** where we use OpenAirInterface and USRP (with UHD) to prepare EnodeB
3. **Core setup** where we use Magma Core to setup LTE core.
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











## 3. Magma Core Network Setup
For deploying Magma core network, we utilized the [v1.8.0](https://magma.github.io/magma/docs/basics/introduction.html) documentation. Like OAI-CN, the documentation in this site is sufficient to run all the functionalities of the core. However, we will mention the required steps we took for a successfull build and run.

### Pre-requisties
First, we installed the [pre-requisites](https://magma.github.io/magma/docs/basics/prerequisites). We used ```UBUNTU 20.04``` operating system for Magma deployment.
**Note**: Pay special attention to the python and golang version as failure to comply to the version might create issues during next steps. We skipped the Deployment Tooling from this page since our goal was not to deploy in real-world testbed.

1. Go Lang
```
# Download and extract Go Lang
wget https://artifactory.magmacore.org/artifactory/generic/go1.18.3.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.18.3.linux-amd64.tar.gz

# Setup PATH variable
echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc
source .bashrc

# Verify proper install
go version
# Output should be something like 'go version go1.18.3 linux/amd64'
```
2. Pyenv
```
# Install necessary dependencies
sudo apt install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python-openssl git

# Clone pyenv repo in _.pyenv_ folder
git clone https://github.com/pyenv/pyenv.git ~/.pyenv

# Configure .bashrc for pyenv variables
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n eval "$(pyenv init -) "\nfi' >> ~/.bashrc
exec "$SHELL"

# Install pyenv according to your requirement. For us, it was 3.8.10
pyenv install 3.8.10
pyenv global 3.8.10
```

3. Download [docker](https://docs.docker.com/engine/install/ubuntu/), [docker-compose](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04), [virtual box](https://www.virtualbox.org/wiki/Linux_Downloads), and [vagrant](https://linuxize.com/post/how-to-install-vagrant-on-ubuntu-20-04/) from their own website (Here we have provided the ones we followed. Official and other sites). It is better to keep these program in their latest version and according to the OS requirement. After successfully installing vagrant, install the following plugins.
```
vagrant plugin install vagrant-vbguest vagrant-disksize vagrant-reload
```
4. Download Magma
```
git clone https://github.com/magma/magma.git
cd magma
git checkout v1.8
```

### AGW Deployment
For AGW, we will use virtualbox for creating the OS for deployment. So, from this section **[HOST]** means the main system where we installed everything. **[AGW]** means the virtual OS inside virtualbox

1. Setting up network configuration in the **[HOST]** system
```
echo "* 192.168.0.0/16" | sudo tee -a /etc/vbox/networks.conf
cd magma/lte/gateway/
```

2. Create VM that will host the access gateway in **[HOST]** system.
```
vagrant up magma
```
3. After completing the previous command, access the virtual environment from the **[HOST]** system.
```
vagrant ssh magma
```
4. Now you are in the **[AGW]** system. Go to the following path.
```
cd ~/magma/lte/gateway
```
5. Compile the source code in **[AGW]**.
```
make run
```
6. Restart magma services in **[AGW]** machine
```
sudo service magma@* stop
sudo service magma@magmad start 
```
7. Check magma services are **active** or not in **[AGW]**.
```
sudo service magma@* status
```







### Orchestrator Deployment
We will deploy the Orchestrator in docker containers in **[HOST]**
1. Access the following path
```
cd magma/orc8r/cloud/docker
```

2. Build orchestrator images in **[HOST]** system.
```
./build.py --all
```

3. Run orchestrator services in **[HOST]** system.
```
./run.py --metrics
```
OUTPUT:
```
Creating orc8r_postgres_1 ... done
Creating orc8r_test_1     ... done
Creating orc8r_maria_1    ... done
Creating elasticsearch    ... done
Creating fluentd          ... done
Creating orc8r_kibana_1   ... done
Creating orc8r_proxy_1      ... done
Creating orc8r_controller_1 ... done
```
4. Check dockers status in **[HOST]** system.
```
docker-compose ps
```
OUTPUT:
```
Creating orc8r_alertmanager_1     ... done
Creating orc8r_maria_1            ... done
Creating elasticsearch            ... done
Creating orc8r_postgres_1         ... done
Creating orc8r_config-manager_1   ... done
Creating orc8r_test_1             ... done
Creating orc8r_prometheus-cache_1 ... done
Creating orc8r_prometheus_1       ... done
Creating orc8r_kibana_1           ... done
Creating fluentd                  ... done
Creating orc8r_proxy_1            ... done
Creating orc8r_controller_1       ... done
```

**Note**: After the orchestrator deployment, we need to add the certificate files to the browser (Firefox in our case) in order to access APIs. The process to add the certificate is given [here](https://magma.github.io/magma/docs/basics/quick_start_guide).

- Access the following path in **[HOST]** system to see the keys.
```
ls ~/magma/.cache/test_certs
```
OUTPUT:
```
admin_operator.key.pem  bootstrapper.key        controller.crt          rootCA.key
admin_operator.pem      certifier.key           controller.csr          rootCA.pem
admin_operator.pfx      certifier.pem           controller.key          rootCA.srl
```

- Since we are using _Firefox_ browser, we imported this _admin_operator.pfx_ file into your browser's installed client certificates. See [here](https://support.globalsign.com/digital-certificates/digital-certificate-installation/install-client-digital-certificate-firefox-windows) for instructions. The ```passwrod``` is ```magma``` for the pfx file.
- After adding the pfx file, you can access the [SWAGGER API](https://localhost:9443/swagger/v1/ui).


### Connecting AGW and Orchestrator
After successfully building and running AGW and orchestrator, we need to connect these two components. The following code is executed in the **[HOST]** system. You will get a success message after proper connection.
```
cd ~/magma/lte/gateway
fab -f dev_tools.py register_vm
```

After success message, access **[AGW]** machine and run the following code to see the success message in AGW.
```
# This is executed in [HOST] machine
vagrant ssh magma

# The following code are executed in [AGW] machine.
sudo service magma@* stop
sudo service magma@magmad restart
sudo tail -f /var/log/syslog
```
OUTPUT:
```
Sep 27 22:57:35 magma-dev magmad[6226]: [2018-09-27 22:57:35,550 INFO root] Checkin Successful!
Sep 27 22:57:55 magma-dev magmad[6226]: [2018-09-27 22:57:55,684 INFO root] Processing config update g1
Sep 27 22:57:55 magma-dev control_proxy[6418]: 2018-09-27T22:57:55.683Z [127.0.0.1 -> streamer-controller.magma.test,8443] "POST /magma.Streamer/GetUpdates HTTP/2" 200 7bytes 0.009s
```

### NMS Deployment
NMS provides UI for configuring and monitoring the network. We use following code to build NMS and connect with orchestrator. This is executed in **[HOST]** system.
```
cd ~/magma/nms
COMPOSE_PROJECT_NAME=magmalte docker-compose build magmalte
docker-compose up -d

# After 1-2 minute, execute this command.
./scripts/dev_setup.sh
```

After successfully running all the code above, the configuration parameters and subscriber can be added through the NMS [website](https://magma-test.localhost/).
- Use ```username=admin@magma.test``` and ```password=password123``` to access NMS dashboard.
- Use the ```Network``` section to update **MCC** and **MNC** values of the system. This should match the value stored in EnodeB configuration file and in SIM card. After this step, you should be able to successfully connect RAN and Core.
- Use the ```Subscriber``` section to add the subscriber with the information mentioned in the UE section. After this, you can successfully connect UE to the network. **NOTE**: If you see authentication error, there is a high chance you have your **OPC** and/or **Ki** keys not matching in **AGW** and **UE**. 


## 4. Routing
Establishing a proper route will be very important when implementing RAN and core network in separate system. The two systems need to identify eachother in order to exchange data. Routing setup is different according to the core network used (OAI or Magma).

### 4.1 Route for RAN and Magma Core
Setting up routes for RAN and Magma core is a bit challenging because the Access Gateway (AGW) is located inside the virtual machine. So we need to add routes on 3 different systems: **RAN system, Magma system,** and **AGW virtual machine**.

#### 4.1.1 RAN System
This step is similar to the previous one.
```
sudo ip route add 192.168.60.0/24 via 192.168.2.171 dev eno1
sudo sysctl net.ipv4.conf.all.forwarding=1
sudo iptables -P FORWARD ACCEPT
```
#### 4.1.2 Magma Core System
```
sudo sysctl net.ipv4.conf.all.forwarding=1
sudo iptables -P FORWARD ACCEPT
sudo iptables -A FORWARD -i vboxnet0 -o eno1 -j ACCEPT
sudo iptables -A FORWARD -i vboxnet0 -o eno1 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -t nat -A POSTROUTING -o vboxnet0 -j MASQUERADE
sudo iptables -A FORWARD -i eno1 -o vboxnet0 -j ACCEPT
```

#### 4.1.3 AGW virtual machine
```
sudo ip route add 192.168.2.0/24 via 192.168.60.142 dev eth1
sudo sysctl net.ipv4.conf.all.forwarding=1
sudo iptables -P FORWARD ACCEPT
```

**Note**: At last, we can ping each system to check if the route is working correctly or not.

Example:
```
# [For Magma setup] from RAN system, ping to AGW
ping 192.168.60.142

# You can also try to ping from AGW to RAN
ping 192.168.2.215

# [For OAI setup] from RAN system, ping to OAI core
ping 192.168.61.149
```

# Video Demonstration

[Video Link](https://www.youtube.com/watch?v=jAqJEjkI3eI)

# Publication
This work was presented as a DEMO paper titled _"Private Cellular Network Deployment: Comparison of OpenAirInterface with Magma Core"_ in CNSM 2022, Thessaloniki, Greece. Paper can be accessed from [this link](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9964713&casa_token=evz-cpsK6xcAAAAA:nbavt4HH-9bHzuoQtepsRbZFy-TSjh8iXKfRoDVXifARMzOlMC-bbe37V5zuMCYuAsOs07WVXL6u) and [this link.](https://www.researchgate.net/publication/365705123_Private_Cellular_Network_Deployment_Comparison_of_OpenAirInterface_with_Magma_Core).

# References
- [OAI RAN](https://gitlab.eurecom.fr/oai/openairinterface5g/-/wikis/OpenAirSoftwareSupport)
- [Kernal Help](https://daobook.github.io/apollo/docs/howto/how_to_install_apollo_kernel.html)
- [Kernal Help](https://super-unix.com/ubuntu/ubuntu-disable-ondemand-cpu-scaling-daemon/)
- [UHD](https://kb.ettus.com/Building_and_Installing_the_USRP_Open-Source_Toolchain_(UHD_and_GNU_Radio)_on_Linux)
- [Magma Core](https://magma.github.io/magma/docs/basics/introduction.html)
