# Как поднять такую же среду на другой машине

Проверено на текущей машине 23.09.2026. Ниже — всё, что отличает эту среду от чистой установки.

## 1. Claude Code

Установить и войти **под тем же аккаунтом**. Скиллы и коннектор документов привязаны к аккаунту и подтягиваются сами — руками их ставить не нужно.

## 2. Настройки

`~/.claude/settings.json` — ровно это:

```json
{
  "model": "opus[1m]",
  "effortLevel": "xhigh",
  "modelSettings": {
    "claude-opus-5": {
      "effortLevel": "xhigh"
    }
  },
  "remoteControlAtStartup": false
}
```

`opus[1m]` — Opus 5 с окном в миллион токенов. `xhigh` — уровень рассуждений. Обе строки важны: на длинной сессии с разбором документов окно расходуется быстро.

## 3. Ultracode

Включается в сессии, в файлах не хранится. Написать в начале работы слово `ultracode` либо включить в `/config`. Даёт оркестрацию подзадач через несколько агентов вместо одиночных ответов — на разборе документов и сверках это основная рабочая лошадь.

## 4. Что приезжает с аккаунтом само

- **Скиллы:** `docs`, `docx`, `pdf`, `pptx`, `xlsx`, `skill-creator`, `import-memory`, `morning`.
- **Коннектор Claude Docs** (MCP). Локального MCP-конфига нет, `mcpServers` пустой — это account-level.

## 4a. Плагины

На эталонной машине их 12 записей (9 уникальных, три задвоены) из трёх маркетплейсов.

**Из официального каталога `claude-plugins-official`** — ставятся сразу:

| Плагин | Зачем |
|---|---|
| `superpowers` | набор скиллов общего назначения |
| `context7` | актуальная документация библиотек по запросу |
| `playwright` | браузерная автоматизация и проверка интерфейса |
| `frontend-design` | работа с версткой и интерфейсом |
| `code-review` | ревью изменений |
| `security-guidance` | проверки безопасности |

**Из сторонних маркетплейсов** — сначала подключить маркетплейс, потом плагин:

| Маркетплейс | Плагин |
|---|---|
| `supabase-agent-skills` | `supabase`, `postgres-best-practices` |
| `n8n-mcp-skills` | `n8n-mcp-skills` |

**Имена маркетплейсов в интерфейсе не совпадают с путями репозиториев** — подставлять нужно вторые:

| Имя в интерфейсе | Репозиторий для `marketplace add` |
|---|---|
| `claude-plugins-official` | `anthropics/claude-plugins-official` |
| `supabase-agent-skills` | `supabase/agent-skills` |
| `n8n-mcp-skills` | `czlonkowski/n8n-skills` |

Через интерфейс: `/plugin` → вкладка **Marketplaces** добавить все три → вкладка **Plugins** установить нужные.

Через CLI — весь набор одной командой:

```bash
claude plugin marketplace add anthropics/claude-plugins-official
claude plugin marketplace add supabase/agent-skills
claude plugin marketplace add czlonkowski/n8n-skills

for p in superpowers context7 playwright frontend-design code-review security-guidance; do
  claude plugin install "$p@claude-plugins-official"
done
claude plugin install supabase@supabase-agent-skills
claude plugin install postgres-best-practices@supabase-agent-skills
claude plugin install n8n-mcp-skills@n8n-mcp-skills
```

Если `claude` не в PATH, он лежит внутри расширения VSCode:
`~/.vscode-server/extensions/anthropic.claude-code-*/resources/native-binary/claude`

После установки нужен перезапуск сессии, чтобы плагины подхватились.

**Два замечания по составу.**

Задвоены `supabase`, `postgres-best-practices` и `n8n-mcp-skills` — один и тот же плагин зарегистрирован дважды. Дубль грузит те же скиллы повторно и тратит контекст на каждом запросе. Стоит снести лишние записи.

`code-review` есть и как встроенный скилл Claude Code, и как плагин — они пересекаются.

Плагины стоят контекста на каждом промпте, поэтому под конкретную задачу разумно держать включёнными только нужные: `supabase`, `postgres-best-practices` и `n8n-mcp-skills` имеют смысл, только если кейс действительно про базу или автоматизацию.

## 5. Что встроено в Claude Code и ставить не надо

Инструменты: Bash, Read, Write, Edit, подагенты, Workflow, WebSearch, WebFetch, Monitor, ToolSearch.
Скиллы: `code-review`, `simplify`, `security-review`, `run`, `init`, `loop`, `schedule`, `workflow-authoring`, `claude-api`, `update-config`, `dataviz`, `artifact-*`.
Типы агентов: `Explore`, `Plan`, `general-purpose`, `claude-code-guide`.

Плагинов не установлено ни одного. Кастомных агентов нет. Проектного `.claude/` нет.

## 6. Память проекта — единственное, что не переезжает само

Лежит вне репозитория: `~/.claude/projects/<путь-проекта-через-дефисы>/memory/`.
Здесь это `/root/.claude/projects/-var-www-hack-d7346464-saudager/memory/`.

Скопировать папку целиком. Если потерялась — в ней четыре заметки:

| Файл | О чём |
|---|---|
| `hackalem-hard-rules.md` | правила Положения, которые нельзя нарушить |
| `hackalem-scoring-levers.md` | как оценивают: общей таблицы баллов нет, критерии из ТЗ |
| `agents-md-not-committed.md` | свод правил коммитится (решение пересмотрено 23.09) |
| `no-claude-attribution.md` | в коммитах не ставить подписи Claude |

Первые две дублируются содержанием `agent/rules.md` в репозитории. Две последние — только в памяти, их стоит перенести руками.

## 7. Что уже в репозитории и приедет с клоном

`AGENTS.md`, `CLAUDE.md` и папка `agent/` — свод правил, детали, регламент постановки задач агенту. Отдельно ставить ничего не нужно.

## Короткая версия для другого ИИ

> Поставь Claude Code, войди под аккаунтом <твой>. Запиши в `~/.claude/settings.json`: модель `opus[1m]`, `effortLevel` `xhigh`, `remoteControlAtStartup` false. В сессии включи ultracode. Скопируй папку памяти проекта из `~/.claude/projects/*/memory/`. Правила работы уже лежат в репозитории — `CLAUDE.md` подтянет `AGENTS.md` сам.
