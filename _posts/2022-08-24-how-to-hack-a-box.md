# How to Hack a Box


## Information Gathering
- **Online Tools**
    - [Security Trails](https://securitytrails.com/)
    - [Shodan](https://www.shodan.io/)


- **Custom Word List generator (CeWL)** : CeWL spiders a given URL, up to a specified depth, and returns a list of words which can then be used for password crackers. 

    `cewl -m 8 -e --email_file ./emails.txt -d 0 -w ./diccio https://twitter.com/hackbysecurity?lang=es`


- **Common User Passwords Profiler (CUPP)**: generate commmon passwords dictionaries based on personal information from the target. You can also import and mutate existing dictionaries. Download from source: [cupp](https://github.com/Mebus/cupp.git)
    
    `python3 cupp.py -i`


- **The Harvester**: using different search engines. Must first configure API Keys:

    `nano /etc/theHarvester/api-keys.yaml`

    `theHarvester -d hackbysecurity.com -s -b google -n -f ./report`

    `--dns-lookup`: check to which IP address corresponds the domain name and will look at the range of IP addresses if there are subdomains.
Direct and inverse domain resolutions


- **Dmitry**: perform whois, subdomaind and mails search.

    `dmitry -w hackbysecurity.com -n -s -e`

## Port and Vulnerability Scanning
### Network Discovery
- **Wireshark**: packet sniffer to visualize traffic in our network interface. Passive method
    - eth0: local wired network
        - ARP packages
        - DHCPv6 packages: with TCP/IP v6
- **netdiscover**: send ARP packages actively or passively
    
    `netdiscover -i eth0 -r 10.0.2/24`

- Check ARP tables from my machine `arp -a`. 
- ARP requests: only works at local network level
- ICMP requests: for exernal petitions, sent 1 package to a sequential network range 
    
    `for i in $(seq 3 254); do ping -c 1 10.0.2.$i; done`
- DNS requests: 
    1. Configure DNS servers include our target dns and ip address 
        
        `nano /etc/resolv.conf`
    2. **wfuzz**: scan, choose a dictionary to use, ignore empty responses  
        `wfuzz -Z -c -w /usr/share/wordlists/wfuzz/general/common.txt --hh=0 FUZZ.hackbysecurity.com`
        - dictionary [secList](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS): common subdomains we can use
- NetBIOS requests
    
    `nbtscan -r 10.0.2.0/24`

### Port Scanning
- **TCP CONN**: a complete connection. 
    - Send request to a closed port (ie 20). Response is connection refused [RST, ACK].

        `nc 10.0.2.4 20`

    - Send request to an open port. Response is [SYN, ACK]

        `nc 10.0.2.4 21`

- **TCP SYN**: similar to TCP CONN but once get's a reply, it aborts the connection
    - Send request, response is [SYN, ACK]

        `hping3 10.0.2.4 --scan 20 -S`

- **TCP Null**: use only in Linux/Unix. Send an empty request. If there is no reply, port is open or filtered by a firewall. If returns an error, port is closed

    `hping3 10.0.2.4 --scan 20 -Y`

- **TCP Fin**: use only in Linux/Unix.  Send a request with End connection value. If no answer, port is open. If error, port is closed or filtered

    `hping3 10.0.2.4 --scan 20 -F`

- **TCP XMAS**: use only in Linux/Unix. Similar to Fin, we receive a RST package if port is closed. We send FIN, URG and PUSH

    `hping3 10.0.2.4 --scan 20 -UPF`

- **TCP ACK**: looks for ports that are filtered by a firewall. When we send an ACK, if we receive reply it resets connection (RST). If there is no reply, port is filtered by a firewall.

    - `hping3 10.0.2.4 --scan 20 -A`

- **UDP**: you may receive corrupted data (i.e. poor audio quality). Much faster than TCP. UDP header size is 8bytes (TCP is 16bytes). Scan takes longer than TCP

   `hping3 10.0.2.4 --scan 20 -2` 
