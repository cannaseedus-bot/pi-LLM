# PowerShell Agentic Orchestration Profile

**Identity:** `PS-AO-1.0.0`

This profile defines how to implement a portable, folder-drop-able, terminal-first orchestration layer for agentic coding workflows using PowerShell.

## 1. Scope

PowerShell is treated as an orchestration runtime (not the model engine). It coordinates:

- Local/remote LLM engines via CLI or HTTP APIs
- File and repository operations
- Terminal UI workflows
- Lightweight service hosting (HTTP/file endpoints)
- Network diagnostics and DNS tooling

## 2. Design Goals

1. **Portable:** single repo/folder execution with minimal setup
2. **Composable:** pluggable engines and command adapters
3. **Deterministic orchestration:** explicit process contracts and logs
4. **Cross-platform:** PowerShell 7+ compatibility
5. **Operator-friendly:** TUI-first experience

## 3. Runtime Capabilities

### 3.1 Terminal UI

Use built-in primitives for interactive UX:

- `PSReadLine` for editing/history/predictions
- `Write-Progress` for long-running activities
- `$Host.UI.RawUI` for dimensions/keypresses/colors
- `$Host.UI.PromptForChoice()` for menu selection
- ANSI escape sequences for rich formatting

For advanced interfaces, integrate .NET TUI frameworks when required.

### 3.2 Process and Engine Orchestration

Core execution primitives:

- `Start-Process` for managed subprocess lifecycle
- `Invoke-RestMethod` for model APIs and control planes
- Structured wrappers per engine (Ollama/CLI providers)
- Unified prompt/result contract across backends

### 3.3 Service Hosting

PowerShell can host lightweight services through .NET classes:

- HTTP endpoints (e.g., `System.Net.HttpListener`)
- Local file-serving endpoints for artifacts
- Optional local state via SQLite/.NET providers

### 3.4 Network and DNS Operations

Native diagnostics and infrastructure checks:

- `Resolve-DnsName`
- `Test-NetConnection`
- platform/module-specific DNS administration cmdlets

## 4. Component Architecture

```text
[ TUI Shell ]
     |
     v
[ Orchestrator Core ] -- [ Config + Profiles ]
     |         |            |
     |         |            +-- engine registry
     |         |
     |         +-- [ Job Manager ]
     |
     +-- [ Engine Adapters ] --> CLI/API providers
     +-- [ File/Repo Services ]
     +-- [ Network Services ]
     +-- [ Optional Local HTTP Server ]
```

## 5. Minimal Command Contracts

### 5.1 Prompt Request

```json
{
  "engine": "ollama|provider-cli|custom",
  "model": "string",
  "prompt": "string",
  "stream": false,
  "cwd": ".",
  "timeout_s": 120
}
```

### 5.2 Prompt Response

```json
{
  "ok": true,
  "engine": "string",
  "model": "string",
  "text": "string",
  "tokens_in": 0,
  "tokens_out": 0,
  "latency_ms": 0,
  "error": null
}
```

### 5.3 Job Event

```json
{
  "job_id": "uuid",
  "phase": "queued|running|completed|failed",
  "timestamp": "iso-8601",
  "metadata": {}
}
```

## 6. Operational Patterns

1. **Drop-in mode:** run from any folder, infer local project context.
2. **Batch mode:** stream files through pipelines for transformations.
3. **Interactive mode:** menu-driven agent loop with explicit action confirmation.
4. **Daemon mode (optional):** local HTTP endpoint for editor/CI integration.

## 7. Safety and Reliability

- Use explicit timeouts for all external calls.
- Capture stdout/stderr separately per process.
- Persist structured logs (JSONL) for replay and debugging.
- Validate paths before file operations.
- Require explicit user confirmation for destructive actions.

## 8. Example Main Loop (Abbreviated)

```powershell
function Show-MainMenu {
  Write-Host "1) Ask local model"
  Write-Host "2) Run file batch"
  Write-Host "3) Start local server"
  Write-Host "4) Network diagnostics"
  Write-Host "5) Exit"
}

do {
  Show-MainMenu
  $choice = Read-Host "Select"
  switch ($choice) {
    '1' { Invoke-AgentPrompt }
    '2' { Invoke-BatchTransform }
    '3' { Start-AgentHttpServer }
    '4' { Resolve-DnsName github.com; Test-NetConnection github.com -Port 443 }
  }
} while ($choice -ne '5')
```

## 9. Fit With π-LLM System Direction

This orchestration profile complements existing π-LLM specs by adding an operator-facing control surface for:

- launching/evaluating benchmark runs,
- invoking distributed workflow steps,
- managing artifacts and logs,
- and coordinating local/remote inference tooling.

It does not replace DCP/QS/TB protocols; it operationalizes them in a terminal-native toolchain.
