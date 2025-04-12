# Стек технологий

## Обзор

StorageFlow построен на основе современного JavaScript/Node.js стека с использованием MongoDB в качестве базы данных. Система реализует архитектуру REST API и использует Express.js в качестве веб-фреймворка.

## Серверная часть

### Основные технологии

- **Node.js** - JavaScript рантайм для создания серверных приложений
- **Express.js** - Веб-фреймворк для Node.js
- **MongoDB** - Документо-ориентированная NoSQL база данных
- **Mongoose** - ODM (Object Document Mapper) для MongoDB

### Дополнительные библиотеки

- **dotenv** - Загрузка переменных окружения из .env файлов
- **express-openapi-validator** - Валидация API запросов на основе OpenAPI спецификации
- **swagger-jsdoc** и **swagger-ui-express** - Документация API в формате OpenAPI
- **redoc-express** - Альтернативный UI для отображения OpenAPI документации

## Версии компонентов

```json
{
  "dependencies": {
    "dotenv": "^16.4.7",
    "express": "^4.21.2",
    "express-openapi-validator": "^5.4.6",
    "mongoose": "^8.12.1",
    "redoc-express": "^2.1.0",
    "swagger-jsdoc": "^6.2.8",
    "swagger-ui-express": "^5.0.1"
  }
}
```

## Структура проекта

```
root/
├── app.js                  # Основной файл приложения
├── index.js                # Точка входа
├── config/
│   └── db.js               # Конфигурация базы данных
├── controllers/            # Контроллеры для обработки запросов
├── models/                 # Mongoose модели
├── routes/                 # Express маршруты
├── swagger.js              # Настройка документации API
└── package.json            # Зависимости и скрипты
```

## Ключевые компоненты

### Express.js маршрутизация

StorageFlow использует модульную структуру маршрутов для организации API:

```javascript
// app.js
app.use("/wms/v1/receiving", receivingRoutes);
app.use("/wms/v1/locations", locationsRoutes);
app.use("/wms/v1/placement-carts", placementCartRoutes);
app.use("/wms/v1/inventory", inventoryRoutes);
app.use("/wms/v1/products", productsRoutes);
app.use("/wms/v1/orders", ordersRoutes);
app.use("/wms/v1/picking", pickingRoutes);
app.use("/wms/v1/picking-carts", pickingCartRoutes);
app.use("/wms/v1/packing", packingRoutes);
app.use("/wms/v1/shipping", shippingRoutes);
```

### MongoDB и Mongoose

Система использует MongoDB для хранения данных и Mongoose для определения схем и работы с базой данных:

```javascript
// models/product.js
import mongoose from "mongoose";

const { Schema } = mongoose;

const productSchema = new Schema({
  sku: { type: String, required: true, unique: true },
  productId: { type: String, required: true },
  name: { type: String, required: true },
  barcode: { type: String },
  category: { type: String, default: "Общая" },
  dimensions: {
    length: { type: Number },
    width: { type: Number },
    height: { type: Number },
    weight: { type: Number },
  },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now },
});

const Product = mongoose.model("Product", productSchema);

export default Product;
```

### Документация API

Документация API генерируется автоматически из JSDoc комментариев в коде с помощью swagger-jsdoc и доступна через Swagger UI и Redoc:

```javascript
// swagger.js
const options = {
  definition: {
    openapi: "3.0.0",
    info: {
      title: "WMS API",
      version: "1.0.0",
      description: "API документация системы управления складом",
    },
    // ...
  },
  apis: [
    "./routes/*.js",
    "./controllers/*.js",
    "./models/*.js",
    "./schemas.js",
  ],
};
```

## Требования к среде

- Node.js 16.x или выше
- MongoDB 5.0 или выше

## Дальнейшее масштабирование

Система спроектирована с учетом возможности масштабирования:

- **Горизонтальное масштабирование** - Благодаря безсостояниевой архитектуре можно запускать несколько экземпляров приложения
- **Шардирование MongoDB** - Для работы с большими объемами данных
- **Микросервисная архитектура** - Возможность разделения на независимые сервисы в будущем

## Рекомендации по развертыванию

- **Контейнеризация** - Система может быть легко контейнеризована с использованием Docker
- **CI/CD** - Рекомендуется настроить конвейер для автоматического тестирования и развертывания
- **Мониторинг** - Интеграция с PM2, Prometheus или другими системами мониторинга
