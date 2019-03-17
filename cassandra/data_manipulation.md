# Data Manipulation
## NOTE
* Consistency: ANY, INTEGER (<= Replication Factor), Quorum = ( RF + 1 ) / 2 
	```
	i.e. for example, Replication Factor of 3, consistency is 2, 
  then once two of the nodes respond that they are successfully written the data, we will get the successful write
  For read, if consistency is 2 of RF = 3, the read will have to be done from 2 nodes, 
  and cassandra will decide which data is the most recent one and it will return the most recent one
  # check consistency by: CONSISTENCY
	```
## CQL
* Insert
	```
	INSERT INTO employee_by_id (id, name, position) VALUES (1, 'JOHN', 'MANAGER');
	INSERT INTO employee_by_id (id, name, position) VALUES (2, 'BOB', 'CEO');
	
	INSERT INTO employee_by_car_make (car_make, id, car_model) VALUES ('BMW', 1, 'Sports Car');
	INSERT INTO employee_by_car_make (car_make, id, car_model) VALUES ('BMW', 2, 'Sports Car');
	INSERT INTO employee_by_car_make (car_make, id, car_model) VALUES ('AUDI', 4, 'Truck');
	INSERT INTO employee_by_car_make (car_make, id, car_model) VALUES ('AUDI', 5, 'Hachback');
	
	INSERT INTO employee_by_car_make_and_model (car_make, car_model, id, name) VALUES ('BMW', 'HATCHBACK', 1, 'Bob');
	INSERT INTO employee_by_car_make_and_model (car_make, car_model, id, name) VALUES ('BMW', 'HATCHBACK', 2, 'Jhon');
	INSERT INTO employee_by_car_make_and_model (car_make, car_model, id) VALUES ('BMW', 'HATCHBACK', 3);
	INSERT INTO employee_by_car_make_and_model (car_make, car_model, id) VALUES ('AUDI', 'HATCHBACK', 3);
	```
* Select
	```
	SELECT * FROM employee_by_id where id = 1;
	
	SELECT * FROM employee_by_car_make where car_make = 'BMW';
	
	-- ERROR because id itself is not primary key nor partion key
	SELECT * FROM employee_by_car_make where id = 1;
	
	-- ERROR order by only supported the columns of the primary keys and not partition key too
	SELECT * FROM employee_by_car_make where car_make = 'BMW' ORDER BY car_model;
	
	-- ERROR because there are two partion keys but we give only one
	SELECT * FROM employee_by_car_make_and_model where car_make = 'BMW';
	
	SELECT * FROM employee_by_car_make_and_model where car_make = 'BMW' and car_model='HATCHBACK';
	```
	
	Update,  Timestamp, TTL
	```
	SELECT car_make, car_model, writetime(car_model) FROM employee_by_car_make;
	
	-- ERROR because car_make is the part of the primary key
	SELECT car_make, car_model, writetime(car_make) FROM employee_by_car_make;
	
	UPDATE employee_by_car_make SET car_model='TRUCK' where car_make='BMW' AND id=1;
	
	-- TTL make the specific value of the set in update valid for some specified time
	-- and it will be null after the expiration
	UPDATE employee_by_car_make USING TTL 10 SET car_model = 'TRUCK' WHERE car_make='BMW' AND id = 2;
  ```
  
  Collections
  ```
  -- add new column to a new table
  ALTER TABLE employee_by_id ADD phone set<text>;
  
  -- update
  update employee_by_id SET phone = {'345', '565'} WHERE id = 1;
  -- add 
  update employee_by_id SET phone = phone + {'852'} WHERE id = 1;
  -- delete
  update employee_by_id SET phone = phone - {'852'} WHERE id = 1;
  -- remove all
  update employee_by_id SET phone = {} WHERE id = 1;
  ```
  
  Secondary Index
  ```
  -- ALLOW FILTERING: not recommended
  -- filering based on non-partition key
  SELECT * FROM employee_by_id WHERE name='JOHN' ALLOW FILTERING;
  
  -- SECONDARY INDEX: not recommended
  -- allow us to query based on non-partition key
  CREATE INDEX ON employee_by_id (name);
  SELECT * FROM employee_by_id WHERE name='JOHN'; 
  ```
  
  UUIDs, Counters
  ```
  -- UUIDs: Unique, Unique based on time
  CREATE TABLE employee_by_uuid (id uuid PRIMARY KEY, first_name text, last_name text);
  INSERT INTO employee_by_uuid (id, first_name, last_name) VALUES (uuid(), 'John', 'Bus');
  
  CREATE TABLE employee_by_timeuuid (id timeuuid PRIMARY KEY, first_name text, last_name text);
  INSERT INTO employee_by_uuid (id, first_name, last_name) VALUES (now(), 'John', 'Bus');
  ```