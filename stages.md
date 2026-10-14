How the five stages should work
A useful improvement from the article is to enforce an evidence requirement before advancing a finding.
1. Recon — Map the attack surface
   Identify endpoints, input sources, authentication checks, file operations, database queries and dangerous functions.
   Output: recon/attack-surface.md
2. Hunt — Generate focused hypotheses
   Investigate one class of vulnerability or one subsystem per task. Avoid feeding the entire repository into every agent.
   Output: hunt/candidates.json
3. Validate — Try to disprove findings
   Check whether the path is reachable, whether sanitization works, and whether authentication or configuration prevents exploitation.
   Output: validate/review.md
4. Trace — Establish evidence
   Record the entry point, transformations, call chain and sink. Where possible, use a harmless runtime test.
   Output: trace/evidence.md
5. Report — Produce defensible results
   Report confirmed vulnerabilities separately from uncertain candidates. Include reproduction evidence and remediation.
   Output: reports/final.md
