CREATE TABLE materials (
    code VARCHAR(10) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    unit VARCHAR(20),
    min_balance INTEGER DEFAULT 0,
    category VARCHAR(50),
    price DECIMAL(10,2) CHECK (price >= 0)
);
-- 2. Создание таблицы "Поставщики"
CREATE TABLE suppliers (
    code VARCHAR(10) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    inn VARCHAR(12),
    contact_person VARCHAR(255),
    phone VARCHAR(20)
);
-- 3. Создание таблицы "Сотрудники"
CREATE TABLE employees (
    code VARCHAR(10) PRIMARY KEY,
    full_name VARCHAR(255) NOT NULL,
    position VARCHAR(100),
    data VARCHAR(20),
    phone VARCHAR(20)
);
-- 4. Создание таблицы "Подразделения"
CREATE TABLE departments (
    code VARCHAR(10) PRIMARY KEY,
    name VARCHAR(255),
    full_name VARCHAR(255),
    phone VARCHAR(20)
);
-- 5. Создание таблицы "Поступление материалов"
CREATE TABLE receipts (
    code VARCHAR(10) PRIMARY KEY,
    date DATE NOT NULL,
    supplier_code VARCHAR(10) REFERENCES suppliers(code),
    responsible_code VARCHAR(10) REFERENCES employees(code)
);
-- 6. Создание таблицы "Строки поступления"
CREATE TABLE receipt_items (
    receipt_code VARCHAR(10) REFERENCES receipts(code) ON DELETE CASCADE,
    material_code VARCHAR(10) REFERENCES materials(code),
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    price DECIMAL(10,2) NOT NULL CHECK (price >= 0),
    amount DECIMAL(10,2) GENERATED ALWAYS AS (quantity * price) STORED,
    PRIMARY KEY (receipt_code, material_code)
);
-- 7. Создание таблицы "Выдача материалов"
CREATE TABLE issues (
    code VARCHAR(10) PRIMARY KEY,
    date DATE NOT NULL,
    employee_code VARCHAR(10) REFERENCES employees(code),
    department_code VARCHAR(10) REFERENCES departments(code)
);
-- 8. Создание таблицы "Строки выдачи"
CREATE TABLE issue_items (
    issue_code VARCHAR(10) REFERENCES issues(code) ON DELETE CASCADE,
    material_code VARCHAR(10) REFERENCES materials(code),
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    price DECIMAL(10,2) NOT NULL CHECK (price >= 0),
    amount DECIMAL(10,2) GENERATED ALWAYS AS (quantity * price) STORED,
    PRIMARY KEY (issue_code, material_code)
);
-- 9. Создание таблицы "Остатки материалов"
CREATE TABLE material_balances (
    period DATE NOT NULL,
    material_code VARCHAR(10) REFERENCES materials(code),
    quantity INTEGER NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (period, material_code)
);
-- 10. Документ "Ввод начальных остатков"
CREATE TABLE opening_balances (
    code VARCHAR(3) PRIMARY KEY,
    date DATE NOT NULL,
    responsible_code VARCHAR(10) REFERENCES employees(code)
);

-- 11. Строки начальных остатков
CREATE TABLE opening_balance_items (
    balance_code VARCHAR(3) REFERENCES opening_balances(code) ON DELETE CASCADE,
    material_code VARCHAR(10) REFERENCES materials(code),
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    amount DECIMAL(10,2) NOT NULL CHECK (amount >= 0),
    price DECIMAL(10,2) GENERATED ALWAYS AS (amount / quantity) STORED,
    PRIMARY KEY (balance_code, material_code)
);

