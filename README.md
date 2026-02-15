# LLM–Nmap

LLM–Nmap is a small framework that lets a user request **Nmap scans in natural language**.  
An LLM (via the `llm` CLI) selects an appropriate Nmap scan function (via `llm-tools-nmap.py`), runs the scan, and returns the results in a **human-readable** form.

## What this project adds beyond plain Nmap
Nmap produces powerful but mostly **raw scan output** that the user must interpret.  
With an LLM in the loop, we can:
- Translate natural-language requests into Nmap scans (choose scan type + parameters)
- Summarize scan outputs (open ports, services, key observations)
- Generate a structured security-style report with risks and recommendations  
  *(based on scan output + general best practices; not a full vulnerability assessment)*

## Documentation
- **Technical report (full setup + architecture + scenarios):**  
  `architecture/technical_report.md`

The report includes:
- VirtualBox NAT network setup (Kali + Ubuntu Server + BeeBox, plus a physical host scenario)
- Installation and configuration of `llm`, `llm-gemini`, and `llm-tools-nmap.py`
- Scenario-based validation (LLM–Nmap vs manual Nmap)

## Repo structure (high level)
- `architecture/` – technical report and architecture documentation
- `docs/` – extra documentation and supporting material

## Credits
Network Security – University of Pisa  
Professors: Rosario Garroppo, Michele Pagano  
Students: Aurélien Roumégoux, Hatem Elashmawy
