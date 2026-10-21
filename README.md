# effort-router

**Let Claude Code pick the reasoning effort for each task.**

[English](#english) · [Монгол](#монгол)

---

## English

Claude Code runs a whole session at one effort level, and a session cannot raise or lower its own effort. With effort-router, Claude sorts each task by how hard it is and routes it:

| Task | Examples | Where it runs |
|---|---|---|
| Simple | questions, small edits, one obvious fix | main session (your current effort) |
| Hard | multi-file debugging, architecture, big refactors | `deep-worker` subagent, **high** effort |
| Hardest | bugs that resisted earlier fixes, concurrency, migrations | `max-worker` subagent, **max** effort |

Keep your main session on **low** or **medium**: everyday work stays fast and cheap, and you only pay for high or max effort when a task needs it.

In the Claude desktop app there is also **[dispatch mode](#dispatch-mode-desktop-app)**, where the session doing the work switches its own effort level for every task.

### Install

In Claude Code (CLI or the desktop app's Code tab):

```
/plugin marketplace add BabuNfr666/claude-effort-router
/plugin install effort-router@effort-router
```

Or from a shell:

```bash
claude plugin marketplace add BabuNfr666/claude-effort-router
claude plugin install effort-router@effort-router
```

Then start a new session (or run `/reload-plugins`).

In the desktop app you can also click **+** next to the prompt box → **Plugins** → **Add plugins**, then add the marketplace `BabuNfr666/claude-effort-router`.

### Use

You don't have to do anything: Claude routes on its own and says so in one line, for example *"Routing to deep-worker (high effort): race condition across three files"*.

To force a level:

```
/effort-router:deep fix the flaky checkout test
/effort-router:max design the migration from REST to gRPC
```

You can also just say "do it yourself" or "use max" in plain words.

### Dispatch mode (desktop app)

Claude Code doesn't let a session change its own effort. That's deliberate: a session shouldn't quietly re-price its own turns. A session can, however, change the effort of a session it started, and dispatch mode is built on that. You type into a **dispatcher** session. For each task it picks a level, sets a **worker** session to that level, and forwards your message, so the worker runs every task at its own level.

1. Open a new Code session in your project and set its effort to **max** in the picker. The dispatcher's own level is the highest it can set without asking you.
2. Run:
   ```
   /effort-router:dispatch add rate limiting to the login endpoint
   ```
3. A worker session opens in split view. Keep typing tasks into the dispatcher, and watch the work happen in the worker.

For each task the dispatcher replies with one line, such as `→ worker @ high: auth change across several files`.

| Level | Typical task |
|---|---|
| low | questions, explanations, trivial edits, running commands |
| medium | an ordinary feature or bug fix in one or two files |
| high | multi-file changes, bugs with an unclear cause, module design |
| xhigh | large refactors, migrations, hard bugs |
| max | the hardest or highest-stakes work: architecture, data-loss risk |

To force a level, start a message with `!low`, `!medium`, `!high`, `!xhigh` or `!max`. To end dispatch mode, say "stop dispatch". The worker stays as a normal session.

Dispatch mode has its own limits:
- It needs local sessions in the Code tab of the Claude desktop app. It doesn't work in the terminal CLI or in cloud sessions.
- Type tasks into the dispatcher. A message typed straight into the worker runs at whatever level the worker is on.
- Raising the worker above the dispatcher's own level asks you each time.
- The dispatcher spends a few tokens per message to classify and forward it.
- If the worker is still busy and the next task needs a different level, the dispatcher holds the task until the worker finishes.

### How it works

- `hooks/routing-policy.md` is loaded into every session by a `SessionStart` hook. It tells Claude how to sort tasks and when to delegate.
- `agents/deep-worker.md` and `agents/max-worker.md` are subagents with `effort: high` and `effort: max` in their frontmatter.
- `skills/deep` and `skills/max` are the slash commands that force a level.
- `skills/dispatch` is dispatch mode. It uses the desktop app's session tools to start the worker, set its effort, and forward your messages.

### Limits

- With subagent routing, the main session's own effort does not change, and hard work runs in a subagent instead. Use dispatch mode if you want the working session itself to switch levels.
- A subagent can't see your conversation. Claude writes it a brief, and you get back a summary, not every step.
- Claude can misjudge how hard a task is. Name a level when it matters.
- Which effort levels exist depends on the model. On a model without `max`, Claude Code falls back to what the model supports.
- Plugins work in the CLI and in local and SSH sessions in the desktop app. They don't work in cloud sessions.

### Update / uninstall

```bash
claude plugin marketplace update effort-router
claude plugin update effort-router@effort-router
claude plugin uninstall effort-router@effort-router
```

---

## Монгол

Claude Code нэг session-ийг бүхэлд нь нэг effort түвшинд ажиллуулдаг бөгөөд session өөрийнхөө effort-ийг өөрөө сольж чаддаггүй. effort-router суулгавал Claude ажил бүрийн хүндийг үнэлээд дараах байдлаар чиглүүлнэ:

| Ажил | Жишээ | Хаана ажиллах |
|---|---|---|
| Энгийн | асуулт, жижиг засвар, илэрхий алдаа | үндсэн session (таны одоогийн effort) |
| Хүнд | олон файлтай debug, архитектур, том refactor | `deep-worker` subagent, **high** effort |
| Хамгийн хүнд | өмнө нь засагдаагүй bug, concurrency, migration | `max-worker` subagent, **max** effort |

Үндсэн session-ээ **low** эсвэл **medium** дээр байлгаарай. Өдөр тутмын ажил хурдан, хямд байх бөгөөд high эсвэл max effort-ийн төлбөрийг зөвхөн хэрэгтэй үед л төлнө.

Claude desktop app дээр мөн **dispatch горим** бий. Энэ горимд ажлыг гүйцэтгэж буй session өөрөө ажил бүрт effort-оо сольдог.

### Суулгах

Claude Code дотор (CLI эсвэл desktop app-ын Code tab):

```
/plugin marketplace add BabuNfr666/claude-effort-router
/plugin install effort-router@effort-router
```

Эсвэл terminal-аас:

```bash
claude plugin marketplace add BabuNfr666/claude-effort-router
claude plugin install effort-router@effort-router
```

Дараа нь шинэ session эхлүүлнэ үү (эсвэл `/reload-plugins` ажиллуулна).

Desktop app дээр prompt-ын хажууд байгаа **+** → **Plugins** → **Add plugins** дарж, `BabuNfr666/claude-effort-router` marketplace-ийг нэмж болно.

### Ашиглах

Юу ч хийх шаардлагагүй. Claude өөрөө чиглүүлээд нэг мөрөөр мэдэгдэнэ, жишээ нь *"Routing to deep-worker (high effort): ..."*.

Түвшинг албадах бол:

```
/effort-router:deep checkout тестийн алдааг засаарай
/effort-router:max REST-ээс gRPC руу шилжих migration-ийг төлөвлө
```

Эсвэл "өөрөө хий", "max ашигла" гэж энгийнээр хэлж болно.

### Dispatch горим (desktop app)

Claude Code нь session-д өөрийнхөө effort-ийг солихыг зөвшөөрдөггүй. Энэ нь санаатай: session өөрийнхөө turn-ийн үнийг чимээгүй өөрчлөх ёсгүй. Харин session өөрийн эхлүүлсэн session-ий effort-ийг сольж чадна, dispatch горим үүн дээр суурилдаг. Та **dispatcher** session-д бичнэ. Dispatcher ажил бүрт түвшин сонгоод, **worker** session-ийг тэр түвшинд тохируулж, таны мессежийг дамжуулна. Ингэснээр worker ажил бүрийг өөр өөрийн түвшинд гүйцэтгэнэ.

1. Төсөл дээрээ шинэ Code session нээгээд effort-ийг нь picker дээр **max** болгоно. Dispatcher-ийн өөрийн түвшин нь танаас асуулгүйгээр тохируулж чадах хамгийн өндөр түвшин юм.
2. Ажиллуулна:
   ```
   /effort-router:dispatch login endpoint-д rate limiting нэм
   ```
3. Worker session split view-д нээгдэнэ. Ажлуудаа dispatcher-т бичсээр байна, гүйцэтгэл нь worker дээр харагдана.

Ажил бүрт dispatcher нэг мөрөөр хариулна, жишээ нь `→ worker @ high: олон файлтай auth өөрчлөлт`.

| Түвшин | Жишээ ажил |
|---|---|
| low | асуулт, тайлбар, жижиг засвар, команд ажиллуулах |
| medium | нэг хоёр файлд хамаарах энгийн feature, bug засвар |
| high | олон файлтай өөрчлөлт, шалтгаан нь тодорхойгүй bug, модулийн дизайн |
| xhigh | том refactor, migration, хэцүү bug |
| max | хамгийн хэцүү, эрсдэл өндөртэй ажил: архитектур, өгөгдөл алдах эрсдэл |

Түвшинг албадах бол мессежээ `!low`, `!medium`, `!high`, `!xhigh` эсвэл `!max`-аар эхлүүлнэ. Dispatch горимыг дуусгах бол "stop dispatch" гэж бичнэ. Worker энгийн session болж үлдэнэ.

Dispatch горимын хязгаарлалт:
- Зөвхөн Claude desktop app-ын Code tab-ын local session-д ажиллана. Terminal CLI болон cloud session-д ажиллахгүй.
- Ажлаа dispatcher-т бичнэ. Worker руу шууд бичсэн мессеж worker-ийн тухайн үеийн түвшинд ажиллана.
- Worker-ийг dispatcher-ийн өөрийн түвшнөөс дээш өсгөх бүрт танаас асууна.
- Dispatcher мессеж бүрийг ангилж дамжуулахад бага зэрэг токен зарцуулна.
- Worker завгүй байхад дараагийн ажилд өөр түвшин хэрэгтэй бол dispatcher тэр ажлыг worker дуустал хүлээлгэнэ.

### Хязгаарлалт

- Subagent чиглүүлэлтийн үед үндсэн session-ий effort өөрчлөгдөхгүй, хүнд ажил subagent дотор ажиллана. Ажиллаж буй session өөрөө түвшнээ солих хэрэгтэй бол dispatch горим ашиглаарай.
- Subagent таны чатыг харахгүй. Claude түүнд даалгаврын тайлбар бичиж өгөх бөгөөд танд бүх алхам биш, хураангуй ирнэ.
- Claude ажлын хүндийг буруу үнэлж болно. Чухал үед түвшинг өөрөө нэрлээрэй.
- Ямар effort түвшин байх нь model-оос хамаарна.
- Plugin нь CLI болон desktop app-ын local, SSH session-д ажиллана. Cloud session-д ажиллахгүй.

---

MIT License
