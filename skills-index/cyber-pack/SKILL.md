---
name: cyber-pack
description: Master cybersecurity knowledge pack for coding agents — 750+ sub-skills spanning network security, penetration testing, offensive security, AppSec, cloud/container security, forensics, malware analysis, cryptography, detection engineering, incident response, CTF, DevSecOps, and hardening. Use when writing/reviewing security-related code, hardening systems, doing security assessments, exploit dev, CTF challenges, or any task touching network/security engineering.
---

# Cyber Skills Pack

Umbrella index over 750+ curated security sub-skills (source: Njones17/AI-agent-master-cyber-skills-list). Each sub-skill is a folder under `pack/skills/<category>/<skill-name>/` containing its own `SKILL.md` (+ optional references).

## How to use

1. Pick the relevant **category** below, then `ls pack/skills/<category>/` to see available topics.
2. Read the specific sub-skill's `SKILL.md` (path relative to this file's dir) before doing that class of work.
3. Multiple categories may apply — load only what the task needs.

## Categories

| Category | Covers |
|---|---|
| `network-security` | firewalls (pfSense/NGFW/palo-alto), VLAN segmentation, NAC, Suricata IPS, TLS 1.3, SSL lifecycle, WAF setup, k8s network policies |
| `penetration-testing` | methodology, tooling, reporting, service-specific tests |
| `offensive` | exploit techniques, adversary infrastructure, red team ops |
| `reconnaissance` | OSINT, scanning, enumeration |
| `application-security` | SAST/DAST (CodeQL, semgrep, ZAP, Burp), API security, GraphQL, mobile, AI red-teaming, secret scanning |
| `cloud-security` | AWS/Azure/GCP posture, IAM, serverless |
| `container-security` | Docker/K8s hardening, image scanning |
| `cryptography` | primitives, protocols, common pitfalls in code |
| `detection` | detection engineering, Sigma/YARA, log pipelines |
| `forensics` | disk/memory/network forensics, artifacts |
| `incident-response` | IR playbooks, containment, eradication |
| `malware-analysis` | static/dynamic analysis, unpacking |
| `hardening` | OS/service hardening baselines |
| `identity-access` | IAM patterns, OAuth/OIDC, SSO |
| `blockchain-security` | smart contract audits, web3 pitfalls |
| `ot-ics-security` | industrial control systems |
| `compliance` | frameworks mapping (SOC2, ISO, NIST) |
| `fuzzing` | fuzzing strategies & harnesses |
| `dependency-scanning` | SCA, supply chain |
| `securing-github-actions-workflows`, `implementing-secrets-management-with-vault`, `implementing-secret-scanning-with-gitleaks`, … | DevSecOps pipeline skills |

## Quick starts (most requested)

- Firewall/VPN/TLS work → `pack/skills/network-security/firewall-config/SKILL.md`, `configuring-pfsense-firewall-rules/`, `configuring-tls-1-3-for-secure-communications/`
- Web/API audit → `pack/skills/application-security/` (burpsuite-project-parser, dast-scanning, graphql-security)
- Exploit/CTF → `pack/skills/offensive/`, `pack/skills/penetration-testing/`
