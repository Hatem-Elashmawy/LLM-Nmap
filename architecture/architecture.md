# Architecture

**NOTE** : This section explains how to configure the architecture required for the project.

## How to set up a personal network using Virtual Box?
*(See the example part for more informations)*

Quick overview :
1) Install Virutal Box / VMware
2) Download VMs *(kali, ubuntu, beebox, ...)*
3) Install each box and put them in the same *"Network NAT"*
4) Once each machine has started up, ping them from a VM to ensure that everything is in place
5) Set up llm-nmap in a VM to perform scans  


## Example of a Network (virutal box)

For this network, we used 3 VMs:
    - a virtual machine to execute requests through llm-nmap : *kali linux*
    - a "classic" target VM : *Ubuntu Server* (to open specific ports) (https://ubuntu.com/download/server/thank-you?version=24.04.3&architecture=amd64&lts=true)
    - a "vurlnerable" target VM : *bee box* (a VM with many ports already open)

**NOTE** : when you install Ubuntu Server with Virtual Box set your credentials before starting the machine.
![UbuntuServerPassword](src/UbuntuServerPassword.png)


To create a Network NAT in Virtual Box :
    - Go to "Tools"
    - Select "NAT Networks"
    - Select "Create"
    - Set up a name and your IP 
![UbuntuServerPassword](src/CreateNAT.png)


Then, add your VMs to the created NAT in *Settings*.
![UbuntuServerPassword](src/AddVMsToNAT.png)


Once done, you can start your VMs and send a ping command to each of them to ensure that they are on the same network.

**NOTE :** on linux, *ifconfig* to show your ip.
![UbuntuServerPassword](src/VMpingToVM.png)


## LLM-Nmap Installation

## Ubuntu Server - Open ports

To open ssh port on Ubuntu Server : 

```
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
```

Then check if the ssh port (port 22) is open
```
systemctl status ssh
```
