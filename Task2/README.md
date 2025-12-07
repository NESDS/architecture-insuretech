# Задание 2. Динамическое масштабирование контейнеров

## Что сделано

### 1. Deployment (`deployment.yaml`)
- Развёртывание тестового приложения `scaletestapp`
- Начальное количество реплик: 1
- Лимит памяти: 30Mi
- Образ: `ghcr.io/yandex-practicum/scaletestapp:latest`

### 2. Service (`service.yaml`)
- Сервис типа NodePort для доступа к приложению
- Проброс порта 80 → 8080

### 3. HPA (`hpa.yaml`)
- Horizontal Pod Autoscaler на основе утилизации памяти
- Порог: 80%
- Диапазон реплик: 1–10

### 4. Нагрузочное тестирование (`locustfile.py`)
- Сценарий Locust для генерации нагрузки на endpoint `/`

## Результат

При нагрузке от Locust HPA автоматически увеличил количество подов с 1 до 2.

Скриншоты:
- `dashboard_screenshot.png` — дашборд Kubernetes с 2 подами
- `locust_screenshot_1.png` — тест с 50 пользователями, 4 RPS
- `locust_screenshot_2.png` — тест с 20 000 пользователями, 85 RPS

