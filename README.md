https://glas.su/du2026-06-13
# demka
          -- 1. Партнёры
          CREATE TABLE Users (
              Id INT IDENTITY(1,1) PRIMARY KEY,
              Username NVARCHAR(100) NOT NULL UNIQUE,  
              Password NVARCHAR(255) NOT NULL,
              FullName NVARCHAR(200) NULL,
              Role NVARCHAR(50) NOT NULL DEFAULT 'user',
              IsActive BIT NOT NULL DEFAULT 1,
              CreatedAt DATETIME NOT NULL DEFAULT GETDATE()                    
          );
          
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
ИМПОРТ JSON

           INSERT INTO partners (id, name, inn, address, phone, salesman, buyer) VALUES
          ('000000001', N'ООО "Поставка"', NULL, N'г.Пятигорск', N'+79198634592', 1, 1),
          ('000000002', N'ООО "Кинотеатр Квант"', N'26320045123', N'г. Железноводск, ул. Мира, 123', N'+79884581555', 1, 0),
          ('000000008', N'ООО "Новый JDTO"', N'26320045111', N'г. Железноводсу', N'+79884581555', 1, 0),
          ('000000003', N'ООО "Ромашка"', N'4140784214', N'г. Омск, ул. Строителей, 294', N'+79882584546', 0, 1),
          ('000000009', N'ООО "Ипподром"', N'5874045632', N'г. Уфа, ул. Набережная,  37', N'+79627486389', 1, 1),
          ('000000010', N'ООО "Ассоль"', N'2629011278', N'г. Калуга, ул. Пушкина, 94', N'+79184572398', 0, 1);

РАССЧЕТ ЗАКАЗА 

          USE DADAP;
          GO
          
          -- Для всех заказов:
          WITH product_material_cost AS (
              SELECT 
                  s.product_id,
                  SUM(s.quantity * m.price) AS material_cost_per_unit
              FROM specifications s
              JOIN items m ON s.material_id = m.id
              GROUP BY s.product_id
          )
          SELECT 
              o.order_number,
              SUM(o.quantity * pmc.material_cost_per_unit) AS total_material_cost,
              SUM(o.amount) AS sales_amount  -- дополнительно продажная сумма (если есть поле amount)
          FROM orders o
          LEFT JOIN product_material_cost pmc ON o.item_id = pmc.product_id
          GROUP BY o.order_number
          ORDER BY o.order_number;
