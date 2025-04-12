# Обзор моделей данных

## Введение

Система управления складом StorageFlow использует MongoDB в качестве базы данных, что обеспечивает гибкую схему данных и эффективную обработку документо-ориентированных данных. Модели данных разработаны с учетом специфики складских процессов и оптимизированы для высокой производительности.

## ER-диаграмма

Ниже представлена ER-диаграмма, отображающая основные сущности системы и их взаимосвязи:

```
┌───────────────┐      ┌────────────────┐      ┌────────────────┐
│   Product     │      │   Inventory    │      │    Location    │
├───────────────┤      ├────────────────┤      ├────────────────┤
│ _id           │◄─────┤ sku            │      │ _id            │◄─────┐
│ sku*          │      │ quantity       │      │ barcode*       │      │
│ productId     │      │ locationId     ├─────►│ zone           │      │
│ name          │      │ status         │      │ aisle          │      │
│ barcode       │      │ createdAt      │      │ rack           │      │
│ category      │      └────────────────┘      │ level          │      │
│ dimensions    │                              │ position       │      │
│ createdAt     │                              │ status         │      │
│ updatedAt     │                              │ capacity       │      │
└───────────────┘                              │ usedCapacity   │      │
        ▲                                      │ createdAt      │      │
        │                                      └────────────────┘      │
        │                                                              │
        │        ┌────────────────┐            ┌────────────────┐      │
        │        │     Order      │            │   PickingTask  │      │
        │        ├────────────────┤            ├────────────────┤      │
        │        │ _id            │◄───────────┤ orderIds       │      │
        │        │ orderNumber*   │            │ assignedTo     │      │
        │        │ externalOrderId│            │ status         │      │
        │        │ customer       │            │ items          ├──────┘
        └────────┤ status         │            │ startedAt      │
                 │ priority       │            │ completedAt    │
                 │ items          │            │ createdAt      │
                 │ createdAt      │            └──────┬─────────┘
                 │ updatedAt      │                   │
                 └────────────────┘                   │
                        ▲                             │
                        │                             │
                        │                             │
             ┌──────────┴─────────┐                   │
             │                    │                   │
             │                    │                   │
┌────────────▼───────┐ ┌──────────▼─────────┐ ┌──────▼─────────────┐
│   PackingTask      │ │   ShippingTask     │ │    PickingCart     │
├────────────────────┤ ├────────────────────┤ ├────────────────────┤
│ _id                │ │ _id                │ │ _id                │
│ orderId            │ │ orderId            │ │ barcode*           │
│ pickingTaskId      │ │ packingTaskId      │ │ status             │
│ pickingCartId      │ │ carrier            │ │ assignedTo         │
│ assignedTo         │ │ trackingNumber     │ │ pickingTaskId      │
│ status             │ │ assignedTo         │ │ items              │
│ packageInfo        │ │ status             │ │ createdAt          │
│ startedAt          │ │ shippingAddress    │ │ updatedAt          │
│ completedAt        │ │ startedAt          │ └────────────────────┘
│ createdAt          │ │ completedAt        │         ▲
└────────────────────┘ │ createdAt          │         │
                       └────────────────────┘         │
                                                      │
                                                      │
                       ┌────────────────┐    ┌────────┴───────────┐
                       │    Invoice     │    │   PlacementCart   │
                       ├────────────────┤    ├────────────────────┤
                       │ _id            │    │ _id                │
                       │ invoiceNumber* │    │ status             │
                       │ barcode        │    │ items              │
                       │ status         │    │ createdAt          │
                       │ items          │    └────────────────────┘
                       │ createdAt      │
                       └────────────────┘
```

## Основные модели данных

### 1. Product (Товар)

**Описание**: Содержит информацию о товарах в каталоге.

**Ключевые поля**:
- `_id`: Уникальный идентификатор MongoDB
- `sku`: Уникальный SKU товара (индексированное поле)
- `productId`: Внешний идентификатор товара
- `name`: Наименование товара
- `barcode`: Штрихкод товара
- `category`: Категория товара
- `dimensions`: Габариты (length, width, height, weight)

**Использование**:
- Хранение каталога товаров
- Связывание с инвентарем через SKU
- Предоставление информации для сборки и размещения

**Пример документа**:
```json
{
  "_id": "60d21b4667d0d8992e610c85",
  "sku": "PROD-001",
  "productId": "EXT-001",
  "name": "Смартфон XYZ",
  "barcode": "4607123456789",
  "category": "Электроника",
  "dimensions": {
    "length": 15,
    "width": 7.5,
    "height": 1.5,
    "weight": 0.2
  },
  "createdAt": "2025-03-01T10:00:00Z",
  "updatedAt": "2025-03-10T14:30:00Z"
}
```

### 2. Location (Ячейка)

**Описание**: Представляет физические места хранения на складе.

**Ключевые поля**:
- `_id`: Уникальный идентификатор MongoDB
- `barcode`: Уникальный штрихкод ячейки (индексированное поле)
- `zone`, `aisle`, `rack`, `level`, `position`: Координаты ячейки в складе
- `status`: Статус ячейки (`available`, `occupied`, `reserved`)
- `capacity`: Максимальная вместимость
- `usedCapacity`: Использованная вместимость

**Использование**:
- Управление пространством склада
- Определение мест размещения товаров
- Оптимизация сборки заказов

**Пример документа**:
```json
{
  "_id": "60d21b4667d0d8992e610c86",
  "barcode": "LOC-A01-01-01",
  "zone": "A",
  "aisle": "01",
  "rack": "01",
  "level": "01",
  "position": "01",
  "status": "available",
  "capacity": 100,
  "usedCapacity": 0,
  "createdAt": "2025-03-01T10:00:00Z"
}
```

### 3. Inventory (Инвентарь)

**Описание**: Связывает товары с их физическим расположением и количеством.

**Ключевые поля**:
- `_id`: Уникальный идентификатор MongoDB
- `sku`: SKU товара (индексированное поле)
- `quantity`: Количество товара
- `locationId`: ID ячейки (индексированное поле)
- `status`: Статус инвентаря (обычно `placed`)

**Использование**:
- Отслеживание фактических остатков
- Определение местоположения товаров
- Резервирование товаров для заказов

**Пример документа**:
```json
{
  "_id": "60d21b4667d0d8992e610c87",
  "sku": "PROD-001",
  "quantity": 10,
  "locationId": "60d21b4667d0d8992e610c86",
  "status": "placed",
  "createdAt": "2025-03-12T10:00:00Z"
}
```

### 4. Order (Заказ)

**Описание**: Содержит информацию о заказах клиентов.

**Ключевые поля**:
- `_id`: Уникальный идентификатор MongoDB
- `orderNumber`: Уникальный номер заказа (индексированное поле)
- `externalOrderId`: Внешний идентификатор заказа
- `customer`: Информация о клиенте
- `status`: Статус заказа (от `new` до `delivered`)
- `priority`: Приоритет обработки
- `items`: Массив заказанных товаров

**Использование**:
- Управление жизненным циклом заказа
- Основа для создания заданий на сборку
- Отслеживание выполнения заказа

**Пример документа**:
```json
{
  "_id": "60d21b4667d0d8992e610c88",
  "orderNumber": "ORD-12345",
  "externalOrderId": "EXT-12345",
  "customer": {
    "name": "Иван Иванов",
    "address": "г. Москва, ул. Примерная, д. 1",
    "phone": "+7 999 123-45-67",
    "email": "example@example.com"
  },
  "status": "new",
  "priority": 1,
  "items": [
    {
      "productId": "60d21b4667d0d8992e610c85",
      "sku": "PROD-001",
      "name": "Смартфон XYZ",
      "quantity": 2,
      "pickedQuantity": 0,
      "status": "pending"
    }
  ],
  "createdAt": "2025-03-10T10:00:00Z",
  "updatedAt": "2025-03-10T10:00:00Z"
}
```

### 5. Invoice (Накладная)

**Описание**: Документы для приемки товаров на склад.

**Ключевые поля**:
- `_id`: Уникальный идентификатор MongoDB
- `invoiceNumber`: Уникальный номер накладной (индексированное поле)
- `barcode`: Штрихкод накладной
- `status`: Статус накладной (`new`, `in_progress`, `accepted`, `accepted_with_discrepancies`)
- `items`: Массив принимаемых товаров

**Использование**:
- Управление процессом приемки
- Контроль расхождений
- Подготовка к размещению товаров

**Пример документа**:
```json
{
  "_id": "60d21b4667d0d8992e610c89",
  "invoiceNumber": "INV-12345",
  "barcode": "4607123456780",
  "status": "new",
  "items": [
    {
      "id": "item-001",
      "productId": "EXT-001",
      "sku": "PROD-001",
      "name": "Смартфон XYZ",
      "barcode": "4607123456789",
      "expectedQuantity": 10,
      "actualQuantity": 0,
      "placedQuantity": 0,
      "status": "pending",
      "placementCartId": null
    }
  ],
  "createdAt": "2025-03-10T10:00:00Z"
}
```

### 6. PickingTask (Задание на сборку)

**Описание**: Задания на сборку товаров для заказов.

**Ключевые поля**:
- `_id`: Уникальный идентификатор MongoDB
- `orderIds`: Массив ID заказов для сборки
- `assignedTo`: ID сотрудника
- `status`: Статус задания (`created`, `assigned`, `in_progress`, `completed`, `cancelled`)
- `items`: Массив товаров для сборки с указанием их местоположения
- `startedAt`, `completedAt`: Время начала и завершения сборки

**Использование**:
- Оптимизация маршрута сборки
- Отслеживание выполнения сборки
- Управление производительностью сборщиков

**Пример документа**:
```json
{
  "_id": "60d21b4667d0d8992e610c90",
  "orderIds": ["60d21b4667d0d8992e610c88"],
  "assignedTo": "emp-001",
  "status": "created",
  "items": [
    {
      "orderId": "60d21b4667d0d8992e610c88",
      "orderItemId": "60d21b4667d0d8992e610c91",
      "productId": "60d21b4667d0d8992e610c85",
      "sku": "PROD-001",
      "name": "Смартфон XYZ",
      "quantity": 2,
      "pickedQuantity": 0,
      "locationId": "60d21b4667d0d8992e610c86",
      "status": "pending"
    }
  ],
  "startedAt": null,
  "completedAt": null,
  "createdAt": "2025-03-12T10:00:00Z"
}
```

### 7. PickingCart (Тележка сборки)

**Описание**: Тележки для сборки товаров.

**Ключевые поля**:
- `_id`: Уникальный идентификатор MongoDB
- `barcode`: Уникальный штрихкод тележки
- `status`: Статус тележки (`free`, `assigned`, `in_use`, `complete`)
- `assignedTo`: ID сотрудника
- `pickingTaskId`: ID задания на сборку
- `items`: Собранные товары в тележке

**Использование**:
- Перемещение товаров при сборке
- Контроль собранных товаров
- Организация процесса упаковки

**Пример документа**:
```json
{
  "_id": "60d21b4667d0d8992e610c92",
  "barcode": "PICK-CART-1626456789",
  "status": "free",
  "assignedTo": null,
  "pickingTaskId": null,
  "items": [],
  "createdAt": "2025-03-10T10:00:00Z",
  "updatedAt": "2025-03-10T10:00:00Z"
}
```

### 8. PlacementCart (Тележка размещения)

**Описание**: Тележки для размещения принятых товаров.

**Ключевые поля**:
- `_id`: Уникальный идентификатор MongoDB
- `status`: Статус тележки (`free`, `occupied`)
- `items`: Товары для размещения

**Использование**:
- Перемещение товаров от зоны приемки к местам хранения
- Контроль процесса размещения

**Пример документа**:
```json
{
  "_id": "60d21b4667d0d8992e610c93",
  "status": "free",
  "items": [],
  "createdAt": "2025-03-10T10:00:00Z"
}
```

### 9. PackingTask (Задание на упаковку)

**Описание**: Задания на упаковку собранных заказов.

**Ключевые поля**:
- `_id`: Уникальный идентификатор MongoDB
- `orderId`: ID заказа
- `pickingTaskId`: ID задания на сборку
- `pickingCartId`: ID тележки сборки
- `assignedTo`: ID сотрудника
- `status`: Статус задания (`created`, `in_progress`, `completed`, `cancelled`)
- `packageInfo`: Информация об упаковке (вес, тип)
- `startedAt`, `completedAt`: Время начала и завершения упаковки

**Использование**:
- Управление процессом упаковки
- Подготовка к отгрузке
- Отслеживание выполнения упаковки

**Пример документа**:
```json
{
  "_id": "60d21b4667d0d8992e610c94",
  "orderId": "60d21b4667d0d8992e610c88",
  "pickingTaskId": "60d21b4667d0d8992e610c90",
  "pickingCartId": "60d21b4667d0d8992e610c92",
  "assignedTo": "emp-001",
  "status": "created",
  "packageInfo": null,
  "startedAt": null,
  "completedAt": null,
  "createdAt": "2025-03-12T11:15:00Z"
}
```

### 10. ShippingTask (Задание на отгрузку)

**Описание**: Задания на отгрузку упакованных заказов.

**Ключевые поля**:
- `_id`: Уникальный идентификатор MongoDB
- `orderId`: ID заказа
- `packingTaskId`: ID задания на упаковку
- `carrier`: Перевозчик
- `trackingNumber`: Номер отслеживания
- `assignedTo`: ID сотрудника
- `status`: Статус задания (`created`, `in_progress`, `completed`, `cancelled`)
- `shippingAddress`: Адрес доставки
- `startedAt`, `completedAt`: Время начала и завершения отгрузки

**Использование**:
- Управление процессом отгрузки
- Отслеживание отправлений
- Интеграция с перевозчиками

**Пример документа**:
```json
{
  "_id": "60d21b4667d0d8992e610c95",
  "orderId": "60d21b4667d0d8992e610c88",
  "packingTaskId": "60d21b4667d0d8992e610c94",
  "carrier": "DHL",
  "trackingNumber": null,
  "assignedTo": "emp-001",
  "status": "created",
  "shippingAddress": {
    "name": "Иван Иванов",
    "address": "г. Москва, ул. Примерная, д. 1"
  },
  "startedAt": null,
  "completedAt": null,
  "createdAt": "2025-03-12T14:00:00Z"
}
```

## Статусные модели и переходы состояний

### Заказ (Order)

**Статусы**:
- `new`: Новый заказ
- `processing`: В обработке
- `picking`: На сборке
- `picked`: Собран
- `packing`: На упаковке
- `packed`: Упакован
- `shipping`: На отгрузке
- `shipped`: Отгружен
- `delivered`: Доставлен
- `cancelled`: Отменен

**Переходы**:
```
new → processing → picking → picked → packing → packed → shipping → shipped → delivered
  └─ cancelled (может быть отменен на любом этапе)
```

### Накладная (Invoice)

**Статусы**:
- `new`: Новая накладная
- `in_progress`: Накладная в процессе приемки
- `accepted`: Принята без расхождений
- `accepted_with_discrepancies`: Принята с расхождениями

**Переходы**:
```
new → in_progress → accepted
           └─ accepted_with_discrepancies
```

### Задание на сборку (PickingTask)

**Статусы**:
- `created`: Создано
- `assigned`: Назначено сборщику
- `in_progress`: В процессе выполнения
- `completed`: Завершено
- `cancelled`: Отменено

**Переходы**:
```
created → assigned → in_progress → completed
     └─ cancelled (может быть отменено до завершения)
```

## Особенности организации данных

### Индексы

Для оптимизации производительности используются следующие индексы:

- **Product**: Индексы по `sku` и `barcode`
- **Inventory**: Индексы по `sku` и `locationId`
- **Location**: Индекс по `barcode`
- **Order**: Индекс по `orderNumber`
- **Invoice**: Индекс по `invoiceNumber`

### Встроенные документы vs. Ссылки

Система использует комбинированный подход:

- **Встроенные документы**: Используются для тесно связанных данных (например, `items` в заказах и накладных)
- **Ссылки**: Используются для связей между основными сущностями (например, `orderId` в заданиях на сборку)

Такой подход обеспечивает баланс между производительностью и гибкостью.

### Временные метки

Все модели включают временные метки:
- `createdAt`: Время создания
- `updatedAt`: Время последнего обновления (где применимо)
- `startedAt`, `completedAt`: Время начала и завершения процессов (для задач)

Это обеспечивает возможность аудита, анализа производительности и отслеживания истории изменений.

## Валидация данных

Mongoose используется для валидации данных на уровне моделей:

- Обязательные поля помечены с `required: true`
- Для полей с ограниченным набором значений используется `enum`
- Применяются кастомные валидаторы для специфических бизнес-правил

Пример валидации в модели Location:
```javascript
const locationSchema = new Schema({
  barcode: { type: String, required: true, unique: true },
  zone: { type: String, required: true },
  aisle: { type: String, required: true },
  rack: { type: String, required: true },
  level: { type: String, required: true },
  position: { type: String, required: true },
  status: { 
    type: String, 
    enum: ['available', 'occupied', 'reserved'], 
    default: 'available' 
  },
  capacity: { type: Number, required: true },
  usedCapacity: { type: Number, default: 0 },
  createdAt: { type: Date, default: Date.now },
});
```

## Пример рабочего процесса и взаимодействия моделей

### Создание и выполнение заказа

1. Создается `Order` со статусом `new`
2. После резервирования товаров статус меняется на `processing`
3. Создается `PickingTask`, связанное с заказом
4. Статус заказа меняется на `picking`
5. После завершения сборки (`PickingTask.status = completed`) статус заказа меняется на `picked`
6. Создается `PackingTask` для упаковки заказа
7. Статус заказа меняется на `packing`
8. После завершения упаковки статус заказа меняется на `packed`
9. Создается `ShippingTask` для отгрузки заказа
10. Статус заказа меняется на `shipping`
11. После отгрузки статус заказа меняется на `shipped`
12. После подтверждения доставки статус меняется на `delivered`

### Приемка и размещение товаров

1. Создается `Invoice` со статусом `new`
2. После начала приемки статус меняется на `in_progress`
3. Товары сканируются, подсчитываются и добавляются в `PlacementCart`
4. После завершения приемки статус меняется на `accepted` или `accepted_with_discrepancies`
5. Товары из `PlacementCart` размещаются по ячейкам
6. Создаются записи в `Inventory`, связывающие товары с ячейками
7. Обновляется `usedCapacity` и `status` в соответствующих ячейках `Location`
