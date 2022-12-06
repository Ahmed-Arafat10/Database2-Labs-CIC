# Database #2 - Lab #3

- We have to create 5 tables
    - `customer`
    - `order`
    - `order_details`
    - `product`
    - `category`

<hr>

- Schema of `product` table :
  - `pid` int primary key
  - `pname` varchar(255) not null unique
  - `price` money not null check(price > 0)
  - `description` varchar(500) not null
  - `cid` int foreign key

> Note: `money` data type exists only in `SQL Server` not `MySQL` 

- Let's start first with `product table` `Create` command


- First we have to create a `database` for our tables
````SQL
CREATE DATABASE cic_ecommerce
````
<hr>

- Second we will create `product` table
````SQL
CREATE TABLE product
(
    pid int PRIMARY KEY,
    pname varchar(255) not null unique,
    price float not null check ( price > 0 ),
    description varchar(500) not null,
    cid int 
)
````
> Note: we will not add `foreign key` at `cid` column cuz we didn't yet create `category` table

<hr>

- Schema of `order` table :
    - `oid` int(12) primary key
    - `order_date` datetime not null default( getdate() )
    - `custid` int


- Let's create `order` table
````SQL
CREATE TABLE order
(
    oid int PRIMARY KEY,
    order_date datetime not null default(getdate()),
    custid int
)
````
> You can get an error if you want to have a table with name `order`
as it is a `keyword` reserved in `SQL` (`ORDER BY`), so can name it `_order`
    
> You can use `Now()` function instead in `MySQL`
<hr>

- Schema of `order_details` table :
    - `pid` int primary key Foreign key
    - `oid` int primary key Foreign key
    - `order_quantity` int not null default(1)

> VIP Note: we can see that we have two `Primary keys` in `order_details`
this is because the relationship between `order` & `product` tables is
`Many-To-Many`, so in this case the `primary key` is called `composite primary key`
where we cannot EVER have the same combination for example if `pid` = 1 & `oid` = 1
we cannot have any record (row) having the same value, but we can have `pid` = 1 & `oid` is 2
or `pid` is 2 & `oid` is 1  <br>
`1 1` YES <br>
`1 2` YES <br>
`2 1` YES <br>
`1 1` NO (already exists) <br>
Also note, as both `pid` & `oid` are primary keys in `order` & `product` tables then
they are taken to `order_details` table as `foreign keys`
So, they both are `primary key` as well as `foreign key`

- Let's create `order_details` table
````SQL
CREATE TABLE order_details
(
    pid int Foreign key REFERENCES product(pid),
    oid int Foreign key REFERENCES order(oid),
    order_quantity int not null default(1),
    PRIMARY KEY(pid,oid)
)
````

In `MySQL`:
````SQL
CREATE TABLE order_details
(
    pid int,
    oid int,
    PRIMARY KEY(pid,oid),
    Foreign key (pid) REFERENCES product(pid),
    Foreign key (oid) REFERENCES _order(oid),
    order_quantity int not null default(1)
)
````
<hr>

- Schema of `category` table :
    - `cid` int primary key 
    - `cname` varchar(255) not null

- Let's create `order_details` table
````SQL
CREATE TABLE category
(
    cid int PRIMARY KEY,
    cname varchar(255) not null
)
````
<hr>

- Schema of `customer` table :
    - `custid` int primary key
    - `name` varchar(255) not null
    - `gender` bit not null
    - `email` varchar(255)

````SQL
CREATE TABLE customer
(
    custid int primary key,
    name varchar(255) not null,
    gender bit not null,
    email varchar(255)
)
````

> Note: we can create tables using `SQL Commands` or using `Wizard` in `SQL Server`

- Now all we have to do is to add a `foreign key constrain` in `product` table

````SQL
ALTER TABLE product
ADD CONSTRAINT FK_product_category
FOREIGN KEY (cid) REFERENCES category(cid);
````

- then add `foreign key constrain` in `_order` table
````SQL
ALTER TABLE _order
ADD CONSTRAINT fk_customer_order
FOREIGN key (custid) REFERENCES customer(custid)
````

<hr>

- To drop a `foreign key constrain`
- In `SQL Server`:
````SQL
ALTER TABLE product
DROP CONSTRAINT FK_product_category;
````

-  In `MySQL`:
````SQL
ALTER TABLE product
DROP FOREIGN KEY FK_product_category;
````

