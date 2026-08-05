---
tags: [deploy]
---

# Continuous Integration (CI)

**Continuous Integration (CI)** — практика часто интегрировать код в общую ветку (main/integration) с последующей автоматической сборкой и тестированием.

## Цель

Обнаруживать ошибки интеграции как можно раньше — на этапе push, а не перед релизом. Частые интеграции (несколько раз в день) сокращают объём изменений между merge'ами и упрощают разрешение конфликтов.

## Ключевые этапы CI-пайплайна

1. **Push кода в репозиторий** — Git (GitHub, GitLab, Bitbucket)
2. **Сборка проекта** — Maven/Gradle для Java, npm/pip для других стеков
3. **Запуск unit- и интеграционных тестов** — JUnit, TestNG
4. **Статический анализ кода** — линтеры, SonarQube, Checkstyle
5. **Генерация отчётов** — покрытие тестами, качество кода, артефакты сборки

При падении любого этапа пайплайн останавливается и команда получает уведомление.

## Пример: CI-конфигурация

### GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
      - name: Build with Maven
        run: mvn -B package
      - name: Run tests
        run: mvn test
      - name: Static analysis (SonarQube)
        run: mvn sonar:sonar
```

### GitLab CI/CD

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - analyze

build:
  stage: build
  script:
    - mvn -B compile
  artifacts:
    paths:
      - target/

test:
  stage: test
  script:
    - mvn test
  artifacts:
    reports:
      junit: target/surefire-reports/TEST-*.xml

analyze:
  stage: analyze
  script:
    - mvn sonar:sonar
```

### Jenkinsfile (Declarative Pipeline)

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { sh 'mvn -B clean compile' }
        }
        stage('Test') {
            steps { sh 'mvn test' }
            post { always { junit 'target/surefire-reports/*.xml' } }
        }
        stage('SonarQube') {
            steps { sh 'mvn sonar:sonar' }
        }
    }
    post {
        failure { emailext subject: 'Build failed', body: 'Check logs', to: 'team@company.com' }
    }
}
```

## Связь с CD и Docker

CI обычно является первой стадией CI/CD пайплайна и связан с [[CD]] (продолжением, доставляющим собранный артефакт дальше). Для изоляции сборочного окружения и воспроизводимости часто применяется [[Docker]].

## Инструменты

Jenkins, GitLab CI/CD, GitHub Actions, TeamCity, CircleCI.

friend:: [[CD]]
friend:: [[Docker]]