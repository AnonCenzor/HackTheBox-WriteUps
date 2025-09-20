# HTB Retired Machine Write Up: Cap

## Enumeration:

- IP: 10.10.10.245

- open ports: 
	- 21/tcp vsftpd 3.0.3
	- 22/tcp OpenSSH Ubuntu 8.2p1
	- 80/tcp http Gunicorn


## Initial Access 

- we found that port 80 was serving http, so we connected using http://10.10.10.245
- we found a Dashboard of sorts while being logged in as "Nathan"
- we found that there was a section that contained PCAP files and when navigating to there we be redirected from /capture -> /data/<ID>
- we are able to retrieve scans by appending different numbers to /data/*
- using [ffuf|https://github.com/ffuf/ffuf], we've extracted some IDs that return information, but most importantly, find the ID 0 (which is typical for admin), to return a 200
- we then append /data/00 and are able to grab the pcap and use wireshark to analyze it
- in there, we find FTP protocol being used
	- in these FTP protocol packets, we find Nathan's password in cleartext: Buck3tH4TF0RM3!
- using this, we login to ftp and grab the user.txt file
- we attempt the same password for ssh, and it works

## Privilege Escalation

- now we need to escalate our privileges
- we use linPEAS [linPEAS|https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS] to find files/file paths we can exploit to escalate our privileges
- using this, we find that /usr/bin/python3.8 has something that certainly escalate us
- it states that cap_setuid & cap_net_bind_service+eip are configured for use(which is not default)
- thus, we can run python3 to create a script that sets our ID to 0 (root) and spawn a bash for us as root
	import os
	os.setuid(0)
	os.system("/bin/bash")
- once we're root, we grab the root flag from the root directory