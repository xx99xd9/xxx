LibraryApi - икросервисная система автоматизации библиотеки
============================================================

инальный проект по дисциплине «ведение в DevSecOps».
Система автоматизирует учет книжного фонда и выдачу книг читателям с сохранением данных в PostgreSQL 16.

## рхитектура системы (C4 Model)
- сходник схемы (draw.io): [docs/architecture.drawio](docs/architecture.drawio)
![C4 Architecture](docs/architecture.png)

### еализованные сервисы и взаимодействие
1. **gateway-service** (порт `8080`) - единая точка входа (API Gateway).
2. **catalog-service** - управление каталогом книг и статусом доступности в PostgreSQL.
3. **loan-service** - оформление выдачи книг; при создании выдачи синхронно вызывает `POST /books/{id}/reserve` у `catalog-service` по HTTP.
4. **db** (`postgres:16`) - база данных PostgreSQL с сохранением данных в именованном томе `db_data` внутри изолированной сети `library_net`.

## апуск проекта с нуля
```bash
docker compose up -d --build
docker compose ps
```

### ывод docker compose ps
```text
NAME              IMAGE                 COMMAND                  SERVICE           STATUS
catalog_service   xxx-catalog-service   "dotnet CatalogServi..."   catalog-service   Up 13 seconds
gateway_service   xxx-gateway-service   "dotnet GatewayServi..."   gateway-service   Up 13 seconds
library_db        postgres:16           "docker-entrypoint.s..."   db                Up 13 seconds (healthy)
loan_service      xxx-loan-service      "dotnet LoanService...."   loan-service      Up 13 seconds
```

## роверка API и сохранения данных (curl)
```bash
# 1. обавление книги
curl -X POST http://localhost:8080/api/books -H "Content-Type: application/json" -d '{"title":"DevSecOps Handbook","author":"Gene Kim"}'

# 2. формление выдачи (loan-service резервирует книгу в catalog-service)
curl -X POST http://localhost:8080/api/loans -H "Content-Type: application/json" -d '{"bookId":1,"readerName":"Ivan Ivanov"}'

# 3. роверка статуса книги и сохранности после перезапуска
curl http://localhost:8080/api/books
docker compose restart
curl http://localhost:8080/api/loans
```

### езультат выполнения сценария
```json
[
  { "id": 1, "title": "DevSecOps Handbook", "author": "Gene Kim", "isAvailable": false }
]
[
  { "id": 1, "bookId": 1, "readerName": "Ivan Ivanov", "createdAt": "2026-11-24T08:25:14.678607" }
]
```
