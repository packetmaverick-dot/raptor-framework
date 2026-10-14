This uses your existing OpenCode model. You do not need a second API key for the harness itself.
Step 1 — Install or verify OpenCode
-----------------------------------------------------------------------------
opencode --version
If already installed, skip installation. Otherwise, use the official installer: curl -fsSL https://opencode.ai/install | bash


Step 2 — Add the five research skills
----------------------------------------------------------------------------
OpenCode supports skills stored under .opencode/skills/, and also recognizes compatible Claude-style skills. 
Copy the Harness Kit skills into your workspace:

cd ~/security-research/roundcube
mkdir -p .opencode/skills
cp -r \
  ~/security-research/tools/harness-kit/skills/0[1-5]-* \
  .opencode/skills/


Inspect what was installed:   find .opencode/skills -name SKILL.md
Open each SKILL.md and customize its placeholders for Roundcube, especially the attack surface, allowed testing methods and report paths.

Step 3 — Create an OpenCode research agent
-----------------------------------------------------------------------------
mkdir -p .opencode/agents
nano .opencode/agents/security-researcher.md

paste the below content 
 
---
description: Multi-stage security research using RAPTOR and Harness Kit
mode: primary
permission:
  edit: ask
  bash: ask
  skill: allow
---
You are an AI security research orchestrator.
Read AGENTS.md before starting.
Use the existing OpenCode session model.
Load the relevant Harness Kit skills for:
recon, hunt, validate, trace, and report.
Use RAPTOR's standalone analysis tools when supported.
Inspect their CLI help before invoking them.
For every candidate vulnerability:
- Identify the attacker-controlled entry point.
- Trace the input to a sensitive sink.
- Identify sanitization and authorization controls.
- Attempt to falsify the hypothesis.
- Reproduce safely in the local test environment.
- Report only supported conclusions.
Keep context focused on relevant files and functions.
Store findings in the research workspace.
Do not claim that an unverified hypothesis is a zero-day.

This agent uses OpenCode's current model because no model override is specified. 




Step 4 — Run OpenCode
------------------------------------------------------------------------------
cd ~/security-research/roundcube
opencode

Select `security-researcher` as your primary agent.

Then provide this task:
Analyze the Roundcube source code at /home/hawkeye/roundcube using the five-stage research harness.
Start with Recon, then Hunt, Validate, Trace, and Report.
Load the relevant skills from .opencode/skills/. Use RAPTOR's standalone scanning tools when appropriate, and inspect their actual CLI help before running them.
Focus on XSS, SQL injection, LFI, path traversal, RCE, unsafe deserialization, and authentication or authorization vulnerabilities.
For each candidate, identify the attacker-controlled source, reachable call chain, sensitive sink, security controls, and exact code references.
Use the existing OpenCode model for reasoning. Do not require a separate LLM API key.
Perform only authorized, non-destructive validation in the local lab. Save structured findings in the research workspace and distinguish confirmed issues from unverified hypotheses.
-------------------------------------------------------------------------------

At this point you have a usable OpenCode-driven research workflow. The model performs the reasoning; RAPTOR and other tools supply mechanical evidence.



Step 5 — Test RAPTOR from OpenCode
-------------------------------------------------------------------------------
Run the following separately in Kali to establish a working baseline:

cd ~/security-research/tools/raptor
uv run python raptor.py doctor
uv run python raptor.py scan --repo /home/hawkeye/roundcube


These are documented standalone commands. They do not require Claude Code. 
RAPTOR writes its own results under its output directory. Have OpenCode inspect the generated findings and copy only relevant summaries into the shared research workspace.
Do not start RAPTOR's agentic mode expecting it to use your OpenCode login automatically. That mode has a separate analysis-model configuration.
