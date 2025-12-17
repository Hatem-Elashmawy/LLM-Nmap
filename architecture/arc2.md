# Architecture

> **Note:** This section explains how to configure the architecture required for the project and describes the scenarios we tested.

---

## 1. Setting up a Personal Network Using VirtualBox

*(See the example section below for more information.)*

Quick overview:

1. Install VirtualBox (or VMware).
2. Download the VM images *(Kali, Ubuntu Server, BeeBox, ...)*.
3. Create each VM and attach them to the same **NAT Network**.
4. Once each machine has started, use `ping` from one VM to another to ensure that everything is correctly configured.
5. Set up LLM–Nmap on a VM (Kali) to perform scans on the other machines.

---

## 2. Example of a VirtualBox Network (Scenario 1: VM-to-VM)

For this network, we used **three VMs**:

- A virtual machine to execute requests through LLM–Nmap: **Kali Linux**  
- A “classic” target VM: **[Ubuntu Server](https://ubuntu.com/download/server/thank-you?version=24.04.3&architecture=amd64&lts=true)**  
  Used to open specific ports (e.g. SSH).
- A **vulnerable** target VM: **BeeBox**  
  A VM with many ports already open (used later in Scenario 3).

**Note:** When you install Ubuntu Server with VirtualBox, it is easier to set your credentials (username and password) in the VirtualBox GUI before starting the machine.

![Ubuntu Server Password](src/UbuntuServerPassword.png)

### 2.1 Creating a NAT Network in VirtualBox

To create a NAT Network in VirtualBox:

- Go to **Tools**
- Select **NAT Networks**
- Click **Create**
- Set a name and an IP range

![Create NAT Network](src/CreateNAT.png)

Then, add your VMs to the created NAT network in **Settings → Network** for each VM.

![Add VMs to NAT Network](src/AddVMsToNAT.png)

### 2.2 Verifying Connectivity

Once done, you can start your VMs and send a `ping` command to each of them to ensure that they are on the same network.

**Note:** On Linux, you can use `ifconfig` (or `ip addr`) to show your IP address.

![VM Ping to VM](src/VMpingToVM.png)

---

## 3. LLM–Nmap Installation (Kali VM)

To set up LLM–Nmap we first need to install `llm`:

```bash
pip install llm
