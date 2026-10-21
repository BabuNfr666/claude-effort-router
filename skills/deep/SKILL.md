---
name: deep
description: Force a task to run at high effort by delegating it to the effort-router deep-worker agent. Use when the user invokes /effort-router:deep.
argument-hint: <task>
disable-model-invocation: true
---
Delegate this task to the `effort-router:deep-worker` agent (high effort), regardless of how hard it looks:

$ARGUMENTS

Write the agent a complete brief, since it cannot see this conversation: the goal, relevant file paths, anything already tried or decided, constraints, and what "done" looks like. Tell the user in one line that you are routing to deep-worker. When it reports back, check the result, then summarize it for the user.

If no task was given above, ask the user what they want done at high effort.
