# Architecture

**NOTE** : This section explains how to configure the architecture required for the project.

## How to set up a personal network using Virtual Box?
*(See the example part for more informations)*

Quick overview :
1) Install virutal box
2) Download VMs *(kali, ubuntu, beebox, ...)*
3) Install each box and put them in the same *"Network NAT"*
4) Once each machine has started up, ping them from a VM to ensure that everything is in place
5) Set up llm-nmap in a VM to perform scans  


## Example of a Network

For this network, I used 3 VMs:
    - a virtual machine to execute requests through llm-nmap : *kali linux*
    - a "classic" target VM : *Ubuntu Server* (to open specific ports)
    - a "vurlnerable" target VM : *bee box* (a VM with many ports already open)

NOTE : when you install Ubuntu Server with Virtual Box set your credentials before starting the machine


![UbuntuServerPassword]("src/UbuntuServerPassword.png")



