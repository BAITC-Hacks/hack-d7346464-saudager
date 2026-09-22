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
