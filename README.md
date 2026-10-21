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

### How it works

- `hooks/routing-policy.md` is loaded into every session by a `SessionStart` hook. It tells Claude how to sort tasks and when to delegate.
- `agents/deep-worker.md` and `agents/max-worker.md` are subagents with `effort: high` and `effort: max` in their frontmatter.
- `skills/deep` and `skills/max` are the slash commands that force a level.

### Limits

- The main session's own effort does not change. Hard work runs in a subagent instead.
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

### Хязгаарлалт

- Үндсэн session-ий effort өөрчлөгдөхгүй. Хүнд ажил subagent дотор ажиллана.
- Subagent таны чатыг харахгүй. Claude түүнд даалгаврын тайлбар бичиж өгөх бөгөөд танд бүх алхам биш, хураангуй ирнэ.
- Claude ажлын хүндийг буруу үнэлж болно. Чухал үед түвшинг өөрөө нэрлээрэй.
- Ямар effort түвшин байх нь model-оос хамаарна.
- Plugin нь CLI болон desktop app-ын local, SSH session-д ажиллана. Cloud session-д ажиллахгүй.

---

MIT License
