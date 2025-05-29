-- View all records
SELECT * FROM Walmart;

-- Count of records
SELECT COUNT(*) FROM Walmart;

-- Distinct payment methods
SELECT DISTINCT w.payment_method FROM Walmart w;

-- Count by payment method
SELECT w.payment_method, COUNT(*) AS cnt
FROM Walmart w
GROUP BY w.payment_method;

-- Count of distinct branches
SELECT COUNT(DISTINCT w.Branch) AS branch_count
FROM Walmart w;

-- Max and Min quantity sold
SELECT MAX(w.quantity) AS max_qty, MIN(w.quantity) AS min_qty
FROM Walmart w;

-- Q1: Payment method analysis
SELECT 
    w.payment_method, 
    COUNT(*) AS transaction_count, 
    SUM(w.quantity) AS total_quantity
FROM Walmart w
GROUP BY w.payment_method;

-- Q2: Highest rated category in each branch
WITH h AS (
    SELECT 
        w.Branch, 
        w.category, 
        AVG(w.rating) AS avg_rating,
        DENSE_RANK() OVER(PARTITION BY w.Branch ORDER BY AVG(w.rating) DESC) AS rank
    FROM Walmart w
    GROUP BY w.Branch, w.category
)
SELECT * 
FROM h 
WHERE h.rank = 1;

-- Q3: Busiest day for each branch
WITH d AS (
    SELECT 
        w.Branch, 
        FORMAT(w.date, 'dddd') AS day_name,
        COUNT(*) AS no_trans,
        DENSE_RANK() OVER(PARTITION BY w.Branch ORDER BY COUNT(*) DESC) AS ranking
    FROM Walmart w
    GROUP BY w.Branch, FORMAT(w.date, 'dddd')
)
SELECT d.Branch, d.day_name, d.no_trans
FROM d 
WHERE d.ranking = 1;

-- Q4: Total quantity sold per payment method
SELECT 
    w.payment_method, 
    SUM(w.quantity) AS total_qty 
FROM Walmart w
GROUP BY w.payment_method;

-- Q5: Rating summary by city and category
SELECT 
    w.City, 
    w.category,
    MIN(w.rating) AS min_rating, 
    MAX(w.rating) AS max_rating,
    ROUND(AVG(w.rating), 2) AS avg_rating
FROM Walmart w
GROUP BY w.City, w.category
ORDER BY w.City;

-- Q6: Total profit by category
SELECT 
    w.category, 
    SUM(w.total) AS revenue,
    SUM(w.total * w.profit_margin) AS total_profit
FROM Walmart w
GROUP BY w.category
ORDER BY total_profit DESC;

-- Q7: Most common payment method per branch
WITH p AS (
    SELECT 
        w.Branch, 
        w.payment_method, 
        COUNT(*) AS trans,
        DENSE_RANK() OVER(PARTITION BY w.Branch ORDER BY COUNT(*) DESC) AS ranking
    FROM Walmart w
    GROUP BY w.Branch, w.payment_method
)
SELECT p.Branch, p.payment_method, p.trans
FROM p 
WHERE p.ranking = 1;

-- Q8: Sales shift analysis (Morning, Afternoon, Evening)
WITH s AS (
    SELECT 
        w.category, 
        w.time, 
        w.invoice_id,
        CASE 
            WHEN DATEPART(HOUR, w.time) < 12 THEN 'Morning'
            WHEN DATEPART(HOUR, w.time) < 17 THEN 'Afternoon'
            ELSE 'Evening'
        END AS Shift
    FROM Walmart w
),
x AS (
    SELECT 
        s.category, 
        s.Shift, 
        COUNT(*) AS inv,
        DENSE_RANK() OVER(PARTITION BY s.category ORDER BY COUNT(*) DESC) AS ranking
    FROM s
    GROUP BY s.category, s.Shift
)
SELECT * 
FROM x 
WHERE x.ranking = 1;


-- Q9: Top 5 branches with highest revenue decrease ratio
WITH last_year AS (
    SELECT 
        w.Branch, 
        SUM(w.total) AS last_rev
    FROM Walmart w
    WHERE DATEPART(YEAR, w.date) = 2022
    GROUP BY w.Branch
),
current_year AS (
    SELECT 
        w.Branch, 
        SUM(w.total) AS cur_rev
    FROM Walmart w
    WHERE DATEPART(YEAR, w.date) = 2023
    GROUP BY w.Branch
),
combined AS (
    SELECT 
        l.Branch,
        last_rev,
        cur_rev,
        ROUND((last_rev - cur_rev) / NULLIF(last_rev, 0) * 100, 2) AS decrease_ratio,
        DENSE_RANK() OVER(ORDER BY ROUND((last_rev - cur_rev) / NULLIF(last_rev, 0) * 100, 2) DESC) AS rank
    FROM last_year l
    FULL JOIN current_year c ON c.Branch = l.Branch
)
SELECT * 
FROM combined 
WHERE rank <= 5;
