# Приемка товаров

## Обзор процесса

Процесс приемки товаров является первым этапом в цепочке складских операций и включает регистрацию, проверку и оприходование поступающих на склад товаров. Правильно организованный процесс приемки обеспечивает точность учета, выявление расхождений и оптимальную скорость обработки входящего потока товаров.

## Участники процесса

- **Менеджер по закупкам**: создает накладную на основании заказа поставщику
- **Кладовщик**: выполняет физическую приемку товаров
- **Супервизор склада**: разрешает спорные ситуации и подтверждает расхождения
- **Система WMS**: обеспечивает учет, контроль и автоматизацию процесса

## Диаграмма процесса

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Создание      │     │   Сканирование  │     │   Сканирование  │
│   накладной     ├────►│   накладной     ├────►│   товаров       │
└─────────────────┘     └─────────────────┘     └────────┬────────┘
                                                         │
                                                         ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Завершение    │     │   Фиксация      │     │   Подсчет       │
│   приемки       │◄────┤   расхождений   │◄────┤   количества    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

## Шаги процесса

### 1. Создание накладной

**Описание**: Накладная создается в системе до физического прибытия товаров на склад.

**Действия**:
- Создание новой накладной с уникальным номером
- Добавление ожидаемых товаров и их количества
- Указание дополнительной информации (поставщик, дата, примечания)

**API Endpoint**: `POST /wms/v1/receiving`

**Входные данные**:
```json
{
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
    },
    {
      "id": "item-002",
      "productId": "EXT-002",
      "sku": "PROD-002",
      "name": "Планшет ABC",
      "barcode": "4607123456790",
      "expectedQuantity": 5
    }
  ]
}
```

**Выходные данные**: Созданная накладная со статусом `new`

**Статус накладной после шага**: `new`

### 2. Сканирование накладной

**Описание**: Кладовщик начинает процесс физической приемки, сканируя штрихкод накладной.

**Действия**:
- Сканирование штрихкода накладной с помощью терминала сбора данных (ТСД)
- Подтверждение начала приемки
- Переход к сканированию товаров

**API Endpoint**: `POST /wms/v1/receiving/{invoiceId}/scan`

**Входные данные**: ID накладной

**Выходные данные**: Обновленная накладная со статусом `in_progress`

**Статус накладной после шага**: `in_progress`

### 3. Сканирование товаров

**Описание**: Кладовщик последовательно сканирует штрихкоды каждого принимаемого товара.

**Действия**:
- Сканирование штрихкода товара
- Проверка соответствия товара накладной
- Переход к подсчету количества

**API Endpoint**: `POST /wms/v1/receiving/{invoiceId}/items/{itemId}/scan`

**Входные данные**:
```json
{
  "barcode": "4607123456789"
}
```

**Выходные данные**: Обновленная позиция накладной со статусом `scanned`

**Статус товара после шага**: `scanned`

### 4. Подсчет количества

**Описание**: Кладовщик подсчитывает фактическое количество принятого товара и фиксирует его в системе.

**Действия**:
- Физический подсчет товара
- Ввод фактического количества в систему
- Опционально: размещение товара в тележку для последующего размещения на складе

**API Endpoint**: `POST /wms/v1/receiving/{invoiceId}/items/{itemId}/count`

**Входные данные**:
```json
{
  "actualQuantity": 8,
  "placementCartId": "60d21b4667d0d8992e610c86"  // Опционально
}
```

**Выходные данные**: Обновленная позиция накладной со статусом `counted`

**Статус товара после шага**: `counted`

### 5. Фиксация расхождений

**Описание**: Если обнаружены расхождения между ожидаемым и фактическим количеством, система фиксирует эти различия.

**Действия**:
- Автоматическое сравнение ожидаемого и фактического количества
- Фиксация расхождений в системе
- При необходимости, утверждение расхождений супервизором

**Расхождения регистрируются автоматически**: Специальный API-эндпоинт не требуется, система фиксирует расхождения на основании разницы между `expectedQuantity` и `actualQuantity`

### 6. Завершение приемки

**Описание**: После обработки всех товаров в накладной кладовщик завершает процесс приемки.

**Действия**:
- Проверка, что все товары в накладной обработаны
- Подтверждение завершения приемки
- Фиксация итогового статуса накладной в зависимости от наличия расхождений

**API Endpoint**: `POST /wms/v1/receiving/{invoiceId}/complete`

**Входные данные**: ID накладной

**Выходные данные**: Обновленная накладная со статусом `accepted` или `accepted_with_discrepancies`

**Статус накладной после шага**: `accepted` (если нет расхождений) или `accepted_with_discrepancies` (если есть расхождения)

## Интеграция с другими процессами

### Связь с процессом размещения

После завершения приемки товары готовы к размещению на складе. Интеграция с процессом размещения происходит через тележки размещения:

1. Во время подсчета товаров кладовщик может указать ID тележки размещения
2. Система автоматически добавляет товар в эту тележку
3. После завершения приемки товары в тележке готовы к размещению

### Связь с управлением товарами

Если в накладной присутствуют новые товары, которых еще нет в системе, они автоматически добавляются в каталог:

1. Система проверяет наличие товара в каталоге по SKU
2. Если товар не найден, он автоматически создается с базовой информацией
3. Впоследствии информация о товаре может быть дополнена

## Обработка исключительных ситуаций

### Товар не соответствует ожидаемому

**Сценарий**: Штрихкод сканируемого товара не соответствует ни одной позиции в накладной.

**Решение**:
1. Система выдает уведомление о несоответствии
2. Кладовщик проверяет товар и накладную
3. Возможные варианты:
   - Отложить товар для дальнейшего выяснения
   - Связаться с супервизором для принятия решения

### Значительные расхождения по количеству

**Сценарий**: Фактическое количество значительно отличается от ожидаемого (например, >20%).

**Решение**:
1. Система помечает расхождение как значительное
2. Уведомляется супервизор склада
3. Требуется дополнительное подтверждение и указание причины расхождения

### Поврежденные товары

**Сценарий**: При приемке обнаружены поврежденные товары.

**Решение**:
1. Фиксация фактического количества неповрежденных товаров
2. Отметка о повреждении в комментариях
3. Создание отдельной заявки на возврат поставщику при необходимости

## Метрики и KPI

### Основные показатели эффективности

1. **Время обработки накладной**:
   - Среднее время от начала сканирования до завершения приемки
   - Целевое значение: < 30 минут на накладную из 20 позиций

2. **Точность приемки**:
   - Процент накладных без расхождений
   - Целевое значение: > 95%

3. **Скорость обработки**:
   - Количество обработанных товаров в час
   - Целевое значение: > 100 единиц/час

### Мониторинг процесса

Система автоматически отслеживает:
- Количество накладных в обработке
- Среднее время обработки
- Процент расхождений
- Производительность каждого кладовщика

## Оптимизация процесса

### Рекомендации по повышению эффективности

1. **Предварительная подготовка**:
   - Заблаговременное создание накладных
   - Подготовка зоны приемки перед прибытием товаров

2. **Использование мобильных устройств**:
   - Применение беспроводных сканеров
   - Использование мобильных терминалов сбора данных

3. **Оптимизация рабочего пространства**:
   - Организация эргономичных рабочих мест
   - Выделение отдельных зон для разных типов товаров

4. **Автоматизация**:
   - Интеграция с системами поставщиков для автоматического создания накладных
   - Использование весового оборудования для быстрого подсчета однотипных товаров

## Примеры использования API

### Пример создания накладной

```bash
curl -X POST https://example.com/wms/v1/receiving \
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

### Пример начала приемки

```bash
curl -X POST https://example.com/wms/v1/receiving/60d21b4667d0d8992e610c85/scan
```

### Пример сканирования товара

```bash
curl -X POST https://example.com/wms/v1/receiving/60d21b4667d0d8992e610c85/items/item-001/scan \
  -H "Content-Type: application/json" \
  -d '{
    "barcode": "4607123456789"
  }'
```

### Пример подсчета товара

```bash
curl -X POST https://example.com/wms/v1/receiving/60d21b4667d0d8992e610c85/items/item-001/count \
  -H "Content-Type: application/json" \
  -d '{
    "actualQuantity": 8,
    "placementCartId": "60d21b4667d0d8992e610c86"
  }'
```

### Пример завершения приемки

```bash
curl -X POST https://example.com/wms/v1/receiving/60d21b4667d0d8992e610c85/complete
```
