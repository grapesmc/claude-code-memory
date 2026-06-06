---
name: feedback-diagnose-first
description: Always ask what the user thinks is broken before taking action on complex systems like OpenClaw
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6b6dfa46-bc02-4dd3-966f-d6c52de43dcb
---

Before touching any running service, plugin, or config — ask what the user thinks is broken. Don't assume from logs or warnings alone.

**Why:** During an OpenClaw Discord troubleshooting session, a "conflicting plugin metadata" warning was treated as a problem to fix. Reinstalling the Discord plugin wiped the channel config, breaking Discord entirely — the thing the user actually wanted to fix. The real issue was just that the channel config was missing `channel:` prefixes in cron prompts.

**How to apply:** On any "let's clean this up" action that touches live service config — pause and ask "what behavior is broken?" first. A warning is not necessarily a problem. Don't fix things that aren't broken.
