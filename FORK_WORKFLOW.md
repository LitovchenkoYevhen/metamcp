# Fork Workflow - Инструкция по работе с форком

Эта инструкция описывает, как правильно работать с форком MetaMCP, вносить свои изменения и получать обновления из оригинального репозитория.

## Схема работы

```
ОРИГИНАЛЬНЫЙ РЕПОЗИТОРИЙ (upstream)
metatool-ai/metamcp
    ↓ (fork)
ТВОЙ ФОРК (origin)
YOUR_USERNAME/metamcp
    ├── main (чистая копия оригинала)
    └── custom-config (твои изменения)
```

## Первоначальная настройка (уже выполнено)

```bash
# 1. Создан Fork на GitHub
# https://github.com/LitovchenkoYevhen/metamcp

# 2. Клонирован твой fork
git clone https://github.com/LitovchenkoYevhen/metamcp.git
cd metamcp

# 3. Добавлен оригинал как upstream
git remote add upstream https://github.com/metatool-ai/metamcp.git

# 4. Проверка remotes
git remote -v
# origin    https://github.com/LitovchenkoYevhen/metamcp.git (твой fork)
# upstream  https://github.com/metatool-ai/metamcp.git (оригинал)
```

## Работа со своими изменениями

### 1. Создание ветки для кастомизаций

```bash
# Создать и переключиться на ветку custom-config
git checkout -b custom-config
```

### 2. Внесение изменений

```bash
# Вноси любые изменения:
# - Редактируй .env
# - Изменяй docker-compose.yml
# - Модифицируй код
# - Добавляй свои файлы

# Проверь что изменилось
git status

# Добавь все изменения
git add .

# Или добавь конкретные файлы
git add .env docker-compose.yml

# Создай коммит с описанием
git commit -m "Описание твоих изменений"
```

### 3. Отправка в свой fork

```bash
# Отправь изменения в свой fork на GitHub
git push origin custom-config
```

## Получение обновлений из оригинала

Когда автор выпустил новую версию, нужно синхронизироваться:

### Шаг 1: Обновить ветку main

```bash
# Переключись на main
git checkout main

# Скачай обновления из оригинала
git fetch upstream

# Слей изменения из оригинала в свой main
git merge upstream/main

# Отправь обновлённый main в свой fork
git push origin main
```

### Шаг 2: Применить обновления к своей ветке

```bash
# Переключись на свою ветку с изменениями
git checkout custom-config

# Примени обновления поверх своих изменений
git rebase main
```

### Шаг 3: Разрешение конфликтов (если есть)

```bash
# Если git показывает конфликты:
# 1. Открой файлы с конфликтами в редакторе
# 2. Найди маркеры конфликтов (<<<<<<< ======= >>>>>>>)
# 3. Вручную разреши конфликты
# 4. Сохрани файлы

# Добавь разрешённые файлы
git add <файл-с-конфликтом>

# Продолжи rebase
git rebase --continue

# Если хочешь отменить rebase:
# git rebase --abort
```

### Шаг 4: Отправить обновлённую ветку

```bash
# Отправь обновлённую ветку в свой fork
# --force-with-lease безопаснее чем --force
git push origin custom-config --force-with-lease
```

## Быстрая шпаргалка команд

### Ежедневная работа

```bash
# Переключиться на рабочую ветку
git checkout custom-config

# Посмотреть статус
git status

# Добавить изменения
git add .

# Закоммитить
git commit -m "Описание изменений"

# Отправить в fork
git push origin custom-config
```

### Синхронизация с оригиналом (раз в неделю/месяц)

```bash
# Обновить main
git checkout main
git fetch upstream
git merge upstream/main
git push origin main

# Обновить рабочую ветку
git checkout custom-config
git rebase main
git push origin custom-config --force-with-lease
```

### Полезные команды

```bash
# Посмотреть все ветки
git branch -a

# Посмотреть историю коммитов
git log --oneline --graph

# Отменить последний коммит (изменения остаются)
git reset --soft HEAD~1

# Посмотреть разницу с main
git diff main

# Временно спрятать незакоммиченные изменения
git stash

# Вернуть спрятанные изменения
git stash pop
```

## Визуальная схема обновлений

```
ДЕНЬ 1 - Начальное состояние:
Оригинал:     [A] → [B]
Твой main:    [A] → [B]
Твоя ветка:   [A] → [B] → [C] (твои изменения)

ДЕНЬ 30 - Вышло обновление:
Оригинал:     [A] → [B] → [D] → [E] (новые коммиты)
Твой main:    [A] → [B] (устарел)
Твоя ветка:   [A] → [B] → [C] (твои изменения)

ПОСЛЕ СИНХРОНИЗАЦИИ:
Оригинал:     [A] → [B] → [D] → [E]
Твой main:    [A] → [B] → [D] → [E] (обновлён)
Твоя ветка:   [A] → [B] → [D] → [E] → [C'] (твои изменения поверх новых)
```

## Преимущества этого подхода

✅ **Ветка main** всегда чистая копия оригинала
✅ **Ветка custom-config** содержит все твои изменения
✅ Можешь получать обновления в любой момент
✅ Твои изменения не потеряются
✅ Если что-то сломалось - легко вернуться на main
✅ Можешь создать несколько веток для разных экспериментов
✅ Можешь предложить свои улучшения автору через Pull Request

## Альтернативный подход для конфигурационных файлов

Если меняешь только `.env` и `docker-compose.yml`:

```bash
# Добавь в .gitignore
echo ".env.custom" >> .gitignore
echo "docker-compose.custom.yml" >> .gitignore

# Создай свои версии
cp .env .env.custom
cp docker-compose.yml docker-compose.custom.yml

# Редактируй .env.custom и docker-compose.custom.yml

# Используй их:
docker compose -f docker-compose.custom.yml up -d

# Обновления получай обычным pull:
git checkout main
git pull upstream main
```

## Важные замечания

⚠️ **Никогда не коммить в main** - она должна быть чистой копией оригинала
⚠️ **Используй --force-with-lease**, не --force (безопаснее)
⚠️ **Делай коммиты часто** с понятными описаниями
⚠️ **Синхронизируйся регулярно** чтобы избежать больших конфликтов
⚠️ **Не добавляй в git секреты** (.env с паролями, API ключи)

## Troubleshooting

### "Конфликты при rebase"
```bash
# Посмотреть какие файлы конфликтуют
git status

# Отменить rebase и попробовать merge вместо него
git rebase --abort
git merge main
```

### "Случайно закоммитил в main"
```bash
# Создай ветку из текущего состояния
git branch backup-branch

# Вернись на main
git checkout main

# Сбрось на upstream/main
git reset --hard upstream/main

# Вернись к своим изменениям
git checkout backup-branch
```

### "Хочу начать с чистого листа"
```bash
# Сохрани важные файлы вне git

# Удали ветку
git checkout main
git branch -D custom-config

# Создай заново
git checkout -b custom-config
```

## Полезные ссылки

- [Git Documentation](https://git-scm.com/doc)
- [GitHub Fork Guide](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks)
- [Resolving Merge Conflicts](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts)
