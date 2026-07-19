---
name: coder-skill
description: "Исполнитель кода. PRE-FLIGHT GATE перед каждым ходом. Делегирование файловых операций."
---

# Coder Skill

## PRE-FLIGHT GATE

> ⚠️ **ПЕРЕД ЛЮБЫМ tool-call → проверь разрешён ли этот инструмент. Нарушение = токены сожжены.**

### Заблокированные инструменты (для main модели / архитектора)

| Инструмент | Статус | Альтернатива |
|---|---|---|
| `read_file` | 🚫 БЛОКИРОВАНО | `delegate_task` |
| `search_files` / `grep` | 🚫 БЛОКИРОВАНО | `delegate_task` |
| `patch` | 🚫 БЛОКИРОВАНО | `delegate_task` |
| `write_file` | 🚫 БЛОКИРОВАНО | `delegate_task` |
| `terminal(build)` | 🚫 БЛОКИРОВАНО | `delegate_task` |
| `execute_code` | 🚫 БЛОКИРОВАНО | `delegate_task` |
| `session_search` | 🚫 БЛОКИРОВАНО | `delegate_task` |

### Разрешённые инструменты

| Инструмент | Статус |
|---|---|
| `clarify` | ✅ |
| `delegate_task` | ✅ **(DEFAULT для всех операций с файлами/кодом)** |
| `todo` | ✅ |
| `memory` | ✅ |
| `cronjob` | ✅ |
| `skill_view` | ✅ |
| `web_search` | ✅ |

---

## Workflow

### 1. Получить задачу от юзера
Выслушай, уточни неясное через `clarify`.

### 2. Загрузить architecture-guardrails
```python
skill_view(name="architecture-guardrails")
```

### 3. Написать план
```python
todo(todos=[...])
```

### 4. Сформировать ТЗ для delegate_task
В `context` обязательно включи:
- PRE-CODE чеклист (из architecture-guardrails)
- Пути файлов для изменений
- Конвенции проекта (ссылка на `principles.md`)
- Критерий готовности

### 5. delegate_task #1: реализация
Передай ТЗ саб-агенту. Убедись что PRE-CODE вставлен.

### 6. delegate_task #2: ревью
Отправь POST-CODE чеклист другому саб-агенту или этой же модели в режиме ревью.
Если ответ `❌ FAIL` → повтори шаг 5 с исправлениями.

### 7. Session-handoff
По завершению — выполни shutdown-ритуал (см. `session-handoff`).

---

## Исключения (когда можно писать самому)

Допускается прямой `write_file` / `patch` без `delegate_task` только когда:

- **Тривиальный stub** < 20 строк в **новом** файле
- **Шаблонные / boilerplate файлы** (конфиги, пустые классы-заглушки)
- **Файлы документации** (README, CHANGELOG, ROADMAP — текстовые правки)

Всё остальное — только через `delegate_task`.
