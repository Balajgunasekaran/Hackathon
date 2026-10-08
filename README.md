# TraceFix AI

## AI Production Investigation and Recovery Copilot

TraceFix AI is a Windows desktop proof of concept that helps production-support teams investigate batch and application incidents. A user selects the application and environment, enters incident details, and pastes or uploads a sanitized production log. TraceFix AI compares the log with a domain-specific JSON knowledge base and produces an explainable investigation report.

> Current implementation: local deterministic analysis using Python, Tkinter, and JSON. No external AI/LLM service is connected, and no data is sent outside the machine.

---

## 1. Problem Statement

Production incident investigation often requires support engineers to:

- Read large log files
- Identify the failed processing stage
- Separate meaningful errors from technical noise
- Search for known incident patterns
- Estimate incident priority and impact
- Determine probable root cause
- Decide whether a restart or rerun is safe
- Identify the correct technical support team

TraceFix AI structures these activities into a single investigation workflow.

---

## 2. Current User Flow

```text
Launch TraceFix AI
        |
        v
Select application and environment
        |
        v
Enter incident details
        |
        v
Paste or upload a sanitized production log
        |
        v
Review the sanitization warning
        |
        v
Click Analyze Incident
        |
        v
Match the log against knowledge-base rules
        |
        v
Display classification, evidence, actions, and recovery guidance
```

---

## 3. Current Features

- Windows desktop interface
- Application selection
- Environment selection: DEV, UAT, or PROD
- Incident-description input
- Production-log paste area
- Log-file upload
- Sanitization warning
- Case-insensitive knowledge-base pattern matching
- Known failure-scenario classification
- Operational priority mapping
- Severity and confidence display
- Failed-stage identification
- Matched log evidence
- Probable root-cause explanation
- Expected incident-impact display
- Investigation steps
- Potential resolution
- Restart or rerun guidance
- Recommended expert teams
- Safe handling of unknown incidents
- Overview, Evidence, and Actions & Experts result tabs

---

## 4. Example Result

For the following log:

```text
Database connection failed while retrieving trades.
DbException occurred during execution.
database connection attempt timed out.
```

TraceFix AI displays:

```text
Classification: X-One trade retrieval database failure
Priority: P1
Severity: Critical
Confidence: 97%
Failed Stage: Trade Retrieval
```

The result tabs provide:

- **Overview:** application, environment, failed stage, probable root cause, resolution, and restart guidance
- **Evidence:** matched log patterns and expected incident impact
- **Actions & Experts:** investigation steps and recommended support teams

---

## 5. Technology Stack

- **Python:** application logic and log analysis
- **Tkinter:** Windows desktop user interface
- **JSON:** domain-specific knowledge base
- **Local rule engine:** deterministic and explainable pattern matching

No additional UI framework is required because Tkinter is included with standard Python installations.

---

## 6. Architecture

```text
Desktop UI
   |
   v
Incident details + production log
   |
   v
Analyzer service
   |
   v
knowledge_base.json
   |
   v
Best matching rule
   |
   v
Classification + evidence + investigation + recovery report
```

The current architecture separates the user interface, analysis logic, and knowledge base so that each component can be improved independently.

---

## 7. Project Structure

```text
Hackathon/
|
|-- data/
|   `-- knowledge_base.json
|
|-- services/
|   |-- knowledge_loader.py
|   |-- analyzer.py
|   `-- report_builder.py        # Optional/prepared for future use
|
|-- ui/
|   `-- main_window.py
|
|-- main.py
|-- test_analyzer.py
|-- README.md
`-- .gitignore
```

---

## 8. What Each File Does

### `main.py`

Application entry point. It imports the desktop UI and starts TraceFix AI.

Typical content:

```python
print("Starting TraceFix AI...")
from ui.main_window import *
print("TraceFix AI window closed.")
```

The final message is printed only after the desktop window is closed.

### `ui/main_window.py`

Contains the complete Tkinter desktop interface and coordinates interaction between the user and the analyzer.

Responsibilities:

- Creates the TraceFix AI window
- Displays the application and environment dropdowns
- Accepts incident details
- Accepts pasted logs
- Uploads text-readable files
- Displays the sanitization warning
- Calls `analyze_log()` when Analyze Incident is selected
- Derives priority from severity
- Selects recommended experts using the failed stage
- Displays summary cards
- Populates Overview, Evidence, and Actions & Experts tabs
- Shows progress and status messages
- Handles missing input and analysis errors

### `services/knowledge_loader.py`

Loads and parses `data/knowledge_base.json`.

Responsibilities:

- Opens the JSON file using UTF-8
- Converts JSON content into Python dictionaries and lists
- Returns the knowledge base to the analyzer

Keeping this logic separate makes it easier to change where knowledge comes from later, such as a database or enterprise search service.

### `services/analyzer.py`

Contains the current deterministic log-analysis engine.

Responsibilities:

- Loads the knowledge base
- Normalizes the production log to lowercase
- Checks every rule pattern against the log
- Counts matching patterns for every rule
- Selects the rule with the highest number of matches
- Returns the matched rule, matched patterns, and match score
- Returns no rule when no known pattern is found

Important: the current analyzer performs substring matching. It is not currently an LLM or semantic-search model.

### `services/report_builder.py`

Optional report-model layer prepared for future use.

Intended responsibilities:

- Convert analyzer output into a consistent report object
- Separate presentation fields from rule-engine internals
- Add Jira, Gerrit, Git, source-code, or AI evidence later
- Support future export to JSON, HTML, PDF, or an API

If this file is not currently imported, it has no effect on the running application.

### `data/knowledge_base.json`

Contains domain-specific investigation knowledge.

The current structure includes:

- Project and batch metadata
- Normal informational or technical-noise patterns
- Known incident rules
- Rule ID and scenario
- Failed stage
- Severity
- Exact log patterns
- Strong indicators
- Success prerequisites where relevant
- Probable root cause
- Expected impact
- Suggested actions
- Recovery action
- Restart guidance
- Expert-defined confidence

Current scenarios include:

- Configured filter missing
- FTB SFTP server unavailable
- X-One trade retrieval database failure
- Invalid report-email configuration
- Null reference during CSV line creation

### `test_analyzer.py`

Console-based test harness for the analyzer.

Use it to:

- Validate knowledge-base changes without opening the UI
- Test new patterns
- Confirm classification, priority, root cause, actions, and experts
- Diagnose analysis logic independently from Tkinter

Run it with:

```bash
python test_analyzer.py
```

### `.gitignore`

Prevents local or sensitive files from being committed.

Recommended content:

```gitignore
__pycache__/
*.pyc
*.pyo
*.log
.env
venv/
.venv/
.vscode/
.idea/
.DS_Store
Thumbs.db
```

Do not commit API keys, passwords, tokens, real production logs, or confidential Jira/Gerrit exports.

---

## 9. Knowledge-Base Rule Format

A typical rule looks like:

```json
{
  "id": "CLF-003",
  "scenario": "X-One trade retrieval database failure",
  "stage": "Trade Retrieval",
  "severity": "Critical",
  "patterns": [
    "Database connection failed while retrieving trades",
    "DbException",
    "database connection attempt timed out"
  ],
  "probableRootCause": "The batch could not retrieve X-One trades because the database connection was unavailable or timed out.",
  "impact": [
    "Eligible trades were not retrieved",
    "CSV generation was not started"
  ],
  "suggestedActions": [
    "Verify X-One trade database availability.",
    "Check connectivity between the Trade Service and database."
  ],
  "recoveryAction": "Restore database connectivity and rerun the batch.",
  "restartGuidance": "Likely safe to rerun because no output or delivery stage was reached.",
  "confidence": 97
}
```

When adding a rule:

1. Use a unique ID.
2. Use specific patterns that are unlikely to cause false matches.
3. Add multiple independent indicators where possible.
4. Clearly describe downstream impact.
5. Separate investigation actions from recovery actions.
6. State whether a complete rerun could create duplicates.
7. Validate the rule using synthetic logs before merging it.

---

## 10. How the Analyzer Works

The current matching algorithm is:

1. Read the submitted production log.
2. Convert the complete log to lowercase.
3. Loop through every rule.
4. Check whether each rule pattern occurs in the log.
5. Count matching patterns.
6. Select the rule with the highest count.
7. Display the rule's configured confidence and evidence.
8. Return `No known pattern` if all rule scores are zero.

### Current limitations

- Matching is based on exact phrases or substrings.
- Similar wording may not match.
- Confidence is stored in JSON, not calculated dynamically.
- Only the best matching rule is returned.
- Incident details are displayed but not yet used for matching.
- Large or binary files are not specially parsed.
- The application does not currently call an external AI model.
- Automatic secret and personal-data masking is not implemented.

---

## 11. Supported Input Files

The upload dialog explicitly offers:

- `.log`
- `.txt`

The All Files option may also read text-based files such as:

- `.out`
- `.err`
- `.trace`
- `.csv`
- `.json`
- `.xml`

The current implementation reads files as UTF-8 text and replaces undecodable characters.

Not yet supported directly:

- `.zip`
- `.gz`
- `.pdf`
- `.docx`
- `.xlsx`
- `.evtx`

---

## 12. Security and Data Handling

The current POC runs locally and does not send logs externally.

Before analyzing any log:

- Remove passwords
- Remove API keys and tokens
- Remove customer and personal information
- Remove account numbers
- Remove internal secrets
- Use synthetic data for demonstrations

The displayed sanitization warning is advisory. Automatic sanitization remains a future enhancement.

Do not add personal or unapproved external AI credentials to the project. Any future LLM integration must use an organization-approved endpoint and approved data-handling controls.

---

## 13. Prerequisites

Required:

- Windows 10 or Windows 11
- Python 3.10 or later recommended
- Tkinter available in the Python installation
- Git, if cloning from a repository
- Visual Studio Code recommended, but not required

No pip package installation is required for the current version.

Verify Python:

```bash
python --version
```

Verify Tkinter:

```bash
python -c "import tkinter; print('Tkinter available')"
```

---

## 14. Run the Existing Project

Open a terminal in the project root:

```bash
cd Hackathon
python main.py
```

Expected terminal output:

```text
Starting TraceFix AI...
```

The desktop window remains open until the user closes it.

Run the console analyzer test:

```bash
python test_analyzer.py
```

---

## 15. Clone and Run on Another Laptop

### Option A: Clone from Git

Install Git and Python, then run:

```bash
git clone <REPOSITORY_URL>
cd <REPOSITORY_FOLDER>
python --version
python -c "import tkinter; print('Tkinter available')"
python main.py
```

Replace `<REPOSITORY_URL>` and `<REPOSITORY_FOLDER>` with the actual values.

### Option B: Copy without Git

1. Zip the project folder.
2. Do not include real logs, secrets, `.env`, or credentials.
3. Copy the ZIP to the other laptop using an approved channel.
4. Extract the ZIP.
5. Open a terminal in the extracted folder.
6. Run:

```bash
python --version
python -c "import tkinter; print('Tkinter available')"
python main.py
```

### If `python` is not recognized

Try:

```bash
py --version
py main.py
```

### If Tkinter is unavailable

Use a standard Python distribution that includes Tcl/Tk, or ask the local IT/software-distribution team for the approved Python package.

### Corporate environment note

Do not bypass organizational package repositories or security controls. The current application requires no external Python packages, making it suitable for restricted environments.

---

## 16. Test Scenarios

### Database failure

Incident details:

```text
Cash loan reconciliation batch failed while retrieving trade data.
```

Log:

```text
Database connection failed while retrieving trades.
DbException occurred during execution.
database connection attempt timed out.
```

Expected:

```text
Classification: X-One trade retrieval database failure
Priority: P1
Severity: Critical
Confidence: 97%
```

### SFTP failure

```text
CSV file successfully created.
Report email sent successfully.
Unable to connect to the configured FTB SFTP server.
SocketException: The target server did not respond.
Failed to send CSV file via SFTP to target.
```

Expected classification:

```text
FTB SFTP server unavailable
```

### Invalid email configuration

```text
Trade streaming completed successfully.
CSV file successfully created.
Invalid report-email recipient configured for the batch.
FormatException: Recipient is not in a valid email-address format.
Error sending report email.
```

Expected classification:

```text
Invalid report-email configuration
```

### CSV line creation failure

```text
Trade retrieval completed successfully.
Error creating CSV line for trade CL-78452.
NullReferenceException occurred in CashLoanEliotFoboExtractCsvLine.
Object reference not set to an instance of an object.
```

Expected classification:

```text
Null reference during CSV line creation
```

### Unknown incident

```text
Worker heartbeat was delayed.
Processing threshold was exceeded.
The execution coordinator terminated the operation.
```

Expected:

```text
No known pattern
```

TraceFix AI should not invent a root cause for an unknown incident.

---

## 17. Recommended Improvement Roadmap

### Phase 1: Improve the local analyzer

- Normalize timestamps, thread IDs, and log levels
- Use weighted strong indicators
- Handle synonyms and alternate technical wording
- Calculate confidence from match coverage
- Detect multiple simultaneous failures
- Rank the top matching scenarios
- Use incident details as additional context
- Separate normal, success, warning, and error evidence

### Phase 2: Improve file processing

- Parse JSON logs by field
- Parse CSV and XML content
- Support ZIP and GZIP extraction
- Add file-size limits
- Add encoding detection
- Add automatic secret and PII masking
- Show a sanitization preview and require user approval

### Phase 3: Add approved AI/LLM support

- Use an organization-approved LLM only
- Send sanitized and minimized context
- Ground the LLM with the matched knowledge-base rule
- Generate an incident summary
- Analyze unfamiliar wording
- Explain evidence relevance
- Suggest new knowledge-base rules for human approval
- Label AI output as preliminary guidance
- Provide a local fallback when the AI endpoint is unavailable

### Phase 4: Add engineering evidence

- Search related Jira incidents
- Search Gerrit reviews
- Search Git commit history
- Search relevant source-code files
- Correlate evidence without claiming causation
- Display source, timestamp, and relevance for every evidence item

### Phase 5: Production readiness

- Add authentication and authorization
- Add role-based access
- Add audit logging
- Add structured configuration
- Add automated tests
- Add packaging as a Windows executable
- Add monitoring and error telemetry
- Complete security, legal, and AI-governance reviews

---

## 18. Prompt for an AI Coding Assistant

Use the following prompt when asking an approved AI coding assistant to improve the project. Attach or provide all current project files, but remove secrets and real production data first.

```text
You are a senior Python engineer, desktop UX engineer, production-support specialist, and responsible-AI architect.

Project name: TraceFix AI

Goal:
Improve an existing Windows desktop proof of concept that analyzes sanitized production logs and produces an explainable investigation and recovery report.

Current stack:
- Python
- Tkinter
- JSON knowledge base
- No external dependencies
- Deterministic substring-based rule matching

Current project structure:
- main.py: application entry point
- ui/main_window.py: Tkinter UI and result rendering
- services/knowledge_loader.py: loads data/knowledge_base.json
- services/analyzer.py: matches log patterns and returns the best rule
- services/report_builder.py: optional report-model layer
- data/knowledge_base.json: incident rules and normal patterns
- test_analyzer.py: console test harness

Current workflow:
1. Select application and environment.
2. Enter incident details.
3. Paste or upload a sanitized log.
4. Click Analyze Incident.
5. Display classification, priority, severity, confidence, failed stage, matched evidence, root cause, impact, actions, recovery guidance, restart guidance, and recommended experts.

Constraints:
- Preserve offline operation.
- Use Python standard library wherever practical.
- Do not add external API calls unless explicitly requested.
- Never hard-code secrets.
- Do not send data outside the machine.
- Do not invent a root cause when there is insufficient evidence.
- Clearly distinguish configured knowledge-base facts from inferred recommendations.
- Preserve safe handling of unknown incidents.
- Keep the application usable on restricted corporate Windows devices.

Please perform these tasks:
1. Review the architecture and identify defects, duplication, tight coupling, and maintainability risks.
2. Refactor UI, analysis, report-building, configuration, and file-reading logic into clear modules.
3. Improve matching with normalization, weighted indicators, match coverage, and ranked candidates.
4. Calculate confidence dynamically while retaining the expert confidence as reference metadata.
5. Use incident details as supporting context without overriding strong log evidence.
6. Detect and display multiple plausible incidents when appropriate.
7. Add structured unknown-incident output with low confidence and safe diagnostic guidance.
8. Process .log, .txt, .json, .csv, .xml, .out, .err, and .trace files safely.
9. Add file-size checks, encoding handling, and clear upload errors.
10. Add automatic secret and PII detection with a sanitization preview before analysis.
11. Improve accessibility, resizing, scrolling, keyboard navigation, and visual hierarchy in Tkinter.
12. Add unit tests using unittest for the loader, analyzer, sanitization, priority mapping, and unknown incidents.
13. Add type hints, docstrings, structured exceptions, and logging without exposing submitted log content.
14. Add a configuration file for applications, environments, severity-to-priority mapping, and expert mapping.
15. Preserve compatibility with the existing knowledge_base.json or provide a safe migration script.
16. Provide updated source code file by file, followed by run instructions and tests.

Optional approved-AI design only:
Design an AIProvider interface with OfflineProvider and EnterpriseLLMProvider implementations. Do not make external network calls. The future EnterpriseLLMProvider must accept only sanitized, minimized context and must return structured JSON. Include prompt-injection defenses, timeouts, fallback behavior, audit metadata, and clear labels for AI-generated preliminary guidance.

Engineering-evidence design only:
Define interfaces for future Jira, Gerrit, Git, and source-code connectors. Do not add live credentials or real endpoints. Return evidence objects containing source, identifier, summary, timestamp, relevance score, and relationship explanation. Never claim a commit caused an incident without verified evidence.

Output format:
- Start with architecture findings.
- Show the proposed folder structure.
- Provide an incremental migration plan.
- Provide complete code for each changed or new file.
- Provide test commands and expected results.
- Explain security and responsible-AI controls.
- Avoid changing working behavior without explaining the reason.
```

---

## 19. Current Status

Completed:

- Desktop UI
- Log paste and upload
- Local knowledge-base loader
- Known-pattern classification
- Priority, severity, and confidence display
- Failed-stage and root-cause output
- Evidence and impact display
- Investigation and recovery guidance
- Restart guidance
- Expert recommendations
- Unknown-pattern handling

Not yet implemented:

- Automatic sanitization
- Dynamic confidence calculation
- Semantic or fuzzy matching
- Multiple-incident ranking
- Approved LLM integration
- Jira, Gerrit, Git, and source-code integration
- Automated test suite
- Windows executable packaging

---

## 20. Responsible Use

TraceFix AI is a support-assistance POC. It must not be treated as the sole authority for production recovery decisions.

- Validate recommendations before acting.
- Follow approved incident-management and change procedures.
- Use synthetic or properly sanitized data.
- Do not use personal external API keys on corporate systems.
- Obtain approval before integrating external AI services.
- Keep a human support engineer responsible for final decisions.
