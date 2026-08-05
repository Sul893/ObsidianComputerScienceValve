---
tags: [deploy]
---

# Continuous Delivery / Continuous Deployment (CD)

**Continuous Delivery / Continuous Deployment (CD)** — автоматическая доставка собранного кода в тестовое или продакшен-окружение после успешного прохождения [[CI]].

## Delivery vs Deployment

Одинаковую аббревиатуру `CD` используют для двух связанных, но разных практик:

- **Continuous Delivery** — код всегда готов к деплою: каждая успешная сборка автоматически доходит до staging, но сам релиз в продакшен запускается вручную (кнопкой). Даёт контроль момента выкатки.
- **Continuous Deployment** — каждая успешная сборка автоматически выкатывается в продакшен без ручного вмешательства. Максимальная автоматизация, требует высокой зрелости тестирования.

## Этапы CD-пайплайна

```
[CI проходит] → Deploy to Staging → Smoke Tests → (Ручное одобрение для Delivery) → Deploy to Prod
[CI проходит] → Deploy to Staging → E2E Tests → Deploy to Prod  (для Continuous Deployment — всё автоматически)
```

## Пример: CD-стадия в GitHub Actions

```yaml
# .github/workflows/cd.yml — Continuous Deployment (auto)
name: CD
on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]
    branches: [main]

jobs:
  deploy-staging:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t myapp:latest .
      - name: Push to registry
        run: |
          echo "${{ secrets.REGISTRY_PASSWORD }}" | docker login -u ${{ secrets.REGISTRY_USER }} --password-stdin
          docker tag myapp:latest registry.io/myorg/myapp:${{ github.sha }}
          docker push registry.io/myorg/myapp:${{ github.sha }}
      - name: Deploy to staging (kubectl)
        run: kubectl set image deployment/myapp app=registry.io/myorg/myapp:${{ github.sha }}

  deploy-prod:
    needs: deploy-staging
    environment: production  # требует ручного approval → Continuous *Delivery*
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to prod
        run: kubectl set image deployment/myapp app=registry.io/myorg/myapp:${{ github.sha }}
```

## Deployment strategies

- **Rolling** — постепенная замена старых подов новыми (по умолчанию в Kubernetes)
- **Blue/Green** — две среды Blue (текущая) и Green (новая); переключение трафика за секунду
- **Canary** — трафик на новую версию пускают частями (1% → 10% → 100%)
- **Recreate** — полностью убрать старую, потом поднять новую (down-time)

## Воспроизводимость окружения

Для устранения проблемы "работает на моей машине" при деплое используются [[Docker]]-образы: одно и то же окружение применяется на всех этапах — от разработки до продакшена.

## Инструменты

Jenkins, GitLab CI/CD, ArgoCD (для Kubernetes, GitOps), Spinnaker, Octopus Deploy.

friend:: [[CI]]
friend:: [[Docker]]