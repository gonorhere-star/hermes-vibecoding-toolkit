# Установка

## Предварительные требования

- **Hermes Agent** — установлен и настроен (см. [документацию](https://hermes-agent.nousresearch.com/docs))
- **DeepSeek API key** (или другой дешёвый LLM для роли исполнителя)
- **GLM API key** (или другая дорогая модель для роли архитектора)

## Пошаговая инструкция

### 1. Установить Hermes Agent

Если ещё не установлен — следуйте официальной инструкции:

```bash
# Пример для pip
pip install hermes-agent

# Или через brew (macOS)
brew install hermes-agent
```

### 2. Создать профиль

```bash
hermes profiles create gamedev
```

Или используйте имя, которое вам удобно:

```bash
hermes profiles create <название-профиля>
```

### 3. Настроить провайдеров

В конфиге профиля (`~/.hermes/profiles/<name>/config.yaml`) нужно настроить два
провайдера:

```yaml
# Основная модель (архитектор) — дорогая, качественная
default_provider: deepseek
default_model: deepseek-chat   # GLM

# Модель для delegate_task (исполнитель) — дешёвая
# Указывается при вызове, настройка API-ключа в provider config
```

Убедитесь, что у DeepSeek API ключ настроен:

```yaml
providers:
  deepseek:
    api_key: "sk-..."
```

### 4. Скопировать навыки

Из папки `templates/skills/` (в этом репозитории) скопируйте навыки в профиль:

```bash
cp -r templates/skills/* ~/.hermes/profiles/<name>/skills/
```

Основные навыки:
- `architecture-guardrails` — PRE-CODE / POST-CODE чеклисты
- `session-handoff` — обновление CHANGELOG, ROADMAP, ADR
- `coder-skill` (или аналог) — PRE-FLIGHT GATE проверка дисциплины

### 5. Создать AGENTS.md

Скопируйте `templates/AGENTS.md` в корень вашего проекта:

```bash
cp templates/AGENTS.md /path/to/your/project/AGENTS.md
```

Это контракт, описывающий роли и границы ответственности.

### 6. Создать game_docs/

В корне проекта создайте папку для документации:

```bash
mkdir -p game_docs/adr
touch game_docs/CHANGELOG.md
touch game_docs/ROADMAP.md
touch game_docs/principles.md
```

- `CHANGELOG.md` — журнал изменений
- `ROADMAP.md` — план развития
- `principles.md` — архитектурные принципы проекта
- `adr/` — Architectural Decision Records

### 7. Настроить memory (блок-лист инструментов)

Создайте memory-запись, которая блокирует GLM от доступа к файлам:

```bash
hermes memory add \
  --content "GLM: НЕ ИСПОЛЬЗОВАТЬ read_file, search_files, patch, write_file, terminal(build), execute_code, session_search" \
  --scope "system"
```

### 8. Настроить coder-skill PRE-FLIGHT GATE

Убедитесь, что coder-skill (или ваш навык исполнителя) проверяет блок-лист ДО начала
работы. Типичная проверка:

```yaml
# coder-skill: PRE-FLIGHT GATE
check:
  - GLM не вызывал read_file за последние N шагов
  - GLM не вызывал patch/write_file самостоятельно
  - Все изменения кода — через delegate_task
```

### 9. Тест

Запустите тестовую задачу через `delegate_task`:

```
GLM: #delegate_task (model: deepseek-reasoner, task: "Прочитай README и скажи
      какие три главные темы в нём описаны")
```

Проверьте, что GLM вернул результат, НЕ прочитав README самостоятельно.

### 10. Obsidian (опционально)

Если используете Obsidian — откройте папку `game_docs/` как хранилище.
Это даст визуальный редактор для CHANGELOG, ROADMAP, ADR и принципов.
