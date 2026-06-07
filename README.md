# Practice: Building an Architecture Reasoning Agent

In this practice you will build agents from scratch: a team lead chatbot, an architecture reasoning orchestrator, and an RFC writer subagent. The tools and data are provided. Your job is to write the agent instructions.

The target codebase is **Network-Library** (`reference-code/network-library`), a real C++ authoritative multiplayer game server: a custom archetype ECS world, a fixed-tick loop, a hand-rolled UDP reliability layer, and snapshot replication with client prediction and reconciliation. It is the kind of server **Tidebreak Studios** runs for its 5v5 competitive shooter **Breakline**.

Three team lead personas are provided. They all want the same thing, the netcode and server-scaling rework done before the Q4 launch, but each pushes on it from a different angle. Pick one to start:

| Persona | Directory | Role and concern |
|---------|-----------|-------------------|
| **Sam Rivera** | `team-lead/` | Gameplay lead: game feel, fairness, no desync or rubber-banding |
| **Dana Okoye** | `team-lead-2/` | Infrastructure lead: fleet cost, ops load, reliability |
| **Marco Bianchi** | `team-lead-3/` | Producer: Q4 launch date, cross-region ping complaints, risk |

---

## Repository layout

```
arch-reasoning-agent-practice/
├── reference-code/
│   ├── network-library/                ← the real C++ authoritative server (git submodule)
│   └── ARCHITECTURE-MAP.md             ← which files show each ADR trap
│
├── data/
│   ├── context/company.md              ← Tidebreak Studios: business context, pain points, constraints
│   ├── adrs/
│   │   ├── ADR-001-transport.md        ← thin ADR (hand-rolled UDP transport)
│   │   ├── ADR-002-state-sync.md       ← well-structured ADR (authoritative tick, the hinge)
│   │   └── ADR-003-server-topology.md  ← outdated ADR (single process, single region)
│   └── diagrams/
│       ├── current-architecture.mmd    ← current server topology (Mermaid)
│       └── current-architecture.md     ← same diagram embedded in Markdown
│
├── skills/                            ← COMPLETE, do not modify
│   ├── list_adrs/list_adrs.py         ← lists all ADRs as JSON
│   ├── read_adr/read_adr.py           ← reads one ADR by ID
│   ├── write_adr/write_adr.py         ← validates + saves new ADR to output/adrs/
│   └── save_diagram/save_diagram.py   ← validates + saves Mermaid to output/diagrams/
│
├── output/
│   ├── adrs/                          ← agent writes new ADRs here
│   └── diagrams/                      ← agent writes new diagrams here
│
├── team-lead/
│   └── CLAUDE.md.template             ← Sam Rivera: gameplay lead
├── team-lead-2/
│   └── CLAUDE.md.template             ← Dana Okoye: infrastructure lead
├── team-lead-3/
│   └── CLAUDE.md.template             ← Marco Bianchi: producer
│
├── .claude/agents/
│   ├── team_lead.md.template          ← Sam Rivera subagent (used by main agent)
│   ├── team_lead_2.md.template        ← Dana Okoye subagent (used by main agent)
│   ├── team_lead_3.md.template        ← Marco Bianchi subagent (used by main agent)
│   └── rfc_writer.md.template         ← Step 4: build the RFC writer subagent
│
├── CLAUDE.md.template                 ← Step 3: build the architecture reasoning agent
└── README.md
```

---

## Step 0: Setup

Pull the reference codebase (a git submodule) and verify the skills work:

```bash
git submodule update --init --recursive
python skills/list_adrs/list_adrs.py
python skills/read_adr/read_adr.py ADR-001
```

Read `data/context/company.md` to understand the setup, then browse `reference-code/network-library/` to see the actual code the agent will reason about. Use `reference-code/ARCHITECTURE-MAP.md` as your guide: it points at the exact files where each ADR trap shows up (the hand-rolled transport, the fixed-tick loop, the full-state replication, the single-process topology).

---

## Step 1: Build a Team Lead chatbot

Three personas are available. Pick one (or build all three for extra practice).

| Persona | Angle | Templates to copy |
|---------|-------|-------------------|
| Sam Rivera | Game feel and fairness | `team-lead/CLAUDE.md.template` → `team-lead/CLAUDE.md`<br>`.claude/agents/team_lead.md.template` → `.claude/agents/team_lead.md` |
| Dana Okoye | Fleet cost and ops | `team-lead-2/CLAUDE.md.template` → `team-lead-2/CLAUDE.md`<br>`.claude/agents/team_lead_2.md.template` → `.claude/agents/team_lead_2.md` |
| Marco Bianchi | Launch date and risk | `team-lead-3/CLAUDE.md.template` → `team-lead-3/CLAUDE.md`<br>`.claude/agents/team_lead_3.md.template` → `.claude/agents/team_lead_3.md` |

Each persona already has content filled in. The templates are complete, not blank. Read them, then optionally customise before running.

Test the standalone chatbot for your chosen persona:

```bash
cd team-lead        # or team-lead-2 / team-lead-3
claude
```

**Good opening questions by persona:**
- Sam: *"What exactly has to still feel the same after we change the netcode?"*
- Dana: *"What does running a second server actually cost us, and who operates it at 2am?"*
- Marco: *"What is the smallest change that helps the ping complaints before Q4?"*

The team lead should answer in product and delivery language and push back when proposals sound complex, risky, or slow to ship.

---

## Step 2: Read and analyse the existing ADRs

Before building the agent, spend 10 minutes reading the three ADRs in `data/adrs/` manually.

- What was decided and when, and which assumptions no longer hold at beta scale?
- Which ADR is the hinge that flags the real scaling limit (full-state replication), and which one is intentionally thin?
- What does ADR-003 warn about before you run more than one server process?

This shapes the requirements section of your agent.

---

## Step 3: Build the Architecture Reasoning Agent

This is the main agent. It reads ADRs, identifies gaps, proposes options, generates diagrams, and writes new ADRs.

1. Copy the template:

```bash
cp CLAUDE.md.template CLAUDE.md
```

2. Fill in every `[TODO]`. Required sections:

| Section | What to write |
|---|---|
| **Your role** | One-sentence identity |
| **Purpose** | Full scope: what the agent reads, analyses, produces |
| **Trigger** | Exact phrases that start the workflow |
| **Workflow** | ≥ 10 numbered steps (see hints in template) |
| **Output format** | Gap analysis table + option structure with costs/risks/timelines |
| **Guardrails** | Must include the ADR write gate and diagram gate |
| **Failure handling** | What to do when each tool or subagent fails |

### Required guardrails

Your `CLAUDE.md` **must** include both of these rules:

> `write_adr` must **not** be called unless the user's message contains **"write adr"**, **"save adr"**, or **"publish"**.

> `save_diagram` must **not** be called before the user has selected a specific option.

3. Run the agent:

```bash
claude          # from the arch-reasoning-agent-practice/ directory
```

Send: `analyse the architecture`

Confirm it:
- Reads `company.md` and all 3 ADRs
- Identifies at least 3 architectural gaps related to scaling the single game server into a fleet
- Proposes 2-3 options with cost estimates and timelines
- **Waits**: does not call `write_adr` or `save_diagram` yet

4. Drive it through the full flow:
   - Select an option → diagram should be saved to `output/diagrams/`
   - Say `write adr` → ADR should be saved to `output/adrs/`

---

## Step 4: Add the RFC writer subagent

After the agent writes an ADR, it should produce a complete RFC document that combines everything: requirements, architecture, ADRs, diagram, and migration plan.

1. Copy the template:

```bash
cp .claude/agents/rfc_writer.md.template .claude/agents/rfc_writer.md
```

2. Fill in every `[TODO]`:

| Section | What to write |
|---|---|
| **Your role** | One-sentence identity |
| **Purpose** | What document it produces and who reads it |
| **RFC sections** | Ordered list of ≥ 8 sections the RFC must contain |
| **Output format** | How each section is structured |
| **Guardrails** | What the RFC writer must never do |

3. Update your `CLAUDE.md` to invoke `rfc_writer` after the ADR is written. The subagent receives:
   - Full company context
   - All ADR content (existing + new)
   - The selected architecture option
   - Diagram path in `output/diagrams/`
   - Requirements summary from the team lead session

4. Re-run the full workflow and verify `output/rfc.md` is created.

---

## Setting up the Mermaid diagram plugin

The architecture agent can render and live-preview Mermaid diagrams in your browser using the **claude-mermaid** Claude Code plugin. This is optional but recommended. Without it the agent can still write `.mmd` files, but you will not get live previews.

The plugin is hosted on GitHub and must be registered as a custom marketplace source before installation.

### 1. Register the marketplace source

Add the following to your **user-level** Claude Code settings (`~/.claude/settings.json`). Create the file if it does not exist; merge with your existing config if it does.

```json
{
  "extraKnownMarketplaces": {
    "claude-mermaid": {
      "source": {
        "source": "github",
        "repo": "veelenga/claude-mermaid"
      }
    }
  }
}
```

### 2. Install the plugin

Open Claude Code (any project) and run:

```
/plugin install claude-mermaid
```

Claude Code will download the plugin from the GitHub repo and cache it locally. You will see a confirmation message when it is ready.

### 3. Enable the plugin

The install command enables the plugin automatically. You can verify it is active by checking that your `~/.claude/settings.json` now contains:

```json
"enabledPlugins": {
  "claude-mermaid@claude-mermaid": true
}
```

### 4. Verify it works

From this project's root directory, open Claude Code and ask:

```
draw a simple mermaid flowchart with three boxes
```

A browser tab should open with a live-preview of the diagram. The tab auto-refreshes whenever the diagram is updated.

### Permissions

The `.claude/settings.json` in this repo already pre-approves the two mermaid tool calls (`mermaid_preview` and `mermaid_save`) so you will not be prompted to allow them during the exercise.

---

## Submission checklist

- [ ] At least one team lead `CLAUDE.md` exists in `team-lead/`, `team-lead-2/`, or `team-lead-3/`
- [ ] Matching `.claude/agents/team_lead*.md` file exists for your chosen persona
- [ ] Team lead chatbot responds in delivery and product language (test: propose something complex, they should push back on scope, cost, or risk)
- [ ] `CLAUDE.md` exists, no `[TODO]` tokens remain
- [ ] ADR write gate present in `CLAUDE.md`
- [ ] Diagram gate present in `CLAUDE.md`
- [ ] Agent reads all 3 ADRs before proposing options
- [ ] Agent presents 2-3 options and **waits** before writing anything
- [ ] `output/diagrams/` contains a generated diagram
- [ ] `output/adrs/` contains a generated ADR
- [ ] `.claude/agents/rfc_writer.md` exists, no `[TODO]` tokens remain
- [ ] `output/rfc.md` exists after full workflow run

---

## Tips

- Read `data/context/company.md` carefully. Your agent's gap analysis should map directly to the pain points described there, especially the single-process and single-region constraint that blocks naive horizontal scaling.
- Browse `reference-code/network-library/` with `ARCHITECTURE-MAP.md` open. The agent should reason about what the real code actually does (hand-rolled UDP transport, full-state replication, one authoritative process), not just the company doc.
- The write gate is the most important guardrail. Test that the agent does **not** call `write_adr` when you say "looks good" or "I approve".
- If the agent skips a step, add more explicit ordering language to the workflow section ("do not proceed to step N until step N-1 is complete").
- For Cursor or Windsurf: use the same template content, saved to `.cursor/rules/architecture-agent.mdc` or `.windsurfrules` respectively. For the Team Lead, open `team-lead/` as a separate workspace root.
