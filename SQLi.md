# SQL injection cheat sheet

big payload cheatsheet can be found [here](https://portswigger.net/web-security/sql-injection/cheat-sheet)

## How to examine SQLi

1. Test input fields for SQLi 
   
   - The key is to trigger an SQL/internal server error
   
   - Try vaious payloads, you can find a lot [here](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
   
   - once an **Internal server error** appears, SQLi confirmed

2. determine the number of columns
   
   - add `ORDER BY 100` and keep decrementing the number until the error disappears 
   
   - or add `UNION SELECT NULL,NULL,NULL...`  and keep adding until the error disappears( see 1st remark)

3. find a column containing text 
   
   - not always all the returned columns are all displayed, thats why to retrieve data we must determine which column are being displayed ( see the 2nd remark)

4. identifying database version

5. determine present databases 

6. determine tables present in database

7. determine columns in table

8. retrieve data :'D

> steps 4 to 8 can be done using [this pdf](./SQLi_htb.pdf) 



## Reading local files

to be able to read local files, the curret user has to have `FILE` privilege. 

1. determine which user we are 
   
   ```sql
   SELECT USER()
   SELECT CURRENT_USER()
   SELECT user from mysql.user
   ```
   
   the output is the **grantee** (format root@localhost) and the user is root here
   
   2.determine our Privileges
   
   ```sql
   SELECT super_priv FROM mysql.user 
   ```
   
   if we have many users we can add `WHERE user= <user>`
   
   this returns `Y` (which means yes we have superuser priv) or `N`
   
   we can dump all priv using 
   
   ```sql
   SELECT grantee, privilege_type FROM information_schema.user_privileges
   ```

        (add `WHERE grantee=<our_grantee>` if needed )

3. LOAD_FILE

payload: 

```sql

```



## Tools

#### sqlmap

ez tool to determine if SQLi is present, 

- try sending request myself adn if Im brave enough exploit it myself

- if no balls -> copy sus request as cURL and modify the request to match sqlmap syntax: 
  
  ```shellsession
  $ sqlmap -u <url> -X <method> --batch 
  ```
  
  flags `--risk <nmuber between 1 and 3> --level<nmuber between 1 and 5>` can help if basic command didnt help, `*` marks the parameter to attack specifically, `--prefix` can help if I suspect a userful perfix 

## Remarks

1. On Oracle databases, every `SELECT` statement must specify a table to select `FROM` otherwise it will result in an error. There is a built-in table on Oracle called `dual` which you can use for this purpose. Example `UNION SELECT NULL,.. FROM dual`and this can be a way to determine the DBMS, if `ORDER BY` determines for example n columns and then `SELECT NULL,NULL,...( n times)` gives an error that can indicate that this is an Oracle database (try addind `FROM dual` and see if the error disappears)

2. in some lab, when I was searchig for  the column containg reflected text (ik there is 3 columns)  I encountered errors 
   
   ```sql
   UNION SELECT NULL,NULL,NULL        --✅ correct number of columns
   UNION SELECT 'N',NULL,NULL         --❌ first column not string-compatible
   UNION SELECT NULL,'N',NULL         --✅ second column string-compatible
   UNION SELECT NULL,NULL,'N'         --❌ third column not string-compatible
   ```
   
   here I tried at first glance to do `UNION SELECT 'a','b','c'` to see which one is being reflected in the page, but it returned an error, that means not all returned columns are `VARCHAR` , they can be `INT` 

3. **AN ERROR MAY MEAN THAT MY PAYLOAD IS NOT PERFECT (IT CAN HAVE SYNTAX FAULTS)** that doesnt mean always im on the wrong track or there is a trick that I dont know

4. most commonly in SQLi the DB is gonna be MYSQL, but in case of erros and idk the reason, try other databases, [here](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection) I can find how to determine the DB type and version

5. a payload like `SELECT * FROM logins WHERE (username='username' AND id > 1) AND password = 'password';`here the suffix `)-- -` is crucial for the payload to work. sometimes the payload need prefix or prefix 
