# Bike Store SQL Project

This project showcases SQL queries designed to analyze and manage the dataset of a bike store. The dataset consists of nine tables, each containing vital information about various aspects of the business such as products, customers, orders, staff, and stores. The queries aim to address business questions ranging from customer behavior to staff performance and inventory management.

## Dataset Description

The dataset includes the following tables:
1. **Products**: Details about the products sold in the store 
2. **Brands**: Information about the product brands.
3. **Categories**: Product categories.
4. **Stocks**: Current stock levels in various stores.
5. **Stores**: Store details.
6. **Staffs**: Information about staff members.
7. **Customers**: Customer details.
8. **Orders**: Information about customer orders.
9. **Order_Items**: Details about items in each order.

## Business Questions Addressed

### Q1: Find the top 3 customers with the highest total purchase value.
- **Objective**: Identify the customers contributing the most to revenue.
  SELECT TOP 3
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS Customer_Name,
    SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS Total_Purchase_Value
FROM 
    customers AS c
JOIN 
    orders AS o
ON 
    c.customer_id = o.customer_id
JOIN 
    order_items AS oi
ON 
    o.order_id = oi.order_id
GROUP BY 
    c.customer_id, c.first_name, c.last_name
ORDER BY 
    Total_Purchase_Value DESC

  

### Q2: Identify the staff member with the highest number of orders processed.
- **Objective**: Recognize high-performing staff members.
SELECT TOP 1
    st.staff_id,
    CONCAT(st.first_name, ' ', st.last_name) AS Staff_Name,
    COUNT(o.order_id) AS Total_Orders_Processed
FROM 
    staffs AS st
JOIN 
    orders AS o
ON 
    st.staff_id = o.staff_id
GROUP BY 
    st.staff_id, st.first_name, st.last_name
ORDER BY 
    Total_Orders_Processed DESC

### Q3: Determine the month with the highest total sales.
- **Objective**: Understand seasonal trends in sales.
SELECT TOP 1
    DATEPART(YEAR, o.order_date) AS Sales_Year,
    DATEPART(MONTH, o.order_date) AS Sales_Month,
    SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS Total_Sales
FROM 
    orders AS o
JOIN 
    order_items AS oi
ON 
    o.order_id = oi.order_id
GROUP BY 
    DATEPART(YEAR, o.order_date), DATEPART(MONTH, o.order_date)
ORDER BY 
    Total_Sales DESC

### Q4: Find the average discount offered across all orders.
- **Objective**: Analyze discounting practices.
SELECT 
    ROUND(AVG(oi.discount), 2) AS Average_Discount
FROM 
    order_items AS oi; 

### Q5: Identify the product category with the highest revenue.
- **Objective**: Highlight the most profitable category.
SELECT TOP 1
    c.category_id,
    c.category_name,
    SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS Total_Revenue
FROM 
    categories AS c
JOIN 
    products AS p
ON 
    c.category_id = p.category_id
JOIN 
    order_items AS oi
ON 
    p.product_id = oi.product_id
GROUP BY 
    c.category_id, c.category_name
ORDER BY 
    Total_Revenue DESC

### Q6: Identify the store with the highest total sales.
- **Objective**: Recognize high-performing stores.
SELECT TOP 1
    s.store_id,
    s.store_name,
    SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS Total_Sales
FROM 
    stores AS s
JOIN 
    staffs AS st
ON 
    s.store_id = st.store_id
JOIN 
    orders AS o
ON 
    st.staff_id = o.staff_id
JOIN 
    order_items AS oi
ON 
    o.order_id = oi.order_id
GROUP BY 
    s.store_id, s.store_name
ORDER BY 
    Total_Sales DESC

### Q7: Identify the most popular product in terms of quantity sold.
- **Objective**: Determine which product is sold the most.

SELECT TOP 1
    p.product_id,
    p.product_name,
    SUM(oi.quantity) AS Total_Quantity_Sold
FROM 
    products AS p
JOIN 
    order_items AS oi
ON 
    p.product_id = oi.product_id
GROUP BY 
    p.product_id, p.product_name
ORDER BY 
    Total_Quantity_Sold DESC

### Q8: Identify the top 5 customers by total spending.
- **Objective**: Expand the focus on high-value customers.
SELECT TOP 5
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS Customer_Name,
    SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS Total_Spending
FROM 
    customers AS c
JOIN 
    orders AS o
ON 
    c.customer_id = o.customer_id
JOIN 
    order_items AS oi
ON 
    o.order_id = oi.order_id
GROUP BY 
    c.customer_id, c.first_name, c.last_name
ORDER BY 
    Total_Spending DESC

### Q9: Calculate the average order value handled by each staff member.
- **Objective**: Evaluate staff efficiency.
SELECT 
    st.staff_id,
    CONCAT(st.first_name, ' ', st.last_name) AS Staff_Name,
    AVG(o.Total_Order_Value) AS Avg_Order_Value
FROM 
    (SELECT 
         o.order_id,
         o.staff_id,
         SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS Total_Order_Value
     FROM 
         orders AS o
     JOIN 
         order_items AS oi
     ON 
         o.order_id = oi.order_id
     GROUP BY 
         o.order_id, o.staff_id) AS o
JOIN 
    staffs AS st
ON 
    o.staff_id = st.staff_id
GROUP BY 
    st.staff_id, st.first_name, st.last_name
ORDER BY 
    Avg_Order_Value DESC;

### Q10: Identify the staff member with the highest sales in each store.
- **Objective**: Understand staff contributions at a store level.
WITH StoreStaffSales AS (
    SELECT 
        s.store_id,
        st.staff_id,
        CONCAT(st.first_name, ' ', st.last_name) AS Staff_Name,
        SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS Total_Sales
    FROM 
        orders AS o
    JOIN 
        order_items AS oi
    ON 
        o.order_id = oi.order_id
    JOIN 
        staffs AS st
    ON 
        o.staff_id = st.staff_id
    JOIN 
        stores AS s
    ON 
        st.store_id = s.store_id
    GROUP BY 
        s.store_id, st.staff_id, st.first_name, st.last_name
)
SELECT 
    st.store_id,
    st.staff_id,
    st.Staff_Name,
    st.Total_Sales
FROM 
    StoreStaffSales AS st
JOIN (
    SELECT 
        store_id,
        MAX(Total_Sales) AS Max_Sales
    FROM 
        StoreStaffSales
    GROUP BY 
        store_id
) AS max_sales
ON 
    st.store_id = max_sales.store_id 
    AND st.Total_Sales = max_sales.Max_Sales
ORDER BY 
    st.store_id, st.Total_Sales DESC;

### Q11: Calculate the total sales handled by each staff member.
- **Objective**: Highlight individual staff contributions.
SELECT 
    st.staff_id,
    CONCAT(st.first_name, ' ', st.last_name) AS Staff_Name,
    SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS Total_Sales
FROM 
    orders AS o
JOIN 
    order_items AS oi
ON 
    o.order_id = oi.order_id
JOIN 
    staffs AS st
ON 
    o.staff_id = st.staff_id
GROUP BY 
    st.staff_id, st.first_name, st.last_name
ORDER BY 
    Total_Sales DESC;

### Q12: Identify the most popular product (highest quantity sold) along with its total quantity.
- **Objective**: Reconfirm top product performance.
SELECT TOP 1
    p.product_id,
    p.product_name,
    SUM(oi.quantity) AS Total_Quantity_Sold
FROM 
    products AS p
JOIN 
    order_items AS oi
ON 
    p.product_id = oi.product_id
GROUP BY 
    p.product_id, p.product_name
ORDER BY 
    Total_Quantity_Sold DESC

### Q13: Calculate the total stock value for each store.
- **Objective**: Evaluate inventory value across stores.
SELECT 
    s.store_name,
    SUM(stk.quantity * p.list_price) AS Total_Stock_Value
FROM 
    stocks AS stk
JOIN 
    products AS p
ON 
    stk.product_id = p.product_id
JOIN 
    stores AS s
ON 
    stk.store_id = s.store_id
GROUP BY 
    s.store_name
ORDER BY 
    Total_Stock_Value DESC;

### Q14: Identify products with low stock (below 10 units) in each store.
- **Objective**: Prevent stockouts by highlighting inventory issues.
SELECT 
    s.store_name,
    p.product_name,
    stk.quantity AS Stock_Quantity
FROM 
    stocks AS stk
JOIN 
    products AS p
ON 
    stk.product_id = p.product_id
JOIN 
    stores AS s
ON 
    stk.store_id = s.store_id
WHERE 
    stk.quantity < 10
ORDER BY 
    s.store_name, p.product_name;

### Q15: Identify categories with the highest total revenue.
- **Objective**: Validate top-performing categories.
SELECT 
    c.category_name,
    SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS Total_Revenue
FROM 
    order_items AS oi
JOIN 
    products AS p
ON 
    oi.product_id = p.product_id
JOIN 
    categories AS c
ON 
    p.category_id = c.category_id
GROUP BY 
    c.category_name
ORDER BY 
    Total_Revenue DESC;

### Q16: List products with the highest revenue in each category.
- **Objective**: Find top-performing products within categories.
WITH CategoryRevenue AS (
    SELECT 
        c.category_name,
        p.product_name,
        SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS Total_Revenue,
        RANK() OVER (PARTITION BY c.category_name ORDER BY SUM(oi.quantity * oi.list_price * (1 - oi.discount)) DESC) AS Rank
    FROM 
        order_items AS oi
    JOIN 
        products AS p
    ON 
        oi.product_id = p.product_id
    JOIN 
        categories AS c
    ON 
        p.category_id = c.category_id
    GROUP BY 
        c.category_name, p.product_name
)
SELECT 
    category_name,
    product_name,
    Total_Revenue
FROM 
    CategoryRevenue
WHERE 
    Rank = 1
ORDER BY 
    category_name;

### Q17: Find the top-selling product in each category.
- **Objective**: Analyze top products by sales volume.
SELECT 
    c.category_name,
    p.product_name,
    SUM(oi.quantity) AS Total_Quantity_Sold
FROM 
    order_items AS oi
JOIN 
    products AS p
ON 
    oi.product_id = p.product_id
JOIN 
    categories AS c
ON 
    p.category_id = c.category_id
GROUP BY 
    c.category_name, p.product_name
ORDER BY 
    c.category_name, Total_Quantity_Sold DESC;

### Q18: Identify customers who purchased products in multiple categories.
- **Objective**: Understand customer buying behavior across categories.
SELECT 
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS Customer_Name,
    COUNT(DISTINCT p.category_id) AS Categories_Purchased
FROM 
    orders AS o
JOIN 
    order_items AS oi
ON 
    o.order_id = oi.order_id
JOIN 
    products AS p
ON 
    oi.product_id = p.product_id
JOIN 
    customers AS c
ON 
    o.customer_id = c.customer_id
GROUP BY 
    c.customer_id, CONCAT(c.first_name, ' ', c.last_name)
HAVING 
    COUNT(DISTINCT p.category_id) > 1
ORDER BY 
    Categories_Purchased DESC;

### Q19: Find the number of unique customers who placed orders in each store.
- **Objective**: Understand customer buying behavior across categories.
SELECT 
    s.store_name,
    COUNT(DISTINCT o.customer_id) AS Unique_Customers
FROM 
    orders AS o
JOIN 
    stores AS s
ON 
    o.store_id = s.store_id
GROUP BY 
    s.store_name
ORDER BY 
    Unique_Customers DESC;

### Q20: List the top 5 customers by total spending and their associated stores.
- **Objective**: Understand customer buying behavior across categories.
SELECT TOP 5
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS Customer_Name,
    s.store_name,
    SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS Total_Spending
FROM 
    order_items AS oi
JOIN 
    orders AS o
ON 
    oi.order_id = o.order_id
JOIN 
    customers AS c
ON 
    o.customer_id = c.customer_id
JOIN 
    stores AS s
ON 
    o.store_id = s.store_id
GROUP BY 
    c.customer_id, 
    CONCAT(c.first_name, ' ', c.last_name),
    s.store_name
ORDER BY 
    Total_Spending DESC


## Highlights
- Comprehensive analysis using SQL queries to derive actionable insights.
- Queries are optimized for performance and readability.
- Covers key areas such as sales, customer behavior, staff performance, and inventory management.

## Technologies Used
- **SQL Server Management Studio (SSMS)**: For executing and testing SQL queries.
- **SQL**: To perform data analysis and answer business questions.

