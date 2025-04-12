# Примеры использования API

## Введение

В этом документе представлены наиболее распространенные примеры использования API StorageFlow. Эти примеры помогут вам быстро начать интеграцию с системой управления складом.

## Примеры работы с товарами

### Получение списка товаров

```bash
curl -X GET http://localhost:3000/wms/v1/products
```

Ответ:

```json
[
  {
    "_id": "60d21b4667d0d8992e610c85",
    "sku": "PROD-001",
    "productId": "EXT-001",
    "name": "Смартфон XYZ",
    "barcode": "4607123456789",
    "category": "Электроника"
  }
]
```

### Получение товара по ID

```bash
curl -X GET http://localhost:3000/wms/v1/products/60d21b4667d0d8992e610c85
```

### Создание нового товара

```bash
curl -X POST http://localhost:3000/wms/v1/products \
  -H "Content-Type: application/json" \
  -d '{
    "sku": "PROD-002",
    "productId": "EXT-002",
    "name": "Планшет ABC",
    "barcode": "4607123456790",
    "category": "Электроника"
  }'
```

## Примеры работы с ячейками

### Получение списка ячеек

```bash
curl -X GET http://localhost:3000/wms/v1/locations
```

### Создание новой ячейки

```bash
curl -X POST http://localhost:3000/wms/v1/locations \
  -H "Content-Type: application/json" \
  -d '{
    "barcode": "LOC-A01-01-02",
    "zone": "A",
    "aisle": "01",
    "rack": "01",
    "level": "01",
    "position": "02",
    "capacity": 100
  }'
```

## Процесс приемки товаров

### 1. Создание накладной

```bash
curl -X POST http://localhost:3000/wms/v1/receiving \
  -H "Content-Type: application/json" \
  -d '{
    "invoiceNumber": "INV-12345",
    "barcode": "4607123456780",
    "items": [
      {
        "id": "item-001",
        "productId": "EXT-001",
        "sku": "PROD-001",
        "name": "Смартфон XYZ",
        "barcode": "4607123456789",
        "expectedQuantity": 10
      }
    ]
  }'
```

### 2. Начало приемки

```bash
curl -X POST http://localhost:3000/wms/v1/receiving/60d21b4667d0d8992e610c89/scan
```

### 3. Сканирование товара

```bash
curl -X POST http://localhost:3000/wms/v1/receiving/60d21b4667d0d8992e610c89/items/item-001/scan \
  -H "Content-Type: application/json" \
  -d '{
    "barcode": "4607123456789"
  }'
```

### 4. Подсчет товара

```bash
curl -X POST http://localhost:3000/wms/v1/receiving/60d21b4667d0d8992e610c89/items/item-001/count \
  -H "Content-Type: application/json" \
  -d '{
    "actualQuantity": 8,
    "placementCartId": "60d21b4667d0d8992e610c93"
  }'
```

### 5. Завершение приемки

```bash
curl -X POST http://localhost:3000/wms/v1/receiving/60d21b4667d0d8992e610c89/complete
```

## Размещение товаров

### Размещение товара в ячейку

```bash
curl -X POST http://localhost:3000/wms/v1/inventory/invoices/60d21b4667d0d8992e610c89/items/item-001/place \
  -H "Content-Type: application/json" \
  -d '{
    "locationId": "60d21b4667d0d8992e610c86",
    "quantity": 8
  }'
```

## Работа с заказами

### Создание заказа

```bash
curl -X POST http://localhost:3000/wms/v1/orders \
  -H "Content-Type: application/json" \
  -d '{
    "orderNumber": "ORD-12345",
    "customer": {
      "name": "Иван Иванов",
      "address": "г. Москва, ул. Примерная, д. 1",
      "phone": "+7 999 123-45-67",
      "email": "example@example.com"
    },
    "items": [
      {
        "sku": "PROD-001",
        "quantity": 2
      }
    ]
  }'
```

### Резервирование товаров для заказа

```bash
curl -X POST http://localhost:3000/wms/v1/orders/60d21b4667d0d8992e610c88/reserve
```

## Процесс сборки заказов

### 1. Создание задания на сборку

```bash
curl -X POST http://localhost:3000/wms/v1/picking \
  -H "Content-Type: application/json" \
  -d '{
    "orderIds": ["60d21b4667d0d8992e610c88"],
    "assignedTo": "emp-001"
  }'
```

### 2. Начало сборки

```bash
curl -X POST http://localhost:3000/wms/v1/picking/60d21b4667d0d8992e610c90/start \
  -H "Content-Type: application/json" \
  -d '{
    "pickingCartId": "60d21b4667d0d8992e610c92"
  }'
```

### 3. Сканирование ячейки

```bash
curl -X POST http://localhost:3000/wms/v1/picking/60d21b4667d0d8992e610c90/scan-location \
  -H "Content-Type: application/json" \
  -d '{
    "locationBarcode": "LOC-A01-01-01",
    "pickingItemId": "60d21b4667d0d8992e610c91"
  }'
```

### 4. Сборка товара

```bash
curl -X POST http://localhost:3000/wms/v1/picking/60d21b4667d0d8992e610c90/pick \
  -H "Content-Type: application/json" \
  -d '{
    "pickingItemId": "60d21b4667d0d8992e610c91",
    "quantity": 2,
    "pickingCartId": "60d21b4667d0d8992e610c92"
  }'
```

## Процесс упаковки заказов

### 1. Создание задания на упаковку

```bash
curl -X POST http://localhost:3000/wms/v1/packing \
  -H "Content-Type: application/json" \
  -d '{
    "orderId": "60d21b4667d0d8992e610c88",
    "pickingTaskId": "60d21b4667d0d8992e610c90",
    "pickingCartId": "60d21b4667d0d8992e610c92",
    "assignedTo": "emp-001"
  }'
```

### 2. Начало упаковки

```bash
curl -X POST http://localhost:3000/wms/v1/packing/60d21b4667d0d8992e610c94/start
```

### 3. Завершение упаковки

```bash
curl -X POST http://localhost:3000/wms/v1/packing/60d21b4667d0d8992e610c94/complete \
  -H "Content-Type: application/json" \
  -d '{
    "weight": 2.5,
    "packageType": "box"
  }'
```

## Процесс отгрузки заказов

### 1. Создание задания на отгрузку

```bash
curl -X POST http://localhost:3000/wms/v1/shipping \
  -H "Content-Type: application/json" \
  -d '{
    "orderId": "60d21b4667d0d8992e610c88",
    "packingTaskId": "60d21b4667d0d8992e610c94",
    "carrier": "DHL",
    "assignedTo": "emp-001"
  }'
```

### 2. Начало отгрузки

```bash
curl -X POST http://localhost:3000/wms/v1/shipping/60d21b4667d0d8992e610c95/start
```

### 3. Завершение отгрузки

```bash
curl -X POST http://localhost:3000/wms/v1/shipping/60d21b4667d0d8992e610c95/complete \
  -H "Content-Type: application/json" \
  -d '{
    "trackingNumber": "DHL1234567890"
  }'
```

## Корректировка инвентаря

```bash
curl -X POST http://localhost:3000/wms/v1/inventory/adjust \
  -H "Content-Type: application/json" \
  -d '{
    "sku": "PROD-001",
    "locationId": "60d21b4667d0d8992e610c86",
    "actualQuantity": 7,
    "notes": "Корректировка после инвентаризации"
  }'
```

## Заключение

Эти примеры покрывают основные операции API StorageFlow. Для более подробной информации обратитесь к полной документации API через Swagger UI или Redoc.
