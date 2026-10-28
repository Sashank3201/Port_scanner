# Learning notes

## Phase 0: Orientation, scope and setup

- Scope comes first. I only scan what I have permission for, the permission belongs to the name (not the IP), and it covers specific activities only. The list lives in my README.
- A SHA-256 hash proves a download wasn't changed on the way to me (MD5 and SHA-1 are broken). Typing `python` ran another app's Python because of PATH order; a venv gives this project its own.
- The .gitignore has to exist before the first commit, and anything committed stays in history, so secrets never go in the repo.
- My nmap baseline: scanme has 22, 80 and 9929 open out of nmap's top 1,000 ports. SERVICE is only a guess from the port number, and checking ports in parallel turned a 4+ minute job into 8 seconds.
- Open question: the "Host is up" reset came back with TTL 126, but scanme runs Linux, which starts at 64. Something near me may have answered for it. Check with a packet capture in Phase 1.
