# Springfield Continuation Suite

> **For Claude:** This is a design document, not an implementation plan. See "Next Steps" for what to build.

**Goal:** Four Stop hook plugins that replace double-shot-latte with smarter continuation decisions. Each uses a different strategy, individually enable/disable via settings.json.

**Architecture:** Simpsons-themed plugins in a single `cameronsjo/continuation-hooks` repo, published via `cameronsjo/superpowers-marketplace`. Each is a standalone Stop hook — enable one, disable the others and double-shot-latte.

**Compatibility:** All four are guaranteed compatible with ralph-loop. Ralph blocks when active (state file present), continuation plugin decides when ralph is inactive. No conflict.

---

## The Four Plugins

### 1. ned-flanders — Deterministic Rules

**Status: BUILD FIRST**

**Strategy:** Grep the transcript for known patterns. No LLM judge, no state file, no skill modifications.

**How it works:**
1. Stop hook reads last ~50 lines of transcript JSONL
2. Extracts last assistant text + last tool calls via jq
3. Applies decision tree:

```
IF last tool call was AskUserQuestion → STOP
IF transcript contains active TodoWrite items with "pending" → CONTINUE
IF last text matches /Task \d+|Step \d+|Phase \d+|Moving to|Next/ → CONTINUE
IF last text matches /complete|finished|done|ready for|let me know/ → STOP
IF announce pattern present ("I'm using the .* skill") → CONTINUE
DEFAULT → STOP
```

4. Same throttle as double-shot-latte — max 3 continuations per 5 minutes

**Pros:**
- Zero API cost (pure bash + jq)
- Zero skill modifications needed
- Works with existing superpowers skills today
- Simple to understand and debug

**Cons:**
- Pattern matching can produce false positives ("Moving to a new topic" conversationally)
- No contextual understanding — just string matching
- Adding new patterns requires updating the hook

**Cost:** $0 per decision.

---

### 2. lisa-simpson — Smarter Transcript Parsing + LLM Judge

**Status: BUILD SECOND**

**Strategy:** Keep the LLM judge but feed it structured, skill-aware data instead of raw JSONL.

**How it works:**
1. Stop hook extracts structured signals from transcript:
   - Active skill (last `Skill` tool invocation)
   - Todo state (last `TodoWrite`/`TaskUpdate` calls, pending count)
   - Last 3 tool calls (name + abbreviated result)
   - Last assistant text message (trimmed to ~500 chars)
2. Formats as structured context
3. Sends to haiku judge with skill-aware evaluation prompt

**What the judge sees (instead of raw JSONL):**
```
CONTEXT:
  Active skill: executing-plans
  Todo state: 2/5 complete, 3 pending
  Last tools: [Bash(npm test → exit 0), TaskUpdate(task2 → completed)]
  Last message: "Task 2 complete. Moving to Task 3."

DECIDE: Should Claude continue working autonomously?
```

**Enhanced judge prompt** is skill-aware — knows that announce patterns mean active workflows, AskUserQuestion means waiting for user, pending todos mean work remains.

**Model configurable** via `LISA_SIMPSON_MODEL` env var (default: haiku).

**Pros:**
- Much higher accuracy than double-shot-latte's raw JSONL approach
- Handles edge cases and ambiguous situations
- Skill-aware decisions
- Graceful degradation (falls back to approve-stop on errors)

**Cons:**
- Still costs haiku per Stop event (~$0.001/decision)
- More complex transcript parsing
- Latency from LLM call

**Cost:** ~$0.001 per decision (haiku).

---

### 3. marge-simpson — State File Convention

**Status: DEFERRED — requires skill modifications that feel bolted-on**

**Strategy:** Skills write structured state to `.claude/workflow-state.json`. Hook reads it.

**State file format:**
```json
{
  "skill": "executing-plans",
  "phase": "batch_execution",
  "tasks_total": 5,
  "tasks_completed": 2,
  "waiting_for_user": false,
  "updated_at": "2026-02-06T21:30:00Z"
}
```

**Why deferred:** Skills writing a JSON state file is infrastructure plumbing that doesn't serve the user — it's purely hook communication. The announce pattern works because it's naturally dual-purpose (tells user AND hook what's happening). A state file is single-purpose. It would feel out of place in skill content.

**Revisit when:** We find a natural dual-purpose reason for skills to write state files (e.g., long-running workflow progress reporting to the user).

---

### 4. bart-simpson — Workflow Markers

**Status: DEFERRED — requires skill modifications that feel bolted-on**

**Strategy:** Skills emit HTML comment markers (`<!-- el-barto:continue -->`). Hook greps for them.

**Why deferred:** HTML comments in conversation output exist solely for a hook to grep. Users see them and wonder "what is that?" The announce pattern works because it's natural language serving dual purposes. Markers are infrastructure leaking into content.

**Convergence note:** A version of Bart where "markers" are enriched announce patterns ("I'm using executing-plans. Task 3/5.") converges with Ned (grepping natural patterns). If we want richer signals than Ned's patterns, Lisa's structured extraction handles it better without requiring skill changes.

---

## Comparison

| | ned-flanders | lisa-simpson | marge-simpson | bart-simpson |
|--|-------------|-------------|---------------|-------------|
| **API cost** | $0 | ~$0.001/stop | $0 | $0 |
| **Skill mods** | None | None | Heavy | Light |
| **Accuracy** | Medium | Highest | High | High |
| **Complexity** | Low | High | Medium | Low |
| **Works today** | Yes | Yes | No | No |
| **Status** | **Build first** | **Build second** | Deferred | Deferred |

## Repo Structure

Single repo: `cameronsjo/continuation-hooks`

```
continuation-hooks/
├── ned-flanders/
│   ├── .claude-plugin/plugin.json
│   ├── hooks/
│   │   ├── hooks.json
│   │   └── stop-hook.sh
│   ├── CLAUDE.md
│   └── README.md
├── lisa-simpson/
│   ├── .claude-plugin/plugin.json
│   ├── hooks/
│   │   ├── hooks.json
│   │   └── stop-hook.sh
│   ├── scripts/
│   │   └── extract-signals.sh
│   ├── CLAUDE.md
│   └── README.md
├── marge-simpson/           # Deferred
│   └── (placeholder)
├── bart-simpson/            # Deferred
│   └── (placeholder)
├── CLAUDE.md
└── README.md
```

Published via `cameronsjo/superpowers-marketplace` with subdirectory references.

## Double-Shot-Latte Integration

When using any Springfield plugin, disable double-shot-latte:

```json
// settings.json
{
  "enabledPlugins": {
    "double-shot-latte@superpowers-marketplace": false,
    "ned-flanders@superpowers-marketplace": true
  }
}
```

## Skill Interaction Reference

These existing superpowers patterns are what Ned and Lisa read:

| Skill Signal | Ned Interpretation | Lisa Interpretation |
|-------------|-------------------|-------------------|
| "I'm using the X skill" | Pattern match → CONTINUE | Structured: active_skill=X → CONTINUE |
| AskUserQuestion tool call | Tool name match → STOP | Structured: waiting_for_user=true → STOP |
| TodoWrite with pending items | Pattern match → CONTINUE | Structured: tasks_remaining=N → CONTINUE |
| "Ready for feedback" | Pattern match → STOP | Context: completion + user wait → STOP |
| "Task N complete. Moving to Task N+1" | Pattern match → CONTINUE | Context: progress + next action → CONTINUE |

## Next Steps

1. Create `cameronsjo/continuation-hooks` repo
2. Build ned-flanders (deterministic rules, zero cost, works today)
3. Test with existing superpowers skills
4. Build lisa-simpson (smarter parsing + LLM judge)
5. Add both to superpowers-marketplace
6. Update MEMORY.md with Springfield suite context
