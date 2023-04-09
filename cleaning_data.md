What issues will you address by cleaning the data?
## 1.Changing column data type in table all_sessions

## 2.Add  PK (for products) and FK (for sales_report,sales_by_sku)

## 3.Create copy tabel sales_by_sku_copy and delete rows from sales_by_sku

## 4.Delete productSKU wihh not exist in table product

## 5.Now is posible create FK for sales_by_sku

## 6.Check duplicate for table with PK 

## 7.Check duplicate for table without PK

## 8.Cleaning hint:The unit cost in the data needs to be divided by 1,000,000.

## 9.update table all_sessions  for all null vaalue of transactios (Q1)

## 10. update table all_sessions  for all null value of productrevenue (Q5)






Queries:
Below, provide the SQL queries you used to clean your data.
```SQL
--1.
-- Create a temporary TIMESTAMP column
ALTER TABLE all_sessions ADD COLUMN create_time_holder TIMESTAMP without time zone NULL;

-- Copy casted value over to the temporary column
UPDATE all_sessions SET create_time_holder = date::TIMESTAMP;

-- Modify original column using the temporary column
ALTER TABLE all_sessions ALTER COLUMN date TYPE TIMESTAMP without time zone USING create_time_holder;

-- Drop the temporary column (after examining altered column values)
ALTER TABLE all_sessions DROP COLUMN create_time_holder;
---

---SQL
--2. add  PK (for products) and FK (for sales_report,sales_by_sku)
ALTER TABLE products ADD PRIMARY KEY (sku);

ALTER TABLE sales_report
    ADD CONSTRAINT fk_productsku FOREIGN KEY (productsku) REFERENCES products (sku);
	
--!!! ISSUE with add FK in sales_by_sku. Does not allow adding because --there are some productSKU in the table sales_by_sku that do not exist in the table products

ALTER TABLE sales_by_sku
    ADD CONSTRAINT fk_sales_by_sku_productsku FOREIGN KEY (productSKU) REFERENCES products (sku);	

--3.create copy tabel sales_by_sku_copy and delete rows from --sales_by_sku

select distinct s.productsku  from sales_by_sku s 
where not  exists (select sku from products where s.productSKU=sku)

--copy of taable sales_by_sku before delefe rows
CREATE TABLE sales_by_sku_copy as 
SELECT * FROM sales_by_sku WHERE 1=0;

insert into sales_by_sku_copy
(SELECT * FROM sales_by_sku)

select * from sales_by_sku_copy

--4.delete productSKU wihh not exist in table products

delete from sales_by_sku
where productSKU in (select distinct s.productsku  from sales_by_sku s 
                     where not  exists (select sku from products where s.productSKU=sku)
					)

-- 8 records
/*"9180753"
"9182182"
"9182763"
"9182779"
"9184663"
"9184677"
"GGOEGALJ057912"
"GGOEYAXR066128"*/

--5. now is posible create FK for sales_by_sku
ALTER TABLE sales_by_sku
    ADD CONSTRAINT fk_sales_by_sku_productsku FOREIGN KEY (productSKU) REFERENCES products (sku);
	

--6. Check duplicate for table with PK 

select sku,count(sku) from products 
group by sku
having count(sku)>1

--7. Check duplicate for table without PK
select name from (
SELECT name, 
ROW_NUMBER() OVER (PARTITION by productSKU, total_ordered, total_ordered,name,stockLevel,restockingLeadTime,sentimentScore,sentimentMagnitude,ratio ORDER BY name) AS duplicateRecCount
FROM sales_report) tmp
where duplicateRecCount>1

select name from (
SELECT name, 
ROW_NUMBER() OVER (PARTITION by productSKU, total_ordered, total_ordered,name,stockLevel,restockingLeadTime,sentimentScore,sentimentMagnitude,ratio ORDER BY name) AS duplicateRecCount
FROM sales_by_sku) tmp
where duplicateRecCount>1


select productSKU from (
SELECT productSKU, 
ROW_NUMBER() OVER (PARTITION by productSKU,total_ordered  ORDER BY productSKU) AS duplicateRecCount
FROM sales_by_sku) tmp
where duplicateRecCount>1

--8.Cleaning hint:The unit cost in the data needs to be divided by 1,000,000.

UPDATE analytics SET unit_price = unit_price / 1000000


select distinct transactions from all_sessions

--9.update table all_sessions  for all null vaalue of transactios

UPDATE all_sessions 
    SET transactions =(case 
                          when t.transactions is null then 1
                          else t.transactions 
	                      end) 
from all_sessions 

--10. update productrevenue for all null velue
select productrevenue,* from all_sessions
where productrevenue is null

UPDATE all_sessions 
set productrevenue=0
where productrevenue is null

select productrevenue,* from all_sessions
```