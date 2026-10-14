Claude Code setup 
Claude Code has the most direct integration with RAPTOR.

Step 1 — Install Claude Code
---------------------------------------------------------------------------------------------------
curl -fsSL https://claude.ai/install.sh | bash
claude --version
Launch claude and authenticate with an eligible account. 
---------------------------------------------------------------------------------------------------



Step 2 — Run RAPTOR natively
---------------------------------------------------------------------------------------------------
cd ~/security-research/tools/raptor
export PATH="$PATH:$PWD/bin"
raptor /home/hawkeye/roundcube
---------------------------------------------------------------------------------------------------
RAPTOR's launcher loads its Claude Code commands, skills and security configuration. 


Inside the interactive RAPTOR session:
/project create roundcube --target /home/hawkeye/roundcube
/project use roundcube
/understand --map
/scan
/agentic --threat-model --validate
/project findings

The /agentic stage may incur analysis-model charges depending on your configuration. Start with /scan if you only want static results.


Step 3 — Use Harness Kit skills with Claude Code
---------------------------------------------------------------------------------------------------
For the separate five-stage workflow:
cd ~/security-research/roundcube
mkdir -p .claude/skills
cp -r ~/security-research/tools/harness-kit/skills/0[1-5]-* .claude/skills/


Create Claude's project instructions:
--
cat > CLAUDE.md <<'EOF'
Read AGENTS.md for the research scope and rules.
Use the five-stage Harness Kit skills.
Write results to the shared research workspace.
Never promote unverified findings to confirmed status.
EOF


Then launch: claude
---------------------------------------------------------------------------------------------------

You can invoke the adapted skills sequentially, such as /01-recon, /02-hunt, /03-validate, /04-trace, and /05-report.
Important: Running claude from your research workspace loads your custom Harness Kit setup, not RAPTOR's complete native configuration. Use the raptor launcher for native RAPTOR functionality.








