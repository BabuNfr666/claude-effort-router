# Effort routing (effort-router plugin)

The user installed the effort-router plugin so that reasoning effort matches each task. A session cannot change its own effort level, so route by delegating. Before starting each task the user gives you, judge how hard it is:

- **Simple** - a question, an explanation, a small or mechanical edit, a single-file change with an obvious fix, running a command. Do it yourself in this session.
- **Hard** - debugging across several files, a bug whose cause is not obvious, an architecture or design decision, a large refactor, a tricky algorithm, security- or data-sensitive changes. Delegate to the `effort-router:deep-worker` agent (high effort).
- **Hardest** - a problem that already resisted an attempt (including one by deep-worker), concurrency or data-corruption bugs, core architecture or migration design where being wrong is costly. Delegate to the `effort-router:max-worker` agent (max effort).

When you delegate:
- Say so in one short line first, e.g. "Routing to deep-worker (high effort): <reason>".
- The agent cannot see this conversation. Give it a complete brief: the goal, relevant file paths, what was already tried, constraints, and what "done" looks like.
- When it reports back, check the result before telling the user it is done, then summarize.

Do not delegate when the task needs back-and-forth with the user, when it is quicker to just do it, or when the user asks you to handle it here. If the user names a level ("use max", "do it yourself"), follow that. The user can also force routing with `/effort-router:deep <task>` or `/effort-router:max <task>`.

Dispatch mode: if the user runs `/effort-router:dispatch`, its rules replace this policy in that session. A session whose brief says it is the worker of effort-router dispatch mode already runs at the effort picked for each task, so it does every task itself and does not delegate to effort-router subagents.
