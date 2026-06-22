# demka
-- 1. Партнёры
CREATE TABLE partners (
    id VARCHAR(20) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    inn VARCHAR(20),
    address TEXT,
    phone VARCHAR(50),
    salesman BIT,
    buyer BIT
);

-- 2. Номенклатура (продукты и материалы)
CREATE TABLE items (
    id INT IDENTITY(1,1) PRIMARY KEY,
    code VARCHAR(50) UNIQUE,
    name VARCHAR(255) NOT NULL,
    unit VARCHAR(20),
    type VARCHAR(20) CHECK (type IN ('product', 'material')),
    price DECIMAL(10, 2)
);

-- 3. Заказы (одна таблица, каждая строка – позиция заказа)
CREATE TABLE orders (
    id INT IDENTITY(1,1) PRIMARY KEY,
    order_number VARCHAR(50) NOT NULL,
    order_date DATE NOT NULL,
    executor_id VARCHAR(20) REFERENCES partners(id),   
    customer_id VARCHAR(20) REFERENCES partners(id),
    item_id INTEGER REFERENCES items(id),
    quantity DECIMAL(10, 3) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,  -- цена на момент заказа
    amount DECIMAL(10, 2)           -- quantity * price (можно вычислять, но оставлено для удобства)
);

-- 4. Производство (строки: тип 'product' – готовая продукция, 'material' – расход материала)
CREATE TABLE productions (
    id INT IDENTITY(1,1) PRIMARY KEY,
    production_number VARCHAR(50) NOT NULL,
    production_date DATE NOT NULL,
    line_type VARCHAR(20) CHECK (line_type IN ('product', 'material')) NOT NULL,
    item_id INTEGER REFERENCES items(id),
    quantity DECIMAL(10, 3) NOT NULL
);

-- 5. Спецификации (рецепты)
CREATE TABLE specifications (
    id INT IDENTITY(1,1) PRIMARY KEY,
    product_id INTEGER REFERENCES items(id),
    material_id INTEGER REFERENCES items(id),
    quantity DECIMAL(10, 3) NOT NULL,
    UNIQUE (product_id, material_id)
);
