# Codebasics-sqlchallenge
This is the sql challenge given by codebasics


##1
1. Provide the list of markets in which customer "Atliq Exclusive" operates its
business in the APAC region.

with cte as 
(select market from dim_customer where region ="APAC" and customer="Atliq Exclusive")
SELECT distinct(market) from cte

##2
2. What is the percentage of unique product increase in 2021 vs. 2020? The
final output contains these fields,
unique_products_2020
unique_products_2021
percentage_chg

with cte as(SELECT count(distinct product_code) as unique_products2020 FROM gdb041.fact_sales_monthly
where fiscal_year=2020),cte2 as (SELECT count(distinct product_code) as unique_products2021 FROM gdb041.fact_sales_monthly
where fiscal_year=2021)
select *,round((unique_products2021-unique_products2020)/unique_products2020*100,2) as pct
from cte cross join cte2

##3
3. Provide a report with all the unique product counts for each segment and
sort them in descending order of product counts. The final output contains
2 fields,
segment
product_count

SELECT segment,count(distinct product) 
as Unique_prod_cnt
FROM gdb041.dim_product
group by segment
order by count(distinct product) desc

##4
4. Follow-up: Which segment had the most increase in unique products in
2021 vs 2020? The final output contains these fields,
segment
product_count_2020
product_count_2021
difference

with cte1 as 
(SELECT dim_product.segment,count(distinct dim_product.product_code) as unique_products2020
 FROM gdb041.fact_sales_monthly left  join  dim_product
 on fact_sales_monthly.product_code=dim_product.product_code
 where fiscal_year=2020
 group by dim_product.segment
 order by unique_products2020 desc),cte2 as
 (SELECT dim_product.segment,count(distinct dim_product.product_code) as unique_products2021
 FROM gdb041.fact_sales_monthly left  join  dim_product
 on fact_sales_monthly.product_code=dim_product.product_code
 where fiscal_year=2021
 group by dim_product.segment
 order by unique_products2021 desc)
 select cte1.segment,cte1.unique_products2020,cte2.unique_products2021,
 round((unique_products2021-unique_products2020)/unique_products2020*100,1) as pct
 from cte1 join cte2 on cte1.segment=cte2.segment
 
 
##5
5. Get the products that have the highest and lowest manufacturing costs.
The final output should contain these fields,
product_code
product
manufacturing_cost

with cte as
(SELECT fact_manufacturing_cost.product_code,dim_product.product,manufacturing_cost
FROM fact_manufacturing_cost join dim_product 
on fact_manufacturing_cost.product_code=dim_product.product_code
union all
SELECT fact_manufacturing_cost.product_code,dim_product.product,manufacturing_cost
FROM fact_manufacturing_cost join dim_product 
on fact_manufacturing_cost.product_code=dim_product.product_code)
select product_code,product,min(manufacturing_cost) as manufacturing_cost from cte
union all
select product_code,product,max(manufacturing_cost) from cte

##6
6. Generate a report which contains the top 5 customers who received an
average high pre_invoice_discount_pct for the fiscal year 2021 and in the
Indian market. The final output contains these fields,
customer_code
customer
average_discount_percentage

SELECT pi.customer_code,customer,avg(pre_invoice_discount_pct) as average_discount_pct
FROM gdb041.fact_pre_invoice_deductions pi
join dim_customer dc
on dc.customer_code=pi.customer_code
where fiscal_year=2021 and market='India'
group by pi.customer_code
order by avg(pre_invoice_discount_pct) desc 
limit 5

##7
7. Get the complete report of the Gross sales amount for the customer “Atliq
Exclusive” for each month. This analysis helps to get an idea of low and
high-performing months and take strategic decisions.
The final report contains these columns:
Month
Year
Gross sales Amount

SELECT gm.fiscal_year,monthname(date) as mnt,month(date) as mno,
(sold_quantity*gross_price) as gross_sales_amount
FROM fact_sales_monthly gm
join fact_gross_price gp
on gm.product_code=gp.product_code
group by fiscal_year,mnt
order by mno

##8
8. In which quarter of 2020, got the maximum total_sold_quantity? The final
output contains these fields sorted by the total_sold_quantity,
Quarter
total_sold_quantity

SELECT quarter(date) as qr,sum(sold_quantity) as ttl 
FROM gdb041.fact_sales_monthly
where fiscal_year=2020
group by qr
order by qr

##9
9. Which channel helped to bring more gross sales in the fiscal year 2021
and the percentage of contribution? The final output contains these fields,
channel
gross_sales_mln
percentage

with cte1 as (SELECT channel,
round(sum(sold_quantity*gross_price)/1000000,0) as total_gross_sales_mln
FROM fact_sales_monthly fm join fact_gross_price fg
on fm.product_code=fg.product_code
join dim_customer dc on fm.customer_code=dc.customer_code
group by channel),cte2 as 
(SELECT 
round(sum(sold_quantity*gross_price)/1000000,0) as total_gross_sales_mln1
FROM fact_sales_monthly fm join fact_gross_price fg
on fm.product_code=fg.product_code
join dim_customer dc on fm.customer_code=dc.customer_code) 
select cte1.channel,cte1.total_gross_sales_mln,
round(100*total_gross_sales_mln/total_gross_sales_mln1,2) as pct
from cte1 join cte2

##10
10. Get the Top 3 products in each division that have a high
total_sold_quantity in the fiscal_year 2021? The final output contains these
fields,
division
product_code
product
total_sold_quantity
rank_order

SELECT fm.product_code,sold_quantity,division
 FROM fact_sales_monthly fm 
join dim_product dm on fm.product_code=dm.product_code
where fiscal_year=2021
order by sold_quantity desc limit 3
