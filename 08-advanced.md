# ⚙08 Advanced — Продвинутая работа с Git

В этом разделе — инструменты переписывания истории, точечного переноса коммитов, временного сохранения изменений и подключения вложенных репозиториев.

---

## `git rebase` — перенос коммитов поверх другой базы

Переписывает историю: «проигрывает» ваши коммиты поверх целевой ветки/коммита.

**Важно:** не делайте `rebase` публичных веток, если коммиты уже ушли на удалённый — это ломает историю коллег.

### Основные приёмы
```bash
git rebase <upstream>                 # Перенести ТЕКУЩУЮ ветку поверх <upstream>
git rebase -i <upstream>              # Интерактивный rebase: squash/fixup/reword/drop
git rebase --onto <newbase> <upstream> [branch]
# Применить коммиты из branch, которые лежат ПОСЛЕ <upstream>, на базу <newbase>

git rebase --continue                 # Продолжить после ручного разрешения конфликтов
git rebase --abort                    # Отменить и вернуть исходное состояние
git rebase --skip                      # Пропустить проблемный коммит
```

### Полезные опции
```bash
git rebase -i --autosquash <upstream> # Автосворачивание fixup!/squash! коммитов
git rebase --rebase-merges            # Сохранить (восстановить) структуру мерджей
git rebase --autostash <upstream>     # Авто-stash до ребейза и авто-восстановление
git rebase --keep-base                # Сохранить исходную базу (полезно при обновлении fork)
```

**Советы:**
- Используйте `fixup!`/`squash!` в сообщениях коммитов, чтобы с `--autosquash` быстро чистить историю.
- Для повседневной тонкой истории: `git config --global pull.rebase true` и (по желанию) `rebase.autosquash=true`, `rebase.autoStash=true`.

---

## `git cherry-pick` — выборочно применить коммиты

Берёт один или несколько **конкретных** коммитов и применяет их поверх текущей ветки.

### Примеры
```bash
git cherry-pick <hash>                # Перенести один коммит
git cherry-pick <A>^..<B>             # Диапазон: с A ВКЛЮЧАЯ по B (порядок сохранится)
git cherry-pick --continue            # После разрешения конфликтов
git cherry-pick --skip                 # Пропустить проблемный коммит
git cherry-pick --abort                # Отменить cherry-pick
```

### Полезные опции
```bash
git cherry-pick -n <hash>             # Применить изменения без создания коммита (accumulate)
git cherry-pick -x <hash>             # Добавить ссылку на оригинальный коммит в сообщении
git cherry-pick --strategy-option theirs|ours  # В сложных конфликтах (точечно)
```

**Помните:** cherry-pick дублирует изменения в новой точке истории — это другой hash. Для переноса «ветки целиком» чаще подходит `rebase`/`merge`.

---

## `git stash` — временно убрать изменения «на полку»

Убирает незакоммиченные изменения, чтобы переключиться на другую задачу, и позволяет вернуть их позже.

### База
```bash
git stash push -m "msg"               # Спрятать изменения (tracked)
git stash push -u -m "msg"            # Включая untracked файлы
git stash push -a -m "msg"            # Включая игнорируемые файлы (.gitignore)
git stash list                         # Список stash'ей
git stash show -p stash@{0}            # Патч конкретного stash
```

### Возврат и управление
```bash
git stash apply stash@{0}              # Применить (stash остаётся в списке)
git stash pop                          # Применить и удалить из списка
git stash drop stash@{0}               # Удалить конкретный stash
git stash clear                        # Очистить все stash'и (осторожно)
git stash branch fixup stash@{0}       # Создать ветку и развернуть на неё stash
```

**Приём:** можно стэшить выборочно по pathspec:
```bash
git stash push -m "only src" -- src/   # Спрятать только изменения в src/
```

---

## Submodules — вложенные репозитории

Позволяют подключать чужие/отдельные репозитории как директории внутри вашего проекта. Родитель хранит **жёсткую ссылку на точный коммит** подмодуля.

### Подключение и клон
```bash
git submodule add <url> path/to/module   # Добавить подмодуль
git submodule init                       # Инициализировать локальные метаданные
git submodule update                     # Выкачать коммит, на который ссылается родитель
git submodule update --init --recursive  # Инициализация и вложенные подмодули

git clone --recurse-submodules <url>     # Клон сразу со всеми подмодулями
```

### Обновления и синхронизация
```bash
cd path/to/module
git fetch
git checkout <new-commit-or-branch>      # Перейти на нужный коммит/ветку
cd ../..
git add path/to/module                   # Зафиксировать новый pointer в родителе
git commit -m "Bump submodule to <hash>"
```

### Полезные приёмы
```bash
git submodule foreach 'git status'       # Выполнить команду во всех подмодулях
git submodule sync --recursive           # Синхронизировать URL'ы после изменений .gitmodules
```

**Подводные камни:**
- Подмодуль обычно находится в состоянии **detached HEAD** — это нормально.  
- Любые изменения **внутри подмодуля** надо коммитить в его репозитории, затем обновлять pointer в родителе.  
- Если хотите «просто положить код» без жёсткой привязки к коммиту — рассмотрите **Git Subtree** как альтернативу.

---

## Типовые сценарии

**1) Приятная история перед merge request:**
```bash
git fetch origin
git rebase -i origin/main          # squash/fixup/reword
git push --force-with-lease
```

**2) Срочно перенести фикс из main в релизную ветку:**
```bash
git checkout release/1.2
git cherry-pick <fix-commit>
git push
```

**3) Быстро спрятать «грязные» правки и переключиться:**
```bash
git stash push -u -m "WIP: feature X"
git switch hotfix/login-crash
# ...фикс, коммит, пуш...
git switch -                        # вернуться назад
git stash pop
```

**4) Обновить подмодуль до свежего релиза:**
```bash
cd libs/some-module
git fetch --tags
git checkout v2.3.0
cd ../..
git add libs/some-module
git commit -m "Bump some-module to v2.3.0"
```
