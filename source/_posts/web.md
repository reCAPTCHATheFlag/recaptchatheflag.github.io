---
title: Web Cheat Sheet
category: cheatsheets
tag: web
author: Chara
---


### Websites

###### Overall Reference
* [🚩Practical CTF](https://book.jorianwoltjer.com/)  - Reference Book
* [HackTricks](https://book.hacktricks.xyz/) - Reference Book
* [PortSwigger](https://portswigger.net/web-security) - Reference Book
* [Google](https://www.google.com/) - Your Best Friend
* [ChatGPT](https://chat.openai.com/) - Last Resort

###### Information Emumeration
* [csp evaluator](https://csp-evaluator.withgoogle.com/) - evalute CSP
* [FontDrop!](https://fontdrop.info/) - Web Font Inspector

###### Helpful Tools
* [interactsh](https://app.interactsh.com/) - OOB tool
* [webhook.site](https://webhook.site/) - Another useful OOB tool
* [CyperChef](https://gchq.github.io/CyberChef/) - en/decryption & en/decoding combination tool
* [dCode](https://dcode.fr/) - decoder and decrypter
* [JWT.io](https://jwt.io/) - JWT manipulation
* [ExploitDB](https://www.exploit-db.com/) - Exploits
* [GTFOBins](https://gtfobins.github.io/) - GTFO Bins
* [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) - Web Payloads
* [PyLingual] (https://pylingual.io/) - great pyc decompiler

### Tools

###### Information Enumeration
* [dirsearch](https://github.com/maurosoria/dirsearch) - Directory Bruteforcing 
`python3 dirsearch.py -e html -u https://target -w /path/to/wordlist`
* [GitTools](https://github.com/internetwache/GitTools) - .git folder enumeration 
`./gitdumper.sh https://target/.git . && ./extractor.sh . .`
* [proxy.py](https://github.com/abhinavsingh/proxy.py) - MITM Proxy `proxy --log-level d`
* [mitmproxy](https://mitmproxy.org/) - MITM Proxy `mitmproxy`
* [Burp Suite](https://portswigger.net/burp) - 👍Web Security Tool

###### Hash / Session Cracker
* [HashCat](https://github.com/hashcat/hashcat) - Password/Hash Cracker 
`hashcat -m 0 -a 0 hash.txt wordlist.txt`
* [jwt-cracker](https://github.com/lmammino/jwt-cracker) - JWT Cracker 
`jwtcrack -t <token> -w <wordlist>`
* [Flask-Unsign](https://github.com/Paradoxis/Flask-Unsign) - Flask Session Cracker 
`flask-unsign -u -c <cookie> --wordlist <wordlist>`

###### SQL Injection
* [SQLMap](https://github.com/sqlmapproject/sqlmap) - Best SQL Injection Tool!
* [NoSQLMap](https://github.com/codingo/NoSQLMap) - NoSQL Injection Tool

