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
7. [Load data into tables](#load-data)

<a name='vm'></a>
## Personal Linux VM
  * A brief tangent to discuss architecture: https://github.com/LinuxAtDuke/Intro-to-MySQL/blob/master/client-server-architecture.pdf  
  * Manage VM: *https://vcm.duke.edu/*  

<a name='access-shell'></a>
## Access MySql  

1. __connect to VM__  
_shell>>_ `ssh spo8@<VMhostname>`  
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
_shell>>_ `mysqldump -p --no-data colab\_class > <file_name.sql>`  
_NOTE: -p prompts user for password which will be hidden_


<a name='tables'></a>
## Tables  
  * Accessing tables
_mysql>>_ `show tables;` to see available tables in selected database  
_mysql>>_ `describe <table_name>;` to display a table in selected database  

  _Return example:_  

	| Field    | Type         | Null | Key | Default | Extra |
	|:---------|:-------------|:-----|:----|:--------|:------|
	| IID      | varchar(16)  | NO   | PRI | NULL    |       |
	| SNPpos   | varchar(512) | NO   | PRI | NULL    |       |
	| rsID     | varchar(256) | NO   | MUL | NULL    |       |
	| genotype | varchar(512) | NO   |     | NULL    |       |  

  
  * Creating tables  
_mysql>>_ 
```
CREATE TABLE `<table_name>` (
`<field1_name>` <field1_datatype> <can field be NULL or NOT NULL>,
`<field2_name>` <field2_datatype> <NULL or NOT NULL>,
...
PRIMARY KEY (`<field_that_is_primary>`, `<field_that_is_primary>`...),
KEY `idx_<field_used_for_index>` (<field_used_for_index>)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```  

  * Adding index to table  
_mysql>>_ `CREATE INDEX idx_<fieldname> ON <table_name>(<fieldname>);` creates index based on chosen field  

_msql>>_ `SHOW INDEX from <table in selected database>;` to get info on index  
 

## Loading DB structure/schema from external source
  ### From github repo
  1. _mysql>>_ `CREATE DATABASE <database_name>` create database
  2. _mysql>>_ `exit` exit to shell 
  3. _shell>>_ `git clone <repo_link.git>` grab file
  4. _shell>>_ `mysql -u root -p <database_name> < /root/<repo/path/to/<filename.sql>` loads file into MySQL instance  
  _NOTE: use \ to escape _ in sql file name_  
  5. _shell>>_ `mysql -u root -p <database_name>` access MySQL database directly from shell


<a name='load-data'></a>
## Loading data
[Click here for helpful documentation!](https://dev.mysql.com/doc/refman/8.0/en/loading-tables.html)  


  2. _shell>>_ `mysql -u root -p colab_class < /root/Intro-to-MySQL/COLAB\_WITHOUT\_DATA.sql` loads file into MySQL instance  
  3. _shell>>_ `mysql -u root -p <database_name>;` opens database in mySQL shell  

  * grab the class files from the github repository
  
	_shell>>_ git clone https://github.com/LinuxAtDuke/Intro-to-MySQL.git

  * load the file into your MySQL instance
	
	_shell>>_ mysql -u root -p colab\_class < /root/Intro-to-MySQL/COLAB\_WITHOUT\_DATA.sql
	
  * now check out the results of the import
	
	_shell>>_ mysql -u root -p colab_class;
	
	_mysql>>_ show tables;
	
	_mysql>>_ DESCRIBE LCL_genotypes;
	
  * now manually modify the table schema
	
	_mysql>>_ ALTER TABLE LCL_genotypes MODIFY genotype VARCHAR(2048) NOT NULL;
	
	_mysql>>_ ALTER TABLE LCL_genotypes MODIFY SNPpos VARCHAR(767) NOT NULL;
	
	_mysql>>_ DESCRIBE LCL_genotypes;
		
		[take note of how this output looks different than it did before]
	
	_mysql>>_ DESCRIBE gwas_results;
	
	_mysql>>_ ALTER TABLE gwas\_results MODIFY study\_population VARCHAR(16) NOT NULL;
		
	_mysql>>_ DESCRIBE gwas_results;
		
		[take note of how this output looks different than it did before]
	
<a name='unit4'></a>
## Unit 4: Populating database with data

  * Data can be added either record by record...
	* _mysql>>_ INSERT INTO tbl\_name () VALUES();
		* E.g., _mysql>>_ INSERT INTO LCL\_genotypes (IID,SNPpos,rsID,Genotype) VALUES('HG02463','10:60523:T:G','rs112920234','TT');
		
	* _mysql>>_ INSERT INTO tbl\_name (a,b,c) VALUES(1,2,3),(4,5,6),(7,8,9);
		* E.g., _mysql>>_ INSERT INTO LCL_genotypes (IID,SNPpos,rsID,Genotype) VALUES('HG02466','10:60523:T:G','rs112920234','TT'),('HG02563','10:60523:T:G','rs112920234','TT'),('HG02567','10:60523:T:G','rs112920234','00');
	
	* _mysql>>_ INSERT INTO tbl\_name SET col\_name=expr, col\_name=expr, ...
		* E.g., _mysql>>_ INSERT INTO phenotypes SET LCL\_ID='HG02461', phenotype='Cells\_ml\_after\_3\_days', phenotypic\_value1='878000', phenotypic\_value2='732000', phenotypic\_value3='805000', phenotypic_mean='805000';
	
	
  * Or in bulk (from an INFILE)
	* _mysql>>_ LOAD DATA LOCAL INFILE '/root/Intro-to-MySQL/snp-data.infile' INTO TABLE snp FIELDS TERMINATED BY '\t';
		
	
  * __WATCH OUT FOR WARNINGS!__ [_NOTE: As of MySQL version 5.7, THIS COMMAND RETURNS A FATAL ERROR AS OPPOSED TO A WARNING_] E.g., _mysql>>_ INSERT INTO LCL\_genotypes (IID,SNPpos,rsID,Genotype) VALUES('HG024638392382903957','10:60523:T:G','rs112920234','TT');

		Query OK, 1 row affected, 1 warning (0.00 sec)
		
		mysql> show warnings;
		+---------+------+------------------------------------------+
		| Level   | Code | Message                                  |
		+---------+------+------------------------------------------+
		| Warning | 1265 | Data truncated for column 'IID' at row 1 |
		+---------+------+------------------------------------------+
		1 row in set (0.00 sec)
		
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
		
  * Also possible (obviously) to change records that already exist (either one at a time or in bunches)...

		mysql> UPDATE tbl_name SET col_name=expr, col_name=expr, ... WHERE where_condition
		E.g., UPDATE LCL_genotypes SET IID='HG0246383' WHERE IID='HG02463839238290';
		Query OK, 1 row affected (0.00 sec)
		Rows matched: 1  Changed: 1  Warnings: 0
	
		mysql> select * from LCL_genotypes;                                          
		+-----------+--------------+-------------+----------+
		| IID       | SNPpos       | rsID        | genotype |
		+-----------+--------------+-------------+----------+
		| HG02463   | 10:60523:T:G | rs112920234 | TT       |
		| HG0246383 | 10:60523:T:G | rs112920234 | TT       |
		| HG02466   | 10:60523:T:G | rs112920234 | TT       |
		| HG02563   | 10:60523:T:G | rs112920234 | TT       |
		| HG02567   | 10:60523:T:G | rs112920234 | 00       |
		+-----------+--------------+-------------+----------+
		5 rows in set (0.00 sec)
	
  * Or to remove records (either one at a time or in bunches).  [First lets look at the table contents BEFOREHAND]

		mysql> select * from phenotypes;
		+---------+-----------------------+--------------------+--------------------+-------------------+--------------------+
		| LCL_ID  | phenotype             | phenotypic_value1  | phenotypic_value2  | phenotypic_value3 | phenotypic_mean    |
		+---------+-----------------------+--------------------+--------------------+-------------------+--------------------+
		| HG02461 | Cells_ml_after_3_days |  878000.0000000000 |  732000.0000000000 | 805000.0000000000 |  805000.0000000000 |
		| HG02462 | Cells_ml_after_3_days |  742000.0000000000 |  453000.0000000000 | 348000.0000000000 |  514333.3000000000 |
		| HG02463 | Cells_ml_after_3_days | 1200000.0000000000 | 1140000.0000000000 | 960000.0000000000 | 1100000.0000000000 |
		+---------+-----------------------+--------------------+--------------------+-------------------+--------------------+
		3 rows in set (0.00 sec)
	
  * Now remove...
	* _mysql>_> DELETE FROM tbl\_name WHERE where\_condition; __MAKE SURE YOU SUPPLY A WHERE CLAUSE UNLESS YOU WANT TO DELETE ALL ROWS!__
		* E.g., mysql> DELETE FROM phenotypes WHERE LCL_ID='HG02463';
		
		Query OK, 1 row affected (0.01 sec)

  * How does it look now?

		mysql> select * from phenotypes;                     
		+---------+-----------------------+-------------------+-------------------+-------------------+-------------------+
		| LCL_ID  | phenotype             | phenotypic_value1 | phenotypic_value2 | phenotypic_value3 | phenotypic_mean   |
		+---------+-----------------------+-------------------+-------------------+-------------------+-------------------+
		| HG02461 | Cells_ml_after_3_days | 878000.0000000000 | 732000.0000000000 | 805000.0000000000 | 805000.0000000000 |
		| HG02462 | Cells_ml_after_3_days | 742000.0000000000 | 453000.0000000000 | 348000.0000000000 | 514333.3000000000 |
		+---------+-----------------------+-------------------+-------------------+-------------------+-------------------+
		2 rows in set (0.00 sec)		


<a name='lab4'></a>
## Lab 4: Adding data to your database

  * First, make the necessary configuration change to allow this functionality:
  
  	_mysql>>_ set global local_infile=true;
  		
  	_mysql>>_ show global variables like 'local_infile';
  		
  		+---------------+-------+
		| Variable_name | Value |
		+---------------+-------+
		| local_infile  | ON    |
		+---------------+-------+
		1 row in set (0.00 sec)
		
  * Re-launch mysql client with ability enabled from the client:
  
	_mysql>>_ exit
  
  	_shell>>_ mysql --local\_infile=1 -u root -p colab_class;

  * Quickly add data to three tables...
  
	_mysql>>_ LOAD DATA LOCAL INFILE '/root/Intro-to-MySQL/snp-data.infile' INTO TABLE snp FIELDS TERMINATED BY '\t';
	
	_mysql>>_ LOAD DATA LOCAL INFILE '/root/Intro-to-MySQL/lcl\_genotypes-data.infile' INTO TABLE LCL\_genotypes FIELDS TERMINATED BY '\t';
		
	_mysql>>_ show warnings;
	
	_mysql>>_ LOAD DATA LOCAL INFILE '/root/Intro-to-MySQL/phenotypes-data.infile' INTO TABLE phenotypes FIELDS TERMINATED BY '\t';

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
	
