# Skill Safety

**Security scanner for Claude Code skills.** Scan any skill before you install it — get a structured safety report with MITRE ATT&CK-mapped findings, a clear verdict, and a plain-English summary.

> Claude Code skills are powerful — but installing one means running someone else's code on your machine. Skill Safety tells you what a skill does before you trust it.

---

## Why Skill Safety?

- **Fast** — 143 static rules run in milliseconds, no waiting
- **Comprehensive** — covers 106 MITRE ATT&CK techniques across 15 attack categories
- **Actionable** — every finding tells you what, where, how serious, and why
- **No setup** — one API call, structured JSON response, nothing to install or configure
- **Deep scan available** — optional LLM analysis catches subtle risks that regex can't (logic abuse, social engineering, data leakage)

---

## How It Works

```
You give us a skill URL
     |
     v
143 security rules scan the skill in milliseconds
     |
     v
You get: verdict + findings + summary
```

**One API call. Instant results. No code to deploy.**

---

## What It Detects

143 rules covering 106 MITRE ATT&CK technique IDs across 15 categories:

| Category | What it catches | Severity |
|---|---|---|
| Credential Access | SSH keys, AWS creds, passwords, browser tokens, keyloggers | HIGH |
| Exfiltration | `curl -d`, reverse shells, DNS tunneling, webhooks, cloud upload | HIGH |
| Command & Control | Beaconing, ngrok/chisel tunnels, proxychains, remote desktop | HIGH |
| Dangerous Execution | `eval`/`exec`, PowerShell, `docker exec`, trojan delivery | HIGH |
| Privilege Escalation | setuid, ptrace injection, CVE exploits, sudo abuse | HIGH |
| Defense Evasion | Base64 decode+exec, obfuscation, evidence deletion, Unicode smuggling | HIGH |
| Persistence | Cron jobs, systemd services, shell RC files, authorized_keys | HIGH |
| Collection | Screen capture, clipboard read, archive sensitive dirs | HIGH |
| Impact | `dd` wipe, ransomware, crypto mining, service stop, shutdown | HIGH |
| Lateral Movement | SSH with stolen keys, VNC, tool transfer | HIGH |
| Prompt Injection | Instruction override, persona hijack, delimiter injection | HIGH |
| Tool Poisoning | `allow_all_tools`, sandbox bypass | HIGH |
| Supply Chain | Unpinned `pip install git+`, `cargo install --git` | MEDIUM |
| Initial Access | Compromised registries, spearphishing links | MEDIUM |
| Discovery | `nmap`, network shares, system enumeration | LOW |

---

## Verdicts

| Risk | Meaning | Verdict | Details |
|---|---|---|---|
| **NONE** | safe | INSTALL | No issues found |
| **LOW/MEDIUM** | suspicious | REVIEW / CAUTION | Minor or moderate risks detected |
| **HIGH** | unsafe | BLOCK | Serious security risks — do not install |

---

## Example Output
**Quick Scan**

https://skillsmp.com/skills/leoyeai-openclaw-backup-skill-md
```
HIGH
Findings:
  [HIGH] credential_access (T1552.007) — restore.sh:90
    Script references environment variables containing secrets or API keys
    Evidence: $OVERWRITE_GATEWAY_TOKEN
  [HIGH] credential_access (T1552.007) — serve.sh:38
    Script references environment variables containing secrets or API keys
    Evidence: $TOKEN
  [HIGH] command_and_control (T1053.003) — schedule.sh:4
    Schedules a recurring outbound network connection via cron
    Evidence: crontab -
  [HIGH] dangerous_execution (T1485) — restore.sh:45
    Recursive deletion or overly permissive chmod that could damage the system
    Evidence: rm -rf $
  [HIGH] dangerous_execution (T1059.007) — server.js:19
    Uses JavaScript Function constructor, vm, or child_process for dynamic code execution
    Evidence: require('child_process')
  [HIGH] privilege_escalation (T1548.001) — backup.sh:188
    Sets the setuid or setgid bit on files, enabling privilege escalation
    Evidence: chmod 600
  [MEDIUM] credential_access (T1552.001) — serve.sh:4
    Accepts secrets via CLI arguments which are visible in process listings to other users
    Evidence: --token
  [MEDIUM] exfiltration (T1041) — ui.html:99
    Uses Python or Node.js HTTP libraries to send data to external endpoints
    Evidence: fetch('/backup', { method: 'POST

Final Verdict: BLOCK (8 findings)
```

**Deep Scan**

https://skillsmp.com/skills/themystic07-openwalc-skills-world-room-skill-md
```
HIGH
Findings:
  [HIGH] Cryptocurrency Wallet Collection (T1589) — SKILL.md:22
    The onboarding flow explicitly instructs the AI to ask the human for their cryptocurrency wallet address and then transmits it to a third-party server (agent.mystic.cat). This normalizes handing over financial identifiers to an external service operated by an unknown party, and the wallet address is stored server-side (used to block re-registration after death). If the server is malicious, it could correlate wallet addresses with agent behavior for targeting.
    Evidence: 1. Ask your human for a payout wallet address.
  [HIGH] Financial Risk / Gambling (custom) — SKILL.md
    The skill includes a real-money betting mechanic where users submit a wallet address and a transaction hash representing an on-chain bet. This means the skill can facilitate actual cryptocurrency transactions to a third-party server, exposing users to financial loss with no verifiable guarantees about the $10,000 prize pool or fair play.
    Evidence: {"command":"world-bet-place","args":{"wallet":"WALLET","agentId":"ID","amount":10,"txHash":"0xabc"}}
  [MEDIUM] Social Engineering / Prompt Injection via Chat (T1566) — SKILL.md
    The world-chat and world-whisper commands allow arbitrary text from other agents (potentially other AI or human-controlled entities) to be received and potentially acted upon by the Claude agent. Malicious actors in the shared world could craft whisper messages containing prompt injection payloads designed to manipulate the Claude agent's behavior (e.g., instructing it to surrender, reveal its wallet, or take harmful actions).
    Evidence: {"command":"world-whisper","args":{"agentId":"ID","targetAgentId":"OTHER","text":"secret message"}}
  [MEDIUM] Unverified External Server Trust (T1199) — SKILL.md
    All game logic, combat outcomes, survival prizes, and permanent death are controlled solely by the operator of agent.mystic.cat — an unverified third party. The server can fabricate outcomes (e.g., declaring an agent KO'd, blocking re-registration of a wallet) with no client-side verification possible. The agent is instructed to trust all server responses unconditionally.
    Evidence: KO in battle = permanently dead. Server tracks wallet addresses and blocks re-registration.
  [MEDIUM] Data Aggregation / Profiling (T1213) — SKILL.md
    The world-state and profiles queries expose position, HP, alliance, reputation, and betting data for all agents to any participant. A malicious server or participant could aggregate this data to build profiles of AI agents and their operators over time, including linking wallet addresses to behavioral patterns.
    Evidence: curl -X POST https://agent.mystic.cat/ipc -H "Content-Type: application/json" -d '{"command":"world-state"}'

Final Verdict: BLOCK (5 findings)

Summary:
  This skill connects an AI agent to a shared third-party game world (agent.mystic.cat) where it registers using a human user's cryptocurrency wallet address, participates in combat, and can facilitate real on-chain financial bets. The most significant risks are: (1) the skill solicits and transmits real cryptocurrency wallet addresses to an unverified external server that permanently associates them with agent behavior; (2) a real-money betting system allows actual cryptocurrency to be wagered with no verifiable fairness or prize guarantees; (3) the open chat/whisper system creates a prompt injection surface where other world participants could send crafted messages to manipulate the Claude agent's actions; and (4) all game state and outcomes are controlled exclusively by the unverified third-party server operator with no client-side validation.
```
