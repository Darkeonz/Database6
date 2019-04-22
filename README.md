# Database6

Excercise1


            query
            select customerName, salesRepEmployeeNumber, customers.city, employees.officeCode, employees.employeeNumber 
            from customers, offices, employees where salesRepEmployeeNumber=employeeNumber and employees.officeCode=offices.officeCode
            group by customerName;

https://i.imgur.com/wxvUdrO.png

As seen above, I have one red table offices that performs a full scan which is costly with only 7 rows, as it has to search through all the elements in the table. The employees and customers table has archived a very low costs performance as it only had to go through 3 and 7 rows. The total cost is 39.80 with 175 rows and the GROUP operation is insignificant in this situation.

Excercise2
Changing the order did not improve the performance. The table offices is searched by primary key so it is a default index. Generally one can improve performance by creating an index to keep the cost low. The relevant columns had pre-created indexes. The columns employeeNumber, reportsTo and officeCode has indexes hence why the performance is low cost. 

            select 
            customerName, salesRepEmployeeNumber, customers.city, employees.officeCode, employees.employeeNumber
            from offices, employees,customers
            where offices.officeCode = employees.officeCode  
            and salesRepEmployeeNumber=employeeNumber
            group by customerName;

            select reportsTo, officeCode from employees;
            select officeCode from offices;

            SELECT customerName, salesRepEmployeeNumber, customers.city, employees.officeCode, employees.employeeNumber
            FROM employees , offices,  customers USE INDEX(salesRepEmployeeNumber)
            where employees.officeCode=offices.officeCode
            and salesRepEmployeeNumber=employeeNumber  
            group by customerName;

Excercise3

            /*group by*/
            select (quantityOrdered * priceEach) as sale, offices.officeCode, offices.country, offices.state, amount, 
            MAX(amount) as maxpayment
            from orderdetails, orders, customers, employees, offices, payments
            where orderdetails.orderNumber = orders.orderNumber
            and orders.customerNumber = customers.customerNumber
            and salesRepEmployeeNumber = employeeNumber
            and payments.customerNumber = customers.customerNumber 
            and employees.officeCode = offices.officeCode 
            group by offices.officeCode order by sale desc;


            /*windowing*/
            select 
            (quantityOrdered*priceEach), priceEach, SUM(priceEach)OVER (PARTITION BY offices.officeCode)total, offices.country, 
            offices.state, amount, offices.officeCode,
            MAX(amount) OVER (PARTITION BY offices.officeCode) maxpayment
            from orderdetails, orders, customers, employees, offices, payments
            where orderdetails.orderNumber = orders.orderNumber
            and orders.customerNumber = customers.customerNumber
            and salesRepEmployeeNumber = employeeNumber
            and payments.customerNumber = customers.customerNumber 
            and employees.officeCode = offices.officeCode; 




The execution plan is for a GROUP BY query.



https://i.imgur.com/0a7aBO5.png
 
 
For windowing
 
https://i.imgur.com/r60s02i.png
 
The cost for windowing is significantly higher than the group query.



Exercise 4
With joins

            use stackoverflow;

            SHOW INDEX FROM posts;
            SHOW INDEX FROM comments;
            SHOW INDEX FROM users;

            DROP procedure IF EXISTS `textsearch`;
            DELIMITER $$
            CREATE PROCEDURE `textsearch` ()
            BEGIN
            select Title, DisplayName, OwnerUserId from posts, users, comments
            where Title LIKE '%grounds%' and posts.Id = comments.PostId and users.Id = OwnerUserId;
            END$$
            DELIMITER ;

            call textsearch();

            select Title, DisplayName, OwnerUserId from posts, users, comments
            where Title LIKE '%grounds%' and posts.Id = comments.PostId and users.Id = OwnerUserId;

            select Title, users.Id, Text, OwnerUserId from posts, users, comments
            where Title LIKE '%grounds%' and posts.Id = comments.PostId and users.Id = OwnerUserId;

https://i.imgur.com/AJDC7n5.png
 
Using joins with UserId instead of DisplayUsername makes no real difference as mysql will not be affected much by use of joins. There might be one full scan table but the other joins will not cost much due to not performing full scan for the joined tables.  

Exercise 5
In natural language full text search, the search is done by relevance. This method also returns no rows if the search string length is less than 4 characters. It will not return query answer if the words are stopwords. In the Boolean full text search the searching is done in word instead of concept as the full text search. 
The Boolean text search and the full text search in this example are very low cost and effective.

 
            Alter table posts ADD FULLTEXT (Title);

            DROP procedure IF EXISTS `fulltextsearch`;
            DELIMITER $$
            CREATE PROCEDURE `fulltextsearch` (keyword varchar(100))
            BEGIN
            Select Title, OwnerUserId, users.Id from posts, users, comments where match(Title) AGAINST (keyword)
            and posts.Id = comments.PostId and users.Id = OwnerUserId;
            END$$
            DELIMITER ;

            call fulltextsearch('grounds');
            call fulltextsearch('coffee');
            call fulltextsearch('should');

            Select Title, OwnerUserId, users.Id from posts, users, comments where match(Title) AGAINST ('coffee')
            and posts.Id = comments.PostId and users.Id = OwnerUserId;

            Select Title, OwnerUserId, users.Id from posts, users, comments where match(Title) AGAINST ('coffee' in boolean mode)
            and posts.Id = comments.PostId and users.Id = OwnerUserId;

https://i.imgur.com/oZXUzyd.png

