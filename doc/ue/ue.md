# Video Link
[YouTube- Setting up User Equipment for Private Cellular Network Testbed | Hardware, Software, SIM Programming](https://youtu.be/00bMo_kaVjc)
# Using Smartphone, and programmable SIM card to set up user equipment
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

