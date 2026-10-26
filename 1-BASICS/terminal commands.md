dig +short google.com

dig google.com


ip addr














bhuvan-ubuntu@bhuvan:~$ ip route
default via 10.131.33.97 dev wlo1 proto dhcp src 10.131.33.102 metric 600 
10.131.33.0/24 dev wlo1 proto kernel scope link src 10.131.33.102 metric 600 
192.168.98.0/24 dev vmnet1 proto kernel scope link src 192.168.98.1 
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown 
192.168.139.0/24 dev vmnet8 proto kernel scope link src 192.168.139.1 
bhuvan-ubuntu@bhuvan:~$ 






bhuvan-ubuntu@bhuvan:~$ resolvectl status
Global
         Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub

Link 2 (wlo1)
    Current Scopes: DNS
         Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 2402:8100:2970:f7fa::fe
       DNS Servers: 8.8.8.8 2402:8100:2970:f7fa::fe
     Default Route: yes

Link 3 (virbr0)
    Current Scopes: none
         Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
     Default Route: no

Link 4 (vmnet1)
    Current Scopes: none
         Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
     Default Route: no

Link 5 (vmnet8)
    Current Scopes: none
         Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
     Default Route: no
bhuvan-ubuntu@bhuvan:~$ 
