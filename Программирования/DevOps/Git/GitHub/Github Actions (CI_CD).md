
# ⚡ GitHub Actions — базовые команды и объяснения

> Все workflow-файлы хранятся в папке:  
> `.github/workflows/имя.yml`

---

## 🔹 1. Триггеры (события, которые запускают пайплайн)

```yaml
on: [push]
```

▶ Запускает пайплайн при каждом `git push`.

```yaml
on: [pull_request]
```

▶ Запускает пайплайн при создании Pull Request.

```yaml
on:
  schedule:
    - cron: "0 0 * * *"
```

▶ Запускает пайплайн по расписанию (здесь — каждый день в 00:00).

```yaml
on:
  workflow_dispatch:
```

▶ Позволяет запускать workflow вручную из интерфейса GitHub.

---

## 🔹 2. Jobs (работы)

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
```

▶ Определяет задачу (`build`) и указывает среду выполнения:

- `ubuntu-latest`
    
- `windows-latest`
    
- `macos-latest`
    

---

## 🔹 3. Steps (шаги внутри job)

### 3.1. Скачивание кода

```yaml
- name: Checkout code
  uses: actions/checkout@v3
```

▶ Загружает исходный код репозитория в runner.

---

### 3.2. Настройка Python

```yaml
- name: Set up Python
  uses: actions/setup-python@v4
  with:
    python-version: '3.11'
```

▶ Устанавливает нужную версию Python.

---

### 3.3. Настройка Node.js

```yaml
- name: Set up Node.js
  uses: actions/setup-node@v3
  with:
    node-version: '18'
```

▶ Устанавливает Node.js указанной версии.

---

### 3.4. Установка зависимостей

Python:

```yaml
- name: Install dependencies
  run: pip install -r requirements.txt
```

▶ Устанавливает зависимости Python из `requirements.txt`.

Node.js:

```yaml
- name: Install dependencies
  run: npm install
```

▶ Устанавливает зависимости Node.js из `package.json`.

---

### 3.5. Запуск тестов

Python:

```yaml
- name: Run tests
  run: pytest
```

▶ Запускает тесты с помощью `pytest`.

Node.js:

```yaml
- name: Run tests
  run: npm test
```

▶ Запускает тесты Node.js.

---

### 3.6. Сборка проекта

```yaml
- name: Build project
  run: npm run build
```

▶ Собирает проект (например, React, Vue, Next.js).

---

### 3.7. Деплой (пример с Docker)

```yaml
- name: Deploy with Docker
  run: docker build -t myapp . && docker push myapp
```

▶ Собирает Docker-образ и отправляет его в DockerHub.

---

## 🔹 4. Полный минимальный CI/CD пример (Node.js)

```yaml
name: CI Pipeline

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test

      - name: Build project
        run: npm run build
```
