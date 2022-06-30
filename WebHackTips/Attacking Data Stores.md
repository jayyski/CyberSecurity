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
