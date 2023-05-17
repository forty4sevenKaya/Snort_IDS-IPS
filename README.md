This is a simple playground to set-up a IDS. 


# Enviroment
Our environment mirrors a simple Ubuntu environment (VM) as the user system and a Kali environment (VM) as the potential attacker. The goal is to seal the Ubuntu machine against information gathering and not to reveal any information to the attacking system.  For this we want to initially detect the attacks (IDS) before we stop them in the next step by preventive measures (IPS).

##### Ubuntu 64-Bit 
- Snort System 

##### Kali Linux 2023 (Linux kali 6.1.0-kali5-amd64)
- Attacker System Linux 


# Configuration
Snort comes with a wide range of predefined rule sets, all of which perform great detections. For understanding, however, it is useful to comment out these rules and develop your own IDS. This way, you can understand simple detection measures and exploit the predefined complex rules in an adapted way later on. 

## Step 1: setting the interface correctly
Within /etc/snort/snort.conf we can establish our initial configurations to allow Snort to read our traffic.

- setting `ipvar HOME_NET 192.* * *.* * *.0/24 
- setting `ipvar EXTERNAL_NET any` 	
- commenting out the pre rules of snort  
	- you can find this unter `Step #7: Customize your rule set`


## Step 2: Testing our config 
To validate our config we run the following command. We need to validate our configurations before each startup of Snort as IDS/IPS. 

- `sudo snort -T -i ens33 -c /etc/snort/snort.conf`

Our config should look like this, because we commented all the `Step #7: Customize your rule set` out, so we can define later our own rules.

![Snort Rules](https://github.com/forty4sevenKaya/Snort_IDS-IPS/blob/main/screens/Pasted%20image%2020230514171510.png)




## Step 3: Adding our rules
Now, to define our own rules, we can break down the existing rules as an aid, read the official documentation, or use a tool to generate the rules. 

The tool Snorpy is an excellent start to create complex rules as easy as possible. The generation can be followed step by step. Ideal for understanding.
[Snorpy 2.0 - Web Based Snort Rule Creator](http://snorpy.cyb3rs3c.net/))

+ to add our rules we can go to `/etc/snort/rules/local.rules` 
	+ typically there are no roles (empty) 

#### ICMP Rule: 
Here we created a simple ICMP alert rule for snort by using Snorpy. As you see, we just specify the `source_ip / source_port` (in our case any) and the `dest_ip` (this is edited in the `snort.conf`)  

![Snorpy](https://github.com/forty4sevenKaya/Snort_IDS-IPS/blob/main/screens/Pasted%20image%2020230514175150.png)

Afterwards we can create much more rules, which can added into the `local.rules`. In our case we create some specific to at least secure our virtual machine against some common stuff. 

1. Block Telnet Connections:
`drop icmp any any -> any any (msg:"Blocked ICMP Echo Request"; icmp-type 8; sid:1000001; rev:1;)`
2. Block SSH Bruteforce Attempts:
`alert tcp any any -> any 22 (msg:"SSH Bruteforce Attempt"; flow:to_server,established; threshold:type threshold, track by_src, count 3, seconds 60; sid:1000003; rev:1;)
`
3. Block FTP Bruteforce Attempts:
`alert tcp any any -> any 21 (msg:"FTP Bruteforce Attempt"; flow:to_server,established; threshold:type threshold, track by_src, count 3, seconds 60; sid:1000004; rev:1;)`

![Own Rules](https://github.com/forty4sevenKaya/Snort_IDS-IPS/blob/main/screens/Pasted%20image%2020230514181001.png)

## Step 4: Starting Snort 

sudo snort -q -l /var/log/snort/ -i ens33 -A console -c /etc/snort/snort.conf 
