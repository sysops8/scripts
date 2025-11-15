# Полный практический курс по Git
## От новичка до эксперта

---

## 🟢 JUNIOR GIT USER

### Задача 1: Инициализация и первый коммит
**Цель:** Создать репозиторий и сделать первые коммиты

**Практика:**
```bash
# Создать директорию проекта
mkdir my-first-project
cd my-first-project

# Инициализировать Git репозиторий
git init

# Настроить имя и email
git config user.name "Your Name"
git config user.email "your.email@example.com"

# Создать файлы
echo "# My First Project" > README.md
echo "Hello, Git!" > hello.txt

# Проверить статус
git status

# Добавить файлы в staging area
git add README.md
git add hello.txt
# Или все сразу:
# git add .

# Сделать коммит
git commit -m "Initial commit: add README and hello.txt"

# Посмотреть историю
git log
git log --oneline
```

**Упражнение:**
1. Создайте новый проект
2. Добавьте 3 разных файла
3. Сделайте 3 отдельных коммита для каждого файла
4. Посмотрите историю коммитов

---

### Задача 2: Работа с изменениями
**Цель:** Научиться отслеживать и коммитить изменения

**Практика:**
```bash
# Изменить существующий файл
echo "New line" >> hello.txt

# Посмотреть изменения
git status
git diff

# Добавить изменения
git add hello.txt

# Посмотреть что в staging
git diff --staged

# Сделать коммит
git commit -m "Add new line to hello.txt"

# Изменить несколько файлов
echo "## Installation" >> README.md
echo "Goodbye, Git!" > goodbye.txt

# Добавить только часть изменений
git add README.md
git commit -m "Update README with installation section"

# Добавить и закоммитить одной командой
git commit -am "Add goodbye.txt"

# Посмотреть историю с изменениями
git log -p
git log --stat
```

**Упражнение:**
1. Измените 3 файла
2. Закоммитьте их по отдельности с описательными сообщениями
3. Используйте `git diff` для просмотра изменений
4. Посмотрите статистику коммитов с `git log --stat`

---

### Задача 3: Отмена изменений
**Цель:** Научиться откатывать изменения на разных этапах

**Практика:**
```bash
# 1. Отменить изменения в рабочей директории (до git add)
echo "Wrong content" > hello.txt
git status
git checkout -- hello.txt
# или в новых версиях Git:
git restore hello.txt

# 2. Убрать файл из staging (после git add)
echo "Some changes" > hello.txt
git add hello.txt
git status
git reset HEAD hello.txt
# или:
git restore --staged hello.txt

# 3. Изменить последний коммит
git commit -m "Wrong message"
echo "Fix" >> hello.txt
git add hello.txt
git commit --amend -m "Correct message with fix"

# 4. Откатить коммит (создать новый коммит с откатом)
git log --oneline
git revert <commit-hash>

# 5. Жесткий откат (ОПАСНО - удаляет изменения)
git reset --hard HEAD~1  # откатить на 1 коммит назад
```

**Упражнение:**
1. Создайте файл с ошибкой и отмените изменения
2. Добавьте файл в staging и уберите его оттуда
3. Сделайте коммит с опечаткой и исправьте через --amend
4. Создайте коммит и откатите его через revert

---

### Задача 4: Работа с ветками (основы)
**Цель:** Научиться создавать и переключаться между ветками

**Практика:**
```bash
# Посмотреть текущую ветку
git branch

# Создать новую ветку
git branch feature/new-feature

# Переключиться на ветку
git checkout feature/new-feature
# или в новых версиях:
git switch feature/new-feature

# Создать и сразу переключиться
git checkout -b feature/another-feature
# или:
git switch -c feature/another-feature

# Посмотреть все ветки
git branch -a

# Сделать изменения в ветке
echo "Feature code" > feature.txt
git add feature.txt
git commit -m "Add feature implementation"

# Вернуться на main
git checkout main
# Заметьте: feature.txt исчез!

# Переключиться обратно на feature
git checkout feature/new-feature
# feature.txt снова здесь!

# Удалить ветку
git branch -d feature/another-feature
# Принудительное удаление:
git branch -D feature/another-feature
```

**Упражнение:**
1. Создайте ветку `feature/login`
2. Добавьте в неё файл `login.js`
3. Создайте ветку `feature/signup`
4. Добавьте в неё файл `signup.js`
5. Переключайтесь между ветками и наблюдайте изменения файлов
6. Вернитесь на main и посмотрите, что там нет новых файлов

---

### Задача 5: Базовое слияние веток
**Цель:** Научиться сливать ветки

**Практика:**
```bash
# Создать и переключиться на feature ветку
git checkout -b feature/header
echo "Header code" > header.js
git add header.js
git commit -m "Add header component"

# Вернуться на main
git checkout main

# Слить feature ветку в main
git merge feature/header

# Посмотреть историю
git log --oneline --graph

# Удалить feature ветку после слияния
git branch -d feature/header

# Fast-forward merge (простое слияние)
git checkout -b feature/footer
echo "Footer code" > footer.js
git add footer.js
git commit -m "Add footer"
git checkout main
git merge feature/footer  # Fast-forward merge

# Merge с созданием merge commit
git checkout -b feature/sidebar
echo "Sidebar" > sidebar.js
git add sidebar.js
git commit -m "Add sidebar"
git checkout main
echo "Main changes" >> README.md
git commit -am "Update README on main"
git merge feature/sidebar --no-ff  # Создаст merge commit
```

**Упражнение:**
1. Создайте 3 feature ветки
2. В каждой добавьте разные файлы
3. Слейте их все в main по очереди
4. Посмотрите историю с `git log --graph --oneline --all`

---

### Задача 6: Работа с удалённым репозиторием
**Цель:** Научиться работать с GitHub/GitLab

**Практика:**
```bash
# Добавить удалённый репозиторий
git remote add origin https://github.com/username/repo.git

# Посмотреть удалённые репозитории
git remote -v

# Отправить изменения
git push -u origin main

# Клонировать существующий репозиторий
git clone https://github.com/username/repo.git
cd repo

# Получить изменения с удалённого репозитория
git fetch origin

# Получить и слить изменения
git pull origin main
# Это эквивалентно:
# git fetch origin
# git merge origin/main

# Отправить ветку на удалённый репозиторий
git checkout -b feature/new
echo "New feature" > feature.txt
git add feature.txt
git commit -m "Add new feature"
git push origin feature/new

# Удалить ветку на удалённом репозитории
git push origin --delete feature/new
```

**Упражнение:**
1. Создайте репозиторий на GitHub
2. Свяжите локальный репозиторий с GitHub
3. Отправьте код на GitHub
4. Создайте новую ветку и отправьте её
5. Клонируйте репозиторий в другую папку
6. Внесите изменения и отправьте их

---

### Задача 7: .gitignore
**Цель:** Научиться игнорировать ненужные файлы

**Практика:**
```bash
# Создать .gitignore файл
cat > .gitignore << EOF
# OS files
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp

# Dependencies
node_modules/
venv/
__pycache__/

# Build files
dist/
build/
*.log

# Environment
.env
.env.local

# Temporary files
*.tmp
*.bak
~$*
EOF

# Добавить в репозиторий
git add .gitignore
git commit -m "Add .gitignore"

# Удалить уже отслеживаемый файл из Git (но оставить локально)
git rm --cached secret.txt
git commit -m "Remove secret.txt from tracking"

# Игнорировать всё кроме определённых файлов
cat > .gitignore << EOF
# Игнорировать всё
*

# Но не игнорировать эти
!.gitignore
!*.js
!*.md
EOF
```

**Упражнение:**
1. Создайте проект Node.js или Python
2. Установите зависимости (появится node_modules или venv)
3. Создайте .gitignore для игнорирования зависимостей
4. Создайте .env файл с секретами и добавьте его в .gitignore
5. Убедитесь, что git status не показывает игнорируемые файлы

---

## 🟡 MIDDLE GIT USER

### Задача 8: Разрешение конфликтов
**Цель:** Научиться разрешать merge conflicts

**Практика:**
```bash
# Создать ситуацию с конфликтом
git checkout -b feature/conflict1
echo "Version 1" > conflict.txt
git add conflict.txt
git commit -m "Add version 1"

git checkout main
echo "Version from main" > conflict.txt
git add conflict.txt
git commit -m "Add version from main"

# Попытка слияния создаст конфликт
git merge feature/conflict1

# Git покажет:
# Auto-merging conflict.txt
# CONFLICT (add/add): Merge conflict in conflict.txt
# Automatic merge failed; fix conflicts and then commit the result.

# Посмотреть конфликтующие файлы
git status

# Открыть файл - увидим:
# <<<<<<< HEAD
# Version from main
# =======
# Version 1
# >>>>>>> feature/conflict1

# Разрешить конфликт (отредактировать файл)
cat > conflict.txt << EOF
Version 1 and main combined
EOF

# Добавить разрешённый файл
git add conflict.txt

# Завершить merge
git commit -m "Merge feature/conflict1 with conflict resolution"

# Посмотреть историю
git log --oneline --graph --all

# Отменить merge если что-то пошло не так
# git merge --abort
```

**Сложное упражнение - 3-way конфликт:**
```bash
# Создать базовую ветку
git checkout -b feature/base
echo "Line 1" > complex.txt
echo "Line 2" >> complex.txt
echo "Line 3" >> complex.txt
git add complex.txt
git commit -m "Add base file"

# Создать две ветки с изменениями
git checkout -b feature/change1
sed -i 's/Line 2/Modified line 2 - version A/' complex.txt
git commit -am "Change line 2 - version A"

git checkout feature/base
git checkout -b feature/change2
sed -i 's/Line 2/Modified line 2 - version B/' complex.txt
git commit -am "Change line 2 - version B"

# Слить в main
git checkout main
git merge feature/change1  # OK
git merge feature/change2  # КОНФЛИКТ!

# Разрешить и закоммитить
```

**Упражнение:**
1. Создайте конфликт в README.md
2. Разрешите его вручную
3. Создайте конфликт в JSON файле
4. Используйте merge tool: `git mergetool`

---

### Задача 9: Rebase
**Цель:** Научиться использовать rebase для линейной истории

**Практика:**
```bash
# Ситуация: feature ветка отстала от main
git checkout main
echo "Main progress" >> main.txt
git add main.txt
git commit -m "Progress on main"

git checkout -b feature/rebase-demo
echo "Feature work" > feature.txt
git add feature.txt
git commit -m "Feature work"

# В это время main ушёл вперёд
git checkout main
echo "More main progress" >> main.txt
git commit -am "More progress on main"

# Вернуться на feature и сделать rebase
git checkout feature/rebase-demo
git rebase main

# История стала линейной!
git log --oneline --graph --all

# Rebase с конфликтами
# Если конфликт:
# 1. Разрешить конфликт
# 2. git add <file>
# 3. git rebase --continue
# Или отменить: git rebase --abort

# Interactive rebase для редактирования истории
git rebase -i HEAD~3

# Откроется редактор с командами:
# pick = использовать коммит
# reword = изменить сообщение коммита
# edit = остановиться для внесения изменений
# squash = объединить с предыдущим коммитом
# fixup = как squash, но отбросить сообщение
# drop = удалить коммит
```

**Пример interactive rebase:**
```bash
# Создать несколько коммитов
echo "A" > file.txt && git add file.txt && git commit -m "Add A"
echo "B" >> file.txt && git commit -am "Add B"
echo "C" >> file.txt && git commit -am "Add C"
echo "D" >> file.txt && git commit -am "Add D"

# Отредактировать последние 4 коммита
git rebase -i HEAD~4

# В редакторе изменить на:
# pick <hash> Add A
# squash <hash> Add B
# reword <hash> Add C
# drop <hash> Add D

# Результат: 2 коммита вместо 4
```

**Упражнение:**
1. Создайте feature ветку
2. Сделайте в ней 5 коммитов
3. Используйте interactive rebase для объединения 3 коммитов в 1
4. Используйте reword для изменения сообщения
5. Используйте drop для удаления ненужного коммита

---

### Задача 10: Cherry-pick
**Цель:** Научиться переносить отдельные коммиты между ветками

**Практика:**
```bash
# Создать feature ветку с несколькими коммитами
git checkout -b feature/multiple
echo "Feature 1" > feature1.txt
git add feature1.txt
git commit -m "Add feature 1"

echo "Feature 2" > feature2.txt
git add feature2.txt
git commit -m "Add feature 2"

echo "Bugfix" > bugfix.txt
git add bugfix.txt
git commit -m "Important bugfix"

# Нужно перенести только bugfix в main
git checkout main
git log feature/multiple --oneline  # Найти hash bugfix коммита
git cherry-pick <bugfix-commit-hash>

# Теперь bugfix в main, но без feature коммитов!

# Cherry-pick нескольких коммитов
git cherry-pick <hash1> <hash2> <hash3>

# Cherry-pick диапазона
git cherry-pick <hash1>..<hash3>

# Cherry-pick с разрешением конфликтов
# Если конфликт:
git status
# Разрешить конфликт
git add <file>
git cherry-pick --continue
# Или отменить:
git cherry-pick --abort
```

**Практический сценарий:**
```bash
# Hotfix который нужен в нескольких ветках
git checkout -b hotfix/critical-bug
echo "Fix critical bug" > fix.txt
git add fix.txt
git commit -m "Fix critical security issue"

# Применить в main
git checkout main
git cherry-pick hotfix/critical-bug

# Применить в release ветку
git checkout release/v1.0
git cherry-pick hotfix/critical-bug

# Применить в develop
git checkout develop
git cherry-pick hotfix/critical-bug
```

**Упражнение:**
1. Создайте ветку с 5 коммитами
2. Cherry-pick только 2-й и 4-й коммит в main
3. Создайте hotfix и примените его в 3 разные ветки
4. Используйте cherry-pick для переноса диапазона коммитов

---

### Задача 11: Stash - временное сохранение
**Цель:** Научиться временно сохранять незавершённые изменения

**Практика:**
```bash
# Начали работу
echo "Work in progress" > wip.txt
git add wip.txt
echo "More changes" >> existing.txt

# Срочно нужно переключиться на другую задачу
git status  # Есть незакоммиченные изменения

# Сохранить изменения в stash
git stash
# или с сообщением:
git stash save "WIP: working on feature X"

# Теперь рабочая директория чистая
git status

# Переключиться и сделать другую работу
git checkout other-branch
# ... делаем срочную работу ...
git checkout main

# Вернуть изменения из stash
git stash pop  # Применить и удалить из stash
# или:
git stash apply  # Применить, но оставить в stash

# Посмотреть список stash
git stash list

# Посмотреть содержимое stash
git stash show
git stash show -p  # С изменениями

# Применить конкретный stash
git stash apply stash@{1}

# Создать ветку из stash
git stash branch feature/from-stash stash@{0}

# Удалить stash
git stash drop stash@{0}
git stash clear  # Удалить все stash
```

**Продвинутые stash техники:**
```bash
# Stash только staged файлов
git stash --keep-index

# Stash включая untracked файлы
git stash --include-untracked
# или короче:
git stash -u

# Stash всего, включая ignored файлы
git stash --all

# Интерактивный stash
git stash --patch
# Спросит для каждого hunk: stash или нет
```

**Упражнение:**
1. Начните работу над фичей
2. Сохраните в stash
3. Переключитесь на hotfix
4. Вернитесь и восстановите из stash
5. Создайте несколько stash и управляйте ими
6. Создайте ветку из stash

---

### Задача 12: Git Tags
**Цель:** Научиться создавать и использовать теги для версий

**Практика:**
```bash
# Легковесный тег (просто указатель)
git tag v1.0

# Аннотированный тег (с сообщением, автором, датой)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Посмотреть все теги
git tag
git tag -l "v1.*"  # Фильтр по паттерну

# Посмотреть информацию о теге
git show v1.0.0

# Создать тег для старого коммита
git tag -a v0.9.0 <commit-hash> -m "Version 0.9.0"

# Отправить теги на удалённый репозиторий
git push origin v1.0.0
git push origin --tags  # Все теги

# Удалить тег локально
git tag -d v1.0.0

# Удалить тег на удалённом репозитории
git push origin --delete v1.0.0

# Checkout тега (detached HEAD)
git checkout v1.0.0

# Создать ветку из тега
git checkout -b hotfix/v1.0.1 v1.0.0
```

**Semantic Versioning теги:**
```bash
# MAJOR.MINOR.PATCH
git tag -a v1.0.0 -m "Initial release"
git tag -a v1.0.1 -m "Patch: bug fixes"
git tag -a v1.1.0 -m "Minor: new features, backward compatible"
git tag -a v2.0.0 -m "Major: breaking changes"

# Pre-release теги
git tag -a v2.0.0-alpha.1 -m "Alpha release"
git tag -a v2.0.0-beta.1 -m "Beta release"
git tag -a v2.0.0-rc.1 -m "Release candidate"

# Build metadata
git tag -a v1.0.0+build.123 -m "Build 123"
```

**Упражнение:**
1. Создайте 5 коммитов
2. Пометьте их тегами v0.1, v0.2, v0.3, v0.4, v0.5
3. Создайте release tag v1.0.0
4. Отправьте теги на GitHub
5. Создайте hotfix ветку из тега v1.0.0

---

### Задача 13: Git Bisect - поиск бага
**Цель:** Найти коммит, который внёс баг, используя бинарный поиск

**Практика:**
```bash
# Подготовка: создать репозиторий с багом
for i in {1..10}; do
  echo "Version $i" > version.txt
  git add version.txt
  if [ $i -eq 7 ]; then
    echo "BUG" >> version.txt  # Баг в коммите 7
  fi
  git commit -m "Commit $i"
done

# Начать bisect
git bisect start

# Отметить текущий коммит как плохой (с багом)
git bisect bad

# Отметить старый коммит как хороший (без бага)
git bisect good HEAD~9

# Git переключится на средний коммит
# Проверить наличие бага
cat version.txt

# Если баг есть:
git bisect bad
# Если бага нет:
git bisect good

# Продолжать пока Git не найдёт первый плохой коммит
# Git покажет: "abc123 is the first bad commit"

# Завершить bisect
git bisect reset

# Автоматический bisect со скриптом
cat > test.sh << 'EOF'
#!/bin/bash
grep -q "BUG" version.txt
exit $?
EOF
chmod +x test.sh

git bisect start HEAD HEAD~9
git bisect run ./test.sh
# Git автоматически найдёт плохой коммит!
```

**Реальный пример с тестами:**
```bash
# Проект с тестами
git bisect start
git bisect bad  # Тесты падают сейчас
git bisect good v1.0.0  # Тесты проходили в v1.0.0

# Автоматический поиск
git bisect run npm test
# или:
git bisect run python -m pytest
# или:
git bisect run make test

# Git найдёт коммит, где тесты начали падать
```

**Упражнение:**
1. Создайте проект с 20 коммитами
2. В случайном коммите введите баг (опечатка в коде)
3. Используйте git bisect для поиска бага
4. Создайте тестовый скрипт и используйте git bisect run
5. Найдите и исправьте баг

---

### Задача 14: Git Reflog - история всех изменений
**Цель:** Восстановить "потерянные" коммиты

**Практика:**
```bash
# Создать коммиты
echo "A" > file.txt && git add file.txt && git commit -m "A"
echo "B" > file.txt && git commit -am "B"
echo "C" > file.txt && git commit -am "C"

# "Потерять" коммиты (hard reset)
git reset --hard HEAD~2

# Коммиты B и C "потеряны"
git log --oneline  # Их нет!

# Но reflog помнит всё
git reflog

# Вывод покажет:
# abc123 HEAD@{0}: reset: moving to HEAD~2
# def456 HEAD@{1}: commit: C
# ghi789 HEAD@{2}: commit: B

# Восстановить коммиты
git reset --hard HEAD@{1}  # Вернуться к коммиту C

# Или использовать hash:
git reset --hard def456

# Посмотреть детальный reflog
git reflog show --all

# Reflog для конкретной ветки
git reflog show main

# Очистка старых reflog записей
git reflog expire --expire=30.days --all
git gc --prune=now
```

**Сценарии восстановления:**
```bash
# Случайно удалили ветку
git branch -D feature/important
# О нет! Вся работа потеряна?

# Найти в reflog
git reflog | grep "feature/important"
# Найдём: checkout: moving from feature/important to main

# Восстановить ветку
git checkout -b feature/important HEAD@{3}
# или:
git branch feature/important <hash>

# Случайный force push
# Коллега сделал force push, перезаписав вашу работу
# Но локальный reflog помнит!
git reflog
git reset --hard HEAD@{5}  # До force push
git push origin main --force-with-lease
```

**Упражнение:**
1. Создайте 5 коммитов
2. Сделайте hard reset на 3 коммита назад
3. Используйте reflog для восстановления
4. Удалите ветку и восстановите её через reflog
5. Экспериментируйте с различными "опасными" операциями, зная что reflog вас спасёт

---

## 🔴 SENIOR GIT USER

### Задача 15: Git Hooks - автоматизация
**Цель:** Настроить автоматические действия при Git событиях

**Pre-commit hook (проверка перед коммитом):**
```bash
# .git/hooks/pre-commit
#!/bin/bash

echo "Running pre-commit checks..."

# Проверка на console.log в JavaScript
if git diff --cached --name-only | grep -E '\.js$' | xargs grep -n 'console\.log' &> /dev/null; then
    echo "❌ Error: console.log statements found"
    echo "Please remove console.log before committing"
    exit 1
fi

# Проверка на TODO комментарии
if git diff --cached | grep -E 'TODO|FIXME' &> /dev/null; then
    echo "⚠️  Warning: TODO/FIXME found in changes"
    read -p "Continue? (y/n) " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        exit 1
    fi
fi

# Запуск линтера
npm run lint --fix
if [ $? -ne 0 ]; then
    echo "❌ Linting failed"
    exit 1
fi

# Запуск тестов
npm test
if [ $? -ne 0 ]; then
    echo "❌ Tests failed"
    exit 1
fi

echo "✅ Pre-commit checks passed"
exit 0
```

```bash
chmod +x .git/hooks/pre-commit
```

**Commit-msg hook (проверка формата сообщения):**
```bash
# .git/hooks/commit-msg
#!/bin/bash

commit_msg_file=$1
commit_msg=$(cat "$commit_msg_file")

# Conventional Commits формат: type(scope): description
# Примеры: feat(api): add user endpoint
#          fix(ui): resolve button alignment

pattern="^(feat|fix|docs|style|refactor|test|chore|perf|ci|build|revert)(\(.+\))?: .{10,}"

if ! echo "$commit_msg" | grep -qE "$pattern"; then
    echo "❌ Invalid commit message format"
    echo ""
    echo "Format: type(scope): description"
    echo ""
    echo "Types:"
    echo "  feat:     New feature"
    echo "  fix:      Bug fix"
    echo "  docs:     Documentation"
    echo "  style:    Code style"
    echo "  refactor: Code refactoring"
    echo "  test:     Tests"
    echo "  chore:    Chores"
    echo "  perf:     Performance"
    echo ""
    echo "Example: feat(auth): add login functionality"
    exit 1
fi

echo "✅ Commit message format is valid"
exit 0
```

```bash
chmod +x .git/hooks/commit-msg
```

**Pre-push hook (проверка перед push):**
```bash
# .git/hooks/pre-push
#!/bin/bash

echo "Running pre-push checks..."

# Проверить что в protected ветках нет коммитов
protected_branch='main'
current_branch=$(git symbolic-ref HEAD | sed -e 's,.*/\(.*\),\1,')

if [ "$current_branch" = "$protected_branch" ]; then
    echo "❌ Direct push to $protected_branch is not allowed"
    echo "Please create a feature branch and use Pull Request"
    exit 1
fi

# Проверить что нет конфликтов с main
git fetch origin main
if ! git merge-base --is-ancestor origin/main HEAD; then
    echo "⚠️  Your branch is behind origin/main"
    echo "Please rebase: git rebase origin/main"
    exit 1
fi

# Запустить полный набор тестов
echo "Running full test suite..."
npm run test:all
if [ $? -ne 0 ]; then
    echo "❌ Tests failed"
    exit 1
fi

echo "✅ Pre-push checks passed"
exit 0
```

**Post-commit hook (уведомления):**
```bash
# .git/hooks/post-commit
#!/bin/bash

commit_msg=$(git log -1 --pretty=%B)
commit_hash=$(git log -1 --pretty=%h)
author=$(git log -1 --pretty=%an)

# Отправить уведомление в Slack
curl -X POST https://hooks.slack.com/services/YOUR/WEBHOOK/URL \
  -H 'Content-Type: application/json' \
  -d "{
    \"text\": \"New commit by $author\",
    \"attachments\": [{
      \"color\": \"good\",
      \"fields\": [{
        \"title\": \"Commit $commit_hash\",
        \"value\": \"$commit_msg\",
        \"short\": false
      }]
    }]
  }"
```

**Упражнение:**
1. Создайте pre-commit hook для запуска линтера
2. Создайте commit-msg hook для проверки формата
3. Создайте pre-push hook для защиты main ветки
4. Попробуйте нарушить каждое правило и убедитесь что hooks работают

---

### Задача 16: Git Submodules
**Цель:** Управлять зависимостями от других Git репозиториев

**Добавление submodule:**
```bash
# Добавить submodule
git submodule add https://github.com/user/library.git libs/library

# Структура:
# project/
# ├── .gitmodules
# └── libs/
#     └── library/

# Просмотр .gitmodules
cat .gitmodules
# [submodule "libs/library"]
#     path = libs/library
#     url = https://github.com/user/library.git

# Закоммитить
git add .gitmodules libs/library
git commit -m "Add library submodule"

# Клонирование проекта с submodules
git clone https://github.com/user/project.git
cd project

# Инициализировать и обновить submodules
git submodule init
git submodule update

# Или одной командой при клонировании
git clone --recurse-submodules https://github.com/user/project.git

# Обновить submodule до последней версии
cd libs/library
git pull origin main
cd ../..
git add libs/library
git commit -m "Update library submodule"

# Обновить все submodules
git submodule update --remote

# Удалить submodule
git submodule deinit libs/library
git rm libs/library
rm -rf .git/modules/libs/library
git commit -m "Remove library submodule"
```

**Работа с submodules:**
```bash
# Foreach команда для всех submodules
git submodule foreach 'git pull origin main'
git submodule foreach 'git checkout develop'

# Статус всех submodules
git submodule status

# Изменения в submodule
cd libs/library
git checkout -b feature/new-feature
echo "changes" >> file.txt
git commit -am "Make changes"
git push origin feature/new-feature
cd ../..

# В основном проекте
git add libs/library
git commit -m "Update library submodule to feature branch"
```

**Альтернатива: Git Subtree:**
```bash
# Добавить subtree (копирует код, не создаёт зависимость)
git subtree add --prefix=libs/library https://github.com/user/library.git main --squash

# Обновить subtree
git subtree pull --prefix=libs/library https://github.com/user/library.git main --squash

# Push изменений обратно в library
git subtree push --prefix=libs/library https://github.com/user/library.git feature/changes
```

**Упражнение:**
1. Создайте главный проект
2. Создайте library репозиторий
3. Добавьте library как submodule
4. Клонируйте проект и инициализируйте submodules
5. Обновите submodule
6. Попробуйте subtree как альтернативу

---

### Задача 17: Git Worktree - множественные рабочие копии
**Цель:** Работать с несколькими ветками одновременно

**Основы worktree:**
```bash
# Текущая структура проекта
# /project (main branch)

# Добавить worktree для другой ветки
git worktree add ../project-feature feature/new-feature

# Теперь структура:
# /project (main branch)
# /project-feature (feature/new-feature branch)

# Можно работать в обеих директориях одновременно!

# Список worktrees
git worktree list

# Вывод:
# /home/user/project         abc123 [main]
# /home/user/project-feature def456 [feature/new-feature]

# Создать worktree с новой веткой
git worktree add -b hotfix/urgent ../project-hotfix

# Удалить worktree
cd /project
git worktree remove ../project-feature

# Или если директория уже удалена
git worktree prune
```

**Практические сценарии:**
```bash
# Сценарий 1: Code review во время разработки
# Работаете над feature, нужно проверить PR
git worktree add ../project-review feature/pr-123
cd ../project-review
# Проверяете код, тестируете
cd ../project
# Продолжаете работу над своей feature

# Сценарий 2: Срочный hotfix во время feature разработки
# Работаете над большой фичей (незакоммиченные изменения)
git worktree add -b hotfix/critical ../project-hotfix main
cd ../project-hotfix
# Делаете hotfix
git commit -am "Fix critical bug"
git push origin hotfix/critical
cd ../project
# Продолжаете работу, изменения не потеряны!

# Сценарий 3: Параллельное тестирование
git worktree add ../project-test1 feature/approach1
git worktree add ../project-test2 feature/approach2
# Тестируете оба подхода параллельно
```

**Продвинутое использование:**
```bash
# Создать worktree из commit
git worktree add ../project-commit abc123

# Создать bare worktree (без рабочей директории)
git worktree add --detach ../project-detached

# Lock worktree (защита от удаления)
git worktree lock ../project-feature
git worktree unlock ../project-feature

# Move worktree
git worktree move ../project-feature ../new-location/project-feature

# Repair worktree (если пути сломаны)
git worktree repair
```

**Упражнение:**
1. Создайте 3 worktree для разных веток
2. Работайте одновременно в разных worktrees
3. Используйте worktree для code review
4. Создайте hotfix worktree во время feature разработки
5. Очистите ненужные worktrees

---

### Задача 18: Git Filter-Branch и BFG - изменение истории
**Цель:** Очистить историю от чувствительных данных или больших файлов

**Удаление файла из истории (filter-branch):**
```bash
# ОПАСНО! Изменяет историю!

# Удалить файл из всей истории
git filter-branch --tree-filter 'rm -f secrets.txt' HEAD

# Удалить директорию
git filter-branch --tree-filter 'rm -rf node_modules' HEAD

# Удалить большие файлы
git filter-branch --tree-filter 'find . -name "*.mp4" -delete' HEAD

# Изменить email автора во всех коммитах
git filter-branch --env-filter '
if [ "$GIT_AUTHOR_EMAIL" = "old@email.com" ]; then
    export GIT_AUTHOR_EMAIL="new@email.com"
    export GIT_COMMITTER_EMAIL="new@email.com"
fi
' HEAD

# После filter-branch нужен force push
git push origin main --force
```

**BFG Repo-Cleaner (быстрее и проще):**
```bash
# Установка BFG
# MacOS: brew install bfg
# Linux: download from https://rtyley.github.io/bfg-repo-cleaner/

# Создать backup!
git clone --mirror git://github.com/user/repo.git

# Удалить файл из истории
bfg --delete-files secrets.txt repo.git

# Удалить все файлы больше 100MB
bfg --strip-blobs-bigger-than 100M repo.git

# Заменить пароли
echo "password123" > passwords.txt
bfg --replace-text passwords.txt repo.git

# Удалить папку из истории
bfg --delete-folders node_modules repo.git

# Применить изменения
cd repo.git
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# Force push
git push --force
```

**Очистка чувствительных данных:**
```bash
# Файл с секретами случайно закоммичен
echo "API_KEY=secret123" > .env
git add .env
git commit -m "Add config"
git push

# О нет! Нужно удалить из истории

# Способ 1: git-filter-repo (современная альтернатива)
pip install git-filter-repo
git filter-repo --path .env --invert-paths

# Способ 2: BFG
bfg --delete-files .env

# Способ 3: Interactive rebase (если недавно)
git rebase -i HEAD~5
# Удалить коммит с секретами

# После любого способа:
git push --force

# ВАЖНО: Поменять все секреты!
# Они в истории GitHub даже после удаления!
```

**Упражнение:**
1. Создайте репозиторий с "секретным" файлом
2. Закоммитьте его в несколько коммитов
3. Используйте BFG для удаления из истории
4. Создайте большой файл и удалите его через filter-branch
5. Попрактикуйтесь в изменении email автора

---

### Задача 19: Git LFS (Large File Storage)
**Цель:** Работать с большими файлами эффективно

**Установка и настройка:**
```bash
# Установить Git LFS
# MacOS: brew install git-lfs
# Ubuntu: apt install git-lfs
# Windows: скачать с git-lfs.github.com

# Инициализировать в репозитории
git lfs install

# Отслеживать типы файлов
git lfs track "*.psd"
git lfs track "*.mp4"
git lfs track "*.zip"

# Или конкретные файлы
git lfs track "large-file.bin"

# Просмотр .gitattributes
cat .gitattributes
# *.psd filter=lfs diff=lfs merge=lfs -text
# *.mp4 filter=lfs diff=lfs merge=lfs -text

# Добавить и закоммитить
git add .gitattributes
git add large-file.mp4
git commit -m "Add large video file"

# Push (файл пойдёт в LFS хранилище)
git push origin main
```

**Работа с LFS:**
```bash
# Посмотреть LFS файлы
git lfs ls-files

# Информация о LFS
git lfs status

# Проверить что файл в LFS
file large-video.mp4
# Вывод будет показывать LFS pointer, не реальный файл

# Просмотр pointer файла
cat large-video.mp4
# version https://git-lfs.github.com/spec/v1
# oid sha256:4665a5ea423900f...
# size 134217728

# Fetch LFS файлы
git lfs fetch

# Pull LFS файлы
git lfs pull

# Клонирование с LFS
git lfs clone https://github.com/user/repo.git

# Просмотр LFS объектов
git lfs ls-files --size

# Удалить старые LFS файлы локально
git lfs prune
```

**Миграция существующих файлов в LFS:**
```bash
# Найти большие файлы в истории
git rev-list --objects --all |
  git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' |
  awk '/^blob/ {print substr($0,6)}' |
  sort --numeric-sort --key=2 |
  tail -20

# Мигрировать в LFS
git lfs migrate import --include="*.psd,*.mp4" --everything

# Или только для конкретных файлов
git lfs migrate import --include="large-file.bin"

# Force push после миграции
git push --force
```

**Управление хранилищем:**
```bash
# Проверить размер LFS объектов
du -sh .git/lfs

# Настроить локальный лимит
git config lfs.fetchrecentalways false
git config lfs.fetchrecentrefsdays 7

# Fetch только нужные ветки
git lfs fetch origin main

# Исключить LFS при клонировании (получить только pointers)
GIT_LFS_SKIP_SMUDGE=1 git clone https://github.com/user/repo.git

# Потом скачать LFS файлы
git lfs pull
```

**Упражнение:**
1. Установите Git LFS
2. Создайте репозиторий с большими файлами
3. Настройте отслеживание типов файлов
4. Закоммитьте и push большие файлы
5. Клонируйте репозиторий и убедитесь что LFS работает
6. Мигрируйте существующие большие файлы в LFS

---

### Задача 20: Advanced Git Config
**Цель:** Настроить Git для максимальной продуктивности

**Глобальные настройки:**
```bash
# Базовые настройки
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Редактор по умолчанию
git config --global core.editor "vim"
git config --global core.editor "code --wait"  # VS Code

# Алиасы
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'

# Продвинутые алиасы
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
git config --global alias.tree "log --graph --oneline --all --decorate"
git config --global alias.contributors "shortlog -s -n"

# Diff и merge инструменты
git config --global merge.tool vimdiff
git config --global diff.tool vimdiff
git config --global difftool.prompt false

# Для VS Code
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'

# Цвета
git config --global color.ui auto
git config --global color.branch.current "yellow reverse"
git config --global color.branch.local yellow
git config --global color.branch.remote green
git config --global color.status.added green
git config --global color.status.changed yellow
git config --global color.status.untracked red

# Push настройки
git config --global push.default current
git config --global push.followTags true

# Pull настройки (rebase вместо merge)
git config --global pull.rebase true

# Автоматическое исправление опечаток
git config --global help.autocorrect 10

# Кеш креденшелов
git config --global credential.helper cache
git config --global credential.helper 'cache --timeout=3600'
# Для MacOS:
git config --global credential.helper osxkeychain
# Для Windows:
git config --global credential.helper wincred
```

**Условные настройки (разные для работы/дома):**
```bash
# ~/.gitconfig
[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work
[includeIf "gitdir:~/personal/"]
    path = ~/.gitconfig-personal

# ~/.gitconfig-work
[user]
    name = "Work Name"
    email = "work@company.com"
[core]
    sshCommand = "ssh -i ~/.ssh/id_rsa_work"

# ~/.gitconfig-personal
[user]
    name = "Personal Name"
    email = "personal@email.com"
[core]
    sshCommand = "ssh -i ~/.ssh/id_rsa_personal"
```

**Полезные настройки производительности:**
```bash
# Оптимизация производительности
git config --global core.preloadindex true
git config --global core.fscache true
git config --global gc.auto 256

# Включить параллельный fetch
git config --global fetch.parallel 0

# Включить rerere (Reuse Recorded Resolution)
git config --global rerere.enabled true

# Настройки whitespace
git config --global core.whitespace trailing-space,space-before-tab
git config --global apply.whitespace fix

# GPG подпись коммитов
git config --global user.signingkey YOUR_GPG_KEY_ID
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

**Проектные настройки (.git/config):**
```bash
# Специфичные для проекта настройки
cd your-project

# Использовать конкретный email для проекта
git config user.email "project-specific@email.com"

# Защита веток
git config branch.main.mergeoptions "--no-ff"

# Автоматический rebase при pull
git config branch.autosetuprebase always

# Защита от force push
git config receive.denyDeletes true
git config receive.denyNonFastForwards true
```

**Упражнение:**
1. Настройте глобальные алиасы для часто используемых команд
2. Настройте условные конфиги для работы и личных проектов
3. Настройте merge/diff инструменты
4. Включите rerere и проверьте его работу
5. Настройте GPG подписи коммитов

---

## 📚 GIT WORKFLOWS

### Задача 21: Git Flow
**Цель:** Организовать работу по Git Flow модели

**Установка Git Flow:**
```bash
# MacOS
brew install git-flow

# Ubuntu
apt-get install git-flow

# Инициализация в проекте
git flow init

# Ответьте на вопросы:
# Production branch: main
# Development branch: develop
# Feature prefix: feature/
# Release prefix: release/
# Hotfix prefix: hotfix/
# Support prefix: support/
# Version tag prefix: v
```

**Работа с features:**
```bash
# Начать новую фичу
git flow feature start user-authentication
# Создаст ветку feature/user-authentication от develop

# Работать над фичей
echo "auth code" > auth.js
git add auth.js
git commit -m "Implement authentication"

# Завершить фичу
git flow feature finish user-authentication
# Сольёт в develop и удалит feature ветку

# Опубликовать фичу (для коллаборации)
git flow feature publish user-authentication

# Получить чужую фичу
git flow feature pull origin user-authentication
```

**Работа с releases:**
```bash
# Начать release
git flow release start 1.0.0
# Создаст ветку release/1.0.0 от develop

# Подготовка к релизу
echo "1.0.0" > VERSION
git commit -am "Bump version to 1.0.0"

# Исправление багов в release
git commit -am "Fix minor bug before release"

# Завершить release
git flow release finish 1.0.0
# Сольёт в main и develop, создаст тег v1.0.0

# Push changes
git push origin main develop --tags
```

**Работа с hotfixes:**
```bash
# Критический баг в production
git flow hotfix start critical-security-fix
# Создаст ветку от main

# Исправить баг
echo "security fix" >> security.js
git commit -am "Fix critical security issue"

# Завершить hotfix
git flow hotfix finish critical-security-fix
# Сольёт в main и develop, создаст тег

# Push
git push origin main develop --tags
```

**Упражнение:**
1. Инициализируйте Git Flow в проекте
2. Создайте 3 feature ветки
3. Создайте release ветку
4. Создайте hotfix для критического бага
5. Визуализируйте историю: `git log --graph --all`

---

### Задача 22: GitHub Flow
**Цель:** Упрощённый workflow для continuous deployment

**Модель:**
```
main (production-ready)
  ↓
feature/new-feature → Pull Request → merge → deploy
```

**Практика:**
```bash
# 1. Создать feature ветку от main
git checkout main
git pull origin main
git checkout -b feature/add-api-endpoint

# 2. Работать над фичей, делать коммиты
git commit -m "feat: add user API endpoint"
git commit -m "test: add API tests"
git commit -m "docs: update API documentation"

# 3. Push ветку
git push -u origin feature/add-api-endpoint

# 4. Создать Pull Request на GitHub
# Зайти на GitHub и создать PR

# 5. Code review, обсуждение, изменения
git commit -m "fix: address review comments"
git push

# 6. После approval - merge в main
# На GitHub: нажать "Merge pull request"

# 7. Удалить feature ветку
git branch -d feature/add-api-endpoint
git push origin --delete feature/add-api-endpoint

# 8. main автоматически деплоится в production
```

**GitHub Actions для автодеплоя:**
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
      
      - name: Deploy
        run: ./deploy.sh
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
```

**Упражнение:**
1. Создайте репозиторий на GitHub
2. Настройте branch protection для main
3. Создайте feature ветку
4. Сделайте Pull Request
5. Настройте GitHub Actions для CI/CD
6. Слейте PR и наблюдайте автодеплой

---

### Задача 23: Trunk-Based Development
**Цель:** Continuous integration с короткоживущими ветками

**Принципы:**
- Все коммитят в main (или в короткоживущие feature ветки)
- Feature flags для незавершённых фич
- Частые коммиты (несколько раз в день)
- Быстрые code reviews

**Практика:**
```bash
# Короткоживущая ветка (живёт < 1 дня)
git checkout -b quick-fix
# Работаем 2-3 часа
git commit -m "fix: resolve issue"
git push origin quick-fix
# Сразу создаём PR и мержим

# Незавершённая фича? Feature flag!
git checkout main
echo "export const NEW_FEATURE_ENABLED = false;" > feature-flags.js
git commit -am "feat: add new feature (disabled)"
git push

# Постепенно включаем
git commit -am "feat: enable new feature for 10% users"
# NEW_FEATURE_ENABLED = Math.random() < 0.1

git commit -am "feat: enable new feature for 50% users"
# NEW_FEATURE_ENABLED = Math.random() < 0.5

git commit -am "feat: enable new feature for all users"
# NEW_FEATURE_ENABLED = true
```

**Feature flags с библиотекой:**
```javascript
// feature-flags.js
const flags = {
  newDashboard: {
    enabled: process.env.NEW_DASHBOARD === 'true',
    rollout: 0.1  // 10% пользователей
  },
  apiV2: {
    enabled: true
  }
};

export function isFeatureEnabled(name, userId) {
  const flag = flags[name];
  if (!flag.enabled) return false;
  if (flag.rollout === undefined) return true;
  
  // Consistent hash для пользователя
  const hash = simpleHash(userId);
  return (hash % 100) < (flag.rollout * 100);
}

// Использование
if (isFeatureEnabled('newDashboard', user.id)) {
  return <NewDashboard />;
} else {
  return <OldDashboard />;
}
```

**Упражнение:**
1. Настройте проект для Trunk-Based Development
2. Делайте частые коммиты в main
3. Используйте feature flags для незавершённых фич
4. Настройте автоматический CI/CD
5. Практикуйте быстрые code reviews

---

## 🔧 TROUBLESHOOTING

### Задача 24: Исправление распространённых проблем

**Проблема: "Detached HEAD state"**
```bash
# Вы в detached HEAD (не на ветке)
git status
# HEAD detached at abc123

# Решение 1: Создать ветку
git checkout -b recovery-branch

# Решение 2: Вернуться на ветку
git checkout main

# Решение 3: Сохранить работу и вернуться
git branch temp-save
git checkout main
```

**Проблема: Случайный commit в wrong branch**
```bash
# Закоммитили в main вместо feature
git log --oneline  # Видим коммит

# Решение:
# 1. Создать ветку с коммитом
git branch feature/correct-branch

# 2. Удалить коммит из main
git reset --hard HEAD~1

# 3. Переключиться на правильную ветку
git checkout feature/correct-branch
```

**Проблема: Merge conflict**
```bash
# Во время merge возник конфликт
git merge feature/branch
# CONFLICT (content): Merge conflict in file.txt

# Решение:
# 1. Посмотреть конфликтующие файлы
git status

# 2. Открыть файл и разрешить
# <<<<<<< HEAD
# Current content
# =======
# Incoming content
# >>>>>>> feature/branch

# 3. Выбрать нужную версию или объединить
# 4. Удалить маркеры конфликта
# 5. Добавить и закоммитить
git add file.txt
git commit

# Или отменить merge
git merge --abort
```

**Проблема: Нужно изменить старый коммит**
```bash
# Ошибка в коммите 5 коммитов назад
git log --oneline

# Решение через interactive rebase
git rebase -i HEAD~5

# В редакторе заменить 'pick' на 'edit' для нужного коммита
# Внести изменения
git add file.txt
git rebase --continue
```

**Проблема: Pushed wrong commit**
```bash
# Отправили коммит с ошибкой
git push origin main

# Решение 1: Revert (безопасно)
git revert HEAD
git push origin main

# Решение 2: Force push (ОПАСНО, если работают другие)
git reset --hard HEAD~1
git push origin main --force-with-lease
```

**Проблема: Lost commits после reset**
```bash
# Сделали hard reset и "потеряли" коммиты
git reset --hard HEAD~5

# Решение через reflog
git reflog
# Найти hash нужного коммита
git reset --hard abc123

# Или создать ветку из потерянного коммита
git branch recovery abc123
```

**Проблема: Огромный .git директория**
```bash
# .git занимает много места

# Решение 1: Очистить историю
git gc --aggressive --prune=now

# Решение 2: Найти большие файлы
git rev-list --objects --all |
  git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' |
  sed -n 's/^blob //p' |
  sort --numeric-sort --key=2 |
  cut -c 1-12,41- |
  tail -20

# Решение 3: Удалить из истории
git filter-branch --tree-filter 'rm -f large-file.bin' HEAD

# Решение 4: Shallow clone (для клонирования)
git clone --depth 1 https://github.com/user/repo.git
```

**Проблема: Cannot push - rejected**
```bash
# Отказано в push
git push origin main
# ! [rejected] main -> main (fetch first)

# Решение 1: Pull и merge
git pull origin main
git push origin main

# Решение 2: Rebase
git pull --rebase origin main
git push origin main

# Решение 3: Force push (осторожно!)
git push origin main --force-with-lease
```

**Проблема: Accidentally committed sensitive data**
```bash
# Закоммитили .env с паролями
git add .env
git commit -m "Add config"
git push

# СРОЧНОЕ решение:
# 1. Удалить из последнего коммита
git rm --cached .env
echo ".env" >> .gitignore
git commit --amend -m "Add config (without .env)"
git push --force-with-lease

# 2. Удалить из истории
git filter-repo --path .env --invert-paths
git push --force

# 3. ВАЖНО: Сменить все пароли!
# Они могут быть в кеше GitHub!
```

---

## 🎯 ПРОДВИНУТЫЕ ТЕХНИКИ

### Задача 25: Git Attributes
**Цель:** Настроить поведение Git для разных типов файлов

**.gitattributes файл:**
```bash
# Создать .gitattributes
cat > .gitattributes << 'EOF'
# Auto detect text files and normalize line endings
* text=auto

# Explicitly declare text files
*.txt text
*.md text
*.js text
*.jsx text
*.ts text
*.tsx text
*.json text
*.yml text
*.yaml text
*.xml text
*.html text
*.css text
*.scss text

# Declare files that will always have LF line endings
*.sh text eol=lf
Makefile text eol=lf

# Declare files that will always have CRLF line endings
*.bat text eol=crlf
*.ps1 text eol=crlf

# Binary files
*.png binary
*.jpg binary
*.gif binary
*.ico binary
*.pdf binary
*.zip binary
*.tar.gz binary

# Archives
*.7z binary
*.gz binary
*.tar binary
*.zip binary

# Fonts
*.ttf binary
*.woff binary
*.woff2 binary

# Diff для специфичных файлов
*.ipynb diff=jupyternotebook
*.pdf diff=pdf
*.docx diff=docx

# Merge strategy
*.json merge=union
package-lock.json merge=ours

# Export-ignore (не включать в git archive)
.gitattributes export-ignore
.gitignore export-ignore
.github/ export-ignore
tests/ export-ignore
docs/ export-ignore
*.test.js export-ignore

# LFS
*.psd filter=lfs diff=lfs merge=lfs -text
*.ai filter=lfs diff=lfs merge=lfs -text
*.mp4 filter=lfs diff=lfs merge=lfs -text
*.zip filter=lfs diff=lfs merge=lfs -text

# Custom diff driver
*.sql diff=sql
EOF
```

**Настройка custom diff drivers:**
```bash
# В .gitconfig
git config --global diff.jupyternotebook.command 'git-nbdiffdriver diff'
git config --global diff.pdf.binary true
git config --global diff.pdf.textconv pdftotext

# Для SQL красивого diff
git config --global diff.sql.xfuncname '^(CREATE|ALTER|DROP).*'

# Для minified JS
git config --global diff.minjs.textconv js-beautify
```

**Упражнение:**
1. Создайте .gitattributes для вашего проекта
2. Настройте line endings для разных файлов
3. Настройте LFS для больших файлов
4. Добавьте export-ignore для тестовых файлов
5. Настройте custom diff для специфичных файлов

---

### Задача 26: Git Bundle - offline репозиторий
**Цель:** Передать репозиторий без сети

**Создание bundle:**
```bash
# Создать bundle всей истории
git bundle create repo.bundle --all

# Bundle определённой ветки
git bundle create main.bundle main

# Bundle диапазона коммитов
git bundle create changes.bundle main ^origin/main
# Все коммиты в main, которых нет в origin/main

# Bundle с тегами
git bundle create release.bundle --tags main

# Проверить bundle
git bundle verify repo.bundle

# Посмотреть содержимое
git bundle list-heads repo.bundle
```

**Использование bundle:**
```bash
# Клонировать из bundle
git clone repo.bundle my-repo
cd my-repo

# Добавить bundle как remote
git remote add bundle ../repo.bundle

# Fetch из bundle
git fetch bundle

# Pull из bundle
git pull bundle main

# Создать инкрементальный bundle
# На компьютере A:
git bundle create initial.bundle --all
# Копируем на компьютер B, потом делаем изменения на A:
git bundle create updates.bundle main ^abc123
# Где abc123 - последний коммит из initial.bundle

# На компьютере B:
git fetch ../updates.bundle main:main
```

**Практический сценарий:**
```bash
# Разработка без интернета
# День 1 (дома, есть интернет)
git clone https://github.com/user/repo.git
cd repo
git bundle create day1.bundle --all
# Копируем bundle на флешку

# День 2 (без интернета)
git clone day1.bundle offline-work
cd offline-work
# Работаем
git commit -am "Work without internet"

# День 3 (снова дома)
git bundle create day2-changes.bundle main ^origin/main
# В основном репозитории:
cd ../repo
git pull ../offline-work/day2-changes.bundle main
git push origin main
```

**Упражнение:**
1. Создайте bundle вашего репозитория
2. Клонируйте проект из bundle
3. Создайте инкрементальный bundle с новыми изменениями
4. Передайте bundle "другому разработчику" (другая папка)
5. Синхронизируйте изменения через bundles

---

### Задача 27: Git Sparse Checkout
**Цель:** Клонировать только часть репозитория

**Partial Clone (Git 2.25+):**
```bash
# Клонировать без файлов
git clone --filter=blob:none --no-checkout https://github.com/user/repo.git
cd repo

# Выбрать нужные директории
git sparse-checkout init --cone
git sparse-checkout set src/frontend

# Добавить ещё директории
git sparse-checkout add docs tests

# Посмотреть список
git sparse-checkout list

# Checkout файлов
git checkout main

# Теперь у вас только src/frontend, docs и tests
```

**Старый способ (до Git 2.25):**
```bash
# Создать пустой репозиторий
git init repo
cd repo

# Добавить remote
git remote add origin https://github.com/user/repo.git

# Включить sparse checkout
git config core.sparseCheckout true

# Указать что скачивать
cat > .git/info/sparse-checkout << EOF
src/frontend/*
docs/*
!docs/internal/*
EOF

# Скачать
git pull origin main
```

**Практические примеры:**
```bash
# Только определённый микросервис из монорепо
git sparse-checkout set services/user-service

# Несколько микросервисов
git sparse-checkout set services/user-service services/api-gateway shared

# С исключениями
git sparse-checkout set src '!src/legacy' '!src/deprecated'

# Вернуться к полному checkout
git sparse-checkout disable
```

**Упражнение:**
1. Клонируйте большой репозиторий с sparse checkout
2. Получите только нужные директории
3. Добавьте и удалите директории из sparse checkout
4. Сравните размер с полным клоном
5. Вернитесь к полному checkout

---

### Задача 28: Git Notes
**Цель:** Добавить метаданные к коммитам без изменения истории

**Основы Git Notes:**
```bash
# Добавить заметку к последнему коммиту
git notes add -m "Reviewed by: John Doe"

# Добавить заметку к конкретному коммиту
git notes add -m "Fixed bug #123" abc123

# Посмотреть заметки
git log --show-notes
git notes show abc123

# Редактировать заметку
git notes edit abc123

# Удалить заметку
git notes remove abc123

# Список всех заметок
git notes list
```

**Разные namespaces для notes:**
```bash
# Создать заметку в namespace
git notes --ref=bugs add -m "Bug #456" abc123
git notes --ref=reviews add -m "LGTM" abc123

# Посмотреть заметки из namespace
git log --show-notes=bugs
git log --show-notes=reviews

# Показать все namespaces
git log --show-notes=*
```

**Синхронизация notes:**
```bash
# Push notes
git push origin refs/notes/*

# Fetch notes
git fetch origin refs/notes/*:refs/notes/*

# Автоматический fetch notes
git config remote.origin.fetch '+refs/notes/*:refs/notes/*'
```

**Практическое использование:**
```bash
# Code review заметки
git notes --ref=review add -m "Approved by: Alice"
git notes --ref=review add -m "Security review: Bob"

# Bug tracking
git notes --ref=bugs add -m "Fixes: #123, #456"

# Deployment tracking
git notes --ref=deploy add -m "Deployed to production: 2024-01-15"

# Performance metrics
git notes --ref=perf add -m "Build time: 45s, Test time: 2m"

# Просмотр всех метаданных
git log --pretty=format:"%h %s" --show-notes=review --show-notes=bugs
```

**Упражнение:**
1. Добавьте review заметки к коммитам
2. Создайте разные namespaces для разных типов заметок
3. Настройте автоматическую синхронизацию notes
4. Используйте notes для tracking deployment
5. Создайте скрипт для автоматического добавления notes

---

## 📊 GIT ANALYTICS

### Задача 29: Анализ репозитория
**Цель:** Получить статистику и insights из Git истории

**Базовая статистика:**
```bash
# Количество коммитов
git rev-list --count HEAD

# Количество коммитов по авторам
git shortlog -s -n

# Количество добавленных/удалённых строк
git log --shortstat

# Статистика по авторам с строками
git log --author="John" --pretty=tformat: --numstat | \
  awk '{ add += $1; subs += $2; loc += $1 - $2 } END \
  { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }'

# Активность по дням недели
git log --date=format:%A --pretty=format:%ad | sort | uniq -c

# Активность по часам
git log --date=format:%H --pretty=format:%ad | sort | uniq -c

# Топ файлов по количеству изменений
git log --pretty=format: --name-only | sort | uniq -c | sort -rg | head -20
```

**Продвинутая аналитика:**
```bash
# Скрипт для детальной статистики
cat > git-stats.sh << 'EOF'
#!/bin/bash

echo "=== Git Repository Statistics ==="
echo ""

echo "Total Commits: $(git rev-list --count HEAD)"
echo "Total Contributors: $(git shortlog -s -n | wc -l)"
echo "Total Files: $(git ls-files | wc -l)"
echo ""

echo "=== Top 10 Contributors ==="
git shortlog -s -n | head -10
echo ""

echo "=== File Type Distribution ==="
git ls-files | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -10
echo ""

echo "=== Largest Files ==="
git ls-files | xargs wc -l 2>/dev/null | sort -rn | head -10
echo ""

echo "=== Commit Activity by Month (last year) ==="
git log --since="1 year ago" --pretty=format:"%ai" | \
  cut -d'-' -f1-2 | sort | uniq -c
echo ""

echo "=== Most Modified Files ==="
git log --all --pretty=format: --name-only | \
  sort | uniq -c | sort -rn | head -10
echo ""

echo "=== Code Churn (files changed frequently) ==="
git log --all --pretty=format: --name-only | \
  sort | uniq -c | sort -rn | \
  awk '$1 > 10 {print $1, $2}' | head -20
EOF

chmod +x git-stats.sh
./git-stats.sh
```

**Визуализация истории:**
```bash
# ASCII график активности
git log --all --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short

# Красивый граф с цветами
git log --graph --abbrev-commit --decorate \
  --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' \
  --all

# Создать граф в DOT формате
git log --graph --all --format='%h %p' | \
  sed 's/\([0-9a-f]\+\)/"\1"/g' | \
  sed 's/"[^"]*" //; s/ /" -> "/g; s/$/"/' > graph.dot
dot -Tpng graph.dot -o graph.png
```

**Анализ качества кода:**
```bash
# Найти большие коммиты (возможно нужен рефакторинг)
git log --all --pretty=format:'%h %an %s' --shortstat | \
  awk '/^[0-9a-f]/ {print msg} /files? changed/ {msg=$0}'| \
  sort -rn -k4 | head -10

# Найти коммиты с большим количеством файлов
git log --all --pretty=format: --name-only --diff-filter=A | \
  awk 'NF{files++} !NF{print files; files=0}' | \
  sort -rn | head -20

# Hotspots - файлы требующие внимания
# (много изменений + много багфиксов = проблемная область)
git log --all --no-merges --pretty=format: --name-only | \
  sort | uniq -c | sort -rn | head -20

# Bugfix коммиты
git log --all --grep="fix\|bug" --pretty=format:"%h %s" | wc -l
```

**Упражнение:**
1. Создайте статистику вашего репозитория
2. Найдите самых активных контрибьюторов
3. Визуализируйте историю коммитов
4. Найдите hotspots в коде
5. Создайте dashboard с метриками

---

### Задача 30: Git для DevOps
**Цель:** Интеграция Git в CI/CD и автоматизацию

**Git в CI/CD pipeline:**
```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0  # Полная история для анализа
      
      - name: Get changed files
        id: changed-files
        uses: tj-actions/changed-files@v35
      
      - name: Run tests only for changed files
        run: |
          for file in ${{ steps.changed-files.outputs.all_changed_files }}; do
            if [[ $file == *.js ]]; then
              npm test $file
            fi
          done
      
      - name: Check commit message format
        run: |
          commit_msg=$(git log -1 --pretty=%B)
          if ! echo "$commit_msg" | grep -qE "^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+"; then
            echo "Invalid commit message format"
            exit 1
          fi
      
      - name: Generate changelog
        if: github.ref == 'refs/heads/main'
        run: |
          git log --pretty=format:"- %s (%h)" $(git describe --tags --abbrev=0)..HEAD > CHANGELOG.md
      
      - name: Tag version
        if: github.ref == 'refs/heads/main'
        run: |
          version=$(node -p "require('./package.json').version")
          git tag -a "v$version" -m "Release $version"
          git push origin "v$version"
```

**Автоматическая версионность:**
```bash
# semantic-release setup
cat > .releaserc.json << 'EOF'
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/npm",
    "@semantic-release/git",
    "@semantic-release/github"
  ]
}
EOF

# Commit types для версионности:
# feat: minor version bump (1.0.0 -> 1.1.0)
# fix: patch version bump (1.0.0 -> 1.0.1)
# BREAKING CHANGE: major version bump (1.0.0 -> 2.0.0)
```

**Git hooks для deployment:**
```bash
# .git/hooks/post-receive (на сервере)
#!/bin/bash

while read oldrev newrev ref
do
    branch=$(git rev-parse --symbolic --abbrev-ref $ref)
    
    if [ "$branch" = "main" ]; then
        echo "Deploying to production..."
        
        # Проверить что тесты проходят
        git checkout main
        npm test || exit 1
        
        # Build
        npm run build
        
        # Deploy
        pm2 restart app
        
        # Отправить уведомление
        curl -X POST https://hooks.slack.com/... \
          -d '{"text":"Deployed to production"}'
    fi
done
```

**Git-based configuration management:**
```bash
# Использование Git для хранения конфигов
# configs/
# ├── production/
# │   ├── app.yml
# │   └── secrets.enc
# ├── staging/
# │   └── app.yml
# └── development/
#     └── app.yml

# Скрипт деплоя читает конфиг из Git
git clone configs-repo /tmp/configs
env=$(cat /etc/environment)
cp /tmp/configs/$env/* /app/config/

# Используйте git-crypt для секретов
git-crypt init
git-crypt add-gpg-user user@example.com
echo "secrets.enc filter=git-crypt diff=git-crypt" >> .gitattributes
```

**Упражнение:**
1. Настройте CI/CD с GitHub Actions
2. Добавьте автоматическую версионность
3. Создайте deployment hooks
4. Настройте автоматическую генерацию changelog
5. Используйте Git для управления конфигами

---

## 🎓 МАСТЕР-КЛАСС

### Задача 31: Создание Git алиасов-суперсил
**Цель:** Создать мощные алиасы для повседневной работы

**Полезные алиасы:**
```bash
# В ~/.gitconfig добавьте:
[alias]
    # Красивые логи
    lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
    
    # Статус в кратком формате
    st = status -sb
    
    # Быстрый commit
    c = commit -m
    ca = commit -am
    
    # Ветки
    br = branch
    co = checkout
    cob = checkout -b
    
    # Отмена изменений
    undo = reset HEAD~1 --soft
    amend = commit --amend --no-edit
    
    # Поиск
    find = "!git ls-files | grep -i"
    grep = grep -Ii
    
    # Aliases для работы с remote
    pushf = push --force-with-lease
    pushu = "!git push -u origin $(git branch --show-current)"
    
    # Очистка
    cleanup = "!git branch --merged | grep -v '\\*\\|main\\|develop' | xargs -n 1 git branch -d"
    
    # Статистика
    stats = shortlog -s -n
    contrib = shortlog -s -n --all --no-merges
    
    # Последние N коммитов
    last = "!f() { git log -${1:-5} --oneline; }; f"
    
    # Изменения с main
    new = !git log main..HEAD --oneline
    
    # Показать игнорируемые файлы
    ignored = !git ls-files -v | grep "^[[:lower:]]"
    
    # WIP (Work In Progress) - быстрое сохранение
    wip = !git add -A && git commit -m "WIP"
    unwip = reset HEAD~1
    
    # Aliases с параметрами
    whoami = "!git config user.name && git config user.email"
    authors = "!git log --format='%aN <%aE>' | sort -u"
    
    # Сложные алиасы
    sync = "!f() { \
        git fetch --all --prune && \
        git pull --rebase origin $(git branch --show-current) && \
        git submodule update --init --recursive; \
    }; f"
    
    # Создать PR (требует GitHub CLI)
    pr = "!gh pr create"
    
    # Поиск коммита по сообщению
    search = "!f() { git log --all --grep=$1; }; f"
```

**Продвинутые алиасы:**
```bash
[alias]
    # Интерактивный rebase с автоматическим подсчётом
    reb = "!f() { git rebase -i HEAD~${1:-10}; }; f"
    
    # Слияние с автоматическим squash
    smerge = merge --squash
    
    # Показать файлы в конфликте
    conflicts = diff --name-only --diff-filter=U
    
    # Создать архив текущей ветки
    archive-branch = "!f() { \
        git archive -o $(git branch --show-current).zip HEAD; \
    }; f"
    
    # Показать размер репозитория
    repo-size = !git gc && du -sh .git
    
    # Найти когда файл был удалён
    find-deleted = "!f() { \
        git log --all --full-history -- $1; \
    }; f"
    
    # Показать коммиты не запушенные
    unpushed = log @{u}..
    
    # Показать коммиты не смерженные в main
    unmerged = log main..HEAD --oneline
    
    # Backup текущей ветки
    backup = "!f() { \
        branch=$(git branch --show-current); \
        git branch backup/$branch-$(date +%Y%m%d-%H%M%S); \
    }; f"
```

**Упражнение:**
1. Добавьте все полезные алиасы в ваш .gitconfig
2. Создайте свои кастомные алиасы
3. Протестируйте каждый алиас
4. Создайте документацию ваших алиасов
5. Поделитесь алиасами с командой

---

## 📖 ШПАРГАЛКА

### Быстрый справочник команд

**Базовые команды:**
```bash
git init                    # Инициализировать репозиторий
git clone <url>             # Клонировать репозиторий
git add <file>              # Добавить файл в staging
git commit -m "message"     # Создать коммит
git push                    # Отправить на remote
git pull                    # Получить изменения
git status                  # Статус изменений
git log                     # История коммитов
```

**Ветки:**
```bash
git branch                  # Список веток
git branch <name>           # Создать ветку
git checkout <branch>       # Переключиться на ветку
git checkout -b <branch>    # Создать и переключиться
git merge <branch>          # Слить ветку
git branch -d <branch>      # Удалить ветку
```

**Откат изменений:**
```bash
git reset HEAD <file>       # Убрать из staging
git checkout -- <file>      # Отменить изменения
git reset --soft HEAD~1     # Откатить коммит, сохранить изменения
git reset --hard HEAD~1     # Откатить коммит, удалить изменения
git revert <commit>         # Создать коммит-откат
```

**Remote:**
```bash
git remote add origin <url> # Добавить remote
git remote -v               # Список remotes
git fetch                   # Получить изменения
git pull origin main        # Pull из конкретной ветки
git push origin main        # Push в конкретную ветку
git push -u origin main     # Push и установить upstream
```

**Продвинутые:**
```bash
git stash                   # Сохранить изменения
git stash pop               # Восстановить изменения
git rebase <branch>         # Rebase на ветку
git cherry-pick <commit>    # Применить коммит
git reflog                  # История HEAD
git bisect                  # Бинарный поиск бага
```

---

## ✅ Чеклист навыков

### Junior Git User
- [ ] Создаю и клонирую репозитории
- [ ] Делаю коммиты с хорошими сообщениями
- [ ] Работаю с ветками (create, checkout, merge)
- [ ] Использую .gitignore
- [ ] Понимаю git status и git log
- [ ] Могу откатить изменения
- [ ] Работаю с remote репозиториями
- [ ] Создаю Pull Requests

### Middle Git User
- [ ] Разрешаю merge conflicts
- [ ] Использую rebase
- [ ] Применяю cherry-pick
- [ ] Работаю с stash
- [ ] Создаю и использую теги
- [ ] Использую git bisect для поиска багов
- [ ] Понимаю reflog и могу восстанавливать коммиты
- [ ] Настраиваю Git hooks
- [ ] Работаю с submodules
- [ ] Использую worktrees

### Senior Git User
- [ ] Изменяю историю (filter-branch, BFG)
- [ ] Настраиваю advanced Git конфиги
- [ ] Использую Git LFS
- [ ] Работаю с Git attributes
- [ ] Создаю Git bundles
- [ ] Использую sparse checkout
- [ ] Применяю Git notes
- [ ] Анализирую репозитории
- [ ] Интегрирую Git в CI/CD
- [ ] Обучаю других Git практикам

---

## 🎯 Практические проекты

### Проект 1: Personal Git Server
Настройте свой Git сервер на VPS

### Проект 2: Automated Changelog Generator
Создайте скрипт автогенерации changelog из коммитов

### Проект 3: Git Analytics Dashboard
Создайте dashboard с метриками репозитория

### Проект 4: Custom Git Workflow
Разработайте и внедрите workflow для вашей команды

### Проект 5: Git-based Wiki
Создайте wiki систему на основе Git

---

## 📚 Дополнительные ресурсы

### Книги
- "Pro Git" by Scott Chacon (бесплатно на git-scm.com)
- "Git Pocket Guide" by Richard Silverman