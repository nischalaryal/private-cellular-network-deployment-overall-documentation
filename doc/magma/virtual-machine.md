# Running AGW on virtual machine
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
