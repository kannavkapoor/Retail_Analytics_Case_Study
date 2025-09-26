
use retail_analytics;

show tables;

select * from customer;
select * from product;
select * from sales;

/*
firstly, we will do the data cleaning process

-- null
-- blank('')
-- since each tables contains primary key hence, no
	need to find duplications
*/

-- first we run

select * from customer
where age is null or age=''
	or gender is null or gender =''
    or location is null or location = ''
    or joindate is null ;
    
-- Now to resolve this we use update statement
update customer
set location = 'na'
where location ='';
-- or
update customer
set location= replace(location,'na','NULL');

-- now we will calculate the outliers
/*
for this we have two ways
1) turkey law
2) using z-score

*/
select * from customer;

-- using turkey law

set @q1=
(with cte as
(select *,
		row_number() over(order by age) as asc_row
	from customer)

select avg(age) as quartile_1 from cte
where asc_row in ( select floor(count(age)/4) from customer 
					union 
				   select ceil(count(age)/4) from customer));

set @q3=
(with cte as
(select *,
		row_number() over(order by age) as asc_row
	from customer)

select avg(age) as quartile_3 from cte
where asc_row in ( select floor(count(age)*3/4) from customer 
					union 
				   select ceil(count(age)*3/4) from customer));

set @iqr=@q3-@q1;

-- the final formula to find the outliers

select * from customer
where age< (@q1-1.5*@iqr) or age>(@q3+1.5*@iqr);

-- now using z-score
/*
z i.e outlier
z= x- mean/ standard deviation
*/
select * from customer 
where age>((select avg(age)+3*stddev(age)from customer))
	or 
	age<((select avg(age)-3*stddev(age) from customer));

-- now we get to know that customerid 668 having age 131 is an outlier which is most likely to be a age entry error
-- so we delete the row where age is 131
delete from customer
where age=131;

-- now due to this there's a  break in sequence of customerid which is normal and no need to resequence it since
-- if its matches with any other tables then wrong matching is done which result to wrong analysis.Hence, leave it as it is
-- with gap

select transactionid from sales
group by transactionid
having count(transactionid)>1;
-- now after running the above query we get to know that the sales data contains some dublicate values too

-- so in order to remove the dublicates and remain the first appeared transactionid only we will do the following process
select * from sales;

alter table sales
add column id int auto_increment primary key first ;
-- since theirs no unique value assigned to rows that why, we add a new column and assigned it as primary key
-- we just need to run a delete command to remove the duplicates
delete from sales
where id not in (select id from (select min(id) as id from sales
				group  by transactionid) s);

-- now we have deleted all the dublicates leaving the one who appeared first
-- so when i got what i wanted then, now i will drop id column

alter table sales
drop id;

-- now after reviewing all the tables, all the tables are not cleaned and ready for analysis


--                                       ************ START ***************
                                         
									   /* PRODUCT PERFORMANCE VARIABILITY */
/*
-- To identify which products are performing well in terms on sales and which are not

	WHY NEED OF THIS INSIGHT?
-- For inventory management
-- For marketing focus
*/

-- now to see the sales or i should say total sales done over each product we need sales table also to specify
-- name of the products then in that case we also need product table

select * from sales;
select * from product;

select productid,sum(price*quantitypurchased) as total_sales
from sales
group by productid;

-- now by this i get all the productid's along with their total_sales done


/*
but here i need the products which are performing well in the market
hence, i will find the top 10% and least 10% of the products which results to highest/lowest total_sales
and also rank them on the basic of highest to lowerest*/

with product_sales as
(select productid,sum(price*quantitypurchased) as total_sales
from sales
group by productid)
,
rankings as
(select 
	dense_rank() over (order by total_sales desc) as RANKINGS,
    productid,
    total_sales
from product_sales)

select r.RANKINGS,
		r.Productid,
		p.ProductName,
        r.Total_sales from rankings r
join product p
on r.productid=p.productid
where r.RANKINGS <= (select round(count(productid)*0.10) from product);  -- total products * 10% 

/* NOW THESE ARE THE TOP PERFORMING PRODUCTS (TOP 10%) */

-- LEAST PERFROMING(BELOW IS THE QUERY)


with product_sales as
(select productid,sum(price*quantitypurchased) as total_sales
from sales
group by productid)
,
rankings as
(select 
	dense_rank() over (order by total_sales) as RANKINGS,
    productid,
    total_sales
from product_sales)

select r.RANKINGS,
		r.Productid,
        p.ProductName,
        r.Total_sales from rankings r
join product p
on r.productid=p.productid
where r.RANKINGS <= (select round(count(productid)*0.10) from product);  -- total products * 10% 

/* NOW THESE ARE THE LEAST PERFORMING PRODUCTS (LEAST 10%) */

--                                        ************** END**************

											/* CUSTOMER SEGMENTATION */

/*
-- Demmographic- age,gender
-- geographic- location
-- behavioral- purchase behaviour,product usage,brand loyalty
*/

/* DEMOGRAPHIC */

 -- WE WILL SEE HOW MANY CUSTOMERS ARE CHILDREN/TEENAGERS, MIDDLE AGED, OLD
--  S0,THE QUERY WE WILL USE IS
SELECT * FROM CUSTOMER;
with AGE_DEMOGRAPHIC AS
(SELECT 
	case when age<=18 then '0-18'
		when age between 19 and 40 then '18-45'
        else '45-Above' end
	as age_demographic
from customer)

select age_demographic,count(age_demographic) as no_of_customer,
		concat(round(count(age_demographic)*100/(select count(*) from customer),2),'%') as percentage_of_customers
from age_demographic
group by age_demographic
order by count(age_demographic) desc;

-- now we will see how many customers are male and female

select gender,count(customerid) as no_of_customer ,
		concat(round(count(customerid)*100/(select count(*) from customer),2),'%') as percentage_of_customers
from customer
group by gender
order by count(customerid) desc ;

-- we will analyse the geographic segmentation
select location,count(customerid) as no_of_customer ,
		concat(round(count(customerid)*100/(select count(*) from customer),2),'%') as percentage_of_customers
from customer
group by location
order by count(customerid) desc;

-- some users haven't specify their location and thats just 1.2% of the overall data 

--                                        ************** END**************

                                          /* CUSTOMER BEHAVIOUR */
-- customers who didnt purchased any product

select customerid from customer c
where not exists
(select 1 from sales s where c.customerid=s.customerid);

-- lets calculate how many products are there in each category

select category,count(productname) as total_products from product
group by category;


-- ONE-TIME-BUYERS
-- customers who purchased only once
select s.customerid,
		min(TransactionDate) as date_purchased,
        sum(s.price*s.QuantityPurchased) as Order_Value
from customer c 
join sales s
on s.customerid=c.customerid
group by s.customerid
having count(s.transactionid)=1
order by s.customerid;

-- now to find count of customers

select count(customerid) as total_one_time_buyers
from
(select customerid from sales
group by customerid
having count(transactionid)=1) t;

-- REPEAT-USERS

select count(customerid) as total_repeat_users 
from
(select customerid from sales
group by customerid
having count(transactionid)>1) t;

-- ACTIVE CUSTOMERS(who does more than 5 purchases over the last 3 months) 

select customerid, 
	sum(price*QuantityPurchased) as total_purchase, 
    max(transactiondate) as latest_purchase_date,
    count(transactionid) as frequency_of_purchase
from sales
where str_to_date(TransactionDate,'%d/%m/%y') between
   date_sub((select max(str_to_date(TransactionDate,'%d/%m/%y')) from sales), interval 3 month) 
		and (select max(str_to_date(TransactionDate,'%d/%m/%y')) from sales)                    -- last 3 months
group by customerid
having count(transactionid)>5;


-- WINDOW /RARE CUSTOMERS

with t as
(select customerid,
		transactiondate,
        lag(str_to_date(transactiondate,'%d/%m/%y')) over(partition by customerid order by str_to_date(transactiondate,'%d/%m/%y')) as last_order,
        datediff(
			str_to_date(transactiondate,'%d/%m/%y'),
			lag(str_to_date(transactiondate,'%d/%m/%y')) over(partition by customerid order by str_to_date(transactiondate,'%d/%m/%y'))) 
        as days_gap
from sales)

select customerid,
		round(avg(days_gap)) as avg_days_gap  
from t
where last_order and days_gap is not null
group by customerid
having round(avg(days_gap))> (select avg(days_gap) from t)
order by avg_days_gap desc;

-- PREMIUM/VIP CUSTOMERS
with t as 
(select customerid, 
	sum(price*quantitypurchased) as total_purchase_amount, 
	count(transactionid) as freq_of_purchase,max(str_to_date(transactiondate,'%d/%m/%y')) as latest_purchase_date
from sales
where str_to_date(transactiondate,'%d/%m/%y') between 
		date_sub((select max(str_to_date(transactiondate,'%d/%m/%y')) from sales), interval 30 day) and 
        (select max(str_to_date(transactiondate,'%d/%m/%y')) from sales)
group by customerid), 

t2 as
(select
	ntile(10) over(order by total_purchase_amount desc) as amount_desc,
    ntile(10) over(order by freq_of_purchase desc) as freq_desc,
    t.*
from t)

select customerid,total_purchase_amount,latest_purchase_date,freq_of_purchase from t2
where amount_desc=1 and freq_desc =1;

-- CHURN-RISK CUSTOMERS
WITH last_purchase AS (
    SELECT customerid,
           MAX(STR_TO_DATE(transactiondate, '%d/%m/%y')) AS last_purchase_date,
           COUNT(transactionid) AS total_purchases
    FROM sales
    GROUP BY customerid
),

latest_date AS (
    SELECT MAX(STR_TO_DATE(transactiondate, '%d/%m/%y')) AS max_date
    FROM sales
)

SELECT l.customerid, 
       l.total_purchases, 
       l.last_purchase_date,
       DATEDIFF(ld.max_date, l.last_purchase_date) AS days_since_last_purchase
FROM last_purchase l
CROSS JOIN latest_date ld
WHERE DATEDIFF(ld.max_date, l.last_purchase_date) > 90   -- last purchase older than 90 days
  AND l.total_purchases > 1                               -- previously active customers
ORDER BY days_since_last_purchase DESC;

-- products which are purchased most number of times

with T AS
(select 
	ntile(10) over(order by count(customerid) desc) as customers_percent,
    productid, count(customerid) as no_of_customers
from sales
group by productid
order by no_of_customers desc)

select productid,no_of_customers from t
where customers_percent=1;


-- Find the avg_order_quantity of the customers

select customerid, round(avg(quantitypurchased)) as avg_order_quantity
from sales
group by customerid;

-- Find the avg_order_quantity of the company

select round(avg(quantitypurchased),2) as avg_order_quantity
from sales;

-- Find the avg_order_price of the customers

select customerid, round(avg(price*quantitypurchased)) as avg_order_price
from sales
group by customerid
order by avg_order_price desc;


-- avg_order_value
select round(avg(price*quantitypurchased)) as avg_order_price
from sales;

-- **************************************************END OF THE PROJECT*****************************************************************
















