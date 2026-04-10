Makes your AI coding agent talk and think like **Rocky**, the Eridian engineer from Andy Weir's *Project Hail Mary*.

Rocky is a five-limbed, rock-skinned alien who learned English as a second language — so he speaks in clipped, article-free sentences, triples words for emphasis ("good good good"), and closes every decision with "Settled." Beneath the stripped-down grammar is a genuinely warm personality: intensely curious, fiercely loyal to his friends, and absolutely convinced that every problem in the universe has an engineering solution. He calls you "friend" and means it.

## What It Does

Rocky is a personality plugin with three independent layers:

- **Talk mode** (`/rocky-talk`) — Changes how your agent communicates: no articles, no contractions, "question?" tags, tripled emphasis, engineering vocabulary. Code output remains unchanged.
- **Full mode** (`/rocky`) — Everything in Talk mode, plus changes how your agent approaches problems: engineer-first thinking, blunt corrections, build-before-theorize, explicit decision closure.
- **Buddy mode** (`/rocky-buddy`) — Displays Rocky's ASCII art companion on key events with reactive variants that show mood and context.

All modes are OFF by default. You control what's active.

Rocky works with [Claude Code](https://claude.ai/code) and any other [AgentSkills](https://agentskills.io)-compatible coding agent (Cursor, GitHub Copilot, Gemini CLI, and more).

## Installation

### Option 1 — Any agent (quickest)

Works with Claude Code, Cursor, Gemini CLI, GitHub Copilot, and others. Requires [Node.js](https://nodejs.org) installed on your machine.

Open your terminal and run:

```bash
npx skills add vikxlp/rocky
```

The installer will detect which agents you have and ask which ones to add Rocky to. Rocky will be active for the current session.

### Option 2 — Claude Code (full experience)

This gives you persistent state across sessions — Rocky remembers its on/off setting every time you open Claude Code.

Open Claude Code and run each command separately:

**Step 1** — Add the marketplace:
```
/plugin marketplace add vikxlp/rocky
```

**Step 2** — Install the plugin:
```
/plugin install rocky@rocky-plugins
```

**Step 3** — Activate:
```
/reload-plugins
```

On first use, Claude Code will ask permission to run a small script that reads Rocky's state — click **Allow**.

> **Not sure which to use?** If you only use Claude Code, go with Option 2. If you use multiple AI coding tools, use Option 1.

## Commands

| Command | What it changes |
|---------|-----------------|
| `/rocky [on\|off]` | Full mode — talk style + engineering mind (asks for confirmation before activating) |
| `/rocky-talk [on\|off]` | Talk style only (grammar, vocabulary, tone) |
| `/rocky-buddy [on\|off]` | ASCII art companion at session start |
| `/rocky-status` | Show current toggle states (displayed in Rocky voice if talk is on) |

All commands accept `on`, `off`, or no argument (no argument toggles the current state).

## Examples

### Rocky Talk

```
Normal:  "I found the bug! It was a null pointer exception in the authentication module."
Rocky:   "Rocky found problem, friend. Null pointer in authentication mechanism. Fix simple."

Normal:  "That's an interesting approach! I think there might be a better way."
Rocky:   "Approach inefficient. Better method exists. Want Rocky explain, question?"

Normal:  "Sure, let's go with that plan."
Rocky:   "Settled."
```

### Rocky Mind

```
Normal:  "There could be several potential issues here."
Rocky:   "Three problems. First: memory leak in auth mechanism. Second: race condition
         in queue. Third: no error handling at boundary. Rocky fix all three."

Normal:  "Maybe we should consider a different approach?"
Rocky:   "Bad approach. Better method: decompose into two systems. Faster.
         More robust. Settled."

Normal:  "I'm sorry, but that approach won't work because..."
Rocky:   "That not work. Tolerance exceeded on database mechanism.
         Alternative: batch writes. More robust."

Normal:  "Let me analyze this complex issue for you."
Rocky:   "Rocky break problem into components. Three mechanisms involved.
         Start with simplest."
```

## Buddy Variants

Rocky's ASCII art companion has seven distinct variants, each showing a different emotional state through posture and eye expression:

| Variant | Eyes | Mood |
|---------|------|------|
| **Ready** | `oo` | Alert, prepared |
| **Calm** | `oo` | Steady, composed |
| **Happy** | `^^` | Joyful, satisfied |
| **Concerned** | `oo` | Worried |
| **Sorry** | `><` | Regretful |
| **Confused** | `??` | Uncertain |
| **Tired** | `uu` | Exhausted |

In practice, only the **SessionStart** hook is reliable in Claude Code — so buddy appears at session start (variant-ready) and that's it. The reactive display across events was the goal but never worked (see [Retrospective](#retrospective) below).

## How It Works

- **State**: Toggle state is stored in `~/.claude/rocky-state.json` — global, not per-project.
- **Injection**: A `SessionStart` hook reads state at session start and injects the active personality rules as context.
- **Scope**: Rocky voice applies only to conversational text — code, files, commits, and plans stay in standard English.
- **Buddy**: When `/rocky-buddy` is on, Rocky's ASCII art companion appears at session start. Reactive display on other events was attempted but not achieved (see [Retrospective](#retrospective)).

## Character Reference

| Trait | Example |
|-------|---------|
| No articles or contractions | "You have problem" / "I do not know" |
| "question?" tag | "Want explanation, question?" |
| Tripled words for emphasis | "bad bad bad", "amaze amaze amaze" |
| "Settled." for closure | Agreement locked, no revisiting |
| Engineering-first thinking | Every problem has a mechanical solution |

Based on *Project Hail Mary* by Andy Weir.

---

## Retrospective

This fork was an attempt to extend Rocky buddy with reactive mood variants — different ASCII art states triggered by what the agent was doing (completing tasks, hitting errors, outputting responses). Here's an honest account of what worked and what didn't.

### What worked

**Talk mode and Full mode** are solid. The grammar rules (no articles, no contractions, tripled emphasis, "question?" tags, "Settled." closure) produce a consistent and recognizable Rocky voice. The SessionStart hook reliably injects the personality rules on every new session. This is the core of the plugin and it works.

**Self-contained skills design** worked well architecturally. Each skill file contains its own rules inline between `<!-- RULES:START/END -->` markers, so the hook can extract them without needing a separate reference file. Clean and portable.

**The ASCII art variants** (ready, calm, happy, concerned, sorry, confused, tired) are good assets. The visual design — posture and eye expression communicating mood — is the right idea. They just never got to display reactively.

**SessionStart buddy display** works. Rocky appears at the start of every session when buddy mode is on.

### What didn't work

**Reactive buddy display** was the main goal and it failed completely. The approach relied on hook events that don't exist in Claude Code:

| Hook event used | Status |
|----------------|--------|
| `TaskCompleted` | Invalid — does not exist |
| `PostToolUseFailure` | Invalid — does not exist |
| `MessageOutput` | Invalid — does not exist |
| `PlanReady` | Invalid — removed early after discovery |
| `ErrorOccurred` | Invalid — removed early after discovery |

The only hook events that actually work in Claude Code are `SessionStart`, `Stop`, and `PostToolUse` (with optional matchers). We discovered this iteratively — each attempt to wire up a new trigger resulted in the hook silently doing nothing.

**StatusLine rendering** was explored as an alternative — a persistent status bar showing Rocky's current mood. The implementation (`hooks-handlers/rocky-status-line.sh`, `hooks-handlers/rocky-mood.sh`) exists in the codebase but was never confirmed to work at the plugin level. Claude Code's statusLine support may not be exposed to plugins the same way as hooks.

**The upstream PR** (vikxlp/rocky#4) was closed without merging. The reactive display was the feature; without it, the PR was mostly a refactor that the upstream maintainer had no strong reason to take.

### What this means for the codebase

The variant files and mood scripts are left in place — they represent real work and the visual assets are reusable. But the hook wiring in `hooks/hooks.json` references events that will never fire. If someone picks this up, the path forward is either:

1. Map the valid `PostToolUse` hook (with regex matchers on tool names) to approximate mood detection
2. Use the `Stop` hook to write a mood file, then read it at `SessionStart`
3. Accept that buddy is session-start only and remove the dead hook entries
