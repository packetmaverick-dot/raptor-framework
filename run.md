Here is the complete copy-and-run workflow for Roundcube vulnerability research using RAPTOR + Semgrep + OpenCode, from scanning to finding results and validating vulnerabilities.

You can reuse it for future scans. No Markdown report files are required.

## Step 1 — Set up paths

Copy this into your Kali terminal:

```
export RC=/home/hawkeye/roundcube
export RAPTOR=/home/hawkeye/security-research/tools/raptor
export OUT=/home/hawkeye/security-research/roundcube/scanner-output

mkdir -p "$OUT"
cd "$RAPTOR"
export RAPTOR_DIR="$RAPTOR"
```

## Step 2 — Run Semgrep (PHP vulnerability scanning)

```
semgrep scan \
  --config p/php \
  --metrics=off \
  --json \
  --output "$OUT/php-findings.json" \
  "$RC"
```

This scans Roundcube using PHP rules and saves the results.

View the total findings:

```
jq '.results | length' "$OUT/php-findings.json"
```

View findings grouped by vulnerability rule:

```
jq -r '.results[].check_id' "$OUT/php-findings.json" \
  | sort | uniq -c | sort -nr
```

View vulnerable file paths and line numbers:

```
jq -r '
.results[] |
[.check_id, .path, (.start.line | tostring), .extra.message] |
@tsv
' "$OUT/php-findings.json"
```

## Step 3 — Run RAPTOR static analysis

```
cd "$RAPTOR"

uv run python raptor.py scan \
  --repo "$RC" \
  --no-codeql \
  --sequential \
  --out "$OUT/raptor-static"
```

RAPTOR runs its own Semgrep analysis and writes the scan results to the specified output directory.

Find the generated results:

```
find "$OUT/raptor-static" -maxdepth 3 -type f \
  \( -name '*.sarif' -o -name '*.json' \)
```

Important: RAPTOR currently misclassifies your PHP Roundcube project as `python.web-app`. Therefore, use the separate PHP-specific Semgrep scan in Step 2 as your main static-analysis source.

## Step 4 — Run RAPTOR dependency vulnerability analysis

```
cd "$RAPTOR"

uv run python raptor.py sca \
  --repo "$RC" \
  --no-llm
```

RAPTOR prints the SCA output directory when it finishes.

Find your latest SCA results:

```
export SCA_DIR=$(find "$RAPTOR/out" -maxdepth 1 \
  -type d -name 'sca_roundcube_*' | sort | tail -1)

echo "$SCA_DIR"

ls -lh "$SCA_DIR"
```

View the vulnerability section:

```
sed -n '/## Vulnerable dependencies/,/## Supply-chain findings/p' \
  "$SCA_DIR/report.md"
```

View all RAPTOR SCA findings by severity:

```
jq -r '
.[] |
[.severity // "unknown", .title // .id // "unknown"] |
@tsv
' "$SCA_DIR/findings.json"
```

The SCA output includes dependency vulnerabilities, supply-chain concerns, hygiene findings and license issues.

## Step 5 — Inspect suspicious source code

For example, to inspect Roundcube's mail retrieval code:

```
nl -ba "$RC/program/actions/mail/get.php" | sed -n '35,75p'
```

For contact export:

```
nl -ba "$RC/program/actions/contacts/export.php" | sed -n '115,150p'
```

For URL redirects:

```
nl -ba "$RC/program/include/rcmail_attachment_handler.php" \
  | sed -n '335,365p'
```

The scanner identifies suspicious patterns; source-code review determines whether they are actually vulnerable.

## Step 6 — Validate findings using OpenCode

Launch OpenCode from the Roundcube directory:

```
cd "$RC"
opencode
```

Copy this prompt into OpenCode:

You are a senior PHP security researcher performing an authorized vulnerability assessment of my local Roundcube Webmail installation.

Target source: /home/hawkeye/roundcube

Semgrep results:\
/home/hawkeye/security-research/roundcube/scanner-output/php-findings.json

RAPTOR static results:\
/home/hawkeye/security-research/roundcube/scanner-output/raptor-static

RAPTOR SCA results:\
Locate the newest sca_roundcube\_\* directory under /home/hawkeye/security-research/tools/raptor/out.

TASKS:

1. Read all Semgrep, RAPTOR static-analysis and SCA findings.
2. Group duplicate findings and prioritize XSS, SQLi, LFI, RCE, path traversal, SSRF and authorization bypass.
3. Identify each source of attacker-controlled input.
4. Trace the input through PHP functions to the sink.
5. Examine validation, escaping, sanitization, access controls and response headers.
6. Check whether each code path is reachable in the actual Roundcube installation.
7. Validate credible vulnerabilities against my local test environment using non-destructive tests.
8. Provide the exact curl commands, request bodies, expected responses and observations needed to reproduce each credible finding.
9. Distinguish confirmed vulnerabilities, unverified candidates and false positives.
10. Show file paths, line numbers, evidence, impact and remediation.

Do not invent vulnerabilities. Do not modify application code or run destructive tests without approval.

IMPORTANT: Display your analysis and validation commands directly in the terminal. Do not create Markdown files or reports.

## Step 7 — Manually verify the results

For each candidate, use this validation sequence:

1. Find the flagged file and line number in Semgrep or RAPTOR.
2. Trace the user input to the dangerous operation.
3. Confirm the relevant functionality is enabled and accessible.
4. Reproduce the behavior with a harmless test input against your local lab.
5. Compare the observed response against the expected secure behavior.

For example, inspect a suspected XSS response's headers:

```
curl -k -i 'http://127.0.0.1/roundcube/'
```

Check `Content-Type`, `Content-Security-Policy`, and whether untrusted content is encoded. A reflected value alone does not establish XSS.

### Complete workflow

```
Roundcube PHP source
        |
        v
Semgrep PHP scan
        |
        v
RAPTOR static scan
        |
        v
RAPTOR SCA dependency scan
        |
        v
Read JSON / SARIF findings
        |
        v
OpenCode source-code analysis
        |
        v
Validate with curl / Burp Suite
        |
        v
Confirmed vulnerability or false positive
```

Use Step 1 → Step 6 whenever you start a new Roundcube research session. Once the scans are finished, OpenCode can help you investigate the findings without requiring RAPTOR to use a separate paid LLM API.
