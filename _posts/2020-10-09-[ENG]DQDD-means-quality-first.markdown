---
layout: post
title:  "[ENG] DQDD means quality first"
date:   2020-10-08
last_modified_at: 2020-10-08
categories: [DQDD,TDD,Quality,Testing]
tags: [DQDD,TDD,Quality,Testing]
---

Testing is beautiful. It gives immediate answer if our actions are valid.
Correctly written tests check everything from A to Z, from the beginning to the end.
It prevents from time-consuming and boring looking into results and hoping for a luck to hit the faulty case.

# Everything costs
A cost of automated tests creation is more or less the same as a creation and one time run of manual tests.
However, manual tests are going to be run at least few times during a project.
Gain from automated tests is huge, and it is raising even more, when a project is going longer. 
This kind of tests let us make a cycle of development, finding a bug and fixing shorter. It makes our work easier, as we don't have to get back to this task in a future, when in the meantime
we've done lots of other things, and we've already forgotten about the details. We need to refresh our mind, and fix fateful code.
Next thing is confidence and sens of security of the programmer, when doing changes in code which he or she sees for the first time or after long time. Tests give a fast respond, that code
after changes is still doing what it should do. 

# Action reaction
Important thing is execution time of tests. The shorter, the better.
It gives us right away information if our task is done properly and put fixes if not before we pass a task to QAs and switch thread in our mind to different task.
From the other hand long execution time could be demotivating and there is a risk to resign making tests at all. 
 
# Inspiration?
In classic, object-oriented programming, one of the best, if not the best, methods to test your own code is Test Driven Development (TDD). Its principals say, that
work should be make of very short cycles, like 1-2 minutes, during which we are doing 3 steps:


1. we are writing a test, which our code fails (for example check if we can call constructor of a class) (phase red)
2. we are writing as little as possible code to pass the test (for example we are declaring a class, without any logic, just empty constructor) (phase green)
3. we are refactoring our code (phase refactor)

TDD assumes, that in each moment only 1 test can be red (fail), so we can't continue to next test or code, until we make the current test to be green (passed).


# DWH i BI
In the world of data warehouses and BI system, especially on database side, when we are writing custom data aggregations, it's difficult to use TDD in exactly the same shape.
However, we can use a technique, which I've called Data Quality Driven Development (DQDD).

DQDD can be used when we are processing data from a source to destination. So always actually. Looking at transformation of data and joins to other tables during processing, we can 
expect some errors during setting up a data flow. We can miss some join condition or make additional one causing extra filtering.
We can duplicate data or even make a Cartesian join. There are few things which can cause these. Everyone can make a typo or by mistake copy alias of a table to left and right join condition, 
which gives always true condition. There is possibility to add not appropriate logic to conditions or there can be some errors in data itself.  

To detect these errors, we can create data quality checks, which usually taking place in further phases of a project, however we can do it during implementation, to get fast feedback, if we are properly setting up our dataflow. 

Important thing during using DQDD is an ability to quick load data from source to destination. To additionally reduce time from writing a code to get feedback from tests, we can create 
proper test data set. Creating minimal data set which can cover all test cases gives us minimal processing time. It also allows us to cover 100% test cases (analogy to code coverage). 
We can't rely on production data, because there could not be all boundary cases. Productional data also causes long time of processing and it is difficult to debug in case of any failure. 
Preparing test data set also can be very helpful when testing UI layer. When we are ready to serve an API to frontend application, we exactly know what output is expected. In case of productional data
expected results in API is different each time, which makes testing harder.

# Example
In next part of this article I'm assuming, that or source is dimensional model (Kimball), while our goal is to create reporting layer for frontend application with predefined reports.
To serve this kind of data as fast as possible, the best option is to aggregate data to the shape in which they will be presented in UI. By that there is no calculation between click in UI and serving the data.
We are going to test this kind of aggregate creation.
To automate tests we will use FitNesse (http://docs.fitnesse.org/)


Source data model:
![Source model](/images/dqdd/source_model.png)

In fact table f_pharma_sales there are sales of products in pharmacies in dollars (sales_dollars).
From fact, we have relations to time dimension d_date, in which we have additional indicator if date is in current month (curr_mth_ind), 
if it is in last 3 months (roll_qtr_ind) and if it is in current year (ytd_ind).
Product dimension d_product holds information about producer, brand and type. 
Shop dimension (pharmacy) d_shop holds geographical information where sales occurred.

There is a requirement to present sum of sales broke by state for different types of product in last 3 months (rolling quarter). For states and types 
with no sales we need to return 0.

Creation of target table r_state_prod_types

![Target model](/images/dqdd/target_model.png)

# Test cases 

To get quick feedback, if our aggregated data meeting all the requirements, firstly we need to define test cases:

1. There are all possible combination of state and type of product 
2. Each combination of state and type of product can exist only once
3. Total sum in target table r_sate_prod_types is equal to quarterly total sum in source table f_pharma_sales
4. Each type sum in r_state_prod_types is equal to quarterly type sum in f_pharma_sales
5. Each state sum in r_state_prod_types is equal to quarterly type sum in f_pharma_sales


Creation of test data set, to let test each possible case:

### d_date 
![data in d_date](/images/dqdd/d_date.png)
Insert only two dates, one in rolling quarter, second out of it.

### d_product 
![data in d_product](/images/dqdd/d_product.png)
3 types of products

### d_shop
![data in d_shop](/images/dqdd/d_shop.png)
Two shops in different states


### f_pharm_sales
Data in f_pharma_sales can handle all cases of positive and negative sales in each type of product in each state.

![data in f_pharma_sales](/images/dqdd/f_pharma_sales.png )
![data in f_pharma_sales translated](/images/dqdd/f_pharma_sales_trans.png )

# Let's roll with tests! 
Going from words to action, let's start to create insert statement and tests.

### test#1
First test which we are defining checks if there are all combinations of state and product.

Query on source data:
```
    SELECT prd.type, shp.state
      FROM d_product prd
CROSS JOIN d_shop shp;
```
Query on target table:
```
    SELECT prod_type, shop_state
      FROM r_states_prod_types;
```	  
Test setup in FitNesse:
![fitness setup](/images/dqdd/1st_test_def.png)

First run:
![First run without code](/images/dqdd/1st_test_fail.png)
Of course the test failed, because there is no data in the target table.

We are creating as little as possible code to load target table to pass the test.
```
CREATE OR REPLACE PROCEDURE load_r_states_prod_types IS

BEGIN
    EXECUTE IMMEDIATE 'TRUNCATE TABLE r_states_prod_types';
   
    INSERT INTO r_states_prod_types (prod_type,shop_state)
        SELECT prd.type AS prod_type
             , shp.state AS shop_state
          FROM d_product prd
    CROSS JOIN d_shop shp;
    
    COMMIT;
END load_r_states_prod_types;
```
Execution of procedure:
```
EXEC load_r_states_prod_types;
```
Execution of test:
![first green test](/images/dqdd/1st_test_pass.png)

Hurray!

### test#2
We are writing second test, which meets second requirement. It is checking DQ, however it will be already passed, as in current state
we are not generating duplicates. However, it will be very helpful when we add more joins, to check after each new portion of code if we are still not duplicate data.
Following query should return 0:
```
SELECT COUNT(*)
  FROM (SELECT COUNT(*)
          FROM r_states_prod_types
      GROUP BY prod_type, shop_state
        HAVING COUNT(*) > 1)
```		
Test run in FitNesse:
![second green test](/images/dqdd/2nd_test_pass.png)

### test#3
Next test checks, if total sum in target table is equal to quarterly sum in source table. Test on source table:
```
    SELECT SUM(sales_dollars) AS sales_dollars
      FROM f_pharma_sales fhs
INNER JOIN d_date dat
        ON dat.date_id = fhs.date_id
       AND dat.roll_qtr_ind = 'Y';
```	   
Test on target table:
```
SELECT SUM(sales_dollars) AS sales_dollars
  FROM r_states_prod_types;
```
![third red test](/images/dqdd/3rd_test_fail.png)

We are writing minimal amount of code (or actually modifying existing query), to pass a test:
```
         SELECT prd.type AS prod_type
              , shp.state AS shop_state
              , SUM(fph.sales_dollars) AS sales_dollars
           FROM d_product prd
     CROSS JOIN d_shop shp
LEFT OUTER JOIN f_pharma_sales fph
             ON fph.product_id = prd.product_id
            AND fph.shop_id = shp.shop_id
     INNER JOIN d_date dat
             ON dat.date_id = dat.date_id
            AND dat.roll_qtr_ind = 'Y'
       GROUP BY prd.type
              , shp.state; 
```			  
test run:
![third red test#2](/images/dqdd/3rd_test_second_fail.png)

And it failed with quite big negative value.
Keeping in mind our test data set, we know, that for some reason, the biggest negative value, which is not in our last 3 months, was included to sum.
Closer look to our code, and we can see aliasing error in condition to join (probably because of copy and paste and not changing to the proper one):
```
     INNER JOIN d_date dat
             ON dat.date_id = dat.date_id
```			 
Quick fix:
```
 ON dat.date_id = fph.date_id
```
Reload of data and test run:
![trzeci zielony test](/images/dqdd/3rd_test_pass.png)

Additionally, we are running all tests to check if we didn't break anything for previous requirements:
![all good](/images/dqdd/all_tests.png)

Last two DQ requirements to check I'm leaving for willing reads.

# Test always!
Even for this simple case, by writing good tests before code we have immediate answer, if our code is proper or not.
Each time it's important to run all tests for current part of code, to have confirmation that previous steps are still ok.

Some of these tests can be used later as acceptance tests. Additionally, we can use them to check DQ on productional data, when before pushing them 
to client application we have time for final tests.

Using DQDD gives a huge gain in comparison to extreme case of not testing at all or manual testing.
And we have 3 big profits. First: it checks our code immediately and reduces time to fix bugs in later phases. Second: by writing tests, 
which can be used in productional phase as DQ tests. Third: it reduces time of multiple rounds of manual tests.
It's very important to remember, that creation of tests and their run should be quick, easy and pleasant. We can't wait for hours to get answer, because
our tests should test quite small changes. Small test data set, which cover all tests cases, can help. Easy in setup automation tool, like FitNesse, can help. 
By testing, we can help.

I recommend. I encourage. I'm sending my best. 

Tomek