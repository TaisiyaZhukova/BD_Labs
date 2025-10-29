# Проектирование базы данных для продажи спортивных товаров

## Исходные данные
Имеется таблица с данными о продажах спортивных товаров по каталогу. Требуется спроектировать реляционную БД, соответствующую третьей нормальной форме (3НФ).

## Анализ предметной области

### Изменения:
- Заказчик идентифицируется уникальным номером
- Товар идентифицируется каталожным номером  
- Цена товара определяется только его номером в каталоге
- В одном заказе несколько штук одного товара указываются в поле Quantity
- Заказ идентифицируется уникальным номером и делается определенным заказчиком

## Спроектированная схема базы данных

### 1. Таблица `Customers` (Заказчики)

| Поле | Тип данных | Ограничения | Описание |
|------|------------|-------------|----------|
| **CustomerNum** | INT | `PRIMARY KEY`, `NOT NULL` | Уникальный номер клиента |
| CustomerName | VARCHAR(100) | `NOT NULL` | ФИО клиента |
| CustomerAddress | VARCHAR(255) | `NOT NULL` | Адрес клиента |
| CustomerPhone | VARCHAR(20) | `NOT NULL` | Телефон клиента |

### 2. Таблица `Products` (Товары)

| Поле | Тип данных | Ограничения | Описание |
|------|------------|-------------|----------|
| **CatalogNum** | INT | `PRIMARY KEY`, `NOT NULL` | Каталожный номер |
| Product | VARCHAR(100) | `NOT NULL` | Название товара |
| Price | DECIMAL(10, 2) | `NOT NULL` | Цена товара |

### 3. Таблица `Orders` (Заказы)

| Поле | Тип данных | Ограничения | Описание |
|------|------------|-------------|----------|
| **OrderNum** | INT | `PRIMARY KEY`, `NOT NULL` | Номер заказа |
| CustomerNum | INT | `FOREIGN KEY`, `NOT NULL` | Номер клиента |
| OrderDate | DATE | `NOT NULL` | Дата заказа |

### 4. Таблица `OrderDetails` (Состав заказа)

| Поле | Тип данных | Ограничения | Описание |
|------|------------|-------------|----------|
| **OrderNum** | INT | `PRIMARY KEY`, `FOREIGN KEY`, `NOT NULL` | Номер заказа |
| **CatalogNum** | INT | `PRIMARY KEY`, `FOREIGN KEY`, `NOT NULL` | Каталожный номер товара |
| Quantity | INT | `NOT NULL` | Количество товара в заказе |

## Связи между таблицами

### 1. `Customers` → `Orders` (1 ко Многим)
- Один клиент может сделать много заказов
- Один заказ принадлежит только одному клиенту

### 2. `Orders` → `OrderDetails` (1 ко Многим)  
- Один заказ может содержать несколько строк с товарами
- Одна строка состава заказа принадлежит только одному заказу

### 3. `Products` → `OrderDetails` (1 ко Многим)
- Один товар может входить в состав многих заказов
- Одна строка состава заказа ссылается только на один товар

## Внешние ключи

- **Таблица `Orders`:**
  - `CustomerNum` → `Customers(CustomerNum)`

- **Таблица `OrderDetails`:**
  - `OrderNum` → `Orders(OrderNum)`
  - `CatalogNum` → `Products(CatalogNum)`

## Обоснование соответствия ЗНФ

Разработанная схема соответствует третьей нормальной форме потому что:

1. **Устранена избыточность данных** - информация о клиентах и товарах хранится в одном месте
2. **Нет транзитивных зависимостей** - все неключевые атрибуты зависят только от первичного ключа
3. **Устранены повторяющиеся группы** - связь "многие ко многим" между заказами и товарами решена через связующую таблицу `OrderDetails`

## Примеры запросов для создания таблиц (SQL)

```sql
CREATE TABLE Customers (
    CustomerNum INT PRIMARY KEY NOT NULL,
    CustomerName VARCHAR(100) NOT NULL,
    CustomerAddress VARCHAR(255) NOT NULL,
    CustomerPhone VARCHAR(20) NOT NULL
);

CREATE TABLE Products (
    CatalogNum INT PRIMARY KEY NOT NULL,
    Product VARCHAR(100) NOT NULL,
    Price DECIMAL(10,2) NOT NULL
);

CREATE TABLE Orders (
    OrderNum INT PRIMARY KEY NOT NULL,
    CustomerNum INT NOT NULL,
    OrderDate DATE NOT NULL,
    FOREIGN KEY (CustomerNum) REFERENCES Customers(CustomerNum)
);

CREATE TABLE OrderDetails (
    OrderNum INT NOT NULL,
    CatalogNum INT NOT NULL,
    Quantity INT NOT NULL,
    PRIMARY KEY (OrderNum, CatalogNum),
    FOREIGN KEY (OrderNum) REFERENCES Orders(OrderNum),
    FOREIGN KEY (CatalogNum) REFERENCES Products(CatalogNum)
);
