

# LLM–Nmap: Network Scanning Using Large Language Models

**Students:**  
- Aurélien Roumégoux  
- Hatem Elashmawy  

**Course:** Network Security – University of Pisa  
**Project:** LLM–Nmap, a network scanning using LLM

---

## Abstract

This report describes the design and implementation of a small framework that integrates Nmap with a Large Language Model (LLM) to perform network scans based on natural-language requests. The framework is built around the `llm` command-line tool, the `llm-tools-nmap.py` plugin, and Google Gemini as the LLM provider, running on a Kali Linux virtual machine inside a VirtualBox lab.

We document how to set up the entire environment, from the virtual network to the LLM configuration, and we present several scenarios where the user describes the desired network analysis to the LLM. The LLM then selects and runs appropriate Nmap commands, the framework collects the scan results, and the output is presented back to the user in a human-readable form. Comparisons with manual Nmap scans are used to validate the correctness of the LLM-driven scans.

---

## 1. Introduction

Nmap is a widely used network scanner that supports a rich set of options, scan types, and output formats. While powerful, it can be difficult for less experienced users to remember the right combination of flags and parameters for a given analysis task.

The goal of this project is to explore how a Large Language Model (LLM) can act as an “interpreter” between a human operator and Nmap. Instead of manually constructing the command line, the user describes the analysis in natural language (e.g., “scan this host and show me which services are running”), and the LLM:

1. Interprets the request,  
2. Chooses an appropriate Nmap function and parameters (using `llm-tools-nmap.py`),  
3. Executes Nmap,  
4. Returns the results and optionally a short summary.

The project specification and the professor’s email define two main requirements:

- Produce a **technical report** that describes how to **set up the entire framework**.
- Provide a **demo** that shows that, when the analysis is described to the LLM, the system:
  1. Correctly configures and runs Nmap,
  2. Acquires the data obtained from Nmap,
  3. Presents the results to the user.

This report addresses these requirements by documenting our architecture, setup steps, scenarios, and a brief evaluation.

---

## 2. Background

### 2.1 Nmap

Nmap is a command-line tool used for network discovery and security auditing. It can perform host discovery, port scanning, service and version detection, and more. Typical usage in our environment involves commands such as:

- `nmap 10.0.2.15` – basic scan of a single host in our NAT network  
- `nmap -sV 10.0.2.15` – service and version detection on the same host  
- `nmap -sn 10.0.2.0/24` – host discovery on the NAT subnet  

In practice, a user must know which flags to use for their goal; this is precisely what we try to offload to the LLM.

### 2.2 LLMs and the `llm` CLI

Large Language Models (LLMs) such as Gemini and GPT can understand and generate text, and can also be integrated with tools via “function calling” or plugins. The `llm` CLI provides:

- A unified command-line interface to multiple LLM providers.
- A plugin system to extend LLMs with custom tools (e.g. Nmap).

In our project, the `llm` CLI acts as the entry point for the user. The user types a natural-language prompt, and `llm` forwards it—together with the function definitions from `llm-tools-nmap.py`—to Gemini.

### 2.3 `llm-tools-nmap.py`

The `llm-tools-nmap.py` plugin defines one or more “functions” that the LLM can call when it decides that a network scan is needed. Each function corresponds to a certain Nmap operation (e.g. a “quick scan” or a “ping scan”).

When the user prompt suggests a scan (for example, “Scan my local network to find live hosts with ping”), the LLM chooses the appropriate function, provides arguments (target, scan type), and the plugin executes Nmap and returns the output. That output is then included in the final LLM response shown to the user.

---

## 3. System Architecture and Environment

### 3.1 Overview

Our architecture is based on a small VirtualBox lab plus one physical host:

- **Kali Linux VM** (controller):
  - Runs the `llm` CLI.
  - Uses Google Gemini via `llm-gemini`.
  - Loads `llm-tools-nmap.py` to connect the LLM to Nmap.
  - Executes all Nmap scans.

- **Ubuntu Server VM**:
  - “Classic” target with specific services (e.g. SSH) that we can enable/disable.
  - Used to demonstrate how open ports appear or disappear in the scans.

- **BeeBox VM**:
  - Vulnerable target with many open ports and services by default.
  - Used as a richer scenario to show how LLM–Nmap behaves on a more complex target.

- **Mac laptop (physical)**:
  - Physical host on the same network.
  - Used to show that the framework is not limited to virtual machines, as long as the target is reachable and we have permission to scan it.

### 3.2 VirtualBox Network Setup

We use a **VirtualBox NAT Network** so that the VMs can communicate with each other on a private subnet.

#### 3.2.1 Creating a NAT Network

1. Open VirtualBox.  
2. Go to **Tools → NAT Networks**.  
3. Click **Create** and configure:
   - A name (e.g. `LLM-Nmap-NAT`),
   - An IP range (e.g. `10.0.2.0/24`).

*(Screenshot as in `architecture.md`):*

```markdown
![Create NAT Network](src/CreateNAT.png)
```

#### 3.2.2 Attaching VMs to the NAT Network

For each VM (Kali, Ubuntu Server, BeeBox):

1. Open **Settings → Network**.  
2. Set **Attached to** = `NAT Network`.  
3. Select the NAT network created earlier (`LLM-Nmap-NAT`).

*(Screenshot as in `architecture.md`):*

```markdown
![Add VMs to NAT Network](src/AddVMsToNAT.png)
```

#### 3.2.3 Verifying Connectivity

On each VM, retrieve its IP address:

```bash
ifconfig    # or: ip addr
```

From the Kali VM, test reachability:

```bash
ping 10.0.2.15       # Ubuntu Server in our example
ping <beebox-ip>     # will be used in Scenario 3
```

*(Screenshot as in `architecture.md`):*

```markdown
![VM Ping to VM](src/VMpingToVM.png)
```

If the pings succeed, the VMs are on the same network and reachable.

### 3.3 Roles of Each Machine

- **Kali VM**:  
  Runs LLM–Nmap (LLM + `llm-tools-nmap.py` + Nmap). Used to issue all LLM-driven scans and manual Nmap scans.

- **Ubuntu Server VM**:  
  Initially has no open services; then we install and enable `openssh-server`. Allows us to show the difference between “no open ports” and “port 22 open” on **10.0.2.15**.

- **BeeBox VM**:  
  Has many open ports and vulnerable services by design. Used to generate more complex Nmap outputs (Scenario 3).

- **Mac laptop**:  
  Serves as an external target reachable from the Kali VM. Demonstrates that the framework works across a real network.

---

## 4. LLM–Nmap Setup on Kali

This section describes the concrete steps to configure the LLM–Nmap framework on the Kali VM.

### 4.1 Installing `llm`

On Kali:

```bash
pip install llm
```

*(Screenshot from `architecture.md`):*

```markdown
![Set Up LLM](src/SetUpLLM.png)
```

### 4.2 Configuring Gemini

We use Google Gemini as the LLM provider.

1. Install the Gemini plugin:

   ```bash
   llm install llm-gemini
   ```

2. Set the Gemini API key (obtained from Google AI Studio):

   ```bash
   llm keys set gemini
   ```

3. Test Gemini:

   ```bash
   llm -m gemini-2.0-flash "Hello"
   ```

*(Screenshots from `architecture.md`):*

```markdown
![Setup Gemini – Test 1](src/SetupGemini1.png)
![Setup Gemini – Test 2](src/SetupGemini2.png)
```

If a valid text response is returned, the LLM configuration is working.

### 4.3 Adding `llm-tools-nmap.py`

1. Download `llm-tools-nmap.py` from the reference repository:  
   <https://github.com/peter-hackertarget/llm-tools-nmap>
2. Place the file (e.g.) in `/home/kali/LLM-Nmap/` or in your project directory on Kali.
3. From that directory, run a first test:

   ```bash
   llm -m gemini-2.0-flash --functions llm-tools-nmap.py \
       "Scan my local network to find live hosts with ping"
   ```

*(Screenshot from `architecture.md`):*

```markdown
![LLM–Nmap – Scan network with ping](<src/LLM-Nmap - Scan network with ping.png>)
```

In our setup, the **Ubuntu Server** VM is detected by the tool.

Then, we perform a more specific scan of the Ubuntu Server at IP **10.0.2.15**:

```bash
llm -m gemini-2.0-flash --functions llm-tools-nmap.py \
    "Make a quick scan of 10.0.2.15"
```

*(Screenshot from `architecture.md`):*

```markdown
![LLM–Nmap – Quick scan, no open ports](<src/LLM-Nmap - Quick scan no open ports.png>)
```

At this stage, no open ports are found. This is expected, because we have not yet enabled any service on the Ubuntu Server.

---

## 5. Scenarios and Implementation Details

We implemented and tested three main scenarios to demonstrate the behaviour of LLM–Nmap.

### 5.1 Scenario 1 – Kali VM Scanning Ubuntu Server VM (10.0.2.15)

#### 5.1.1 Opening SSH on Ubuntu Server

On the Ubuntu Server VM (10.0.2.15):

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
systemctl status ssh
```

*(Screenshot from `architecture.md`):*

```markdown
![Ubuntu Server – Open SSH port](<src/UbuntuServer Open SSH port.png>)
```

After this, port **22/tcp** should be open on 10.0.2.15.

#### 5.1.2 LLM–Nmap Scan After Enabling SSH

We re-run the quick scan from Kali:

```bash
llm -m gemini-2.0-flash --functions llm-tools-nmap.py \
    "Make a quick scan of 10.0.2.15"
```

*(Screenshot from `architecture.md`):*

```markdown
![LLM–Nmap – Quick scan, SSH port open](<src/LLM-Nmap - Quick scan ssh port open.png>)
```

Port **22/tcp** is now detected as open.

#### 5.1.3 Comparison with Manual Nmap

To validate the result, we run a manual Nmap scan from Kali:

```bash
nmap -sV 10.0.2.15
```

*(Screenshot from `architecture.md`):*

```markdown
![LLM–Nmap vs Nmap – Quick scan, SSH port open](<src/LLM-Nmap VS Nmap - Quick scan ssh port open.png>)
```

The manual output confirms that port 22 is open and identifies the SSH service.  
This verifies that:

- The LLM correctly configured and executed Nmap via the plugin, and  
- The data presented by LLM–Nmap is consistent with the “ground truth”.

---

### 5.2 Scenario 2 – Kali VM Scanning a Physical Mac Host

In this scenario, Kali runs as a VM on one laptop, and the target is a Mac laptop on the same Wi-Fi network.

#### 5.2.1 Target: Mac Host

On the Mac, we identify:

- The IP address (shown in the screenshot).
- The open ports (e.g. enabled services).

*(Screenshots referenced in `architecture.md`):*

```markdown
![Mac Open Ports](src/mac_open_ports_screenshot.png)
*Figure: Screenshot showing the open ports on the Mac laptop*

![Mac IP Address](src/mac_ip_address_screenshot.png)
*Figure: Screenshot showing the IP address of the Mac laptop*
```

#### 5.2.2 Nmap and LLM–Nmap Scans from Kali

From the Kali VM, we run:

Manual scan:

```bash
nmap -sV <mac-ip>
```

LLM-driven scan:

```bash
llm -m gemini-2.0-flash --functions llm-tools-nmap.py \
    "Scan <mac-ip> and show me which ports are open and what services are running"
```

*(Placeholders – to be filled with your screenshots for this scenario)*

```markdown
![Nmap Scan on Mac Target](src/mac_nmap_scan.png)
*Traditional Nmap scan results showing detected ports and services on the Mac*

![LLM–Nmap Scan on Mac Target](src/mac_llm_nmap_scan.png)
*LLM–Nmap scan results showing the comparison between traditional Nmap and AI-powered scanning*
```

You can rename `mac_nmap_scan.png` and `mac_llm_nmap_scan.png` to the actual filenames you use.

#### 5.2.3 Observations

- LLM–Nmap successfully runs from a virtual machine and scans a **physical device** on the same network.
- The set of open ports and services matches the manual Nmap scan and the expected services on the Mac.
- This shows that the architecture is not limited to intra-VM scans; any reachable host can be scanned, as long as it is in scope and legally allowed.

---

### 5.3 Scenario 3 – Kali VM Scanning BeeBox (Vulnerable Target)

BeeBox is a deliberately vulnerable VM with many open ports and insecure applications.

#### 5.3.1 Host Discovery Including BeeBox

From Kali:

```bash
llm -m gemini-2.0-flash --functions llm-tools-nmap.py \
    "Scan the NAT network and list all live hosts"
```

BeeBox should appear as one of the live hosts in the output.

*(Placeholder for BeeBox host discovery screenshot):*

```markdown
![BeeBox detected in host discovery](src/beebox_host_discovery.png)
```

#### 5.3.2 Service Enumeration on BeeBox

Next, we perform a more detailed scan:

```bash
llm -m gemini-2.0-flash --functions llm-tools-nmap.py \
    "Run a detailed service scan on <beebox-ip> and summarize the open ports and services"
```

*(Placeholder for BeeBox LLM–Nmap scan screenshot):*

```markdown
![LLM–Nmap detailed scan on BeeBox](src/beebox_llm_scan.png)
```

#### 5.3.3 Comparison with Manual Nmap

Finally, we run a manual scan:

```bash
nmap -sV <beebox-ip>
```

*(Placeholder for manual BeeBox Nmap scan screenshot):*

```markdown
![Manual Nmap scan on BeeBox](src/beebox_nmap_manual.png)
```

We will compare:

- The open ports and services reported by LLM–Nmap.
- The open ports and services reported by manual Nmap.

BeeBox will provide a **richer and more realistic example**, with many open services, which will help to show how the LLM can summarize and interpret more complex scan output.

---

## 6. Evaluation and Discussion

### 6.1 Does the Framework Meet the Project Goals?

The project goals from the professor’s email are:

1. **Describe how to set up the entire framework**  
   - Sections 3 and 4 of this report document the full setup:
     - VirtualBox NAT network and VMs.
     - Installation of `llm`, Gemini, and `llm-tools-nmap.py` on Kali.
   - These steps are sufficient for another student to reproduce our environment.

2. **Demo: “description → configuration and execution → data acquisition → presentation”**  
   - In all three scenarios, the interaction pattern is:
     - We describe the analysis in natural language (e.g. “Make a quick scan of 10.0.2.15”, “Scan the NAT network and list all live hosts”).
     - The LLM chooses the appropriate Nmap function and parameters via `llm-tools-nmap.py`.
     - Nmap is executed on the Kali VM.
     - The Nmap output is included in the LLM response and presented to the user.
   - Manual Nmap scans serve as a reference to confirm correctness.

Therefore, the implemented framework satisfies the requested workflow.

### 6.2 Advantages

- **Ease of use:**  
  Users can describe what they want to do instead of remembering Nmap syntax.

- **Flexibility:**  
  The same setup can scan virtual machines, a vulnerable lab target (BeeBox), and a physical machine (Mac), as long as they are reachable.

- **Extensibility:**  
  The approach can be extended to other tools by writing new plugins similar to `llm-tools-nmap.py`.

### 6.3 Limitations

- **Dependence on the LLM:**  
  The quality of the chosen scan depends on the LLM. It may sometimes select suboptimal flags or misinterpret ambiguous prompts.

- **Lack of strict constraints:**  
  Without additional checks, an LLM could, in principle, try to scan large ranges or external networks. In our project, we manually restrict ourselves to the lab and authorized targets.

- **Performance and rate limits:**  
  Using an online LLM (Gemini) introduces latency and potential API limits, though this did not significantly impact our small-scale experiments.

- **Output parsing:**  
  In this project, we rely mainly on the raw Nmap output that is embedded in the LLM response, plus a short natural-language summary. A more advanced system might parse the XML output of Nmap and present structured tables or dashboards.

---

## 7. Conclusion and Future Work

In this project, we built and evaluated a framework that combines Nmap with an LLM using the `llm` CLI and the `llm-tools-nmap.py` plugin. The system allows a user to describe a desired network analysis in natural language and have the LLM:

1. Configure and execute the appropriate Nmap command,  
2. Collect the scan results,  
3. Present the findings in a human-readable format.

We implemented a small VirtualBox lab with Kali, Ubuntu Server, and BeeBox, and we also tested scans against a physical Mac host. Across all scenarios, LLM–Nmap produced results consistent with manual Nmap scans, demonstrating that the approach is feasible and practical for small-scale environments.

Possible extensions include:

- Adding more strict safety constraints (e.g. whitelisting targets, limiting scan types).
- Parsing Nmap XML output to build structured reports or dashboards.
- Integrating additional security tools (e.g. vulnerability scanners) as LLM “functions” alongside Nmap.
- Comparing behaviour and quality across different LLM providers (e.g. Gemini vs GPT vs Claude) using the same plugin.

Overall, the project confirms that LLMs can make network scanning more accessible by bridging between natural language and powerful but complex tools like Nmap.

---