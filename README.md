# Port Scanner and Subdomain Enumerator

A port scanner and subdomain enumerator written in Python (standard library only), built step by step as a learning project: Beginner project 01 of my cybersecurity roadmap.

**Status:** Phase 0 of 8 (setup and scope). No scanner code yet.

## Scope and authorization

Scanning a system without its owner's permission can be illegal. This tool is only ever pointed at the targets below. Anything not listed is out of scope. To add a target, I get written permission first and add it to this table before running anything against it.

| Target | Why it's allowed | Allowed activity | Phases |
| --- | --- | --- | --- |
| localhost (127.0.0.1) | My own machine | Any scanning | All |
| My own lab VMs (e.g. Metasploitable 2) | I own them | Any scanning, kept on my own machine or network | 0-4, 7-8 |
| scanme.nmap.org | The Nmap Project allows it: http://scanme.nmap.org/ | Light port scans, a few per day. No brute-forcing. Big benchmarks run on localhost instead. | 0-4, 7-8 |
| zonetransfer.me | Published by security researcher DigiNinja for practicing DNS zone transfers | DNS queries only. No port scans. | 5-6 |
| Public certificate logs (crt.sh) | Public data; the target is never contacted | Passive lookups. A host found this way is not in scope until it is in this table. | 5-8 |
| Domains I own, or have written permission to test | Ownership, or written permission (stored privately, not in this repo) | Only what the permission covers | 5-8 |

### Rules of engagement

- **Permission belongs to the name.** If a target's IP address changes, the new address is in scope and the old one is not.
- **Permission for one activity doesn't cover another.** Being allowed to query zonetransfer.me's DNS doesn't mean I may port-scan its hosts.
- **Active scans leave traces.** Targets log what I send them, so I assume someone will read those logs.

## Setup (Windows)

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Needs Python 3.10 or newer. nmap is used only for comparison.

## Files

- `baseline.txt`: nmap's results for scanme.nmap.org, made with `nmap -sT --reason -oN baseline.txt scanme.nmap.org`. It's the answer key my scanner is checked against.
- `LEARNING.md`: what I learned in each phase, in my own words.
