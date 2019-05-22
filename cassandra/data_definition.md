# Basics of CQL

## Terminology
* Partition Key: practically, partition key is used to partition the data with the same key to the same note so it will increase read performance.
	```
	e.g. 'Audi' > Hash Function() > Token (64 bit integer value)
  * This token will be used to store data across the nodes in the cluster.
  * For example, node 1 will store from token range 0 to 2^64, node 2 will store from 2^64 till 2^128 and so on
	```
* Virtual Node: unrealistic node to be added in the partition range to make the real node store more data within that virtual node's token range
	```
	e.g. Node 1 has 256 GB, then we want to store more data on node 1, 
	we can do this by adding more virtual nodes to node 1 so it will cover more ranges of token
	```
* Data Center consists of Racks and Rack consists of nodes
	```
	e.g. the following means that there is only one rack1 in the datacenter1, that rack1 holds all the token partitions (vnodes) of 256
	C:\Users\chhat>nodetool status
  Datacenter: datacenter1
  ========================
  Status=Up/Down
  |/ State=Normal/Leaving/Joining/Moving
  --  Address    Load       Tokens       Owns (effective)  Host ID                               Rack
  UN  127.0.0.1  193.09 KiB  256          100.0%            a19b200f-fee1-449c-a9bc-72ef198c430f  rack1
  ```


## NOTE
* In cassandra data model design, the query first approach is used.
* Either the model is designed to be fit for the query to be executed on the same table or not at all
* The model must be de-normalized
* Accessing the data by using partition key only. 
* If we want to access the data by primary key, then a new table is needed to be created with primary key as the partition key 

## CQL DDL

1. Keyspace
	* show Keyspace: `desc keyspaces;`
	* switch to different keyspace: `use <keyspace_name>`;
	* create keyspace: 
		```cql
		# Strategy Name: SimpleStrategy, NetworkTopologyStrategy
		# create keyspace <name> with <properties>
		create keyspace <keyspace_name> with replication={'class':'<strategy_name>','replication_factor': <num>} and dualable_writes <bool>;
		```
	* alter keyspace:
		```
		# alter keyspace <name> with <properties>
		alter keyspace <keyspace_name> with replication={'class':'<strategy_name>','replication_factor': <num>} and dualable_writes <bool>;
		```
	* drop keyspace: `drop keyspace <name>`
2. Tables
	* creating table: 
	```
	create table students(roll_num int, first_name varchar, last_name varchar, dob varchar, batch_id int, PRIMARY KEY(roll_num));
	```
	* alter table:
	```
	# alter table <name> <alter_instruction>
	alter table students add graduation_year int;
	```
	* to see tables
	```
	desc table;
	desc <table_name>;
	# e.g.
	desc students;
	```
	* to drop a table
	```
	drop table if exists <table_name>;
	# e.g.
	drop table if exists testing;
	```
	* to truncate table: `truncate table <name>`
3. Examples
```
CREATE KEYSPACE employee WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '1'} AND durable_writes = 'true';
-- split the records based on id
CREATE TABLE employee_by_id (id int PRIMARY KEY, name text, position text);
-- split the records based on car_make which is the partion key, and order it by id
CREATE TABLE employee_by_car_make (car_make text, id int, car_model text, PRIMARY KEY(car_make, id));
-- split the records based on car_make and order it by (age, id)
CREATE TABLE employee_by_car_make_sorted (car_make text, age int, id int, name text, PRIMARY KEY(car_make, age, id));
-- split the records based on car_make and car_model. 
-- this will help in partition data better like only the employee record with the same car_make and car_model will be stored in the same node
CREATE TABLE employee_by_car_make_and_model (car_make text, car_model text, id int, name text, PRIMARY KEY((car_make, car_model), id));
```
```
CREATE KEYSPACE university WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '1'} AND durable_writes = 'true';
create table if not exists students(roll_num varchar, first_name varchar, last_name varchar, dob varchar, batch_id int, graduation_year int, PRIMARY KEY(roll_num));
create table if not exists course(id int, name varchar, start_date varchar, end_date varchar, price int, PRIMARY KEY(id));
create table if not exists enrollment(id int, roll_num varchar, course_id int, PRIMARY KEY(roll_num, course_id)); 
insert into students(roll_num, first_name, last_name, dob, batch_id, graduation_year)
values ('1101601001', 'chhatra', 'chhorm', '12/22/1998', 3, 2020) 
if not exists; 
```
```
CREATE MATERIALIZED VIEW employee_by_id_name 
AS SELECT * FROM employee_by_id
WHERE name IS NOT NULL
PRIMARY KEY (name, id); 
```