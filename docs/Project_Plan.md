# LLM–Nmap Project Plan

## 1. Project Overview

**Goal:**  
Build a small framework where a user can describe a network analysis task in natural language to an LLM, and the system will:

1. Let the LLM choose appropriate Nmap parameters (target, ports, scan type).
2. Run Nmap with those parameters.
3. Capture Nmap’s output.
4. Present the results to the user in a clear, summarized way.

**Deliverables:**

- A working command-line demo.
- A technical report explaining how to set up and use the framework.
- Example demo scenarios with real outputs.

---

## 2. Professor’s Requirements (from email)

- The project’s main goal is to produce a **technical report** that describes how to **set up the entire framework**.
- The **demo must show** that, when the analysis is described to the LLM:
  1. The LLM **correctly configures and runs Nmap**.
  2. The system **acquires the data** obtained from Nmap.
  3. The system **presents the results** to the user.

We will explicitly design our architecture, code, and demo to satisfy these three points.

---

## 3. Architecture (High-Level)

**Components:**

- **User**
  - Writes a natural-language request (e.g. “Scan 192.168.56.101 and show me open HTTP and SSH ports”).

- **LLM Layer**
  - Implemented using the `llm` CLI + a provider (Google Gemini).
  - Receives the user prompt and a description of the Nmap tool.
  - Decides when to call the Nmap tool and with which arguments.

- **Nmap Tool (Python)**
  - Function that builds and runs an Nmap command (`subprocess`).
  - Returns:
    - The exact command used.
    - The raw Nmap output.
    - A status/return code.

- **Result Processing / Presentation**
  - Stores raw Nmap output under `results/`.
  - Optionally parses or sends the output back to the LLM to produce a human-friendly summary (list of hosts/ports/services).

- **Test Network (Lab)**
  - 1–2 target machines (VirtualBox VMs) with known open ports.
  - Used to validate that LLM-driven Nmap matches manual Nmap scans.

**Data Flow (text version):**

1. User → writes prompt.
2. LLM → interprets prompt, chooses Nmap tool + arguments.
3. Nmap tool → runs Nmap with those arguments.
4. Nmap output → saved and optionally summarized.
5. User → sees the Nmap command and a clear summary of results.

---

## 4. Technology Stack

- **OS / Environment**
  - Kali Linux running inside a VirtualBox VM (host machine: local laptop/desktop).

- **Scanning**
  - `nmap` (CLI) installed on the Kali VM.

- **Programming Language**
  - Python 3 (Nmap wrapper + orchestration scripts) running on the Kali VM.

- **LLM Integration**
  - `llm` CLI by Simon Willison.
  - `llm-gemini` plugin as the provider integration.
  - Google Gemini model accessed via API key (configured in the Kali environment).
  - Tool/function-calling to let the LLM invoke our Python Nmap wrapper.

- **Documentation**
  - Markdown files in `docs/`.
  - Optional diagrams created with any external tool (exported as PNG and referenced in the report).

---

## 5. Planned Demo Scenarios (Draft)

We will implement at least **two** scenarios:

1. **Host Discovery on a Subnet**
   - Prompt example:  
     “Scan 192.168.56.0/24 and list all live hosts.”
   - Expected behavior:
     - LLM chooses an Nmap host discovery scan (e.g. `nmap -sn 192.168.56.0/24`).
     - Output: list of IPs that responded.

2. **Service Enumeration on a Single Host**
   - Prompt example:  
     “On 192.168.56.101, run a detailed service scan and tell me which services and versions are running.”
   - Expected behavior:
     - LLM chooses an Nmap service scan (e.g. `nmap -sV 192.168.56.101`).
     - Output: list of open ports with service names and versions.

(Optional third scenario later)  
3. **Targeted Ports Check**
   - Prompt example:  
     “Check if 192.168.56.102 has SSH and HTTP open and show only those ports.”
   - Expected behavior:
     - LLM calls Nmap with `-p 22,80`.
     - Output: only SSH/HTTP if open.

---

## 6. Work Split (Students)

Student A (Hatem)

- Co-implemented and exercised the LLM–Nmap framework from the user / operator side:
  - Configured and used the `llm` CLI with Gemini to run LLM-driven Nmap scans.
  - Executed and validated multiple scanning scenarios (VM–VM, VM–Mac, BeeBox) based on the framework.
  - Actively participated in debugging connectivity, NAT networking, and scan behaviour during the tests.
- Designed and documented the experiments:
  - Described the exact steps to reproduce the lab (commands, parameters, IPs, tools).
  - Structured the scenarios and highlighted how the LLM translates natural-language prompts into Nmap commands.
- Wrote the technical report:
  - System Architecture and Environment (from the lab point of view).
  - Setup and Implementation of LLM–Nmap (Kali, `llm`, Gemini, `llm-tools-nmap.py`).
  - Detailed description of the scenarios, comparisons with manual Nmap, and discussion of advantages/limitations.
  - Drafted the initial outline for the final presentation.

Student B (Aurélien)

- Implemented and maintained the practical framework:
  - Set up the VirtualBox lab (Kali, Ubuntu Server, BeeBox, NAT network, IP addressing).
  - Installed and configured the `llm` CLI, Gemini, and the existing `llm-tools-nmap.py` plugin on Kali.
  - Defined and ran the initial scanning scenarios and collected raw outputs and screenshots.
- Contributed to the experimental design:
  - Helped refine the demo scenarios (what to scan, which ports/services to highlight, which screenshots to capture).
  - Provided the baseline Nmap runs used for comparison with LLM-driven scans.

Both (Hatem & Aurélien)

- Tested and debugged the end-to-end demo on Kali inside VirtualBox and across the local network.
- Verified that, given a natural-language description of an analysis, the LLM:
  - correctly configures and runs Nmap,
  - acquires the scan results,
  - and presents them back to the user.
- Polished the technical report and prepared the presentation slides.
---

## 7. Open Questions / To Decide

- Which **exact Gemini model** we will use (e.g. a faster/cheaper model vs a higher-quality model).
- Which Nmap scan types we will officially support in the tool:
  - `default` (basic TCP scan),
  - `service`/`-sV` (service and version detection),
  - `host_discovery`/`-sn` (ping/host discovery), etc.
- How strict we want to be with safety constraints:
  - Restricting targets to private IP ranges only.
  - Restricting the maximum subnet size (e.g. disallowing very large scans).
  - Limiting which Nmap flags the LLM is allowed to use.
- Whether we will parse Nmap output ourselves (basic parsing of ports/services) or rely entirely on the LLM to interpret the raw text when generating summaries.