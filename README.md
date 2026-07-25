# Momo Store — Docker-контейнеризация

Проектная работа по дисциплине «Docker-контейнеризация и хранение данных».

## Описание

Momo Store — это приложение для заказа пельменей, состоящее из:

- **Backend** — Go (Chi router, Prometheus метрики)
- **Frontend** — Vue.js 3 (SPA)

## Требования

- Docker Engine 20.10+
- Docker Compose v2.0+

## Быстрый старт

```bash
# Клонировать репозиторий
git clone <repo-url>
cd cloud-services-engineer-docker-project-sem2

# Собрать и запустить все сервисы
docker compose up --build

# Или в фоновом режиме
docker compose up --build -d
```

После запуска:

- **Frontend**: http://localhost:80
- **Backend API**: http://localhost:8081
- **Healthcheck**: http://localhost:8081/health
- **Метрики**: http://localhost:8081/metrics

## Структура проекта

```
.
├── backend/                    # Go backend
│   ├── Dockerfile              # Multi-stage сборка
│   ├── .dockerignore
│   └── cmd/api/                # Точка входа
├── frontend/                   # Vue.js frontend
│   ├── Dockerfile              # Multi-stage сборка
│   ├── .dockerignore
│   ├── nginx.conf              # Nginx конфигурация
│   └── src/                    # Исходный код
├── docker-compose.yml          # Оркестрация сервисов
├── secrets/                    # Docker Secrets (пример)
└── .github/workflows/deploy.yaml  # CI/CD пайплайн
```

## Dockerfile

### Backend (Go)

- **Multi-stage сборка**: первый этап — сборка бинарника в `golang:1.17-alpine`, второй этап — минимальный `alpine:3.15`
- **Не-root пользователь**: приложение запускается от `appuser` (UID 1001)
- **Healthcheck**: проверка `/health` эндпоинта каждые 30 секунд
- **Статическая линковка**: `CGO_ENABLED=0` для уменьшения размера

**Размер образа**: ~15 MB

### Frontend (Vue.js + Nginx)

- **Multi-stage сборка**: первый этап — сборка статики в `node:16-alpine`, второй этап — `nginx:1.21-alpine`
- **Не-root пользователь**: nginx запускается от `nginxuser` (UID 1001)
- **Healthcheck**: проверка доступности порта 80 каждые 30 секунд
- **Кастомный nginx.conf**: SPA routing, кэширование статики, security-заголовки

**Размер образа**: ~25 MB

## Docker Compose

### Сервисы

| Сервис    | Порт | Зависимости         | Сеть          |
|-----------|------|---------------------|---------------|
| backend   | 8081 | —                   | backend_net   |
| frontend  | 80   | backend (healthy)   | frontend_net  |

### Сети

- `backend_net` — внутренняя (internal), изолированная сеть для бэкенда
- `frontend_net` — внешняя сеть для фронтенда

### Volumes

- `backend_data` — для хранения данных бэкенда

### Secrets

- `jwt_secret` — секретный ключ для JWT
- `csrf_token` — токен для CSRF защиты

Примеры файлов находятся в `./secrets/`. В production замените на реальные значения.

### Профили

- **dev** — профиль для разработки с дополнительными сервисами:
  ```bash
  docker compose --profile dev up --build
  ```

## Безопасность

### Реализованные меры

1. **Не-root пользователи** — все сервисы запускаются от непривилегированных пользователей
2. **Docker Secrets** — чувствительные данные передаются через secrets, не попадают в образы
3. **Read-only файловые системы** — корневая ФС контейнеров в read-only режиме
4. **Ограничение capabilities** — у контейнеров отозваны все capabilities, добавлены только необходимые
5. **Ограничение ресурсов** — настроены лимиты CPU и памяти
6. **Политика перезапуска** — `unless-stopped` для автоматического восстановления
7. **Security-опции** — `no-new-privileges` для предотвращения эскалации привилегий
8. **Минимальные базовые образы** — Alpine Linux для уменьшения поверхности атаки
9. **Сканирование уязвимостей** — Trivy в CI/CD пайплайне
10. **Healthcheck** — проверка работоспособности для всех сервисов
11. **Trivy fixes** — исправлены уязвимости найденные Trivy в CI/CD пайплайне

## CI/CD

Пайплайн (`.github/workflows/deploy.yaml`) включает:

1. **Security scan** — сканирование кода на уязвимости с помощью Trivy
2. **Build & Push** — сборка и публикация образов в DockerHub
3. **Docker Compose** — проверка сборки через Docker Compose

## Команды

```bash
# Сборка и запуск
docker compose up --build -d

# Остановка
docker compose down

# Остановка с удалением томов
docker compose down -v

# Просмотр логов
docker compose logs -f

# Запуск в dev режиме
docker compose --profile dev up --build

# Сборка только бэкенда
docker compose build backend

# Сборка только фронтенда
docker compose build frontend

# Проверка healthcheck
curl http://localhost:8081/health

# Просмотр метрик
curl http://localhost:8081/metrics