<p align="center">
  <img src="static/hound.png" alt="Hound Banner" width="75%">
</p>
<h1 align="center">Hound</h1>

<p align="center"><strong>Autonomous agents for code security auditing</strong></p>

<p align="center">
  <a href="https://github.com/muellerberndt/hound/actions"><img src="https://github.com/muellerberndt/hound/workflows/Tests/badge.svg" alt="Tests"></a>
  <a href="LICENSE.txt"><img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg" alt="License: Apache 2.0"></a>
  <a href="https://www.python.org/downloads/"><img src="https://img.shields.io/badge/python-3.8%2B-blue" alt="Python 3.8+"/></a>
  <a href="https://openai.com"><img src="https://img.shields.io/badge/OpenAI-Compatible-74aa9c" alt="OpenAI"/></a>
  <a href="https://ai.google.dev/"><img src="https://img.shields.io/badge/Gemini-Compatible-4285F4" alt="Gemini"/></a>
  <a href="https://anthropic.com"><img src="https://img.shields.io/badge/Anthropic-Compatible-6B46C1" alt="Anthropic"/></a>
</p>

<p align="center">
  <sub>
    <a href="#overview"><b>Overview</b></a>
    • <a href="#configuration"><b>Configuration</b></a>
    • <a href="#complete-audit-workflow"><b>Workflow</b></a>
    • <a href="#chatbot-telemetry-ui"><b>Chatbot</b></a>
    • <a href="#contributing"><b>Contributing</b></a>
  </sub>
</p>

---

## Overview

Hound is a security audit automation pipeline for AI-assisted code review that mirrors how expert auditors think, learn, and collaborate. 

### Key Features

- Graph-driven analysis: adaptive architecture/access control/value-flow graphs with code-grounded annotations
- Senior/Junior loop: Strategist plans investigations; Scout executes targeted code reads
- Precise evidence: findings reference exact files, functions, and code spans
- Sessionized audits: resumable runs with coverage metrics and token usage
- Provider‑agnostic models: OpenAI, Anthropic, Google, XAI, plus mock for offline

### How It Works

Hound’s analysis loop is organized around graphs, beliefs, and focused investigations:

1. **Relational Knowledge Graphs (adaptive, language‑agnostic)**
   - A model‑driven graph builder constructs and refines interconnected graphs of the system (architecture, access control, value flows, state). It adapts to the target scope and programming language without relying on brittle parsers or CFG generators.
   - The graph evolves as the audit progresses: nodes accrue observations and assumptions; relationships are added or revised as new code is inspected.
   - The agent reasons at a high level, then “zooms in” on a subgraph to pull only the precise code slices needed at that moment. Rather than pure embedding search, the graph provides structured context for targeted retrieval and reasoning.

2. **Belief System and Hypotheses**
   - Instead of one‑shot judgments, Hound maintains beliefs (hypotheses) that evolve as evidence accumulates. Confidence adjusts up or down when new observations support or contradict an assumption.
   - This lets the agent keep weak but plausible leads “alive” without overcommitting, then promote or prune them as the audit uncovers more code and context.
   - The result is steadier calibration over longer runs: fewer premature rejections, better recall of subtle issues, and a transparent trail from initial hunch to conclusion.

3. **Precise Code Grounding**
   - Every graph element and annotation links to exact files, functions, and call sites. Investigations retrieve only the relevant code spans, maintaining attention on concrete implementation details rather than broad semantic similarity.

4. **Adaptive Planning**
   - Planning reacts to discoveries: finding one issue seeds targeted searches for related classes of bugs (e.g., privilege checks, reentrancy surfaces, value transfer patterns).
   - Coverage tracking ensures systematic exploration while allowing strategic pivots toward the most promising areas.

The system employs a **senior/junior auditor pattern**: the Scout (junior) actively navigates the codebase and annotates the knowledge graphs as it explores, while the Strategist (senior) handles high‑level planning and vulnerability analysis, directing and refocusing the Scout as needed. This mirrors real audit teams where seniors guide and juniors investigate.

**Codebase size considerations:** While Hound is language-agnostic and can analyze any codebase, it's optimized for small-to-medium sized projects like typical smart contract applications. Large enterprise codebases may exceed context limits and require selective analysis of specific subsystems.

### Links

- [Original blog post](https://muellerberndt.medium.com/unleashing-the-hound-how-ai-agents-find-deep-logic-bugs-in-any-codebase-64c2110e3a6f)
- [Smart contract audit benchmarks datasets and tooling](https://github.com/muellerberndt/scabench)

## Installation

```bash
pip install -r requirements.txt
```

## Configuration

Set up your API keys, e.g.:

```bash
export OPENAI_API_KEY=your_key_here
```

Copy the example configuration and edit as needed:

```bash
cp hound/config.yaml.example hound/config.yaml
# then edit hound/config.yaml to select providers/models and options
```

Notes:
- Defaults work out-of-the-box; you can override many options via CLI flags.
- Keep API keys out of the repo; `API_KEYS.txt` is gitignored and can be sourced.

<!-- Quick Start and Repository Layout removed to avoid duplication; see Complete Audit Workflow below. -->

**Note:** Audit quality scales with time and model capability. Use longer runs and advanced models for more complete results.

## Complete Audit Workflow

### Step 1: Create a Project

Projects organize your audits and store all analysis data:

```bash
# Create a project from local code
./hound.py project create myaudit /path/to/code

# List all projects
./hound.py project list

# View project details and coverage
./hound.py project info myaudit
```

### Step 2: Build Knowledge Graphs

Hound analyzes your codebase and builds aspect-oriented knowledge graphs that serve as the foundation for all subsequent analysis:

```bash
# Build graphs (uses scout model by default)
./hound.py graph build myaudit

# Customize graph types and depth
./hound.py graph build myaudit --graphs 5 --iterations 3

# View generated graphs
./hound.py graph list myaudit
```

**What happens:** Hound inspects the codebase and creates specialized graphs for different aspects (e.g., access control, value flows, state management). Each graph contains:
- **Nodes**: Key concepts, functions, and state variables
- **Edges**: Relationships between components
- **Annotations**: Observations and assumptions tied to specific code locations
- **Code cards**: Extracted code snippets linked to graph elements

These graphs enable Hound to reason about high-level patterns while maintaining precise code grounding.

### Step 3: Run the Audit

The audit phase uses the **senior/junior pattern** with planning and investigation:

```bash
# Run a full audit with strategic planning (new session)
./hound.py agent audit myaudit

# Set time limit (in minutes)
./hound.py agent audit myaudit --time-limit 30

# Start with telemetry (connect the Chatbot UI to steer)
./hound.py agent audit myaudit --telemetry --time-limit 30

# Enable debug logging (captures all prompts/responses)
./hound.py agent audit myaudit --debug

# Attach to an existing session and continue where you left off
./hound.py agent audit myaudit --session <session_id>
```

Tip: When started with `--telemetry`, you can connect the Chatbot UI and steer the audit interactively (see Chatbot section above).

**Key parameters:**
- **--time-limit**: Stop after N minutes (useful for incremental audits)
- **--plan-n**: Number of investigations per planning batch
- **--session**: Resume a specific session (continues coverage/planning)
- **--debug**: Save all LLM interactions to `.hound_debug/`

**Audit duration and depth:**
Hound is designed to deliver increasingly complete results with longer audits. The analyze step can range from:
- **Quick scan**: 1 hour with fast models (gpt-4o-mini) for initial findings
- **Standard audit**: 4-8 hours with balanced models for comprehensive coverage
- **Deep audit**: Multiple days with advanced models (GPT-5) for exhaustive analysis

The quality and duration depend heavily on the models used. Faster models provide quick results but may miss subtle issues, while advanced reasoning models find deeper vulnerabilities but require more time.

**What happens during an audit:**

The audit is a **dynamic, iterative process** with continuous interaction between Strategist and Scout:

1. **Initial Planning** (Strategist)
   - Reviews all knowledge graphs and annotations
   - Identifies contradictions between assumptions and observations
   - Creates a batch of prioritized investigations (default: 5)
   - Focus areas: access control violations, value transfer risks, state inconsistencies

2. **Investigation Loop** (Scout + Strategist collaboration)
   
   For each investigation in the batch:
   - **Scout explores**: Loads relevant graph nodes, analyzes code
   - **Scout escalates**: When deep analysis needed, calls Strategist via `deep_think`
   - **Strategist analyzes**: Reviews Scout's collected context, forms vulnerability hypotheses
   - **Hypotheses form**: Findings are added to global store
   - **Coverage updates**: Tracks visited nodes and analyzed code

3. **Adaptive Replanning**
   
   After completing a batch:
   - Strategist reviews new findings and updated coverage
   - Reorganizes priorities based on discoveries
   - If vulnerability found, searches for related issues
   - Plans next batch of investigations
   - Continues until coverage goals met or no promising leads remain

4. **Session Management**
   - Unique session ID tracks the entire audit lifecycle
   - Coverage metrics show exploration progress
   - All findings accumulate in hypothesis store
   - Token usage tracked per model and investigation

**Example output:**
```
Planning Next Investigations...
1. [P10] Investigate role management bypass vulnerabilities
2. [P9] Check for reentrancy in value transfer functions
3. [P8] Analyze emergency function privilege escalation

Coverage Statistics:
  Nodes visited: 23/45 (51.1%)
  Cards analyzed: 12/30 (40.0%)

Hypotheses Status:
  Total: 15
  High confidence: 8
  Confirmed: 3
```

### Step 4: Monitor Progress

Check audit progress and findings at any time during the audit. If you started the agent with `--telemetry`, you can also monitor and steer via the Chatbot UI:

- Open http://127.0.0.1:5280 and attach to the running instance
- Watch live Activity, Plan, and Findings
- Use the Steer form to guide the next investigations

```bash
# View current hypotheses (findings)
./hound.py hypotheses list myaudit

# See detailed hypothesis information
./hound.py hypotheses list myaudit --verbose

# Filter by confidence level
./hound.py hypotheses list myaudit --min-confidence 0.8

# Check coverage statistics
./hound.py project coverage myaudit

# View session details
./hound.py project info myaudit
```

**Understanding hypotheses:** Each hypothesis represents a potential vulnerability with:
- **Confidence score**: 0.0-1.0 indicating likelihood of being a real issue
- **Status**: `proposed` (initial), `investigating`, `confirmed`, `rejected`
- **Severity**: critical, high, medium, low
- **Type**: reentrancy, access control, logic error, etc.
- **Annotations**: Exact code locations and evidence

### Step 5: Run Targeted Investigations (Optional)

For specific concerns, run focused investigations without full planning:

```bash
# Investigate a specific concern
./hound.py agent investigate "Check for reentrancy in withdraw function" myaudit

# Quick investigation with fewer iterations
./hound.py agent investigate "Analyze access control in admin functions" myaudit \
  --iterations 5

# Use specific models for investigation
./hound.py agent investigate "Review emergency functions" myaudit \
  --model gpt-4o \
  --strategist-model gpt-5
```

**When to use targeted investigations:**
- Following up on specific concerns after initial audit
- Testing a hypothesis about a particular vulnerability
- Quick checks before full audit
- Investigating areas not covered by automatic planning

**Note:** These investigations still update the hypothesis store and coverage tracking.

### Step 6: Quality Assurance

A reasoning model reviews all hypotheses and updates their status based on evidence:

```bash
# Run finalization with quality review
./hound.py finalize myaudit

# Customize confidence threshold
./hound.py finalize myaudit \
  --confidence-threshold 0.7 \
  --model gpt-4o

# Include all findings (not just confirmed)
./hound.py finalize myaudit --include-all
```

**What happens during finalization:**
1. A reasoning model (default: GPT-5) reviews each hypothesis
2. Evaluates the evidence and code context
3. Updates status to `confirmed` or `rejected` based on analysis
4. Adjusts confidence scores based on evidence strength
5. Prepares findings for report generation

**Important:** By default, only `confirmed` findings appear in the final report. Use `--include-all` to include all hypotheses regardless of status.

### Step 7: Generate Proof-of-Concepts

Create and manage proof-of-concept exploits for confirmed vulnerabilities:

```bash
# Generate PoC prompts for confirmed vulnerabilities
./hound.py poc make-prompt myaudit

# Generate for a specific hypothesis
./hound.py poc make-prompt myaudit --hypothesis hyp_12345

# Import existing PoC files
./hound.py poc import myaudit hyp_12345 exploit.sol test.js \
  --description "Demonstrates reentrancy exploit"

# List all imported PoCs
./hound.py poc list myaudit
```

**The PoC workflow:**
1. **make-prompt**: Generates detailed prompts for coding agents (like Claude Code)
   - Includes vulnerable file paths (project-relative)
   - Specifies exact functions to target
   - Provides clear exploit requirements
   - Saves prompts to `poc_prompts/` directory

2. **import**: Links PoC files to specific vulnerabilities
   - Files stored in `poc/[hypothesis-id]/`
   - Metadata tracks descriptions and timestamps
   - Multiple files per vulnerability supported

3. **Automatic inclusion**: Imported PoCs appear in reports with syntax highlighting

### Step 8: Generate Professional Reports

Produce comprehensive audit reports with all findings and PoCs:

```bash
# Generate HTML report (includes imported PoCs)
./hound.py report myaudit

# Include all hypotheses, not just confirmed
./hound.py report myaudit --include-all

# View the generated report
./hound.py report view myaudit

# Export report to specific location
./hound.py report myaudit --output /path/to/report.html
```

**Report contents:**
- **Executive summary**: High-level overview and risk assessment
- **System architecture**: Understanding of the codebase structure
- **Findings**: Detailed vulnerability descriptions (only `confirmed` by default)
- **Code snippets**: Relevant vulnerable code with line numbers
- **Proof-of-concepts**: Any imported PoCs with syntax highlighting
- **Severity distribution**: Visual breakdown of finding severities
- **Recommendations**: Suggested fixes and improvements

**Note:** The report uses a professional dark theme and includes all imported PoCs automatically.

<!-- Removed duplicate "Complete Example Workflow" in favor of the detailed Complete Audit Workflow. -->

## Session Management

Each audit run operates under a session with comprehensive tracking and per-session planning:

- Planning is stored in a per-session PlanStore with statuses: `planned`, `in_progress`, `done`, `dropped`, `superseded`.
- Existing `planned` items are executed first; Strategist only tops up new items to reach your `--plan-n`.
- On resume, any stale `in_progress` items are reset to `planned`; completed items remain `done` and are not duplicated.
- Completed investigations, coverage, and hypotheses are fed back into planning to avoid repeats and guide prioritization.

```bash
# View session details
./hound.py project info myaudit

# List and inspect sessions
./hound.py project sessions myaudit --list
./hound.py project sessions myaudit <session_id>

# Show planned investigations for a session (Strategist PlanStore)
./hound.py project plan myaudit <session_id>

# Session data includes:
# - Coverage statistics (nodes/cards visited)
# - Investigation history
# - Token usage by model
# - Planning decisions
# - Hypothesis formation
```

Sessions are stored in `~/.hound/projects/myaudit/sessions/` and contain:
- `session_id`: Unique identifier
- `coverage`: Visited nodes and analyzed code
- `investigations`: All executed investigations
- `planning_history`: Strategic decisions made
- `token_usage`: Detailed API usage metrics

Resume/attach to an existing session during an audit run by passing the session ID:

```bash
# Attach to a specific session and continue auditing under it
./hound.py agent audit myaudit --session <session_id>
```

When you attach to a session, its status is set to `active` while the audit runs and finalized on completion (`completed` or `interrupted` if a time limit was hit). Any `in_progress` plan items are reset to `planned` so you can continue cleanly.

### Simple Planning Examples

```bash
# Start an audit (creates a session automatically)
./hound.py agent audit myaudit

# List sessions to get the session id
./hound.py project sessions myaudit --list

# Show planned investigations for that session
./hound.py project plan myaudit <session_id>

# Attach later and continue planning/execution under the same session
./hound.py agent audit myaudit --session <session_id>
```

## Chatbot (Telemetry UI)

Hound ships with a lightweight web UI for steering and monitoring a running audit session. It discovers local runs via a simple telemetry registry and streams status/decisions live.

Prerequisites:
- Set API keys (at least `OPENAI_API_KEY`): `source ../API_KEYS.txt` or export manually
- Install Python deps in this submodule: `pip install -r requirements.txt`

1) Start the agent with telemetry enabled

```bash
# From the hound/ directory
./hound.py agent audit myaudit --telemetry --debug

# Notes
# - The --telemetry flag exposes a local SSE/control endpoint and registers the run
# - Optional: ensure the registry dir matches the chatbot by setting:
#   export HOUND_REGISTRY_DIR="$HOME/.local/state/hound/instances"
```

2) Launch the chatbot server

```bash
# From the hound/ directory
python chatbot/run.py

# Optional: customize host/port
HOST=0.0.0.0 PORT=5280 python chatbot/run.py
```

Open the UI: http://127.0.0.1:5280

3) Select the running instance and stream activity

- The input next to “Start” lists detected instances as `project_path | instance_id`.
- Click “Start” to attach; the UI auto‑connects the realtime channel and begins streaming decisions/results.
- The lower panel has tabs:
  - Activity: live status/decisions
  - Plan: current strategist plan (✓ done, ▶ active, • pending)
  - Findings: hypotheses with confidence; you can Confirm/Reject manually

4) Steer the audit

- Use the “Steer” form (e.g., “Investigate reentrancy across the whole app next”).
- Steering is queued at `<project>/.hound/steering.jsonl` and consumed exactly once when applied.
- Broad, global instructions may preempt the current investigation and trigger immediate replanning.

Troubleshooting
- No instances in dropdown: ensure you started the agent with `--telemetry`.
- Wrong or stale project shown: clear the input; the UI defaults to the most recent alive instance.
- Registry mismatch: confirm both processes print the same `Using registry dir:` or set `HOUND_REGISTRY_DIR` for both.
- Raw API: open `/api/instances` in the browser to inspect entries (includes `alive` flag and registry path).

## Managing Hypotheses

Hypotheses are the core findings that accumulate across sessions:

```bash
# List all hypotheses with confidence scores
./hound.py hypotheses list myaudit

# View with full details
./hound.py hypotheses list myaudit --verbose

# Filter by status or confidence
./hound.py hypotheses list myaudit --status confirmed
./hound.py hypotheses list myaudit --min-confidence 0.8

# Update hypothesis status
./hound.py hypotheses update myaudit hyp_12345 --status confirmed

# Reset hypotheses (creates backup)
./hound.py hypotheses reset myaudit

# Force reset without confirmation
./hound.py hypotheses reset myaudit --force
```

Hypothesis statuses:
- **proposed**: Initial finding, needs review
- **investigating**: Under active investigation
- **confirmed**: Verified vulnerability
- **rejected**: False positive
- **resolved**: Fixed in code

## Advanced Features

### Model Selection

Override default models per component:

```bash
# Use different models for each role
./hound.py agent audit myaudit \
  --platform openai --model gpt-4o-mini \           # Scout
  --strategist-platform anthropic --strategist-model claude-3-opus \  # Strategist
  --finalizer-platform openai --finalizer-model gpt-4o  # Finalizer
```

### Debug Mode

Capture all LLM interactions for analysis:

```bash
# Enable debug logging
./hound.py agent audit myaudit --debug

# Debug logs saved to .hound_debug/
# Includes HTML reports with all prompts and responses
```

### Coverage Tracking

Monitor audit progress and completeness:

```bash
# View coverage statistics
./hound.py project coverage myaudit

# Coverage shows:
# - Graph nodes visited vs total
# - Code cards analyzed vs total
# - Percentage completion
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for development setup and guidelines.

## License

Apache 2.0 with additional terms:

You may use Hound however you want, except selling it as an online service or as an appliance - that requires written permission from the author.

- See [LICENSE](LICENSE) for details.
