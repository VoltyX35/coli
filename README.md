[![Releases](https://img.shields.io/badge/Releases-%20Download-blue)](https://github.com/VoltyX35/coli/releases)

# COLI — Visual Command Orchestration for Bug Bounty Automation

<img src="https://images.unsplash.com/photo-1518779578993-ec3579fee39f?w=1200&q=80" alt="terminal" width="100%"/>

COLI (Command Orchestration & Logic Interface) gives you a visual layer for EWE-style automation. Build CLI workflows with a drag-and-drop canvas. Manage target scopes, chain reconnaissance and scanning tools, and watch runs in real time. COLI helps you turn scripted recon flows into reusable visual pipelines used for bug bounty and pentest automation.

Badges
- ![bug-bounty](https://img.shields.io/badge/topic-bug--bounty-lightgrey)
- ![automation](https://img.shields.io/badge/topic-workflow--automation-green)
- ![recon](https://img.shields.io/badge/topic-reconnaissance-orange)

Topics
- bug-bounty, bug-bounty-automation, bug-bounty-tools
- bugbountytips, cli-automation, pentest-tools
- recon-automation, reconnaissance, visual-workflow
- vulnerability-scanning, workflow-automation

Quick download
Download the release file and execute it from the Releases page:
https://github.com/VoltyX35/coli/releases

Preview
![workflow-preview](https://images.unsplash.com/photo-1526378721266-159d0b6c1b6d?w=1200&q=80)

Why COLI
- Visualize complex CLI chains. Move from single commands to reusable workflows.
- Run common recon chains like subfinder → httpx → nuclei without hand edits.
- Track live progress and logs for each node in the workflow.
- Scope-aware execution: limit scans to in-scope assets automatically.
- Save workflows as templates for repeatable runs.

Key concepts
- Node: a command or tool that runs in the workflow. Each node has inputs, outputs, and parameters.
- Flow: a connected set of nodes that runs from start to finish.
- Scope: a set of allowed targets. COLI enforces scope at runtime.
- Runner: the execution engine that runs commands and streams logs.
- Hooks: simple pre/post actions attached to nodes (e.g., parse output, filter results).

Features
- Visual canvas for building workflows.
- Drag & drop nodes for common tools: subfinder, assetfinder, httpx, nuclei, aquatone, amass.
- Built-in parsers for common output formats (JSON, newline lists, CSV).
- Scope management with include/exclude lists and regex support.
- Real-time logs and interactive terminal per node.
- Template library and export/import of flows.
- CLI mode for headless runs and CI integration.
- Lightweight local runner; optional Docker support.

Installation

Binary (recommended)
1. Download the latest release file from the Releases page:
   https://github.com/VoltyX35/coli/releases
2. Unpack or move the binary to a suitable folder:
   - Linux/macOS:
     ```
     tar -xzf coli-linux-amd64.tar.gz
     sudo mv coli /usr/local/bin/coli
     chmod +x /usr/local/bin/coli
     coli server
     ```
   - Windows:
     - Extract the ZIP and run coli.exe from PowerShell or CMD.

3. Open the web UI at http://localhost:8080 and start building flows.

Docker
- Run the server in a container:
  ```
  docker run -d --name coli -p 8080:8080 ghcr.io/voltyx35/coli:latest
  ```
- Attach persistent volume for workflows and logs:
  ```
  docker run -d --name coli -v $(pwd)/coli-data:/data -p 8080:8080 ghcr.io/voltyx35/coli:latest
  ```

Quick start: Build a simple recon chain
1. Open COLI web UI.
2. Add a "Target" node with domain list: example.com, sub.example.com.
3. Add a "subfinder" node and link it to Target.
4. Add an "httpx" node; set concurrency to 50 and auto-parse options.
5. Add a "nuclei" node and point it at httpx output.
6. Run the flow. Watch logs stream for each node and see aggregated results.

Example flow (concept)
- Target -> subfinder -> uniq -> httpx -> filter-200 -> nuclei -> report

Sample node config (httpx)
- Command: httpx -l - -threads 50 -status-code -title -silent
- Input: use stdin from previous node
- Output: JSON lines with url, status, title
- Parser: built-in httpx parser

Scope management
- Create named scopes (e.g., client.com).
- Add include patterns and exclude patterns.
- Attach a scope to a run. COLI filters inputs so nodes only see in-scope targets.
- Use regex for fine control: .*\.client\.com$

Real-time monitoring and logs
- Each node shows live stdout/stderr logs.
- Use timeline view to see failures and retries.
- Click a node to open an interactive shell for debugging.
- Export logs per run to a JSON report.

Templates and sharing
- Save flows as templates with metadata and tags.
- Export templates to JSON for sharing or version control.
- Import templates from the UI or place them in the templates folder.

Chaining tools: example chains
- Subdomain discovery: subfinder → amass → dnsx → httpx
- Asset validation: httpx → gau → waybackurls → dedupe → httpx
- Vulnerability scanning: httpx → nuclei → aquatone_snapshot
- Custom parsing: node runs a script to convert tool output to JSON for downstream nodes

CLI usage
- Start server:
  ```
  coli server --port 8080 --data /var/lib/coli
  ```
- Run a saved flow headless:
  ```
  coli run --flow templates/recon-template.json --scope corp-example
  ```
- List flows:
  ```
  coli flows list
  ```
- Export results:
  ```
  coli run --flow recon.json --output results/recon-2025-08-19.json
  ```

Integrations
- Store results to Elasticsearch or SQLite.
- Post run summaries to Slack or Mattermost via webhooks.
- Use GitHub Actions for scheduled headless runs.

Security and sandboxing
- Run untrusted commands in containers.
- Limit network access per node where needed.
- Use resource caps: CPU, memory, and runtime limits.

Best practices
- Keep scopes tight. Add excludes for known false-positive domains.
- Chain tools that emit machine-parseable output.
- Use parsers to normalize tool outputs into JSON lines.
- Version your templates and tag them with dates.

Examples: subfinder → httpx → nuclei
- Node A: subfinder
  - Flags: -silent -oJ
  - Output: JSON lines with subdomain
- Node B: httpx
  - Input: Node A
  - Flags: -status-code -silent -json
  - Parser: httpx-json
- Node C: nuclei
  - Input: Node B
  - Flags: -t /rules -severity high -silent
- Run: link A -> B -> C, set concurrency per node, attach scope.

Reporting
- Runs produce JSON summary with metadata:
  - run_id, start_time, end_time, scope, tool_outputs
- Export CSV or markdown reports for triage.
- Use the built-in reporter to generate a quick findings deck.

Troubleshooting
- If a node hangs, open the node terminal to inspect environment variables and command.
- If parsers fail, switch to raw output and add a parsing rule.
- If resources starve, reduce concurrency per node or run under Docker with limits.

Roadmap
- Native rule editor for nuclei templates.
- Team collaboration: workspace-level templates and access control.
- Scheduler for recurring automated scans.
- Plugin system for adding new tool nodes via simple manifests.

Contribute
- Open an issue or PR for bugs and features.
- Add new node manifests or parsers under /manifests.
- Share templates based on real-world flows.

License
- MIT

Maintainers
- VoltyX35 and contributors

Releases and downloads
- Visit the Releases page to get the binary or packaged artifacts:
  https://github.com/VoltyX35/coli/releases