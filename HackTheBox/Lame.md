# Lame
>URL : https://app.hackthebox.eu/machines/Lame 


IP : 10.129.197.50

## Info gathering
- Port Scanning
	- ![](https://i.imgur.com/aqhaBG8.png)
		- 21,22,139,445,3632
	- ![](https://i.imgur.com/4nP95qL.png)
		- Anonymous FTP
		- SMB
		- 3632 Port distccd

## File Protocol
- Try anonymous login FTP
	- ![](https://i.imgur.com/4SbeeBH.png)
	- It's empty
- Try anonymous login SMB 
	- ![](https://i.imgur.com/tAFlRJ1.png)
	- There are `tmp` and `opt` folder
	- Access tmp folder
		- ![](https://i.imgur.com/tM6RvLY.png)
		- ![](https://i.imgur.com/nfdeBLb.png)
			- Download all file
## Exploit Distccd
- Distccd_rce_CVE-2004-2687
	- https://gist.github.com/DarkCoderSc/4dbf6229a93e75c3bdf6b467e67a9855
	- ![](https://i.imgur.com/Q1Y4t7j.png)
- Run Reverse shell
	- ![](https://i.imgur.com/mtPM4ei.png)
	- ![](https://i.imgur.com/83ysiHm.png)
- Get User Flag
	- ![](https://i.imgur.com/40sakTp.png)
## Privilege escalation
- Run LinPEAS
	- ![](https://i.imgur.com/yxrx9q6.png)
		- NFS Exploit?
	- ![](https://i.imgur.com/tp6ptzW.png)
		- Eterm SGID Binary?
	- ![](https://i.imgur.com/zAYCC6R.png)
		- nmap SUID !!
- Nmap GTFOBins
	- Shell (2) Interactive shell
		- https://gtfobins.github.io/gtfobins/nmap/#suid
		- `nmap --interactive`
		- `nmap> !sh`
	- ![](https://i.imgur.com/Gw41QSr.png)
- Get Root Flag
	- ![](https://i.imgur.com/WXIvv23.png)
