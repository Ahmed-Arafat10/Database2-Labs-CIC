
-- 6232B Demonstration 2A - Data and Domain Integrity

-- Step 1: Open the Demonstration 2A Script File
--         Review the requirements for a table design

/*
Table called dbo.ProspectiveHire

Columns and requirements are:

ProspectiveHireID (identifies the row)
GivenNames
FamilyName
FullName
CountryCode   (used to hold a 3 letter ISO code)
StateOrRegion (used to hold details of a state or region  if the countrycode is USA, this must be 2 characters long)
DateOfBirth
LikelihoodOfJoining (value from 1 to 5 with 5 being most likely but 3 as the default if not specified)

 What constraints should be put in place?

*/
 
-- Step 2: Determine the data types, nullability, default and check constraints
--         that should be put in place

-- Step 3: Check the outcome with this proposed solution
````sql
USE tempdb;
GO

CREATE TABLE dbo.ProspectiveHire
(
	ProspectiveHireID int NOT NULL,
	GivenNames nvarchar(30) NOT NULL,
	FamilyName nvarchar(30) NULL,
	FullName nvarchar(50) NOT NULL,
	CountryCode nchar(3) NOT NULL
	  CONSTRAINT CHK_ProspectiveHire_CountryCode_Length3
	  CHECK (LEN(CountryCode) = 3),
	StateOrRegion nvarchar(20) NULL,
	DateOfBirth date NOT NULL
	  CONSTRAINT CHK_ProspectiveHire_DateOfBirth_NotFuture
	  CHECK (DateOfBirth < SYSDATETIME()),
	LikelihoodOfJoining int NOT NULL
	  CONSTRAINT CHK_ProspectiveHire_LikelihoodOfJoining_Range1To5
	  CHECK (LikelihoodOfJoining BETWEEN 1 AND 5)
	  CONSTRAINT DF_ProspectiveHire_LikelihoodOfJoining
	  DEFAULT (3),
	CONSTRAINT CHK_ProspectiveHire_US_States_Length2
	CHECK (CountryCode <> N'USA' OR LEN(StateOrRegion) = 2)
);
GO

-- Step 4: Execute statements to test the actions of the integrity constraints

-- INSERT a row providing all values ok:

INSERT INTO dbo.ProspectiveHire 
  ( ProspectiveHireID, GivenNames, FamilyName, FullName,
    CountryCode, StateOrRegion, DateOfBirth, LikelihoodOfJoining)
  VALUES (1, 'Jon','Jaffe','Jon Jaffe',
          'USA', 'WA', '19730402', 4);
GO
SELECT * FROM dbo.ProspectiveHire;
GO

-- Step 5: INSERT rows that test the nullability and constraints

-- INSERT a row that fails a nullability test

INSERT INTO dbo.ProspectiveHire 
  ( ProspectiveHireID, GivenNames, FamilyName, 
    CountryCode, StateOrRegion, DateOfBirth, LikelihoodOfJoining)
  VALUES (2, 'Jacobsen','Lola',
          'USA', 'W', '19730405', 2);
GO

-- INSERT a row that fails the country code length test

INSERT INTO dbo.ProspectiveHire 
  ( ProspectiveHireID, GivenNames, FamilyName, FullName,
    CountryCode, StateOrRegion, DateOfBirth, LikelihoodOfJoining)
  VALUES (2, 'Jacobsen','Lola', 'Lola Jacobsen',
          'US', 'WA', '20600405', 2);
GO

-- INSERT a row that fails the date of birth test

INSERT INTO dbo.ProspectiveHire 
  ( ProspectiveHireID, GivenNames, FamilyName, FullName,
    CountryCode, StateOrRegion, DateOfBirth, LikelihoodOfJoining)
  VALUES (2, 'Jacobsen','Lola', 'Lola Jacobsen',
          'USA', 'WA', '20600405', 2);
GO

-- INSERT a row that fails the US State length test

INSERT INTO dbo.ProspectiveHire 
  ( ProspectiveHireID, GivenNames, FamilyName, FullName,
    CountryCode, StateOrRegion, DateOfBirth, LikelihoodOfJoining)
  VALUES (2, 'Jacobsen','Lola', 'Lola Jacobsen',
          'USA', 'W', '19730405', 2);
GO

-- Step 5: Query sys.sysconstraints to see the list of constraints 
--         that have been cataloged

sp_helpconstraint prospectivehire

--------------------------------------------------------------------------------------------------
-- 6232B Demonstration 3A - Entity and Referential Integrity

-- Step 1: Open a new query window to tempdb

USE tempdb;
GO

-- Step 2: Create the Customer and CustomerOrder tables
--         and populate them

CREATE TABLE dbo.Customer
(
	CustomerID int IDENTITY(1,1) PRIMARY KEY,
	CustomerName nvarchar(50) NOT NULL
);
GO
INSERT dbo.Customer
  VALUES (' Marcin Jankowski'),('Darcy Jayne');
GO

CREATE TABLE dbo.CustomerOrder
(
	CustomerOrderID int IDENTITY(1,1) PRIMARY KEY,
	CustomerID int NOT NULL
	  FOREIGN KEY REFERENCES dbo.Customer (CustomerID),
	OrderAmount decimal(18,2) NOT NULL
);
GO

-- Step 3: Select the list of customers and 
--         perform a valid insert into the CustomerOrder table

SELECT * FROM dbo.Customer;
GO

INSERT INTO dbo.CustomerOrder (CustomerID, OrderAmount)
  VALUES (1, 12.50), (2, 14.70);
GO

-- Step 4: Try to insert a CustomerOrder row for an invalid customer
--         Note how poor the error messages look when constraints are 
--         not named appropriately

INSERT INTO dbo.CustomerOrder (CustomerID, OrderAmount)
  VALUES (3, 15.50);
GO

-- Step 5: Try to remove a customer that has an order
--         Again note how the poor naming doesn�t help much.

DELETE FROM dbo.Customer WHERE CustomerID = 1;
GO

-- Step 6: Remove the foreign key constraint and 
--         replace it with a named constraint with cascade. 
--         Note that you will need to copy into this code the 
--         name of the constraint returned in the error from the 
--         previous statement. This is part of the problem 
--         when constraints are not named.

ALTER TABLE dbo.CustomerOrder
  DROP CONSTRAINT FK__CustomerO__Custo__66603565;
GO

ALTER TABLE dbo.CustomerOrder
  ADD CONSTRAINT FK_CustomerOrder_Customer
  FOREIGN KEY (CustomerID) 
  REFERENCES dbo.Customer (CustomerID)
  ON DELETE CASCADE;
GO

-- Step 7: Select the list of customer orders, try a delete again
--         and note that the delete is now possible

SELECT * FROM dbo.CustomerOrder;
GO
DELETE FROM dbo.Customer WHERE CustomerID = 1;
GO

-- Step 8: Note how the cascade option caused the orders for the 
--         deleted customer to also be deleted

SELECT * FROM dbo.CustomerOrder;
GO

-- Step 9: Try to drop the referenced table and note the error:
DROP TABLE dbo.Customer;
GO

----------------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------------------------


-------------------------------Check Constraints--------------------------------------------------
use tempdb
go
--multiple constraints
--one is table level constraint
--data must satisfy ALL constraints
create table test2000 
  ( id int not null constraint idNotGtr100 check(id > 100),
    name char(10) null,
    constraint idNotEven check(id%2=0))
go
sp_helpconstraint test2000
go
select * from test2000
--try 1
insert into test2000 values( 1,'mohamed')

insert into test2000 values (101,'Mohamed')

insert into test2000 values (102,'Mohamed')
go
select * from test
go
drop table test
go

--Using like clause for formatting
create table test2011
  (id int not null,
   phone_no char(13) null
     constraint phoneformat
       check (phone_no like 
          '([0-9][0-9][0-9])[0-9][0-9][0-9]-[0-9][0-9][0-9][0-9]')
)
go
--note constraint with null values
insert into test2011 values(1,null)
select * from test2011
go
insert into test2011 values(2,'(704)933-5555')
select * from test2011
go
insert into test2011 values(3,'(704)9335555')
insert into test2011 values(4,'704 933-5555')
select * from test
go
-- you may also temporarily turn off check constraints
alter table test2011 nocheck constraint phoneformat
go

drop table test
go

--do a state_cd example
create table test2012
  (id int not null,
   gender char(6) not null
    constraint validgender check(gender in ('male','female')))
go
sp_helpconstraint test2012
insert into test2012 values (1, 'male')
insert into test2012 values (2, 'female')
insert into test2012 values (3, 'sgr')
select * from test2012
go

alter table test drop constraint validstates
go
alter table test 
  add constraint validstates check(state_cd in ('NC','UT','CA'))
go

insert into test values (1, 'UT')
select * from test
go
drop table test
go
-----------------------------------------------------------------------------------------------------------

---------------Default-------------------------------

========================
2)DEFAULT Constraints:
======================
use nicegroup
CREATE TABLE Location(
LocationID smallint IDENTITY(1,1) NOT NULL,
[Name] nvarchar(50) NOT NULL,
[CostRate] smallmoney NOT NULL CONSTRAINT DF_Location_CostRate DEFAULT ((0.00)),
Availability decimal(8, 2) NOT NULL CONSTRAINT DF_Location_Availability DEFAULT ((0.00)),
ModifiedDate datetime NOT NULL CONSTRAINT [DF_Location_ModifiedDate] DEFAULT (getdate())
)

--create the table with no default
drop table defaulttest
create table defaulttest
  (id int not null,
   comment varchar(128) null)
go
--add a record,provide NO comment value
insert into  defaulttest (id) values (1)
go
--what default was used?
select * from defaulttest
go
--now add a default constraint
alter table defaulttest
   add default 'No Comment' for comment
go
--insert another record
insert into  defaulttest (id) values (2)
go
--confirm the default was used
select * from defaulttest
go
--insert another record
insert into  defaulttest (id,comment)
  values (3, 'I have a comment')
go
--confirm the data was added
select * from defaulttest
go
--can you put a null value in a column with a default?
insert into defaulttest values (4, null)
go
select * from defaulttest
go
drop table defaulttest
------------------------------------------------------------------------------------
-------------------Unique
--------------------------------------------------------------------------------------
create table test
 (id int not null,
  name char(10) null unique)
go
sp_helpconstraint test
go
--Note index name same as constraint name
sp_help test
go
drop table test
go

create table test
 (id int not null,
  name char(10) null 
    constraint NameMustBeUnique 
      unique nonclustered (name))
go

insert into test values (1,'Vickie')
go
insert into test values (2,'Wayne')
go
select * from test
go
--Will not allow duplicate name
insert into test values (3,'Vickie')
go
select * from test
go
insert into test values (4,null)
go
--treats nulls as unique values
--can only add one null value
insert into test values (5,null)
go
select * from test
go
drop table test
go


create table test
 (id int not null,
  fname char(10) null,
  lname char(10) null 
    constraint NameMustBeUnique 
      unique nonclustered (fname,lname))
go
sp_help test
go
insert into test values(1,'Wayne','Snyder')
go
insert into test values (2,'Vickie','Snyder')
go
insert into test values(3,'Vickie','Hubbard')
go
select * from test
go
--The whole key must be a duplicate to be denied
insert into test values (2,'Vickie','Snyder')
go
select * from test
go
drop table test
-------------------------------------------------------------------------
------------------PK
--------------------------------------------------------------------------
create table test
 (id int not null
    constraint PK_id Primary Key Nonclustered(id),
  name char(10) null)
go
--Note index name same as constraint name
sp_help test
go

--can't drop via drop command
drop index test.pk_id
go

insert into test values (1,null)
go
insert into test values (1,null)
go
insert into test values (2,null)
go
select * from test
go
drop table test
go


--Use Alter table
create table test
 (id int not null,
  name char(10) null)
go

alter table test
  add constraint PK_id Primary Key Nonclustered(id)
go
insert into test values (1,null)
go
insert into test values (1,null)
go
insert into test values (2,null)
go
select * from test
go
drop table test
go

--Now add data first
create table test
 (id int not null,
  name char(10) null)
go
insert into test values (1,null)
go
insert into test values (1,null)
go
insert into test values (2,null)
go
select * from test
go
alter table test
  add constraint PK_id Primary Key Nonclustered(id)
go
drop table test
go

--multi column key
create table test
 (id int not null,
  orderid int not null,
  name char(10) null,
   constraint PK_id Primary Key 
      Nonclustered(id,orderid))
go
sp_helpconstraint test
go

insert into test values (1,3,null)
go
insert into test values (1,3,null)
go
insert into test values (1,2,null)
go
insert into test values (2,3,null)
go
select * from test
go
drop table test
go


-------------------------------------------------------------------------------------------
--FK
-------------------------------------------------------------------------------------------
--self referencing table 
create table emp
(emp_id int not null Primary key,
 emp_name char(10) not null,
 boss_id int null
  constraint boss_FK 
    references emp (emp_id))
go
sp_help emp
go
insert into emp values(1,'Mr.Smith',null)
go
insert into emp values (2, 'Wayne', 3)
go
insert into emp values (2, 'Wayne',1)
go
select * from emp
drop table emp
go

--Can replace a check constraint
create table state
(state_cd char(2) not null unique,
 state_name varchar(128) not null)
go
insert into state values('NC','North Carolina')
insert into state values('UT','Utah')
insert into state values('SC','South Carolina')
go

--column level constraint
create table emp
(emp_id int not null,
 emp_birth_state_cd char(2) null
  constraint state_cd_FK 
    Foreign Key 
    references state(state_cd),
 emp_name char(10) null)
go
drop table emp
go

--table level constraint
create table emp
(emp_id int not null,
 emp_birth_state_cd char(2) null,
 emp_name char(10) null,
   constraint state_cd_FK 
      Foreign Key (emp_birth_state_cd)
      references state(state_cd) )
go
drop table emp
go

--alter table 
create table emp
(emp_id int not null,
 emp_birth_state_cd char(2) null,
 emp_name char(10) null)
go
alter table emp
  add
   constraint state_cd_FK 
      Foreign Key (emp_birth_state_cd)
      references state(state_cd) 
go
--try to add GA
insert into emp values(1,'GA','Vickie')
go
--try to add SC
insert into emp values(1,'SC','Vickie')
select * from emp
go

--try to change state_cd
update emp set emp_birth_state_cd = 'NC'
select * From emp
go

--try to change it to a bad state
update emp set emp_birth_state_cd = 'FL'
select * From emp
go

--This will not work...why?
update state set state_cd = 'NO' where state_cd = 'NC'
go
select * from emp
select * from state
go
update state set state_cd = 'SO' where state_cd = 'SC'
go
select * from emp
select * from state
go
delete from state where state_cd = 'NC'
go

-- drop constraints
alter table emp drop constraint state_cd_FK
go
alter table emp
  add
   constraint state_cd_FK 
      Foreign Key (emp_birth_state_cd)
      references state(state_cd) 
      on update cascade
      on delete cascade
go
update state set state_cd = 'NO' where state_cd = 'NC'
go
select * from emp
select * from state
go
delete from state where state_cd = 'NO'
go
select * from emp
select * from state
go

--clean up
drop table emp
drop table state

--------------------------------------------------------------------

````
