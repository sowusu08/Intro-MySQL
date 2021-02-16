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
	* [Adding index](#adding-index-to-table)  
	* [Altering tables](#altering-table-schema)
7. [Load database structure](#load-schema)
8. [Populate tables](#load-data)  
9. [Querying records](#querying-records)  
10. [Check for warnings](#check-for-warnings)

<a name='vm'></a>
## Personal Linux VM
  * A brief tangent to discuss architecture: https://github.com/LinuxAtDuke/Intro-to-MySQL/blob/master/client-server-architecture.pdf  
  * Manage VM: *https://vcm.duke.edu/*  

<a name='access-shell'></a>
## Access MySql  

1. __connect to VM__  
_shell>>_ `ssh NetID@<VMhostname>`  
_shell>>_ `<NetID pswd>`

2. __open MySql shell__  
_shell>>_ `sudo -i`  
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

  ### Adding index to table  
_mysql>>_ `CREATE INDEX idx_<fieldname> ON <table_name>(<fieldname>);` creates index based on chosen field  

_msql>>_ `SHOW INDEX from <table in selected database>;` to get info on index  

  ### Altering table schema
  * altering individual columns  
_mysql>>_ `ALTER TABLE <table_name> MODIFY <column_name> <modified_attribute1> <modified_attribute2> ...` MODIFY changes column definition   
(i.e. INT (Type), NOT (NULL), NULL (Default)) but not its name  
_mysql>>_ `ALTER TABLE <table_name> RENAME COLUMN <old-name> TO <new_name>` renames column    
 

<a name='load-schema'></a>
## Loading DB structure/schema from external source
  ### From github repo
  1. _mysql>>_ `CREATE DATABASE <database_name>` create database
  2. _mysql>>_ `exit` exit to shell 
  3. _shell>>_ `git clone <repo_link.git>` grab file
  4. _shell>>_ `mysql -u root -p <database_name> < /root/<repo/path/to/filename.sql>` loads file into MySQL instance  
  5. _shell>>_ `mysql -u root -p <database_name>` access MySQL database directly from shell


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
	Ex: ```
		Value1	Value2	Value3 \N
		Value1	\N	Value3	Value4
		Value1	Value2	Value3	Value4
	    ```  
	    _NOTE: \N indicates null values_  
	    <br></br>
  	2. _mysql>>_ `LOAD DATA LOCAL INFILE '<path/to/data.txt>' INTO TABLE <table_name>;`  
	
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

### Store query results in file


		
## Check for warnings
_mysql>>_ `show warnings;`


<a name='unit5'></a>
## Unit 5: Writing queries to retrieve data

  * Simplest queries
	
		mysql> select * from LCL_genotypes;
		+------------------+--------------+-------------+----------+
		| IID              | SNPpos       | rsID        | genotype |
		+------------------+--------------+-------------+----------+
		| HG02463          | 10:60523:T:G | rs112920234 | TT       |
		| HG02463839238290 | 10:60523:T:G | rs112920234 | TT       |
		| HG02466          | 10:60523:T:G | rs112920234 | TT       |
		| HG02563          | 10:60523:T:G | rs112920234 | TT       |
		| HG02567          | 10:60523:T:G | rs112920234 | 00       |
		+------------------+--------------+-------------+----------+
		5 rows in set (0.00 sec)
	
		mysql> SELECT IID,rsID from LCL_genotypes WHERE genotype = 'TT';
		+------------------+-------------+
		| IID              | rsID        |
		+------------------+-------------+
		| HG02463          | rs112920234 |
		| HG02463839238290 | rs112920234 |
		| HG02466          | rs112920234 |
		| HG02563          | rs112920234 |
		+------------------+-------------+
		4 rows in set (0.00 sec)
	
		mysql> SELECT COUNT(*) from snp;
		+----------+
		| COUNT(*) |
		+----------+
		|        5 |
		+----------+
		1 row in set (0.04 sec)
	
		mysql> select * from snp;
		+-------------+------------+----------+---------+---------+----------------------+------------+------------+
		| rsID        | Chromosome | Position | Allele1 | Allele2 | DistanceToNearGene   | Gene       | SNPtype    |
		+-------------+------------+----------+---------+---------+----------------------+------------+------------+
		| rs112920234 |         10 |    60523 | G       | T       | dist=NONE;dist=32305 | NONE,TUBB8 | intergenic |
		| rs147855157 |         10 |    61372 | CA      | C       | .                    | .          | .          |
		| rs536439816 |         10 |    61386 | A       | G       | dist=NONE;dist=31442 | NONE,TUBB8 | intergenic |
		| rs536478188 |         10 |    60803 | G       | T       | dist=NONE;dist=32025 | NONE,TUBB8 | intergenic |
		| rs569167217 |         10 |    60684 | C       | A       | dist=NONE;dist=32144 | NONE,TUBB8 | intergenic |
		+-------------+------------+----------+---------+---------+----------------------+------------+------------+
		5 rows in set (0.00 sec)

  * Slightly more complex queries
                   
		mysql> select * from LCL_genotypes WHERE IID LIKE 'HG0246%'; 
		+------------------+--------------+-------------+----------+
		| IID              | SNPpos       | rsID        | genotype |
		+------------------+--------------+-------------+----------+
		| HG02463          | 10:60523:T:G | rs112920234 | TT       |
		| HG02463839238290 | 10:60523:T:G | rs112920234 | TT       |
		| HG02466          | 10:60523:T:G | rs112920234 | TT       |
		+------------------+--------------+-------------+----------+
		3 rows in set (0.00 sec)

  * "JOIN" GUIDANCE:  https://stackoverflow.com/questions/6294778/mysql-quick-breakdown-of-the-types-of-joins
  
  * MORE "JOIN" GUIDANCE:  https://www.javatpoint.com/mysql-join
  
  * (The default "JOIN" in MySQL is an "INNER JOIN")
  
		mysql> SELECT * FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID;
		+------------------+--------------+-------------+----------+-------------+------------+----------+---------+---------+----------------------+------------+------------+
		| IID              | SNPpos       | rsID        | genotype | rsID        | Chromosome | Position | Allele1 | Allele2 | DistanceToNearGene   | Gene       | SNPtype    |
		+------------------+--------------+-------------+----------+-------------+------------+----------+---------+---------+----------------------+------------+------------+
		| HG02463          | 10:60523:T:G | rs112920234 | TT       | rs112920234 |         10 |    60523 | G       | T       | dist=NONE;dist=32305 | NONE,TUBB8 | intergenic |
		| HG02463839238290 | 10:60523:T:G | rs112920234 | TT       | rs112920234 |         10 |    60523 | G       | T       | dist=NONE;dist=32305 | NONE,TUBB8 | intergenic |
		| HG02466          | 10:60523:T:G | rs112920234 | TT       | rs112920234 |         10 |    60523 | G       | T       | dist=NONE;dist=32305 | NONE,TUBB8 | intergenic |
		| HG02563          | 10:60523:T:G | rs112920234 | TT       | rs112920234 |         10 |    60523 | G       | T       | dist=NONE;dist=32305 | NONE,TUBB8 | intergenic |
		| HG02567          | 10:60523:T:G | rs112920234 | 00       | rs112920234 |         10 |    60523 | G       | T       | dist=NONE;dist=32305 | NONE,TUBB8 | intergenic |
		+------------------+--------------+-------------+----------+-------------+------------+----------+---------+---------+----------------------+------------+------------+
		5 rows in set (0.00 sec)
		
		mysql> SELECT IID,Position,Gene FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID;
		+------------------+----------+------------+
		| IID              | Position | Gene       |
		+------------------+----------+------------+
		| HG02463          |    60523 | NONE,TUBB8 |
		| HG02463839238290 |    60523 | NONE,TUBB8 |
		| HG02466          |    60523 | NONE,TUBB8 |
		| HG02563          |    60523 | NONE,TUBB8 |
		| HG02567          |    60523 | NONE,TUBB8 |
		+------------------+----------+------------+
		5 rows in set (0.00 sec)

		mysql> SELECT IID,Position,Gene FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID where LCL_genotypes.rsID = 'rs536478188';
		Empty set (0.00 sec)

		mysql> SELECT IID,Position,Gene FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID where snp.rsID = 'rs536478188';
		Empty set (0.00 sec)
		
		mysql> SELECT IID,Position,Gene FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID where IID = 'HG02466';
		+---------+----------+------------+
		| IID     | Position | Gene       |
		+---------+----------+------------+
		| HG02466 |    60523 | NONE,TUBB8 |
		+---------+----------+------------+
		1 row in set (0.00 sec)

  * What if I want the output to go directly into a file instead of to the screen?
	
		mysql> SELECT * INTO OUTFILE '/var/lib/mysql-files/colab_class_result.txt' \
		         FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' \
		         LINES TERMINATED BY '\n' \
		         FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID;
		Query OK, 5 rows affected (0.00 sec)

		mysql> SELECT IID,Position,Gene INTO OUTFILE '/var/lib/mysql-files/colab_class_result2.txt' \
		         FIELDS TERMINATED BY '\t' OPTIONALLY ENCLOSED BY '' ESCAPED BY '' \
		         LINES TERMINATED BY '\n' \
		         FROM LCL_genotypes JOIN snp ON LCL_genotypes.rsID = snp.rsID;
		Query OK, 5 rows affected (0.00 sec)
		
		mysql> exit
		Bye
		root@vcm-XXXX:~$ cat /var/lib/mysql-files/colab_class_result.txt
		"HG02463","10:60523:T:G","rs112920234","TT","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"
		"HG02463839238290","10:60523:T:G","rs112920234","TT","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"
		"HG02466","10:60523:T:G","rs112920234","TT","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"
		"HG02563","10:60523:T:G","rs112920234","TT","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"
		"HG02567","10:60523:T:G","rs112920234","00","rs112920234",10,60523,"G","T","dist=NONE;dist=32305","NONE,TUBB8","intergenic"

		root@vcm-XXXX:~$ cat /var/lib/mysql-files/colab_class_result2.txt
		HG02463	60523	NONE,TUBB8
		HG02463839238290	60523	NONE,TUBB8
		HG02466	60523	NONE,TUBB8
		HG02563	60523	NONE,TUBB8
		HG02567	60523	NONE,TUBB8
	

<a name='lab5'></a>
## Lab 5: Practice with INSERT, UPDATE, DELETE, and SELECT (with JOIN!)

  * Take some time to play around with queries we've talked about above...

<a name='unit6'></a>
## Unit 6: Useful ancillary information

  * please note that VCM VMs now default to powering down every morning at 06:00 am, so if you can't connect (starting tomorrow), the first thing to do is to login to https://vcm.duke.edu to verify that your VM is actually powered on.

  * sudo -- allows certain commands to be run with elevated privileges.  First, without:

		vcm@vcm-XXXX:~$ service mysql restart
		==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
		Authentication is required to restart 'mysql.service'.
		Authenticating as: root
		Password:
	
  * And now, with:

		vcm@vcm-XXXX:~$ sudo service mysql restart
		vcm@vcm-XXXX:~$ ps -aef | grep mysql 

  * To REBOOT the server itself: _note that this can also be done from the VCM webUI via "Power off" and then "Power on"_

		vcm@vcm-XXXX:~$ sudo shutdown -r now
		Connection to vcm-XXXX.vm.duke.edu closed by remote host.
		Connection to vcm-XXXX.vm.duke.edu closed
	
  * To change the configuration of the MySQL server, edit the "my.cnf" file __AND THEN RESTART THE mysql PROCESS!!__

		vcm@vcm-XXXX:~$ sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
			[making necessary edits to the file and saving them]
		vcm@vcm-XXXX:~$ sudo service mysql restart
	
  * To check the error log, use "cat" (or "more" or "less"...)

		vcm@vcm-XXXX:~$ sudo cat /var/log/mysql/error.log
	
