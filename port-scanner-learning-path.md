# Port Scanner & Subdomain Enumerator — Learning Path

## How to use this path

Nine phases, in order, one prompt per session. Each prompt is self-contained, so it works in a fresh chat.

1. Copy the phase's prompt and paste it as your first message.
2. In a new chat, paste your code from the previous phase right after the prompt.
3. Answer the check questions before moving on. Each phase assumes the last one is solid.
4. At the end of each phase, commit your code (`git commit -m "phase N: ..."`) and write 3–5 lines in a `LEARNING.md` in your own words.

Every prompt enforces the same teaching order: **why → approach → concepts → pause → build → break it → check**. If a session drifts or skips ahead, reply "Back to the format" and name the step you want.

## Phase overview

Phases 0–4 build the port scanner, 5–6 the subdomain enumerator, and 7–8 join them into one finished, tested tool.

| Phase | Focus | You walk away with |
| --- | --- | --- |
| 0 | Orientation, scope, setup | Project repo, test targets confirmed, an nmap baseline to compare against |
| 1 | Networking foundations | The handshake and port states seen with your own eyes in a packet capture |
| 2 | Single-port checker | A function that reports open, closed or filtered for one port |
| 3 | Scanning many ports fast | A concurrent scanner and a benchmark proving why concurrency matters |
| 4 | Service detection | A service guess and banner for every open port |
| 5 | DNS foundations | How subdomains are found, seen through dig and certificate logs |
| 6 | Subdomain enumerator | Passive + brute-force discovery with wildcard detection |
| 7 | CLI and chaining | One command: find subdomains, resolve them, scan them, output JSON |
| 8 | Validate and write up | Tests, accuracy check against nmap, README and a sample report |

---

## Phase 0 — Orientation, scope and setup

You can't judge whether your tool works until you know what "working" looks like, where it fits in real recon, and what you're allowed to point it at.

**Done when:** your repo exists, your test targets respond, and you've saved an nmap baseline of scanme.nmap.org.

```text
I'm building a Python port scanner + subdomain enumerator as a learning project (Beginner project 01 of my cybersecurity roadmap). This is Phase 0 of 8: Orientation, scope and setup. No scanner code yet.

GOAL OF THIS PHASE:
Understand what the finished tool will do and why, where it fits in a real security assessment, what I'm legally and ethically allowed to scan, and get my project and test targets ready.

CONCEPTS TO COVER:
1. Reconnaissance: what it is, passive vs active recon, and where port scanning and subdomain enumeration fit in the stages of a pentest.
2. Attack surface: what it means, and why every open port and forgotten subdomain adds to it.
3. Scope and authorization: rules of engagement, why scanning without permission is a legal problem, and how real pentesters document scope.
4. The tool landscape: what nmap, masscan, subfinder and amass do, and why building my own is still worth it.
5. The finished tool: what it will take as input, what it will output, and the phase-by-phase plan to get there.
6. Project setup: folder layout, Python virtual environments (what problem they solve), and git from the first commit.

FOR THE BUILD STEP (no scanner code yet):
- Set up the project folder, a virtual environment and a git repo.
- Confirm my test targets: a local server (python3 -m http.server 8000) and scanme.nmap.org.
- Run nmap against scanme.nmap.org and save the output as baseline.txt. Explain each part of the output.
- Write a README stub with a short scope statement saying what this tool may be used against.

HOW TO TEACH ME (same order every time):
1. Why: the problem this phase solves and where it shows up in real recon or pentest work.
2. Approach: the plan for this phase, the realistic alternatives, and why we're choosing this one.
3. Concepts: one labeled section per concept above, each fully explained in plain English with an analogy where it helps. Cover every concept before any code or commands. Don't compress or skip any.
   Then STOP and ask if I'm ready to build.
4. Build: small steps. For each step: what we're doing and why, then the commands or code with comments explaining why (not what), how to run it, and the output I should expect.
5. Break it: 2-3 experiments where I change something on purpose. Ask me to predict the result before I run it.
6. Check: 3-5 questions I answer in my own words. Wait for my answers and correct anything I get wrong.

RULES:
- Only this phase. Don't jump ahead.
- Python 3.10+, standard library only unless this phase says otherwise.
- Targets: only localhost, my own lab VMs, or scanme.nmap.org (light scans only).
```

---

## Phase 1 — Networking foundations for scanning

A scanner is just a machine that asks network questions. If you can't read the answers yourself, you can't trust or debug the tool that reads them for you.

**Done when:** you've captured a full handshake and an RST in Wireshark or tcpdump, and can explain open, closed and filtered without notes.

```text
I'm building a Python port scanner + subdomain enumerator as a learning project (Beginner project 01 of my cybersecurity roadmap). This is Phase 1 of 8: Networking foundations for scanning. Phase 0 is done: my repo is set up and I have an nmap baseline of scanme.nmap.org.

GOAL OF THIS PHASE:
Understand exactly what happens on the network when you probe a port, so that every result my scanner prints later means something concrete to me.

CONCEPTS TO COVER:
1. IP address vs port vs service: how one machine runs many services, and what well-known ports are.
2. TCP vs UDP: connection-based vs fire-and-forget, and why TCP scanning gives clearer answers.
3. The TCP three-way handshake: SYN, SYN-ACK, ACK, what each packet means and why TCP needs it.
4. Port states: how a host responds when a port is open, closed or filtered (SYN-ACK, RST, silence, ICMP unreachable), and who sends each reply.
5. What a socket is: the operating system's role, and what my program does vs what the OS does.
6. Connect scan vs SYN (half-open) scan: how they differ on the wire, why one needs root, and which one my Python scanner will be.

FOR THE BUILD STEP (observation, no Python yet):
- Start a local server on port 8000, then capture traffic with Wireshark or tcpdump on the loopback interface.
- Connect to port 8000 (open) and port 8001 (closed) using nc or curl. Show me how to find the SYN, SYN-ACK, ACK and the RST in the capture.
- Run nmap -sT and sudo nmap -sS against one port on scanme.nmap.org while capturing, and show me the difference in the packets.

HOW TO TEACH ME (same order every time):
1. Why: the problem this phase solves and where it shows up in real recon or pentest work.
2. Approach: the plan for this phase, the realistic alternatives, and why we're choosing this one.
3. Concepts: one labeled section per concept above, each fully explained in plain English with an analogy where it helps. Cover every concept before any code or commands. Don't compress or skip any.
   Then STOP and ask if I'm ready to build.
4. Build: small steps. For each step: what we're doing and why, then the commands or code with comments explaining why (not what), how to run it, and the output I should expect.
5. Break it: 2-3 experiments where I change something on purpose. Ask me to predict the result before I run it.
6. Check: 3-5 questions I answer in my own words. Wait for my answers and correct anything I get wrong.

RULES:
- Only this phase. Don't jump ahead.
- Python 3.10+, standard library only unless this phase says otherwise.
- Targets: only localhost, my own lab VMs, or scanme.nmap.org (light scans only).
```

---

## Phase 2 — Single-port checker

Every scanner is one reliable port check repeated many times. Get one port exactly right first, and scaling up later is just plumbing.

**Done when:** `check_port(host, port, timeout)` returns the correct state for an open and a closed port on localhost and on scanme.nmap.org, and you can explain every line.

```text
I'm building a Python port scanner + subdomain enumerator as a learning project (Beginner project 01 of my cybersecurity roadmap). This is Phase 2 of 8: Single-port checker. Phases 0-1 are done: my project is set up, and I understand the TCP handshake and the open, closed and filtered states from packet captures.

GOAL OF THIS PHASE:
Write one function that takes a host, a port and a timeout, and correctly reports whether the port is open, closed or filtered.

CONCEPTS TO COVER:
1. Creating a socket in Python: what AF_INET and SOCK_STREAM mean and what the alternatives are.
2. connect() vs connect_ex(): how each reports failure, and which fits our goal better.
3. Timeouts: why a filtered port would hang forever without one, and how to pick a sensible value (the speed vs accuracy tradeoff).
4. Turning errors into answers: mapping ConnectionRefusedError, socket.timeout and other OSErrors to port states.
5. Cleaning up: why every socket must be closed, and how a with-block guarantees it.
6. Hostname resolution: turning scanme.nmap.org into an IP once, and why not to resolve it again for every port.

FOR THE BUILD STEP:
- Build check_port() one piece at a time, running it after each piece.
- Test it against python3 -m http.server 8000 (open), port 8001 (closed), and ports 22, 80 and 9999 on scanme.nmap.org.
- Compare my results with my nmap baseline from Phase 0.

HOW TO TEACH ME (same order every time):
1. Why: the problem this phase solves and where it shows up in real recon or pentest work.
2. Approach: the plan for this phase, the realistic alternatives, and why we're choosing this one.
3. Concepts: one labeled section per concept above, each fully explained in plain English with an analogy where it helps. Cover every concept before any code or commands. Don't compress or skip any.
   Then STOP and ask if I'm ready to build.
4. Build: small steps. For each step: what we're doing and why, then the commands or code with comments explaining why (not what), how to run it, and the output I should expect.
5. Break it: 2-3 experiments where I change something on purpose. Ask me to predict the result before I run it.
6. Check: 3-5 questions I answer in my own words. Wait for my answers and correct anything I get wrong.

RULES:
- Only this phase. Don't jump ahead.
- Python 3.10+, standard library only unless this phase says otherwise.
- Targets: only localhost, my own lab VMs, or scanme.nmap.org (light scans only).
- If I paste code from an earlier phase, build on it instead of rewriting it.
```

---

## Phase 3 — Scanning many ports fast

Checking 1,000 ports one at a time with a 1-second timeout can take over 16 minutes. This phase is about why that happens and how to make it take seconds without hammering the target.

**Done when:** your scanner takes a port range, scans it concurrently, prints open ports in order, and you've benchmarked it against a one-at-a-time version.

```text
I'm building a Python port scanner + subdomain enumerator as a learning project (Beginner project 01 of my cybersecurity roadmap). This is Phase 3 of 8: Scanning many ports fast. Phases 0-2 are done: I have a working check_port(host, port, timeout) that returns open, closed or filtered.

GOAL OF THIS PHASE:
Turn my single-port checker into a scanner that takes a list or range of ports, scans them concurrently, and reports results in order, while staying polite to the target.

CONCEPTS TO COVER:
1. Parsing port input: handling formats like 22,80,443 and 1-1024, and validating the 1-65535 range.
2. Why one-at-a-time is slow: work through the math with real numbers, and show where the time actually goes (waiting, not computing).
3. I/O-bound vs CPU-bound work, and why it decides which concurrency tool fits.
4. Threads vs processes vs asyncio: how each works, the tradeoffs, and why we'll start with ThreadPoolExecutor.
5. Choosing the worker count: what happens with too few or too many, and the OS limit on open sockets (ulimit).
6. Being a polite scanner: rate limits, why aggressive scans get you blocked or flagged by intrusion detection.
7. Collecting results from concurrent workers safely, sorting them, and showing progress.

FOR THE BUILD STEP:
- Write the port parser first and test it with good and bad inputs.
- Benchmark a one-at-a-time scan of ports 1-1024 on localhost before adding concurrency.
- Add ThreadPoolExecutor, re-run the same benchmark, and compare times.
- Add a configurable worker count and timeout.

HOW TO TEACH ME (same order every time):
1. Why: the problem this phase solves and where it shows up in real recon or pentest work.
2. Approach: the plan for this phase, the realistic alternatives, and why we're choosing this one.
3. Concepts: one labeled section per concept above, each fully explained in plain English with an analogy where it helps. Cover every concept before any code or commands. Don't compress or skip any.
   Then STOP and ask if I'm ready to build.
4. Build: small steps. For each step: what we're doing and why, then the commands or code with comments explaining why (not what), how to run it, and the output I should expect.
5. Break it: 2-3 experiments where I change something on purpose. Ask me to predict the result before I run it.
6. Check: 3-5 questions I answer in my own words. Wait for my answers and correct anything I get wrong.

RULES:
- Only this phase. Don't jump ahead.
- Python 3.10+, standard library only unless this phase says otherwise.
- Targets: only localhost, my own lab VMs, or scanme.nmap.org (light scans only; keep big benchmarks on localhost).
- If I paste code from an earlier phase, build on it instead of rewriting it.
```

---

## Phase 4 — Service detection and banner grabbing

"Port 22 is open" is a guess about SSH, not a fact. Attackers care about what is actually running and which version, because that is what maps to known vulnerabilities.

**Done when:** every open port in your output shows a service guess and a banner (or a clear "no banner"), and you've compared it with `nmap -sV`.

```text
I'm building a Python port scanner + subdomain enumerator as a learning project (Beginner project 01 of my cybersecurity roadmap). This is Phase 4 of 8: Service detection and banner grabbing. Phases 0-3 are done: I have a concurrent scanner that finds open ports on a host.

GOAL OF THIS PHASE:
For each open port, find out what service is actually running and, where possible, its version, instead of assuming from the port number.

CONCEPTS TO COVER:
1. Port number vs actual service: why the well-known port list is only a starting guess, and how services get moved to unusual ports.
2. Banner grabbing: services that speak first (SSH, FTP, SMTP) vs services that wait for you (HTTP), and what to send to each.
3. Receiving data: how recv() works, why one call may not return the whole message, read timeouts, and decoding bytes safely.
4. Why versions matter: how a version string leads to known CVEs, with one real example.
5. TLS on ports like 443: why a plain socket gets nothing useful, and what the ssl module lets me read (certificate subject, issuer, expiry).
6. How nmap -sV does this at scale with its probe database, and the limits of my simpler approach.
7. Banners can lie: how and why admins fake or hide them.

FOR THE BUILD STEP:
- Add a grab_banner() function and plug it into the scanner only for open ports.
- Add a small probe for HTTP so web servers return a header.
- Test against my local http.server, scanme.nmap.org, and a lab VM like Metasploitable 2 if I have one.
- Compare my results with nmap -sV on the same target.

HOW TO TEACH ME (same order every time):
1. Why: the problem this phase solves and where it shows up in real recon or pentest work.
2. Approach: the plan for this phase, the realistic alternatives, and why we're choosing this one.
3. Concepts: one labeled section per concept above, each fully explained in plain English with an analogy where it helps. Cover every concept before any code or commands. Don't compress or skip any.
   Then STOP and ask if I'm ready to build.
4. Build: small steps. For each step: what we're doing and why, then the commands or code with comments explaining why (not what), how to run it, and the output I should expect.
5. Break it: 2-3 experiments where I change something on purpose. Ask me to predict the result before I run it. Include one where I fake a banner with nc and see if my tool is fooled.
6. Check: 3-5 questions I answer in my own words. Wait for my answers and correct anything I get wrong.

RULES:
- Only this phase. Don't jump ahead.
- Python 3.10+, standard library only unless this phase says otherwise.
- Targets: only localhost, my own lab VMs, or scanme.nmap.org (light scans only).
- If I paste code from an earlier phase, build on it instead of rewriting it.
```

---

## Phase 5 — DNS foundations for subdomain enumeration

The juiciest targets are often the ones nobody remembers: an old `dev.` or `staging.` host. DNS is how you find them, so you need to understand it before automating it.

**Done when:** you can explain each DNS record type you queried, show what a non-existent subdomain returns, and find subdomains through certificate transparency by hand.

```text
I'm building a Python port scanner + subdomain enumerator as a learning project (Beginner project 01 of my cybersecurity roadmap). This is Phase 5 of 8: DNS foundations for subdomain enumeration. Phases 0-4 are done: my port scanner works, including banner grabbing. Now I'm starting the subdomain half of the tool.

GOAL OF THIS PHASE:
Understand how DNS works and every way subdomains can be discovered, so I know what my enumerator is doing and where it can go wrong.

CONCEPTS TO COVER:
1. What DNS does and the lookup path: my machine, the resolver, root, TLD and authoritative servers, plus caching.
2. The name hierarchy: root, TLD, domain, subdomain.
3. Record types: A, AAAA, CNAME, NS, MX, TXT, what each is for and which matter for recon.
4. DNS responses: NXDOMAIN vs NOERROR, and what each tells me.
5. Why subdomains matter for attack surface: forgotten dev, staging and admin hosts, with a real-world style example.
6. Passive vs active enumeration: certificate transparency logs and public datasets vs brute-forcing with a wordlist, and the tradeoffs of each.
7. Wildcard DNS: what it is and why it fools naive brute-forcing.
8. Zone transfers (AXFR): what they are and why they're almost always blocked.

FOR THE BUILD STEP (observation, no Python yet):
- Use dig (or nslookup) to query A, CNAME, NS, MX and TXT records, and read each part of the output.
- Query a subdomain that doesn't exist and compare the response with a real one.
- Look up a domain on crt.sh and explain why certificates leak subdomain names.
- Try a zone transfer against zonetransfer.me, which exists for practicing this.

HOW TO TEACH ME (same order every time):
1. Why: the problem this phase solves and where it shows up in real recon or pentest work.
2. Approach: the plan for this phase, the realistic alternatives, and why we're choosing this one.
3. Concepts: one labeled section per concept above, each fully explained in plain English with an analogy where it helps. Cover every concept before any code or commands. Don't compress or skip any.
   Then STOP and ask if I'm ready to build.
4. Build: small steps. For each step: what we're doing and why, then the commands or code with comments explaining why (not what), how to run it, and the output I should expect.
5. Break it: 2-3 experiments where I change something on purpose. Ask me to predict the result before I run it.
6. Check: 3-5 questions I answer in my own words. Wait for my answers and correct anything I get wrong.

RULES:
- Only this phase. Don't jump ahead.
- Targets: domains I own, domains I have permission to test, or zonetransfer.me. Looking up public certificate logs is fine for learning.
```

---

## Phase 6 — Subdomain enumerator

This is where the manual DNS work from Phase 5 becomes a tool: two discovery methods combined, with the false positives filtered out.

**Done when:** given a domain and a wordlist, your enumerator returns unique, live subdomains with their IPs, and doesn't get fooled by a wildcard domain.

```text
I'm building a Python port scanner + subdomain enumerator as a learning project (Beginner project 01 of my cybersecurity roadmap). This is Phase 6 of 8: Subdomain enumerator. Phases 0-5 are done: my port scanner works, and I understand DNS, record types, wildcards and passive vs active enumeration from hands-on dig and crt.sh work.

GOAL OF THIS PHASE:
Build an enumerator that takes a domain, discovers subdomains passively (certificate transparency) and actively (wordlist brute force), filters out false positives, and returns unique live subdomains with their IP addresses.

CONCEPTS TO COVER:
1. Resolving names in Python: what the standard library can do, its limits, and when a DNS library like dnspython is worth adding. Explain the tradeoff and let me decide.
2. Wordlists: where good ones come from (e.g. SecLists), and why size vs speed is a real tradeoff.
3. Reusing concurrency from Phase 3 for DNS lookups, and how DNS rate limits differ from port scanning.
4. Wildcard detection: resolving a random name first, and filtering results that match the wildcard answer.
5. Querying crt.sh from Python: fetching JSON, handling slow or failed responses, and parsing it.
6. Cleaning results: lowercasing, stripping *. entries, removing duplicates, keeping only names inside the target domain.
7. Merging passive and active results, and recording which method found each subdomain.

FOR THE BUILD STEP:
- Build the passive (crt.sh) source first and test it on its own.
- Build the brute-force resolver with a small wordlist, then add concurrency.
- Add wildcard detection and prove it works, either on a wildcard domain I'm allowed to test or with a faked resolver if I don't have one.
- Merge both sources into one clean result list.

HOW TO TEACH ME (same order every time):
1. Why: the problem this phase solves and where it shows up in real recon or pentest work.
2. Approach: the plan for this phase, the realistic alternatives, and why we're choosing this one.
3. Concepts: one labeled section per concept above, each fully explained in plain English with an analogy where it helps. Cover every concept before any code or commands. Don't compress or skip any.
   Then STOP and ask if I'm ready to build.
4. Build: small steps. For each step: what we're doing and why, then the commands or code with comments explaining why (not what), how to run it, and the output I should expect.
5. Break it: 2-3 experiments where I change something on purpose. Ask me to predict the result before I run it.
6. Check: 3-5 questions I answer in my own words. Wait for my answers and correct anything I get wrong.

RULES:
- Only this phase. Don't jump ahead.
- Python 3.10+, standard library only unless we agree on a library in concept 1.
- Targets: active brute force only against domains I own, domains I have permission to test, or zonetransfer.me. Passive certificate log lookups are fine for learning.
- If I paste code from an earlier phase, build on it instead of rewriting it.
```

---

## Phase 7 — CLI, structure and chaining the two tools

Two scripts that work separately aren't a tool yet. This phase turns them into one clean command whose output other tools (and you, later) can rely on.

**Done when:** one command finds subdomains, resolves them, removes duplicate IPs, scans each one, and writes the results as JSON.

```text
I'm building a Python port scanner + subdomain enumerator as a learning project (Beginner project 01 of my cybersecurity roadmap). This is Phase 7 of 8: CLI, structure and chaining the two tools. Phases 0-6 are done: I have a working concurrent port scanner with banner grabbing, and a subdomain enumerator with passive and brute-force sources and wildcard detection.

GOAL OF THIS PHASE:
Turn my code into a proper command-line tool with a clean project structure, consistent output, and a mode that chains subdomain discovery into port scanning.

CONCEPTS TO COVER:
1. Project structure: splitting code into modules (ports, services, dns, output, cli) and why separating logic from input/output makes code testable.
2. Running it as a package with python -m, and what __init__.py and __main__.py do.
3. argparse: arguments, options, defaults, help text, and subcommands (scan, subdomains, recon).
4. Input validation: catching bad hosts, ports and domains early with clear error messages.
5. Output formats: a readable table for humans vs JSON and CSV for machines, and why structured output matters for chaining tools.
6. Logging vs print: log levels, a --verbose flag, and keeping results separate from status messages.
7. Error handling and exit codes: what a tool should do when a host is down or a lookup fails, and why exit codes matter in scripts.
8. The recon pipeline: subdomains, then resolve, then deduplicate IPs, then scan, then report.

FOR THE BUILD STEP:
- Restructure my existing code into the package layout without changing behavior, and run it after the move.
- Add the CLI with the three subcommands.
- Add JSON and table output.
- Build the recon subcommand that chains everything, and run it end to end against an allowed target.

HOW TO TEACH ME (same order every time):
1. Why: the problem this phase solves and where it shows up in real recon or pentest work.
2. Approach: the plan for this phase, the realistic alternatives, and why we're choosing this one.
3. Concepts: one labeled section per concept above, each fully explained in plain English with an analogy where it helps. Cover every concept before any code or commands. Don't compress or skip any.
   Then STOP and ask if I'm ready to build.
4. Build: small steps. For each step: what we're doing and why, then the commands or code with comments explaining why (not what), how to run it, and the output I should expect.
5. Break it: 2-3 experiments where I change something on purpose. Ask me to predict the result before I run it.
6. Check: 3-5 questions I answer in my own words. Wait for my answers and correct anything I get wrong.

RULES:
- Only this phase. Don't jump ahead.
- Python 3.10+, standard library only unless we already added a library in Phase 6.
- Targets: only localhost, my own lab VMs, scanme.nmap.org (light scans), or domains I own or have permission to test.
- If I paste code from an earlier phase, build on it instead of rewriting it.
```

---

## Phase 8 — Validate, test and write it up

A security tool you can't prove is accurate is a liability, and a project nobody can understand from the repo won't help your portfolio. This phase closes both gaps.

**Done when:** your tests pass, you've measured accuracy against nmap, and the repo has a README and a sample report you'd be comfortable showing in an interview.

```text
I'm building a Python port scanner + subdomain enumerator as a learning project (Beginner project 01 of my cybersecurity roadmap). This is Phase 8 of 8, the final phase: Validate, test and write it up. Phases 0-7 are done: my tool is a package with a CLI (scan, subdomains, recon), JSON output, and a working recon pipeline.

GOAL OF THIS PHASE:
Prove my tool is accurate, add automated tests, and document it so the repo works as a portfolio piece.

CONCEPTS TO COVER:
1. Ground truth: comparing my results with nmap and a public subdomain tool, and what false positives and false negatives mean here.
2. Why scanners disagree: timeouts, rate limits, firewalls, and DNS caching, and how to investigate a mismatch.
3. Automated testing with pytest: what to test, and how to test network code without the internet (a tiny local server inside the test, and faking DNS answers).
4. Testing the edge cases: bad input, unreachable hosts, timeouts, wildcard domains.
5. Writing a good README: what it does, install, usage examples, sample output, design decisions, limits, and a responsible-use statement.
6. Writing a short recon report: findings, evidence, and recommendations, the way a pentester would hand it to a client.
7. Where to take it next: SYN scanning with raw packets, asyncio, UDP scanning, and what each would teach me.
8. Talking about it in an interview: the design decisions and tradeoffs I should be able to explain.

FOR THE BUILD STEP:
- Run my tool and nmap against the same target and build a comparison table, then explain every difference.
- Write pytest tests for the port parser, check_port, banner grabbing, wildcard detection and the CLI.
- Write the README and a one-page sample report against an allowed target.
- Push the finished repo to GitHub.

HOW TO TEACH ME (same order every time):
1. Why: the problem this phase solves and where it shows up in real recon or pentest work.
2. Approach: the plan for this phase, the realistic alternatives, and why we're choosing this one.
3. Concepts: one labeled section per concept above, each fully explained in plain English with an analogy where it helps. Cover every concept before any code or commands. Don't compress or skip any.
   Then STOP and ask if I'm ready to build.
4. Build: small steps. For each step: what we're doing and why, then the commands or code with comments explaining why (not what), how to run it, and the output I should expect.
5. Break it: 2-3 experiments where I change something on purpose. Ask me to predict the result before I run it.
6. Check: 3-5 questions I answer in my own words, including two mock interview questions about my design choices. Wait for my answers and correct anything I get wrong.

RULES:
- Only this phase.
- pytest is allowed as a dev dependency; the tool itself stays as it is.
- Targets: only localhost, my own lab VMs, scanme.nmap.org (light scans), or domains I own or have permission to test.
- If I paste code from an earlier phase, build on it instead of rewriting it.
```
