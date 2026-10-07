SELECT
    m.code AS "Код",
    m.name AS "Наименование",
    m.unit AS "Ед.изм",
    mb.quantity AS "Количество",
    mb.amount AS "Сумма"
FROM material_balances mb
JOIN materials m ON mb.material_code = m.code
WHERE mb.period = '2026-02-06'
ORDER BY m.code;
----------------------------------------------------
SELECT
    m.code AS "Код",
    m.name AS "Наименование",
    COALESCE((
        SELECT SUM(quantity) FROM receipt_items WHERE material_code = m.code
    ), 0) AS "Приход кол-во",
    COALESCE((
        SELECT SUM(amount) FROM receipt_items WHERE material_code = m.code
    ), 0) AS "Приход сумма",
    COALESCE((
        SELECT SUM(quantity) FROM issue_items WHERE material_code = m.code
    ), 0) AS "Расход кол-во",
    COALESCE((
        SELECT SUM(amount) FROM issue_items WHERE material_code = m.code
    ), 0) AS "Расход сумма",
    COALESCE((
        SELECT SUM(quantity) FROM material_balances WHERE material_code = m.code AND period = '2026-02-06'
    ), 0) AS "Остаток кол-во",
    COALESCE((
        SELECT SUM(amount) FROM material_balances WHERE material_code = m.code AND period = '2026-02-06'
    ), 0) AS "Остаток сумма"
FROM materials m
ORDER BY m.code;
------------------------------------------------------------
SELECT
    s.name AS "Поставщик",
    COUNT(DISTINCT r.code) AS "Количество документов",
    SUM(ri.quantity) AS "Всего количество",
    SUM(ri.amount) AS "Всего сумма"
FROM receipts r
JOIN suppliers s ON r.supplier_code = s.code
JOIN receipt_items ri ON r.code = ri.receipt_code
GROUP BY s.name
ORDER BY SUM(ri.amount) DESC;
---------------------------------------------------------
SELECT
    d.name AS "Подразделение",
    COUNT(DISTINCT i.code) AS "Количество документов",
    SUM(ii.quantity) AS "Всего количество",
    SUM(ii.amount) AS "Всего сумма"
FROM issues i
JOIN departments d ON i.department_code = d.code
JOIN issue_items ii ON i.code = ii.issue_code
GROUP BY d.name
ORDER BY SUM(ii.amount) DESC;
-----------------------------------------------------
SELECT
    m.name AS "Наименование",
    mb.quantity AS "Количество",
    mb.amount AS "Стоимость остатка",
    ROUND(mb.amount / mb.quantity, 2) AS "Средняя цена"
FROM material_balances mb
JOIN materials m ON mb.material_code = m.code
WHERE mb.period = '2026-02-06' AND mb.quantity > 0
ORDER BY mb.amount DESC
LIMIT 5;
-----------------------------------------------------
SELECT
    m.code AS "Код",
    m.name AS "Наименование",
    m.unit AS "Ед.изм",
    m.min_balance AS "Мин.остаток"
FROM materials m
LEFT JOIN material_balances mb ON m.code = mb.material_code AND mb.period = '2026-02-06'
WHERE mb.quantity IS NULL OR mb.quantity = 0
ORDER BY m.code;
---------------------------------------------------
SELECT
    code AS "Код",
    name AS "Наименование",
    price AS "Цена",
    category AS "Категория"
FROM materials
WHERE price > (SELECT AVG(price) FROM materials)
ORDER BY price DESC;
--------------------------------------------------
SELECT
    'Поступление' AS "Тип операции",
    r.date AS "Дата",
    ri.quantity AS "Количество",
    ri.price AS "Цена",
    ri.amount AS "Сумма",
    s.name AS "Контрагент"
FROM receipt_items ri
JOIN receipts r ON ri.receipt_code = r.code
JOIN suppliers s ON r.supplier_code = s.code
WHERE ri.material_code = '001'
UNION ALL
SELECT
    'Выдача' AS "Тип операции",
    i.date AS "Дата",
    ii.quantity AS "Количество",
    ii.price AS "Цена",
    ii.amount AS "Сумма",
    d.name AS "Контрагент"
FROM issue_items ii
JOIN issues i ON ii.issue_code = i.code
JOIN departments d ON i.department_code = d.code
WHERE ii.material_code = '001'
ORDER BY "Дата";
---------------------------------------------------
SELECT
    COUNT(DISTINCT material_code) AS "Количество позиций",
    SUM(quantity) AS "Всего единиц товара",
    SUM(amount) AS "Общая стоимость склада"
FROM material_balances
WHERE period = '2026-02-06';
