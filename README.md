Ansible Role: Iptables
=========

Installs a simple iptables-classic firewall for RHEL/CentOS or Debian/Ubunty systems.  
For EL it uses traditional iptables package and iptables service for those who doesn't like firewalld.  
For Debian/Ubuntu role uses iptables-persistent package and service as well.  
This role for those who has a good knowledges of iptables and prefer to write complex rules on yourself.  
Also, role operates with firewall lists and  allows to define group and custom variables for fine tuning of your servers.  

Requirements
------------

None.

Role Variables
--------------

Available variables are listed below, along with default values (see defaults/main.yml):

```
### list of default rulesets - filter table
iptables_rules_default:
  initial:
    - INPUT -i lo -j ACCEPT
    - INPUT -p icmp  --icmp-type echo-request -j ACCEPT
    - INPUT -p icmp  --icmp-type echo-reply  -j ACCEPT
    - INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
    - INPUT -f -j DROP
    - INPUT -p tcp -m state --state INVALID -j DROP
    - INPUT -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 1/s -j ACCEPT
  whitelist:
    - INPUT -s 192.168.0.0/24 -j ACCEPT
  ssh:
    - INPUT -p tcp -m tcp --dport 22 -m state --state NEW -j ACCEPT
  http:
    - INPUT -p tcp -m tcp --dport 80 -m state --state NEW -j ACCEPT
  https:
    - INPUT -p tcp -m tcp --dport 443 -m state --state NEW -j ACCEPT
  reject:
    - INPUT -j REJECT --reject-with icmp-host-prohibited

iptables_rules_nat_default:
  snat:
    - POSTROUTING -s 192.168.0.0/24 -o extInt -j SNAT --to-source my_real_ip
  

### enabled rules
iptables_rules_enabled:
  - initial
  - whitelist
  - ssh
  - reject

### rules for nat table
# disabled by default
iptables_rules_nat_enabled: []
```

You can define as many lists as you want for different groups and servers, and activate them via 
`iptables_rules_enabled` variable.  


Dependencies
------------

None.

Example Playbook
----------------

```
    - hosts: servers
      vars_files:
        - vars/main.yml
      roles:
         - { role: skandyla.iptables }
```

Inside *vars/main.yml:*  
```
# define lists:
iptables_rules:                                                                      
  whitelist:                                                                         
    - INPUT -s 192.168.33.0/24 -j ACCEPT                                             
  custom:                                                                            
    - INPUT -p tcp -m tcp --dport 8443 -m state --state NEW -j ACCEPT                
                                                                                     
                                                                                     
# enabled rules order take matter!                                                 
iptables_rules_enabled:                                                              
  - initial                                                                          
  - whitelist                                                                        
  - http                                                                             
  - ssh                                                                              
  - custom                                                                           
  - reject   
```


License
-------

BSD

Author Information
------------------

Sergey Kandyla
