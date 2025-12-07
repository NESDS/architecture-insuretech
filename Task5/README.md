# Задание 5. Проектирование GraphQL API

## Описание задачи

Перевод REST API сервиса `client-info` на GraphQL для оптимизации взаимодействия между потребителями данных и сервисом.

## Проблема текущего REST API

Существующий REST API имеет три отдельных эндпоинта:
- `GET /clients/{id}` — базовая информация о клиенте
- `GET /clients/{id}/documents` — документы клиента
- `GET /clients/{id}/relatives` — родственники клиента

**Недостатки такого подхода:**
1. **Множество запросов (Under-fetching):** Для получения полных данных клиента требуется 3 отдельных HTTP-запроса
2. **Повышенная нагрузка:** Каждый дополнительный запрос увеличивает RPS сервиса
3. **Неэффективность:** Невозможно запросить только нужные поля — всегда возвращается полная структура объекта

## Решение: GraphQL Schema

Файл: `schema.graphql`

### Структура схемы

#### Типы данных

| Тип | Описание | Поля |
|-----|----------|------|
| `Client` | Клиент сервиса | id, name, age, documents, relatives |
| `Document` | Документ клиента | id, type, number, issueDate, expiryDate |
| `Relative` | Родственник клиента | id, relationType, name, age |

#### Запросы (Queries)

| Запрос | Параметры | Описание | REST-эквивалент |
|--------|-----------|----------|-----------------|
| `client` | id: ID! | Получить клиента по ID | GET /clients/{id} |
| `documents` | clientId: ID! | Получить документы клиента | GET /clients/{id}/documents |
| `relatives` | clientId: ID! | Получить родственников клиента | GET /clients/{id}/relatives |

### Ключевое преимущество: Вложенные типы

Тип `Client` содержит вложенные поля `documents` и `relatives`, что позволяет получить все данные **одним запросом**:

```graphql
query {
  client(id: "123") {
    id
    name
    age
    documents {
      type
      number
    }
    relatives {
      relationType
      name
    }
  }
}
```

## Примеры использования

### 1. Только базовые данные клиента (минимальный запрос)

```graphql
query GetClientBasicInfo {
  client(id: "123") {
    id
    name
  }
}
```

**Ответ:**
```json
{
  "data": {
    "client": {
      "id": "123",
      "name": "Иванов Иван Иванович"
    }
  }
}
```

### 2. Клиент с документами для оформления страховки

```graphql
query GetClientWithDocuments {
  client(id: "123") {
    id
    name
    age
    documents {
      type
      number
      expiryDate
    }
  }
}
```

### 3. Полная информация о клиенте

```graphql
query GetClientFullInfo {
  client(id: "123") {
    id
    name
    age
    documents {
      id
      type
      number
      issueDate
      expiryDate
    }
    relatives {
      id
      relationType
      name
      age
    }
  }
}
```

### 4. Только документы (эквивалент REST endpoint)

```graphql
query GetDocuments {
  documents(clientId: "123") {
    id
    type
    number
  }
}
```

## Сравнение REST vs GraphQL

| Критерий | REST API | GraphQL |
|----------|----------|---------|
| Запросов для полных данных | 3 | 1 |
| Выбор полей | Нет | Да |
| Over-fetching | Да | Нет |
| Under-fetching | Да | Нет |
| Гибкость для потребителей | Низкая | Высокая |

## Преимущества перехода на GraphQL

1. **Снижение RPS:** Вместо 3 запросов — 1 запрос для получения всех данных
2. **Оптимизация трафика:** Клиент запрашивает только нужные поля
3. **Гибкость:** Разные сценарии (продажа, обслуживание) используют один endpoint
4. **Самодокументируемость:** Схема служит контрактом и документацией
5. **Типизация:** Строгая типизация снижает количество ошибок

