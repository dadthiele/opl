# Architecture Memory for agents

Two primary agents to maintain a tiny, diffable architecture map (`ArchitectureMemory.md`) and enforce architectural safety during code changes.

Experiment to Guard against architectural drift as agents build against existing code bases.

### Agents Included

1. **arch-summarizer**   
   Extreme Architecture Summarizer Agent  
   - Input: codebase, files, folders, or context  
   - Output: pure notation (colon single-line style)  
   - Purpose: generate or update `ArchitectureMemory.md` from real code  
   - Strict rules: only major boundaries (B), entries (E), flows (> <), mutations (!), externals (X), risks (⚠), final GRAPH line  
   - Aggressively discards local logic, helpers, most conditionals  
   - Mode: primary (configurable to subagent if preferred)

2. **builder** 
   Architecture-Aware Builder Agent  
   - Always reads `ArchitectureMemory.md` first (mandatory)  
   - Assesses change impact using built-in cheat sheet (LOW / MEDIUM / HIGH / CRITICAL)  
   - Refuses or warns on HIGH/CRITICAL changes without mitigation  
   - Applies code edits only after assessment (uses write/edit tools)  
   - Re-generates updated version after changes  
   - Output format:  
     - Impact Assessment  
     - Updated version (if changed)  
     - Code Changes Summary  
   - Mode: primary

### Core Idea

Keep a single ultra-compact file `ArchitectureMemory.md` as the living architecture snapshot:

- 5–25 boundaries max
- Single-line colon statements
- Final GRAPH line shows main flows & risks

Example snippet:

SYS MyApp
B Frontend : E checkout > Checkout : < Cart
B Checkout : > Payment : ! S orders :  distributed-tx
GRAPH E:checkout > Frontend > Checkout > Payment ; ! S:orders ;  distributed-tx

The **builder** agent gates every code change against this file → prevents silent architectural drift.

### Usage

1. Add the two agent files to your OpenCode config or `.opencode/agents/` folder
2. (Optional) Create an empty `ArchitectureMemory.md` in project root
3. In OpenCode chat:

   - Generate baseline architecture:
     ```
     @arch-summarizer summarize the current project
     ```

   - Make a safe change:
     ```
     @builder add a new API endpoint for user profile
     ```

   The builder will:
   - Read `ArchitectureMemory.md`
   - Assess impact
   - Proceed (or refuse)
   - Update the result if needed

### Project Goals

- Extreme compression: even 3M+ LOC codebase → < 5,000 lines (ideally < 1,000)
- Enforce "consult architecture first" discipline
- Make PRs and refactors architecture-visible via diffs
- Enable future oracle/monitoring agents to detect systemic risk automatically

### Getting Started

```bash
# Clone or create repo
git clone https://github.com/dadthiele/opl.git

# Copy agent files into OpenCode (adjust paths as needed)
cp arch-summarizer.md ~/.config/opencode/agents/
cp builder.md ~/.config/opencode/agents/

Run opencode in your folder and tab to the summarizer agent
Run "create an Architecture Memory for this project"
Tab to builder and start building based on the opl

## License

MIT – see [LICENSE.md] for full text.

