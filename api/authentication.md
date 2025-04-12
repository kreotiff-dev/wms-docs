# Аутентификация API

## Введение

Для обеспечения безопасности API StorageFlow использует механизм аутентификации на основе JWT (JSON Web Tokens). Этот документ описывает процесс аутентификации и авторизации при работе с API.

## Обзор механизма аутентификации

StorageFlow использует токены доступа JWT для проверки подлинности запросов. Каждый запрос к защищенным эндпоинтам должен включать действительный токен в заголовке Authorization.

## Получение токена доступа

### Эндпоинт для аутентификации

```
POST /wms/v1/auth/login
```

### Тело запроса

```json
{
  "username": "user@example.com",
  "password": "securePassword123"
}
```

### Ответ при успешной аутентификации

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 3600,
  "tokenType": "Bearer"
}
```

### Пример запроса

```bash
curl -X POST http://localhost:3000/wms/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "user@example.com",
    "password": "securePassword123"
  }'
```

## Использование токена в запросах

Для доступа к защищенным эндпоинтам необходимо добавить полученный токен в заголовок Authorization:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Пример запроса с токеном

```bash
curl -X GET http://localhost:3000/wms/v1/products \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

## Обновление токена

Когда срок действия токена истекает, его необходимо обновить.

### Эндпоинт для обновления токена

```
POST /wms/v1/auth/refresh
```

### Тело запроса

```json
{
  "refreshToken": "your-refresh-token"
}
```

### Ответ

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 3600,
  "tokenType": "Bearer",
  "refreshToken": "new-refresh-token"
}
```

## Выход из системы

Для завершения сессии и инвалидации токена:

### Эндпоинт для выхода

```
POST /wms/v1/auth/logout
```

### Тело запроса

```json
{
  "refreshToken": "your-refresh-token"
}
```

## Защищенные эндпоинты

Все эндпоинты API, кроме `/wms/v1/auth/*`, требуют аутентификации.

## Уровни доступа и роли

Система StorageFlow поддерживает следующие роли пользователей:

1. **admin** - Полный доступ ко всем функциям
2. **manager** - Управление заказами, отчеты, ограниченный доступ к настройкам
3. **operator** - Базовые операции склада (приемка, размещение, сборка)
4. **viewer** - Только просмотр данных

Доступ к определенным эндпоинтам контролируется в зависимости от роли пользователя.

### Примеры прав доступа

| Эндпоинт       | admin | manager | operator | viewer |
| -------------- | ----- | ------- | -------- | ------ |
| GET /products  | ✓     | ✓       | ✓        | ✓      |
| POST /products | ✓     | ✓       | ✗        | ✗      |
| GET /orders    | ✓     | ✓       | ✓        | ✓      |
| POST /orders   | ✓     | ✓       | ✗        | ✗      |
| GET /reports   | ✓     | ✓       | ✗        | ✓      |
| POST /users    | ✓     | ✗       | ✗        | ✗      |

## Обработка ошибок аутентификации

### Отсутствие токена

```json
{
  "status": 401,
  "message": "Authentication token is missing"
}
```

### Недействительный токен

```json
{
  "status": 401,
  "message": "Invalid authentication token"
}
```

### Истек срок действия токена

```json
{
  "status": 401,
  "message": "Token has expired"
}
```

### Недостаточно прав

```json
{
  "status": 403,
  "message": "Insufficient permissions to access this resource"
}
```

## Рекомендации по безопасности

1. **Передача токенов**: Всегда используйте HTTPS для передачи токенов.
2. **Хранение токенов**: Храните токены в безопасном месте (например, в HttpOnly cookie).
3. **Время жизни токенов**: Используйте короткое время жизни для токенов доступа (1-24 часа).
4. **Регулярная смена ключей**: Периодически меняйте ключи для подписи JWT.

## Примеры использования с различными клиентами

### JavaScript (Fetch API)

```javascript
// Получение токена
async function login(username, password) {
  const response = await fetch("http://localhost:3000/wms/v1/auth/login", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ username, password }),
  });

  return await response.json();
}

// Использование токена
async function getProducts(token) {
  const response = await fetch("http://localhost:3000/wms/v1/products", {
    headers: {
      Authorization: `Bearer ${token}`,
    },
  });

  return await response.json();
}
```

### Python (Requests)

```python
import requests

# Получение токена
def login(username, password):
    response = requests.post(
        'http://localhost:3000/wms/v1/auth/login',
        json={'username': username, 'password': password}
    )
    return response.json()

# Использование токена
def get_products(token):
    headers = {'Authorization': f'Bearer {token}'}
    response = requests.get(
        'http://localhost:3000/wms/v1/products',
        headers=headers
    )
    return response.json()
```

## Устранение неполадок

### Проблема: Постоянные ошибки 401 Unauthorized

**Возможные причины**:

- Токен отсутствует или указан неверно
- Токен просрочен
- Неверный формат заголовка Authorization

**Решение**:

1. Убедитесь, что вы включаете токен в заголовок в правильном формате
2. Проверьте, не истек ли срок действия токена
3. Получите новый токен через эндпоинт /auth/login

### Проблема: Ошибки 403 Forbidden

**Возможные причины**:

- Недостаточно прав для выполнения операции
- Попытка доступа к ресурсам другого пользователя

**Решение**:

1. Проверьте роль пользователя
2. Убедитесь, что пользователь имеет необходимые права
3. Запросите необходимые права у администратора
