# Directional Prompting

<p align="center">
  <img src="plugins/directional-prompting/skills/directional-prompting/assets/hero.jpeg" alt="Vibe Bot walking a glowing path that Bootoshi is pointing toward, with the words 'Directional Prompting — name the path. the trees disappear.'" width="900">
</p>

A two-layer skill for writing prompts, agent directives, skill descriptions, slash commands, and anywhere else an LLM reads instructions. Works the same in **Claude Code** and **OpenAI Codex CLI**.

Same `SKILL.md`, same trigger surface, same outcome. Drop it into `~/.claude/skills/directional-prompting/` and `~/.codex/skills/directional-prompting/` and both agents pick it up natively.

## The two layers

**Layer 1 — Outcome.** Every non-trivial prompt opens with a block that names the destination: the goal, what "done" looks like, when to stop, the true invariants. This is the frame.

**Layer 2 — Direction.** Inside that frame, every sentence names the path forward with positive verbs. "Trace", "build", "use", "read", "return", "ask", "check". The correct behavior is described so clearly and completely that the wrong behavior has no room to exist.

Outcome without direction reads as wishful — the model knows where to go but not how to step. Direction without outcome wanders — the model walks crisp paths to nowhere. Both layers together: a model that knows the destination and walks toward it on every token.

## Why both labs converge here

Modern frontier models follow instructions literally. The Claude 4.7 guide: *"Positive examples showing how Claude can communicate with the appropriate level of concision tend to be more effective than negative examples or instructions that tell the model what not to do."* The GPT-5.5 guide: *"GPT-5.5 is strongest when the prompt defines the target outcome, success criteria, constraints, and available context, then lets the model choose the path."*

Both labs converge on the same shape. Name the destination. Name the path. Skip the prohibitions.

## What this skill does

When the agent sees prompt-authoring language (`write a prompt`, `improve this prompt`, `audit this system prompt`, `outcome-first`, `make this prompt positive`, or any new skill, agent, or directive being authored), it:

1. Opens the draft with an explicit outcome block: `Goal:`, `Success means:`, `Stop when:` (plus `Constraints:` when needed).
2. Rewrites the body so every sentence leads with a positive verb naming the correct action.
3. Audits the draft for `don't`, `do not`, `never`, `avoid`, `refrain`, `instead of`, `rather than`, `not allowed`, `prohibited`, `forbidden`, `won't`, `shouldn't` — and rewrites each as the positive replacement.
4. Demotes decorative `ALWAYS` / `NEVER` / `MUST` down to plain prose so the absolute-rule signal stays loud where it matters.
5. Keeps negation only in four narrow cases: hard safety boundaries, disambiguating near-identical paths, acceptable spaces too large to enumerate, and specific banned items where the positive form is genuinely ambiguous.

## The before-and-after that proves it

**Before** — no outcome block, mostly negatives, seven `don't`s:

```text
You are a code reviewer. Don't be too harsh. Don't nitpick formatting.
Avoid making assumptions about the author's intent. Never approve code
with obvious bugs. Don't suggest changes that aren't actionable. Try
not to be vague. Avoid emojis.
```

**After** — outcome on top, directional inside:

```text
Goal: Review the PR diff and decide whether to approve, request changes, or block.

Success means:
  - Verdict is one of: APPROVE, REQUEST_CHANGES, BLOCK
  - Each comment names the file, line, and replacement code
  - Comments cover correctness, security, clarity (skip formatting — the linter handles that)

Stop when: A verdict is issued and every comment is actionable.

Focus on bugs you can reproduce, security boundaries, and unclear logic.
Ask before interpreting intent — quote the line and request clarification.
Block merges on reproducible bugs. Write in plain text.
```

Same constraints, half the length. The model knows the destination (verdict plus actionable comments), how to stop (verdict issued), and every sentence in the body pulls forward.

## Install

### Claude Code

Add the marketplace, then install the plugin:

```text
/plugin marketplace add kingbootoshi/directional-prompting
/plugin install directional-prompting
```

Restart Claude Code if the skill does not appear immediately. After install, the skill triggers automatically on prompt-authoring language, or you can invoke it explicitly.

### OpenAI Codex CLI

Codex looks for skills at `~/.codex/skills/<name>/`. Drop the skill directory there:

```bash
git clone https://github.com/kingbootoshi/directional-prompting.git ~/.directional-prompting
mkdir -p ~/.codex/skills
ln -sfn ~/.directional-prompting/plugins/directional-prompting/skills/directional-prompting ~/.codex/skills/directional-prompting
```

Restart Codex if needed. Invoke explicitly by asking Codex to "use directional prompting on this" or it will auto-select on prompt-authoring language.

### One canonical location for both agents

Keep the skill in one place and symlink it into both agents:

```bash
# clone once
git clone https://github.com/kingbootoshi/directional-prompting.git ~/.directional-prompting

# Claude Code
mkdir -p ~/.claude/skills
ln -sfn ~/.directional-prompting/plugins/directional-prompting/skills/directional-prompting ~/.claude/skills/directional-prompting

# Codex CLI
mkdir -p ~/.codex/skills
ln -sfn ~/.directional-prompting/plugins/directional-prompting/skills/directional-prompting ~/.codex/skills/directional-prompting
```

Both agents follow symlinks for skill discovery, so updates to the cloned repo flow to both automatically.

## When to fire it

Run this skill whenever you write or audit any of the following:

- System prompts for agents
- `AGENTS.md`, `CLAUDE.md`, project instructions
- Skill descriptions and `SKILL.md` bodies
- Tool descriptions in JSONSchema
- Slash command bodies
- Cursor rules, Continue rules, any IDE-agent ruleset
- Sub-agent prompts in orchestration code
- Eval rubric instructions

For each draft, run the four-check pass inside the skill:

1. **Outcome check.** Does the prompt open with goal plus success criteria plus stopping condition? If not, add the block.
2. **Direction check.** Count negations in the body. Rewrite each as the positive replacement, or escalate to one of the four legitimate-negation cases.
3. **Absolute-rule check.** Is every `ALWAYS` / `NEVER` / `MUST` a true invariant? Demote the decorative ones to plain prose.
4. **Read-back.** Read the final prompt aloud. Every sentence should name a destination or a step toward it. Cut anything that does neither.

## Why this matters more for agents

A coding agent reads its system prompt on every turn. A negation that plants the wrong concept gets re-planted dozens of times per session. A vague outcome lets the agent's notion of "done" drift turn-by-turn.

Outcome plus direction together re-load the correct frame on every turn — the agent's attention is structurally aimed at the destination, and every instruction in the body points toward it.

## Layout

```text
directional-prompting/
  .claude-plugin/
    marketplace.json
  plugins/
    directional-prompting/
      skills/
        directional-prompting/
          SKILL.md              # the skill itself
          agents/
            openai.yaml         # Codex-specific UI metadata
          assets/
            hero.jpeg
```

## Compatibility

Built against the open [Agent Skills](https://developers.openai.com/codex/skills) standard. Works in any agent that scans a skills directory for `SKILL.md`:

| Agent | Personal path | Project path |
| --- | --- | --- |
| Claude Code | `~/.claude/skills/directional-prompting/` | `.claude/skills/directional-prompting/` |
| OpenAI Codex CLI | `~/.codex/skills/directional-prompting/` | `.agents/skills/directional-prompting/` or `.codex/skills/directional-prompting/` |
| Cursor | n/a | `.cursor/skills/directional-prompting/` |
| Google Antigravity | `~/.gemini/antigravity/skills/directional-prompting/` | `.agent/skills/directional-prompting/` |

The `agents/openai.yaml` sidecar is Codex-specific UI metadata. Other agents ignore it.

## License

MIT.

## Contributing
PRs welcome!
