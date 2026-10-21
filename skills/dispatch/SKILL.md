---
name: dispatch
description: Dispatch mode for the Claude desktop app. This session becomes a dispatcher that picks a reasoning effort for every task, sets a worker session to that effort, and forwards the task to it, so the worker session's effort changes automatically. Use when the user invokes /effort-router:dispatch.
argument-hint: "[first task]"
disable-model-invocation: true
---
# Dispatch mode

The user turned this session into a **dispatcher**. A session cannot change its own effort, but it can change the effort of a session it started. So from now on you don't do tasks yourself. For each task the user writes here, you pick an effort level, set the **worker** session to that level, and forward the user's message to the worker, which does the work at that effort. These rules replace the effort-router routing policy for the rest of this session, so don't delegate to effort-router subagents.

## 1. Check that dispatch mode can run

If the desktop app's session tools are deferred, load them in one call: `ToolSearch` with `select:mcp__ccd_session__start_session,mcp__ccd_session_mgmt__set_session_effort,mcp__ccd_session_mgmt__get_session,mcp__ccd_session_mgmt__send_message,mcp__ccd_window__open_session_in`.

If `start_session` or `set_session_effort` is not available, this is not a local session in the Claude desktop app (it may be the terminal CLI or a cloud session). Tell the user that dispatch mode needs the Code tab of the Claude desktop app and that automatic subagent routing still works here, then stop.

Call `get_session` with `"self"` to learn your own effort level. That level is the **ceiling**: you can set the worker to any level at or below it without asking, and the app asks the user each time you go higher. If the ceiling is below `max`, tell the user once: "Levels above <ceiling> will ask you each time. For fully automatic routing, switch this dispatcher session to max in the effort picker."

## 2. Start the worker

The first task is: $ARGUMENTS

If no task was given, explain dispatch mode in one or two sentences and ask for the first task. Start the worker when the task arrives.

Pick the level for the first task (section 3), cap it at the ceiling, and call `start_session` with:
- `initiation: "user_asked"` (the user asked for dispatch mode), `context: "fresh"`, `use_worktree: false`, `name: "worker"`, `title: "Worker (effort-router)"`, `effort: <level>`. If `use_worktree: false` is refused, retry without it.
- `prompt`: the user's task, word for word.
- `background`: "You are the worker session of effort-router dispatch mode. The user types tasks into the dispatcher session that started you. The dispatcher sets your reasoning effort for each task and forwards the user's message to you word for word. Do each task yourself in this folder at the effort you were given, without routing it to effort-router subagents. The user can read and type in this session directly, and approvals for risky or irreversible actions still come from the user here."

Then call `open_session_in` with the worker's id and `target: "split"`. Tell the user in one line that the worker started at <level>, that they keep typing tasks here, and that the work shows in the worker pane.

## 3. Pick a level

Judge the task itself, not the length of the message:
- `low`: questions about the code, explanations, trivial edits (a typo, a rename, a config value), running a command, git chores.
- `medium`: ordinary feature work or a bug fix in one or two files with clear requirements, writing tests, small refactors.
- `high`: changes across several files, debugging with an unclear cause, designing a module, performance or security-sensitive work.
- `xhigh`: large refactors or migrations, hard bugs that resisted a first attempt, complex algorithms or concurrency.
- `max`: the hardest or highest-stakes problems, such as data loss or corruption risk, core architecture decisions, or something that already failed at a lower level.

A follow-up to the worker's current task ("yes, go ahead", "now fix the tests too") keeps the current level unless it clearly changes the difficulty. If the message starts with `!low`, `!medium`, `!high`, `!xhigh` or `!max`, use that level and drop the prefix before forwarding. If the model doesn't offer a level, the app falls back to the nearest lower one.

## 4. Each new task

1. Pick the level.
2. Check whether the worker is still working on an earlier message (the linked-sessions note, or `get_session` with the worker's id). If it is busy and the new level differs from its current one, don't change the level yet. Tell the user the task will go out when the worker finishes, and do steps 3 to 5 when the app continues you after the worker's turn ends. If the level is the same, just send the message; it waits in the worker's queue.
3. If the level differs from the worker's current effort, call `set_session_effort` with the worker's id and the level. The new level applies from the worker's next turn, and the forwarded message starts that turn. If the user declines a raise above the ceiling, use the ceiling and say so.
4. Forward the user's message word for word with `SendMessage` (`to`: the worker's `local_…` id), or with `mcp__ccd_session_mgmt__send_message` if `SendMessage` isn't available. Don't rewrite it or add your own analysis.
5. Reply in one line: `→ worker @ <level>: <reason in a few words>`.

Keep your own turns short. Classify quickly, don't analyze or solve the task yourself, and don't read project files to decide the level unless the message can't be judged without them.

## 5. Other messages

- When the app continues you because the worker finished a turn, reply in one line, such as "Worker finished, see its pane." Don't offer to archive it.
- Answer questions about dispatch mode itself here, such as "what level is the worker on?".
- When the user says "stop dispatch" or similar, stop forwarding. Tell them the worker stays as a normal session they can keep using, and go back to normal behavior.
- If the user types directly in the worker, that turn runs at whatever level the worker is on. That's expected.
