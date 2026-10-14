Codex CLI setup 
-
Codex can use the same research workspace, instructions and standalone tools.

Step 1 — Install Codex
-------------------------------------------------------------------------
npm install -g @openai/codex
codex --version

Alternatively, use the official Linux installer. 
-------------------------------------------------------------------------


Step 2 — Authenticate
-------------------------------------------------------------------------
codex
Follow the sign-in instructions. Codex supports account-based authentication as well as API-based workflows; you do not necessarily need to purchase a separate API key.

Step 3 — Start in the shared workspace
-------------------------------------------------------------------------
cd ~/security-research/roundcube
codex


Codex can use the AGENTS.md instructions you created earlier.
-------------------------------------------------------------------------

Give Codex this prompt:
-------------------------------------------------------------------------
Act as a security research orchestrator for the authorized local Roundcube source tree.
Read AGENTS.md and the five Harness Kit stage prompts located under /home/hawkeye/security-research/tools/harness-kit/prompts.
Perform the research sequentially:
Recon → Hunt → Validate → Trace → Report.
Use the existing Codex model for reasoning and RAPTOR's standalone Python CLI for mechanical scanning.
Prioritize XSS, SQLi, LFI, path traversal, RCE, and authorization flaws.
Investigate findings with exact source-to-sink traces. Challenge each hypothesis before reporting it. Use safe reproduction tests against the local lab only.
Save intermediate artifacts and final reports to the shared research workspace. Do not modify the target application's source code.
-------------------------------------------------------------------------



Codex will not automatically acquire RAPTOR's Claude-specific slash commands. It can instead run the standalone Python tools and interpret their output.
