# Домашнее задание "Микросервисы: подходы"
# Решение для обеспечения непрерывной интеграции и поставки (CI/CD)

## Заголовок задачи

**Задача:** Разработать архитектуру процесса разработки программного обеспечения с использованием облачных технологий, системы контроля версий Git, непрерывной интеграции (CI) и непрерывной поставки (CD).

---

## 1. Предлагаемое решение

### Стек технологий

**Основные компоненты:**

| Компонент | Решение | Роль |
|-----------|---------|------|
| **Система контроля версий** | GitHub / GitLab | Хранение исходного кода и триггеры для сборок |
| **Платформа CI/CD** | GitLab CI / GitHub Actions | Оркестрация, управление пайплайнами и сборками |
| **Контейнеризация** | Docker | Создание и управление образами для сборки |
| **Оркестрация сборок** | Runners (GitLab CI) / Self-hosted Runners (GitHub Actions) | Выполнение задач сборки и тестирования |
| **Хранилище артефактов** | Docker Registry / Container Registry (встроенное) | Хранение собранных Docker-образов |
| **Управление секретами** | HashiCorp Vault / встроенное хранилище (GitHub Secrets) | Безопасное хранение учетных данных |

---

## 2. Архитектура решения

```
┌─────────────────────────────────────────────────────────────────┐
│                   РАЗРАБОТЧИК                                    │
│              (локальная рабочая среда)                          │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              СИСТЕМА КОНТРОЛЯ ВЕРСИЙ (Git)                       │
│  ┌──────────────────┐  ┌──────────────────┐                    │
│  │ Репозиторий      │  │ Репозиторий      │  ...               │
│  │ сервис-1         │  │ сервис-2         │                    │
│  └────────┬─────────┘  └────────┬─────────┘                    │
└───────────┼──────────────────────┼──────────────────────────────┘
            │                      │
    (git push /              (git push /
     git pull request)        git pull request)
            │                      │
            ▼                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ПЛАТФОРМА CI/CD                               │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │         Webhook триггеры из Git                         │   │
│  │  • Push в main/develop                                  │   │
│  │  • Pull requests                                        │   │
│  │  • Manual trigger с параметрами                         │   │
│  └──────────────────┬──────────────────────────────────────┘   │
│                     │                                           │
│  ┌──────────────────▼──────────────────────────────────────┐   │
│  │     Управление пайплайнами (YAML конфиги)              │   │
│  │  • .gitlab-ci.yml или .github/workflows/                │   │
│  │  • Шаблоны для разных конфигураций                      │   │
│  │  • Параметризованные сборки                             │   │
│  └──────────────────┬──────────────────────────────────────┘   │
│                     │                                           │
│  ┌──────────────────▼──────────────────────────────────────┐   │
│  │      Хранилище переменных и секретов                    │   │
│  │  • Environment variables                                │   │
│  │  • Encrypted secrets                                    │   │
│  │  • Привязка к каждой сборке                             │   │
│  └──────────────────┬──────────────────────────────────────┘   │
└─────────────────────┼──────────────────────────────────────────┘
                      │
    ┌─────────────────┼─────────────────┐
    │                 │                 │
    ▼                 ▼                 ▼
┌───────────┐ ┌──────────────┐ ┌──────────────┐
│ Runner 1  │ │ Runner 2     │ │ Runner N ...  │
│(облачный) │ │(self-hosted) │ │(self-hosted)  │
└─────┬─────┘ └──────┬───────┘ └──────┬───────┘
      │              │               │
      │    ┌─────────┴───────┐       │
      │    │                 │       │
      ▼    ▼                 ▼       ▼
   ┌─────────────────────────────────────┐
   │  ВЫПОЛНЕНИЕ СБОРОК                  │
   │  • Build (Компиляция)               │
   │  • Test (Параллельное тестирование) │
   │  • Scan (SAST, SCA)                 │
   │  • Package (Создание артефактов)    │
   └──────────────────┬──────────────────┘
                      │
     ┌────────────────┼────────────────┐
     │                │                │
     ▼                ▼                ▼
┌──────────────┐ ┌──────────────┐ ┌─────────────┐
│   Docker     │ │   Maven/NPM  │ │   Другие    │
│   Registry   │ │   Registry   │ │ артефакты   │
└──────────────┘ └──────────────┘ └─────────────┘
                      │
     ┌────────────────┼────────────────┐
     │                │                │
     ▼                ▼                ▼
┌──────────────┐ ┌──────────────┐ ┌─────────────┐
│ DEV среда    │ │ TEST среда   │ │ PROD среда  │
│ (развертыв.) │ │ (развертыв.) │ │ (развертыв) │
└──────────────┘ └──────────────┘ └─────────────┘
```

---

## 3. Детальное описание компонентов

### 3.1 Система контроля версий

**Выбор: GitHub или GitLab**

#### GitHub
- **Преимущества:**
  - Встроенная платформа GitHub Actions
  - Лучший выбор для открытого кода и экосистемы
  - Простая интеграция с Actions Marketplace
  - Поддержка Pull Requests с автоматическими проверками
  
- **Недостатки:**
  - Платный для приватных проектов при большом использовании Actions
  - Менее гибкий для self-hosted runner'ов (по сравнению с GitLab)

#### GitLab (рекомендуется для корпоративных сценариев)
- **Преимущества:**
  - Встроенная платформа GitLab CI/CD
  - Отличная поддержка для self-hosted runners
  - Более полный контроль над инфраструктурой
  - Встроенное управление секретами
  - Бесплатный self-hosted вариант
  
- **Недостатки:**
  - Требует больше администрирования при self-hosted варианте
  - Меньше готовых интеграций по сравнению с GitHub

**Структура репозиториев:**
```
GitHub.com/Company/
├── service-api/                    # Микросервис API
│   ├── .github/workflows/
│   │   ├── build-and-test.yml
│   │   ├── deploy-dev.yml
│   │   └── deploy-prod.yml
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── src/
├── service-web/                    # Веб-фронтенд
│   ├── .github/workflows/
│   ├── Dockerfile
│   └── src/
├── service-worker/                 # Worker сервис
│   ├── .github/workflows/
│   ├── Dockerfile
│   └── src/
└── infrastructure/                 # IaC конфигурации
    ├── terraform/
    └── kubernetes/
```

---

### 3.2 Платформа CI/CD - GitLab CI / GitHub Actions

#### Основные возможности

**Триггеры сборки:**

1. **Автоматические триггеры (events):**
   ```yaml
   # GitLab CI пример
   workflow:
     rules:
       - if: '$CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_BRANCH == "main"'
       - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
   ```

   ```yaml
   # GitHub Actions пример
   on:
     push:
       branches: [main, develop]
     pull_request:
       branches: [main]
   ```

2. **Ручные триггеры с параметрами:**
   ```yaml
   # GitLab CI
   manual_deploy:
     script:
       - echo "Deploying to $DEPLOY_ENV"
     when: manual
     environment: $DEPLOY_ENV
   ```

   ```yaml
   # GitHub Actions
   on:
     workflow_dispatch:
       inputs:
         environment:
           description: 'Target environment'
           required: true
           default: 'staging'
   ```

**Привязка настроек к сборке:**
```yaml
# GitLab CI с переменными
variables:
  DEPLOY_ENV: "staging"
  DOCKER_REGISTRY: "registry.gitlab.com"
  BUILD_TIMEOUT: "1800"

stages:
  - build
  - test
  - deploy

build_job:
  stage: build
  variables:
    BUILD_TYPE: "release"
  script:
    - docker build -t $DOCKER_REGISTRY/app:$CI_COMMIT_SHA .
```

**Шаблоны для различных конфигураций:**
```yaml
# .gitlab-ci.yml с повторяемыми блоками
.base_docker_build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $DOCKER_USER -p $DOCKER_PASSWORD $DOCKER_REGISTRY

.build_service_template:
  extends: .base_docker_build
  script:
    - docker build -f Dockerfile.$BUILD_TYPE -t $IMAGE_NAME:$CI_COMMIT_SHA .
    - docker push $IMAGE_NAME:$CI_COMMIT_SHA

build_dev:
  extends: .build_service_template
  variables:
    BUILD_TYPE: "dev"
    IMAGE_NAME: "$DOCKER_REGISTRY/app-dev"
  only:
    - develop

build_prod:
  extends: .build_service_template
  variables:
    BUILD_TYPE: "prod"
    IMAGE_NAME: "$DOCKER_REGISTRY/app-prod"
  only:
    - main
```

---

### 3.3 Управление секретными данными

**Решение: Встроенное хранилище + HashiCorp Vault (опционально)**

#### GitHub Secrets
```yaml
# Использование в GitHub Actions
name: Deploy with Secrets
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy
        env:
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
          API_KEY: ${{ secrets.API_KEY }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        run: |
          ./deploy.sh
```

#### GitLab CI Variables
```yaml
# Использование в GitLab CI
stages:
  - deploy

deploy_prod:
  stage: deploy
  environment:
    name: production
  script:
    - echo "Deploying with credentials: $DB_PASSWORD"
    - ./deploy-prod.sh
  variables:
    DB_PASSWORD: $SECRET_DB_PASSWORD
    API_TOKEN: $SECRET_API_TOKEN
  only:
    - main
```

**Иерархия секретов:**
- Глобальные переменные (на уровне организации)
- Переменные проекта (на уровне репозитория)
- Переменные окружения (для каждого развертывания)
- Переменные группы (для нескольких проектов)

**Лучшие практики:**
1. Никогда не коммитить секреты в Git
2. Использовать role-based access control (RBAC)
3. Регулярно ротировать ключи
4. Логировать доступ к секретам (audit logs)
5. Использовать Vault для очень чувствительных данных

---

### 3.4 Несколько конфигураций из одного репозитория

```yaml
# .gitlab-ci.yml с множественными вариантами
variables:
  REGISTRY: "registry.gitlab.com"

stages:
  - build
  - test
  - deploy

# Параллельные сборки разных конфигураций
build:python:
  stage: build
  image: python:3.11
  script:
    - pip install -r requirements.txt
    - python setup.py build

build:nodejs:
  stage: build
  image: node:18
  script:
    - npm install
    - npm run build

build:docker:
  stage: build
  image: docker:latest
  parallel:
    matrix:
      - FLAVOR: ["slim", "full"]
        ARCH: ["amd64", "arm64"]
  script:
    - docker build --build-arg FLAVOR=$FLAVOR -t app:$FLAVOR-$ARCH .

# Параллельное тестирование
test:unit:
  stage: test
  image: python:3.11
  parallel: 5
  script:
    - pip install -r requirements-test.txt
    - pytest tests/unit --split-index=$CI_NODE_INDEX --split-total=$CI_NODE_TOTAL

test:integration:
  stage: test
  image: docker:latest
  services:
    - postgres:13
    - redis:7
  script:
    - docker-compose -f docker-compose.test.yml up --abort-on-container-exit

test:e2e:
  stage: test
  image: cypress/base:latest
  script:
    - npm install
    - npx cypress run --parallel --record
```

---

### 3.5 Кастомные шаги при сборке

```yaml
# Полный пайплайн с кастомными шагами
stages:
  - prepare
  - build
  - test
  - scan
  - deploy
  - cleanup

# Кастомный шаг подготовки
prepare:environment:
  stage: prepare
  image: ubuntu:22.04
  before_script:
    - apt-get update && apt-get install -y curl jq
  script:
    - echo "Preparing build environment..."
    - mkdir -p /builds/artifacts
    - ./scripts/setup-env.sh
    - source /builds/artifacts/build.env
  artifacts:
    paths:
      - /builds/artifacts/
    expire_in: 1 day

# Кастомная сборка с разными шагами
build:app:
  stage: build
  image: ubuntu:22.04
  dependencies:
    - prepare:environment
  script:
    - echo "Step 1: Validate code..."
    - ./scripts/validate.sh
    - echo "Step 2: Compile..."
    - make build
    - echo "Step 3: Create artifacts..."
    - tar -czf app-$CI_COMMIT_SHA.tar.gz dist/
    - echo "Step 4: Upload to storage..."
    - curl -X POST -F "file=@app-$CI_COMMIT_SHA.tar.gz" $ARTIFACT_SERVER
  after_script:
    - echo "Build completed with status: $CI_JOB_STATUS"
  artifacts:
    paths:
      - dist/
      - app-*.tar.gz
    expire_in: 30 days

# Кастомный шаг анализа кода
scan:sast:
  stage: scan
  image: returntocorp/semgrep
  script:
    - semgrep --config=p/security-audit src/
  allow_failure: true

# Кастомный шаг с уведомлениями
deploy:with-approval:
  stage: deploy
  when: manual
  environment:
    name: production
    url: https://prod.example.com
  script:
    - echo "Deploying to production..."
    - kubectl apply -f k8s/prod.yaml
    - kubectl rollout status deployment/app-prod
  after_script:
    - curl -X POST https://hooks.slack.com/services/... -d "Deployment completed"
```

---

### 3.6 Собственные Docker-образы для сборок

```dockerfile
# Dockerfile для builder образа
FROM ubuntu:22.04

# Установка необходимых инструментов
RUN apt-get update && apt-get install -y \
    build-essential \
    git \
    curl \
    wget \
    docker.io \
    python3 \
    pip \
    nodejs \
    npm \
    openjdk-11-jdk \
    maven \
    && rm -rf /var/lib/apt/lists/*

# Установка дополнительных инструментов
RUN pip install \
    pytest \
    pylint \
    black \
    mypy

# Копирование скриптов сборки
COPY scripts/ /opt/scripts/
RUN chmod +x /opt/scripts/*.sh

# Установка рабочей директории
WORKDIR /builds

# Точка входа
ENTRYPOINT ["/opt/scripts/build.sh"]
```

**Использование custom-образов в CI/CD:**
```yaml
# GitLab CI с custom образом
stages:
  - build
  - test

build:custom:
  stage: build
  image: registry.company.com/ci-builder:v1.0.0
  script:
    - /opt/scripts/build.sh
    - /opt/scripts/test.sh

# GitHub Actions с custom образом
name: Custom Builder
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: ghcr.io/company/ci-builder:v1.0.0
      credentials:
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    steps:
      - uses: actions/checkout@v3
      - run: /opt/scripts/build.sh
      - run: /opt/scripts/test.sh
```

---

### 3.7 Self-hosted Runners

#### Развертывание собственных агентов сборки

**GitLab Runner (self-hosted):**
```bash
# Установка GitLab Runner
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
sudo apt-get install gitlab-runner

# Регистрация runner'а
sudo gitlab-runner register \
  --url https://gitlab.com/ \
  --registration-token $RUNNER_TOKEN \
  --executor docker \
  --docker-image ubuntu:22.04 \
  --description "Docker runner on company server"

# Конфигурация для параллельных сборок
sudo cat > /etc/gitlab-runner/config.toml << EOF
[[runners]]
  name = "Company Docker Runner"
  url = "https://gitlab.com/"
  token = "xxxxxxxxxxxxx"
  executor = "docker"
  [runners.docker]
    image = "ubuntu:22.04"
    privileged = true
    volumes = ["/var/run/docker.sock:/var/run/docker.sock"]
    max_builds = 10
  [runners.machine]
    idle_count = 2
    idle_time = 600
    max_builds = 10
EOF

sudo gitlab-runner restart
```

**GitHub Self-hosted Runner:**
```bash
# Скачивание и установка
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.310.0.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.310.0/actions-runner-linux-x64-2.310.0.tar.gz
tar xzf ./actions-runner-linux-x64-2.310.0.tar.gz

# Конфигурация
./config.sh --url https://github.com/company/repo --token $RUNNER_TOKEN

# Установка как сервис и запуск
sudo ./svc.sh install
sudo ./svc.sh start
```

**Конфигурация GitHub Actions с self-hosted runner:**
```yaml
name: Self-hosted Build
on: [push]

jobs:
  build:
    runs-on: [self-hosted, linux, docker]
    steps:
      - uses: actions/checkout@v3
      - name: Build
        run: |
          docker build -t myapp:${{ github.sha }} .
          docker push myapp:${{ github.sha }}
```

---

### 3.8 Параллельное выполнение сборок и тестов

```yaml
# GitLab CI - Параллельное выполнение
stages:
  - build
  - test
  - deploy

# Параллельные сборки (матрица)
build:
  stage: build
  parallel:
    matrix:
      - PYTHON_VERSION: ["3.9", "3.10", "3.11"]
        DJANGO_VERSION: ["3.2", "4.0", "4.1"]
  image: python:${PYTHON_VERSION}
  script:
    - pip install django==${DJANGO_VERSION}
    - python setup.py build
  artifacts:
    paths:
      - dist/

# Параллельное тестирование
test:unit:
  stage: test
  needs: ["build"]
  parallel: 10  # 10 параллельных job'ов
  script:
    - pytest tests/unit --split-index=$CI_NODE_INDEX --split-total=$CI_NODE_TOTAL

test:integration:
  stage: test
  needs: ["build"]
  parallel:
    matrix:
      - DB: ["postgres", "mysql", "mariadb"]
        CACHE: ["redis", "memcached"]
  services:
    - ${DB}:latest
    - ${CACHE}:latest
  script:
    - pytest tests/integration -v

# Фаза развертывания с зависимостями от параллельных сборок
deploy:
  stage: deploy
  needs: ["test:unit", "test:integration"]
  environment:
    name: production
  script:
    - ./deploy.sh
```

**GitHub Actions - Параллельное выполнение:**
```yaml
name: Parallel Tests

on: [push, pull_request]

jobs:
  # Матрица для параллельного выполнения
  test:
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]
        os: [ubuntu-latest, windows-latest, macos-latest]
      max-parallel: 6
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test

  # Параллельные тесты (шаркирование)
  integration:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        test-group: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npm run test:integration -- --shard ${{ matrix.test-group }}/4
      - uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: failures-shard-${{ matrix.test-group }}
          path: test-results/

  # Объединение результатов
  report:
    if: always()
    needs: [test, integration]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v3
      - run: ./scripts/combine-results.sh
      - uses: codecov/codecov-action@v3
```

---

## 4. Пример полного пайплайна

```yaml
# Полный пример .gitlab-ci.yml для микросервиса
image: docker:latest

variables:
  REGISTRY: "registry.gitlab.com"
  APP_NAME: "my-service"
  VERSION: "$CI_COMMIT_SHORT_SHA"

stages:
  - build
  - test
  - scan
  - deploy

# ============ СБОРКА ============
build:image:
  stage: build
  services:
    - docker:dind
  script:
    - docker build -t $REGISTRY/$APP_NAME:$VERSION .
    - docker push $REGISTRY/$APP_NAME:$VERSION
  only:
    - main
    - develop

# ============ ТЕСТИРОВАНИЕ ============
test:unit:
  stage: test
  image: node:18
  parallel: 4
  script:
    - npm ci
    - npm run test:unit -- --shard=$CI_NODE_INDEX/$CI_NODE_TOTAL
  coverage: '/Coverage: (\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

test:integration:
  stage: test
  services:
    - postgres:14
    - redis:7
  script:
    - npm ci
    - npm run test:integration
  environment:
    name: test
    kubernetes:
      namespace: testing

# ============ СКАНИРОВАНИЕ ============
scan:sast:
  stage: scan
  image: returntocorp/semgrep
  script:
    - semgrep --config=p/security-audit --json . > sast-report.json
  artifacts:
    reports:
      sast: sast-report.json

scan:dependencies:
  stage: scan
  image: aquasec/trivy:latest
  script:
    - trivy image --format json --output deps-report.json $REGISTRY/$APP_NAME:$VERSION
  allow_failure: true

# ============ РАЗВЕРТЫВАНИЕ ============
deploy:dev:
  stage: deploy
  environment:
    name: development
    url: https://dev.example.com
    kubernetes:
      namespace: development
  script:
    - kubectl set image deployment/my-service my-service=$REGISTRY/$APP_NAME:$VERSION
    - kubectl rollout status deployment/my-service
  only:
    - develop

deploy:staging:
  stage: deploy
  environment:
    name: staging
    url: https://staging.example.com
    kubernetes:
      namespace: staging
  script:
    - helm upgrade --install my-service ./helm/my-service -f values-staging.yaml
  only:
    - main

deploy:production:
  stage: deploy
  when: manual
  environment:
    name: production
    url: https://example.com
    kubernetes:
      namespace: production
  script:
    - helm upgrade --install my-service ./helm/my-service -f values-prod.yaml
    - kubectl rollout status deployment/my-service
  only:
    - main
  needs: ["test:unit", "test:integration", "scan:sast"]
```

---

## 5. Соответствие требованиям

| # | Требование | Решение | Статус |
|---|-----------|---------|--------|
| 1 | Облачная система | GitHub / GitLab cloud | ✓ |
| 2 | Система контроля версий Git | GitHub / GitLab | ✓ |
| 3 | Репозиторий на каждый сервис | Отдельные репо для каждого микросервиса | ✓ |
| 4 | Запуск сборки по событию Git | Webhooks, triggers (push, PR) | ✓ |
| 5 | Запуск сборки по кнопке с параметрами | `when: manual` в GitLab CI / `workflow_dispatch` в GitHub Actions | ✓ |
| 6 | Привязка настроек к сборке | Variables, environments, build parameters | ✓ |
| 7 | Шаблоны для различных конфигураций | `.extends`, `include`, reusable workflows | ✓ |
| 8 | Безопасное хранение секретных данных | GitHub Secrets / GitLab Variables (encrypted) | ✓ |
| 9 | Несколько конфигураций из одного репо | Matrix builds, parallel stages | ✓ |
| 10 | Кастомные шаги при сборке | Custom scripts, before_script, after_script | ✓ |
| 11 | Собственные Docker-образы | Custom Docker images в registry | ✓ |
| 12 | Развертывание агентов на своих серверах | GitLab Runner / GitHub self-hosted runner | ✓ |
| 13 | Параллельное запуск сборок | Parallel jobs, matrix strategy | ✓ |
| 14 | Параллельное запуск тестов | Test splitting, parallel: N | ✓ |

---

## 6. Обоснование выбора

### Почему GitLab CI/CD?

**Преимущества для корпоративного использования:**

1. **Полная интеграция с Git:**
   - CI/CD встроен в платформу GitLab
   - Нет необходимости интегрировать отдельные инструменты
   - Единая точка управления исходным кодом и сборками

2. **Гибкость Self-hosted Runner'ов:**
   - Полный контроль над инфраструктурой
   - Возможность развернуть runner'ы на собственных серверах
   - Выполнение сборок в закрытой сети компании

3. **Встроенное управление секретами:**
   - Зашифрованное хранилище переменных
   - Привязка переменных к окружениям
   - Логирование доступа (audit logs)

4. **Масштабируемость:**
   - Поддержка параллельных сборок из коробки
   - Встроенное кэширование артефактов
   - Эффективное распределение нагрузки между runner'ами

5. **Разработчик-ориентированность:**
   - YAML конфигурация легко читается и поддерживается
   - Обширная документация
   - Активное сообщество

### Альтернативы и почему не они

**GitHub Actions:**
- ✓ Хорош для open source
- ✗ Дорогой для приватных проектов при интенсивном использовании
- ✗ Менее гибкий для self-hosted инфраструктуры
- ✗ Привязан к GitHub (сложна миграция)

**Jenkins:**
- ✓ Высокая гибкость и расширяемость
- ✗ Требует значительного администрирования
- ✗ Устаревший интерфейс
- ✗ Проблемы с безопасностью при неправильной настройке

**CircleCI:**
- ✓ Облачное решение с быстрым стартом
- ✗ Лимитирован облачными ресурсами
- ✗ Сложнее развернуть свои runner'ы
- ✗ Дорогой при масштабировании

---

## 7. План внедрения

### Фаза 1: Подготовка (1-2 недели)
- [ ] Создать GitLab инстанс или использовать GitLab.com
- [ ] Мигрировать репозитории
- [ ] Создать структуру групп и проектов
- [ ] Настроить RBAC (Role-Based Access Control)

### Фаза 2: Настройка CI/CD (2-3 недели)
- [ ] Развернуть первые GitLab Runner'ы
- [ ] Создать базовые пайплайны для микросервисов
- [ ] Настроить Docker Registry
- [ ] Создать шаблоны для разных типов проектов

### Фаза 3: Интеграция сборок (1-2 недели)
- [ ] Настроить сборку Docker-образов
- [ ] Настроить автоматизированное тестирование
- [ ] Интегрировать анализ кода (SAST, SCA)
- [ ] Настроить артефакты и кэширование

### Фаза 4: Развертывание и мониторинг (1-2 недели)
- [ ] Настроить развертывание в различные среды
- [ ] Интегрировать Kubernetes (если используется)
- [ ] Настроить мониторинг и логирование сборок
- [ ] Обучение команды

---

## 8. Заключение

**Предлагаемое решение** на основе **GitLab CI/CD** с self-hosted runner'ами обеспечивает:

✅ **Полный контроль** над процессом разработки и сборок  
✅ **Высокую масштабируемость** для растущих команд  
✅ **Безопасность** данных и учетных данных  
✅ **Гибкость** в конфигурациях и интеграциях  
✅ **Экономичность** при использовании самостоятельной инфраструктуры  

Это решение подходит как для стартапов, так и для крупных корпоративных организаций и соответствует всем указанным в задаче требованиям.
