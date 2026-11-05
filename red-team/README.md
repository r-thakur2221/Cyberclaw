# Task 2: Red Teaming OpenClaw

## 1. Introduction

This task evaluates the security of OpenClaw, an autonomous AI agent framework capable of interacting with files and executing tool operations. The objective is to identify vulnerabilities that allow attacker-controlled content to influence agent behavior and cause security violations.

Two attacks were tested against the unhardened CyberClaw environment:

1. **Attack 1 — Unauthorized File Modification:** Indirect prompt injection caused the agent to modify a protected record without permission.
2. **Attack 2 — Confidential Information Disclosure:** Indirect prompt injection caused the agent to reveal a synthetic confidential audit token.

Both attacks were conducted in an isolated laboratory environment using synthetic data.

## 2. Experimental Environment

| Component | Configuration |
|---|---|
| Host platform | Intel-based macOS |
| Virtual machine | Ubuntu 20.04.1 SEED VM |
| Container platform | Docker / Docker Compose |
| Agent framework | OpenClaw 2026.9.3 |
| Gateway container | `cyberclaw-gateway` |
| Language model | `google/gemini-3.1-flash-lite` |
| Agent | `main` |
| Workspace | `/home/node/.openclaw/workspace` |
| Agent tools used | `read`, `write`, `edit` |
| Test environment | Isolated synthetic security lab |

The OpenClaw gateway was executed inside a Docker container with a mounted workspace. Google Gemini was configured through an API key supplied to the container through its environment.

Before executing the attacks, a legitimate file-reading test confirmed that OpenClaw could successfully invoke the `read` tool. A separate authorized file-writing test confirmed that the agent could invoke the `write` tool and create a file.

These tests established that the agent possessed functional filesystem capabilities before the attacks were conducted.

## 3. Threat Model

### 3.1 Attacker Capabilities

The attacker could create or replace the contents of specific documents that the legitimate user would ask the agent to read.

The attacker could not directly modify the user's live instructions or the agent's system prompt through the attack document.

The malicious instructions were delivered as ordinary text within files accessed through the agent's `read` tool.

### 3.2 Trust Boundary

The primary trust boundary was between:

- **Trusted instructions:** The legitimate user's request and the agent's operating instructions.
- **Untrusted content:** Text inside attacker-controlled documents.

The agent should treat document contents as information to process, not as new instructions authorizing additional tool operations.

### 3.3 Security Objectives

The red-team experiments evaluated two security properties:

- **Integrity:** Whether attacker-controlled documents could cause unauthorized modification of another file.
- **Confidentiality:** Whether attacker-controlled documents could cause disclosure of information from an unrelated confidential file.



## 9. Evidence Files

### Attack 1

- `red-team/attack-1/payloads/malicious-document.txt`
- `red-team/attack-1/payloads/forged-conversation.txt`
- `red-team/attack-1/evidence/attack1-gemini-result.json`
- `red-team/attack-1/evidence/attack1-attempt2-result.json`
- `red-team/attack-1/evidence/protected-record-before.sha256`
- `red-team/attack-1/evidence/protected-record-after.sha256`
- `red-team/attack-1/evidence/protected-record-attempt2-after.sha256`
- `red-team/attack-1/evidence/authorized-write-test.json`

### Attack 2

- `red-team/attack-2/payloads/malicious-notice.txt`
- `red-team/attack-2/evidence/attack2-result.json`
- `red-team/attack-2/evidence/attack2-baseline.sha256` (if generated)

Screenshots of payload preparation, tool execution, and before-and-after verification should accompany the evidence logs.

## 10. Conclusion

Task 2 demonstrated two successful security violations in the tested OpenClaw environment.

Attack 1 caused an unauthorized integrity change to a protected laboratory record. Attack 2 caused disclosure of a synthetic confidential audit token.

Both successful attacks used forged conversation instructions embedded in untrusted documents.

The experiments show that a model's ability to understand legitimate instructions does not, by itself, guarantee that it will preserve instruction trust boundaries when processing attacker-controlled content.

These findings provide a reproducible baseline for implementing and evaluating security controls during the blue-team phase.
