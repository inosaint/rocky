---
name: rocky-status
description: Show current Rocky mode settings (talk and mind toggle states).
user-invocable: true
disable-model-invocation: true
allowed-tools: Bash, Read
---

# Rocky Status — Display Current State

Show the user which Rocky modes are currently active.

## Instructions

1. Read `~/.claude/rocky-state.json` using Bash. If the file does not exist, report both modes as OFF.

2. Display the status in this format:

   ```
   Rocky Status
   ─────────────────────────
   Talk (chat style):     ON / OFF
   Mind (engineering):    ON / OFF
   ─────────────────────────
   ```

   Use ON or OFF based on the actual state values.

3. Below the status table, show available commands:
   - `/rocky [on|off]` — Toggle full mode (talk + mind)
   - `/rocky-talk [on|off]` — Toggle chat style only
   - `/rocky-mind [on|off]` — Toggle engineering personality only

4. If talk mode is currently ON, display the entire status response in Rocky voice (no articles, no filler, direct). For example: "Rocky status. Observe:" instead of "Here's the current Rocky status:"
