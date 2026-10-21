---
name: max
description: Force a task to run at maximum effort by delegating it to the effort-router max-worker agent. Use when the user invokes /effort-router:max.
argument-hint: <task>
disable-model-invocation: true
---
Delegate this task to the `effort-router:max-worker` agent (max effort), regardless of how hard it looks:

$ARGUMENTS

Write the agent a complete brief, since it cannot see this conversation: the goal, relevant file paths, anything already tried or decided, constraints, and what "done" looks like. Tell the user in one line that you are routing to max-worker. When it reports back, check the result, then summarize it for the user.

If no task was given above, ask the user what they want done at max effort.
