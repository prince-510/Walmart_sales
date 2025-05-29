# Walmart_sales

```

select * from Walmart ```

-- count
select count(*) from Walmart

-- distinct
select distinct w.payment_method 
from Walmart w;

select w.payment_method,count(*) as cnt
from Walmart w
group by w.payment_method;


select count(distinct w.Branch) as branch
from Walmart w;

select max(w.quantity) as max, min(w.quantity) as min_qty from Walmart w

--Business Problems

--Q1 Find different payment method and number of transactions,no.of qty sold

```select w.payment_method,count(*) as cnt,sum(w.quantity) as tot_qty
from Walmart w
group by w.payment_method```

--Q2 Identify the highest rated category in each branch,
--displaying the branch,categroy,avg rating


select * from Walmart wl;

with h as (
select w.Branch,w.category,AVG(w.rating) as avg_rating,
DENSE_RANK() over(partition by w.branch order by avg(w.rating) desc) as rank
from Walmart w
group by w.Branch,w.category)
select *
from h
where h.rank = '1';

--Q3 Identify the busiest day for each branch based on the number of transaction

with d as (
SELECT w.Branch, FORMAT(w.date , 'dddd') AS day_name,
count(*) as no_tans,
DENSE_RANK() over(partition by w.branch order by count(*) desc) as ranking
FROM Walmart w
group by w.Branch,FORMAT(w.date , 'dddd')
)
select  d.Branch, 
    d.day_name, 
    d.no_tans
	from d
where d.ranking = '1';

--Q4 Calculate the total quantity of items sold per payment method.
--List payment method and total quantity

select w.payment_method,sum(w.quantity) as total_qty from Walmart w
group by w.payment_method;

--Q5 Determine the average,minimum and maximum rating of category for each city
--List the city, average_rating,min_rating,max_rating

select w.City,w.category,
MIN(w.rating) as min_rat, max(w.rating) as max_rat
,round(avg(w.rating),2) as avg_rat
from Walmart w
group by w.City,w.category
order by w.City;

--Q6 Calculate the total profit for each category by considering total_profit 
-- as unit_price*quantity*profit_margin --or-- total*profit_margin
--List category, total_profit where ordered from highest to lowest


select w.category,sum(w.total) as revenue,
sum(w.total*w.profit_margin) as total_profit
from Walmart w
group by w.category
order by sum(w.total*w.profit_margin) desc

--Q7 Determine the most common payment method for each branch,
-- display branch and the preferred payment method

with p as (
select w.Branch,w.payment_method,count(*) as trans,
DENSE_RANK() over(partition by w.branch order by count(*) desc) as ranking
from Walmart w
group by w.Branch,w.payment_method
)
select p.* from p
where p.ranking = 1

--Q8 Categorize sales into 3 group Morning,Afternoon,Evening
--Find out which of the shift and number of invoices

with s as (
select w.category,w.time,w.invoice_id,
case when DATEPART(HOUR,w.time) <12 then 'Morning'
     when DATEPART(Hour,w.time) <17  then 'Afternoon'
	 else 'Evening'
	 end as 'Shift'
from 
Walmart w),x as 
(select s.category,s.Shift,count(*) as inv,
DENSE_RANK() over(partition by s.category order by count(*) desc) as ranking
from s
 group by s.category,s.Shift
 )
 select * from x
 where x.ranking = 1;

 
select * from (
select k.category,k.Shift,count(*) as inv,
DENSE_RANK() over(partition by k.category order by count(*) desc) as ranking
from (
select w.category,w.time,w.invoice_id,
case when DATEPART(HOUR,w.time) <12 then 'Morning'
     when DATEPART(Hour,w.time) <17  then 'Afternoon'
	 else 'Evening'
	 end as 'Shift'
from 
Walmart w) as k
group by k.category,k.Shift) as x
where x.ranking = 1
;
--Q9 Identify 5 branches with highest decrease ratio in revenue
--compare to last year(current year 2023 and last year 2022)

-- revenue decrease ratio =  (last_rev - current_rev)/last_rev * 100

 with last_year as (
select w.Branch,sum(w.total) as last_rev
from Walmart w
where datepart(year,w.date) = 2022
group by w.Branch),
current_year as (
select w.Branch, sum(w.total) as cur_rev
from Walmart w
where DATEPART(year,w.date) = 2023
group by w.Branch)
select * from (
select l.Branch,last_rev,cur_rev, round((last_rev-cur_rev)/nullif(last_rev,0)*100,2) as decrease_ratio,
DENSE_RANK() over(order by round((last_rev-cur_rev)/nullif(last_rev,0)*100,2) desc) as ran
from last_year l
full join current_year c on c.Branch = l.Branch ) as k
where k.ran<=5


