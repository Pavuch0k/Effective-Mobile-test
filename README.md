# Effective Mobile Test Task

## Описание

Проект демонстрирует базовую архитектуру микросервисного приложения с использованием Docker, состоящую из HTTP-сервера на Python и Nginx в качестве reverse proxy.

## Архитектура

```
Клиент → Nginx (порт 80) → Backend (порт 8080, внутренний)
```

Nginx принимает HTTP-запросы на порт 80 и проксирует их на backend-сервис, работающий на порту 8080 внутри Docker-сети. Backend недоступен напрямую с хоста, что обеспечивает дополнительный уровень безопасности.

## Структура проекта

```
.
├── backend/
│   ├── Dockerfile          # Образ Python-приложения
│   ├── .dockerignore       # Исключения для Docker
│   └── app.py              # HTTP-сервер
├── nginx/
│   ├── Dockerfile          # Образ Nginx с кастомной конфигурацией
│   ├── .dockerignore       # Исключения для Docker
│   └── nginx.conf          # Конфигурация reverse proxy
├── docker-compose.yml      # Оркестрация сервисов
├── .gitignore              # Исключения для Git
└── README.md               # Документация
```

## Технологии

- **Backend**: Python 3.11 (стандартная библиотека http.server)
- **Reverse Proxy**: Nginx 1.27-alpine
- **Контейнеризация**: Docker, Docker Compose
- **Сеть**: Изолированная bridge-сеть

## Требования

- Docker 20.10+
- Docker Compose 2.0+

## Запуск проекта

1. Клонируйте репозиторий:
```bash
git clone git@github.com:Pavuch0k/Effective-Mobile-test.git
cd Effective-Mobile-test
```

2. Запустите сервисы:
```bash
docker-compose up -d
```

3. Проверьте статус контейнеров:
```bash
docker-compose ps
```

Ожидаемый вывод:
```
NAME      IMAGE              STATUS                    PORTS
backend   effective-backend  Up (healthy)              8080/tcp
nginx     effective-nginx    Up (healthy)              0.0.0.0:80->80/tcp
```

## Проверка работоспособности

Выполните команду:
```bash
curl http://localhost
```

Ожидаемый ответ:
```
Hello from Effective Mobile!
```

Проверка healthcheck endpoints:
```bash
# Backend (доступен только из Docker-сети)
docker exec backend curl http://localhost:8080/health

# Nginx
curl http://localhost/health
```

## Остановка проекта

```bash
docker-compose down
```

## Особенности реализации

### Безопасность

- Backend-приложение запускается от непривилегированного пользователя (appuser, UID 1000)
- Backend-сервис изолирован и недоступен с хоста
- Только Nginx доступен извне на порту 80
- Отсутствие секретов в репозитории

### Docker

- Использованы официальные базовые образы (python:3.11-slim, nginx:1.27-alpine)
- Явно указан WORKDIR
- Порты документированы через EXPOSE
- Реализованы healthcheck для мониторинга состояния сервисов

### Networking

- Выделенная Docker-сеть app-network типа bridge
- Взаимодействие между сервисами по DNS-именам
- Отсутствие хардкода IP-адресов

### Nginx

- Конфигурация upstream для backend
- Передача стандартных заголовков:
  - Host
  - X-Real-IP
  - X-Forwarded-For
  - X-Forwarded-Proto


Сервер будет доступен на http://localhost:8080
