# Write-up for HTB Expressway

### Initial recon

```
> sudo nmap -sU <IP>
OpenSSH 10.0p2 Debian 8 (protocol 2.0)

PORT      STATE         SERVICE
68/udp    open|filtered dhcpc
69/udp    open|filtered tftp
161/udp   open|filtered snmp
500/udp   open          isakmp <-----------------------
1101/udp  open|filtered pt2-discover
4500/udp  open|filtered nat-t-ike
5003/udp  open|filtered filemaker
9200/udp  open|filtered wap-wsp
```

Leads to IPSEC/IKE vpn which has some misconfigurations.
https://book.hacktricks.wiki/en/network-services-pentesting/ipsec-ike-vpn-pentesting.html

Download ike-scan to get some information about it
```
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.253.211  Main Mode Handshake returned
        HDR=(CKY-R=53a6713a18bdcf26)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
        VID=09002689dfd6b712 (XAUTH)
        VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)
```

IKE Backoff Patterns:

```
IP Address      No.     Recv time               Delta Time
10.129.253.211  1       1758692517.703869       0.000000
10.129.253.211  Implementation guess: Linksys Etherfast

ID(Type=ID_USER_FQDN, Value=ike@expressway.htb)
```
Use hashcat to crack the PSK: `hashcat ./hash.txt rockyou.txt`
PSK: <!-- freakingrockstarontheroad -->

### Creds
User: ike
Pass: <!-- freakingrockstarontheroad -->

(not useful. IDK what this is lol but I found it in the running processes)
bash -c $'(( ( printf "\xCF\xC9\x01\x00\x00\x01\x00\x00\x00\x00\x00\x00\x0Aduckduckgo\x03com\x00\x00\x01\x00\x01" >&3; dd bs=9000 count=1 <&3 2>/dev/null | xxd ) 3>/dev/udp/1.1.1.1/53 && echo "DNS accessible") | grep "accessible" && exit 0 ) 2>/dev/null || echo "DNS is not accessible"'

### Sudo
got a nudge to look at sudo, so I checked the version and there were multiple CVEs for it. I normally don't check sudo versions for vulns cause
linpeas always says it's "potentially vulnerable". This time it actually was. This CVE worked to get root:
https://github.com/pr0v3rbs/CVE-2025-32463_chwoot/tree/main

EZ root.



