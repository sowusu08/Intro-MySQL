Intro to MySQL 
=====================
Documentation: https://dev.mysql.com/doc/refman/8.0/en/tutorial.html  
Practice: *https://www.reddit.com/r/SQL/comments/b5pbij/any_recommendation_of_how_to_practice_your_mysql/*  

**Table of Contents**

1. [Personal Linux VM](#vm)
2. [Check structure of DBMS](#check-structure)
3. [Access mySQL](#access-shell)
4. [Add password to DBMS](#password) 
5. [Databases](#databases)
6. [Tables](#tables)  
	* [Accessing tables](#acessing-tables)  
	* [Creating tables](#creating-tables)  
	* [Deleting table](#delete-table)  
	* [Indexes/Keys](#keys)  
	* [Altering tables](#altering-table-schema)
7. [Load database structure](#load-schema)
8. [Populate tables](#load-data)  
	* [Manually](#adding-data-manually)  
	* [Loading in data](#loading-in-data)
10. [Querying records](#querying-records)  
11. [Check for warnings](#check-for-warnings)

<a name='vm'></a>
## Personal Linux VM
  * A brief tangent to discuss architecture: https://github.com/LinuxAtDuke/Intro-to-MySQL/blob/master/client-server-architecture.pdf  
  * Manage VM: *https://vcm.duke.edu/*  
  * disconnect terminal from VM: _vcm@vcm-XXXX:~$_ `sudo shutdown -r now`

<a name='access-shell'></a>
## Access MySql  

1. __connect to VM__  
_shell>>_ `ssh NetID@<VMhostname>`  
_shell>>_ `<NetID pswd>`

2. __open MySql shell__  
_shell>>_ `sudo -i` opens "super user" (root) prompt  
_shell>>_ `mysql -u root`  

* _If error install mysql first:_  
_shell>>_ `apt install -y mysql-server`  
_shell>>_ `mysql -u root`  

<a name='check-structure'></a>
## Check structure of DBMS
_mysql>>_ `status;`  for general info including current database and current user  
_mysql>>_ `show databases;` to see available databases on server    


<a name='password'></a>
## Add password
1. __check that no password is set__  
_mysql>>_ `SELECT Host, User, plugin, authentication_string from mysql.user where User='root';`  

_Returns:_
| Host      | User | plugin      | authentication\_string |
|:----------|:-----|:------------|:-----------------------|
| localhost | root | auth_socket |                        |  


2. __change root user's password__  
_mysql>>_ `update mysql.user set plugin='mysql_native_password' where user='root' and host='localhost';`  
_mysql>>_ `flush privileges;`  
_mysql>>_ `SET PASSWORD FOR 'root'@'localhost' = '<PASSWORD>';`  

3. __check that password is set__
_mysql>>_ `SELECT Host, User, plugin, authentication_string from mysql.user where User='root';`  


<a name='databases'></a>
## Databases, schema
  * Removing or creating databases  
_mysql>>_ `CREATE DATABASE <new_database>;` creates database  
_mysql>>_ `DROP DATABASE <database_name>;` removes database  

  * Accessing databases  
_mysql>>_ `use <database_name>;` to access a database   

  * Backing up databases  
_shell>>_ `mysqldump -p --no-data <database_name> > <file_name.sql>` --no-data doesn't include data in backup  
_NOTE: -p prompts user for password which will be hidden_


<a name='tables'></a>
## Tables  
  ### Accessing tables
_mysql>>_ `show tables;` to see available tables in selected database  
_mysql>>_ `describe <table_name>;` to display a table in selected database  

  _Return example:_  

	| Field    | Type         | Null | Key | Default | Extra |
	|:---------|:-------------|:-----|:----|:--------|:------|
	| IID      | varchar(16)  | NO   | PRI | NULL    |       |
	| SNPpos   | varchar(512) | NO   | PRI | NULL    |       |
	| rsID     | varchar(256) | NO   | MUL | NULL    |       |
	| genotype | varchar(512) | NO   |     | NULL    |       |  

  
  ### Creating tables  
_mysql>>_ 
```
CREATE TABLE <table_name> (
<field1_name> <field1_datatype> <can field be NULL or NOT NULL>,
<field2_name> <field2_datatype> <NULL or NOT NULL>,
...
PRIMARY KEY (`<field_that_is_primary>`, `<field_that_is_primary>`...),
KEY `idx_<field_used_for_index>` (<field_used_for_index>)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```  
<a name='delete-table'></a>
  ### Deleting tables
  _mysql>>_ `DROP TABLE <table_name>;`

  ### Keys  
* adding keys  
_mysql>>_ `ALTER TABLE <table_name> ADD INDEX <index_name> (<field_name>);` adds non-unique index (key)  
_mysql>>_ `ALTER TABLE <table_name> ADD PRIMARY KEY (<field_name>, <if_desired_field2_name>);` adds Primary keys  
_mysql>>_ `ALTER TABLE <table_name> ADD UNIQUE KEY (<field_name>, <if_desired_field2_name>);` adds Unique keys    

_NOTE: can also add unique and primary keys through new column defintion:_    
_mysql>>_ `ALTER TABLE <table_name> ADD COLUMN <column_name> <column_definition> UNIQUE KEY;` adds Unique key  

* removing keys  
_mysql>>_ `ALTER TABLE <table_name> DROP PRIMARY KEY (<field_name>);` removes ALL Primary keys  
_mysql>>_ `ALTER TABLE <table_name> DROP INDEX <index_name>;` removes adds non-unique index (key) where <index_name> is NOT Field name  
_mysql>>_ `ALTER TABLE <table_name> DROP INDEX <field_name>;` removes Unique key  

* to get info on index  
_msql>>_ `SHOW INDEX from <table in selected database>;`  

  ### Altering table schema
  * altering individual columns  
_mysql>>_ `ALTER TABLE <table_name> MODIFY <column_name> <modified_attribute1> <modified_attribute2> ...` MODIFY changes column definition   
(i.e. INT (Type), NOT NULL (field is not optional)) but not its name  
_mysql>>_ `ALTER TABLE <table_name> RENAME COLUMN <old-name> TO <new_name>` renames column    

  * adding and dropping columns    
_mysql>>_ `ALTER TABLE <table_name> ADD <new_column_name> <type> <NULL | NOT NULL>;`  
_mysql>>_ `ALTER TABLE <table_name> DROP <old_column_name>;`  

  * changing column order  
  after dropping column...  
_mysql>>_ `ALTER TABLE <table_name> ADD <column_name> <column_definition> FIRST;` puts column first  
_mysql>>_ `ALTER TABLE <table_name> ADD <column_name> <column_definition> AFTER <column_name>;` puts column after another column    


<a name='load-schema'></a>
## Loading DB structure/schema from external source
  ### From github repo
  1. _mysql>>_ `CREATE DATABASE <database_name>` create database
  2. _mysql>>_ `exit` exit to shell 
  3. _shell>>_ `git clone <repo_link.git>` grab file
  4. _shell>>_ `mysql -u root -p <database_name> < /root/<repo/path/to/filename.sql>` loads file into MySQL instance  
  5. _shell>>_ `mysql -u root -p <database_name>` access MySQL database directly from shell  

  ### From local .sql file
  1. _mysql>>_ `CREATE DATABASE <database_name>` create database  
  2. _shell>>_ `mysql -u root -p <database_name> < </local/path/to/file.sql>` loads file into MySQL instance  



<a name='load-data'></a>
## Populating tables
[Helpful documentation!](https://dev.mysql.com/doc/refman/8.0/en/loading-tables.html)  
  
  ### Adding data manually
  * Adding values one column at a time   
  _mysql>>_ `INSERT INTO <table_name> VALUES(<field1>, <field2>, <field3>, ...);`  
  
  * Adding values to multiple columns  
  _mysql>>_ `INSERT INTO <table_name> VALUES(<field1>, <field2>, <field3>, ...), (<field1>, <field2>, <field3>, ...), (<field1>, <field2>, <field3>, ...)...;`  
  
  ### Loading in data
  a. from github repo  
  	1. _mysql>>_ `set global local_infile=true;` change config to allow use of infile  
	2. _mysql>>_ `show global variables like 'local_infile';` check that config changed  
	3. _mysql>>_ `exit` relaunch mysql with new ability enabled  
	4. _shell>>_ `git clone <repo_link.git>` grab file  
	5. _shell>>_ `mysql --local_infile=1 -u root -p <database_name>` open mysql shell  
	5. _mysql>>_ `LOAD DATA LOCAL INFILE '/root/<repo/path/to/<data.infile>' INTO TABLE <table_name> FIELDS TERMINATED BY '\t';` load data  
	_NOTE: FIELDS TERMINATED BY '\t' indicates that in infile each column is separated by tab_  
	
  b. from local txt file  
  	1. Create txt file where each record is one line and each value is separated by tabs  
		Value1	Value2 	Value3 	\N   
		Value1	\N	Value3	Value4  
		Value1	Value2	Value3	Value4  
	 _NOTE: \N indicates null values_  
	 2. _mysql>>_ `LOAD DATA LOCAL INFILE '<path/to/data.txt>' INTO TABLE <table_name>;`  
	 _NOTE: Primary Key can not have empty field in data file therefore if using auto_increment   
	 use a Unique Key (can take empty fields in data file which it will auto fill)_
	
  ### Altering data values
  * Changing records  
  _mysql>>_ `UPDATE <table_name> SET <col_name>=<new_value>, <col_name>=<new_value>, <col_name>=<new_value>, ... WHERE <col_name>=<value_at_target_location>;`  
  
  * Removing records  
  _mysql>>_ `DELETE FROM <table_name>;` removes all records form table. 
  _mysql>>_ `DELETE FROM <table_name> WHERE <col_name>=<value_at_target_location>;` removes record from one row  


## Querying records
[Helpful Documentation here!](https://dev.mysql.com/doc/refman/8.0/en/retrieving-data.html)  
### Return specific records
_mysql>>_ `select * from <table_name>;` returns ALL records (NOT schema) from named table  
_mysql>>_ `select <col_name>, <col_name>, ... from <table_name> WHERE <col_name> = <target_value>;` returns records in specific columns  
_mysql>>_ `select * from <table_name> WHERE <col_name> LIKE 'first_few_characters%';` uses regex to query records  

### Join tables
* "JOIN" GUIDANCE:  https://stackoverflow.com/questions/6294778/mysql-quick-breakdown-of-the-types-of-joins
* MORE "JOIN" GUIDANCE:  https://www.javatpoint.com/mysql-join  
_mysql>>_ `SELECT <what-to-select> FROM <table1> JOIN <table2> ON <table1>.<table1_column_name> = <table2_name>.<table2_column_name>`  
joins tables by matching values of table1_column_name and table2_column_name and only returns selected columns (aka only returns shared data for slected columns)  

### Store query results in file
_mysql>>_ 
```
SELECT <select-statement> INTO OUTFILE '<path/to/store/file.txt>' \
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' \
LINES TERMINATED BY '\n' \
<JOIN-statement-if-necessary>;
```  
_NOTE: each column will be separated by ',' and each row by a return/enter_  

		
## Check for warnings
_mysql>>_ `show warnings;`
