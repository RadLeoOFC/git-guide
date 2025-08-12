# Установка и настройка Git

## Установка на Windows, macOS, Linux

### Windows
1. Перейдите на официальный сайт: [https://git-scm.com](https://git-scm.com)
2. Скачайте установщик для Windows.
3. Запустите установку и следуйте инструкциям (оставьте настройки по умолчанию, если не уверены).
4. После установки проверьте:
   ```bash
   git --version
   ```

*Совет:* В установщике есть опция **Git Bash** — это удобный терминал с поддержкой Git-команд.

---

### macOS
1. Откройте терминал.
2. Установите Git через Homebrew:
   ```bash
   brew install git
   ```
   Или через Xcode Command Line Tools:
   ```bash
   xcode-select --install
   ```
3. Проверьте версию:
   ```bash
   git --version
   ```

---

### Linux
Для разных дистрибутивов — разные команды:

**Ubuntu/Debian**
```bash
sudo apt update
sudo apt install git
```

**Fedora**
```bash
sudo dnf install git
```

**Arch Linux**
```bash
sudo pacman -S git
```

Проверка версии:
```bash
git --version
```

---

## Настройка имени и email

Git использует имя и email для подписи ваших коммитов.

```bash
git config --global user.name "Ваше Имя"
git config --global user.email "you@example.com"
```

Проверить настройки:
```bash
git config --list
```

---

## Настройка SSH-ключей

SSH-ключи позволяют подключаться к GitHub/GitLab без ввода пароля.

1. **Проверяем наличие ключей**
   ```bash
   ls -al ~/.ssh
   ```
   Если есть файлы `id_rsa` и `id_rsa.pub` — ключ уже создан.

2. **Создаём новый ключ**
   ```bash
   ssh-keygen -t rsa -b 4096 -C "you@example.com"
   ```
   Нажмите Enter для сохранения в стандартную папку, при желании задайте пароль.

3. **Добавляем ключ в ssh-agent**
   ```bash
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_rsa
   ```

4. **Добавляем ключ на GitHub**
   - Скопируйте содержимое файла:
     ```bash
     cat ~/.ssh/id_rsa.pub
     ```
   - Перейдите в [GitHub → Settings → SSH and GPG keys](https://github.com/settings/keys)  
     Нажмите **New SSH key** и вставьте скопированный ключ.

5. **Проверяем подключение**
   ```bash
   ssh -T git@github.com
   ```
   Должно появиться сообщение:
   ```
   Hi username! You've successfully authenticated...
   ```

---

## Краткий вывод
- Установите Git в зависимости от вашей ОС.
- Настройте имя и email — без этого Git будет ругаться.
- Для удобной работы с GitHub/GitLab используйте SSH-ключи.

---

➡️ [Следующий раздел: Основные команды Git](03-basic-commands.md)
