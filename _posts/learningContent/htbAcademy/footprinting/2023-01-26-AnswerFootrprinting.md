---
title:  "Reponse d'empreinte"
---
# DNS
For anyone else still struggling with this specific question, like others have mentioned: start by doing a dig Zone Transfer command on the main domain using the target machine’s IP as the DNS server. Then record all the subdomains you get back. Then use dig to try and Zone Transfer on those subdomains (app.inlanefreight.htb, internal.inlanefreight.htb, dev.inlanefreight.htb, etc.) and record any that you cannot get records for.

Finally, use either the bash script or the DNSEnum tool to brute force the subdomains you couldn’t get records for using a very “fierce” wordlist. Good luck and hope you learn something new like I did!

# SMTP
1. Enumerate the SMTP service and submit the banner, including its version as the answer. 

- classic nmap with -sC -sV

2. Enumerate the SMTP service even further and find the username that exists on the system. Submit it as the answer. 
- try the script smtp-enum-users.nse
- use smtp-user-enum :
smtp-user-enum.pl -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t 10.0.0.1

# IMAP/POP
Voir les messages 


A1 LOGIN username password
A1 LIST "" *
A1 SELECT INBOX
A1 FETCH 1:*
A1 FETCH 1 body[text] (voir messages)
A1 fetch 1 all (voir aussi headers donc email )
* 1 FETCH (FLAGS (\Seen) INTERNALDATE "08-Nov-2021 23:51:24 +0000" RFC822.SIZE 167 ENVELOPE ("Wed, 03 Nov 2021 16:13:27 +0200" "Flag" (("CTO" NIL "devadmin" "inlanefreight.htb")) (("CTO" NIL "devadmin" "inlanefreight.htb")) (("CTO" NIL "devadmin" "inlanefreight.htb")) (("Robin" NIL "robin" "inlanefreight.htb")) NIL NIL NIL NIL))
