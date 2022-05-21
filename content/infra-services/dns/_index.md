---
title: "DNS"
weight: 1
---


## Domain Name

DNS Hierarchy
- Domain root
- Top Level Domain (TLD)
- Subdomain
- Host or resource name

Wildcard Domains: start with *
	

## DNS Message Format

Protocol Transport
- UDP Port 53. 
- If answer exceeds 512 bytes, can initiate failover response for re-query with TCP.
- Extension mechanism for DNS (EDNS): Use OPT pseudo-RR in Additional Data section.

Queries and Replies both have the same format.
- Header
- Question
- Answer
- Authority
- Additional
	
| Header Field | Length (bytes)
| --- | ---
| Identification (match response with query)	| 2
| **Flags**	| 2
| Number of Question Resource Records (RRs)	| 2
| Number of Answers RRs	| 2
| Number of Authority RRs	| 2
| Number of Additional RRs	| 2
	
| Flags Sub-Field	| Description	| Length (bits)
| --- | --- | ---
| QR	| 0: Query<br>1: Reply | 1
| OPCODE |	0: QUERY - Standard query<br>1: IQUERY - Inverse query<br>2: STATUS - Server Status Request | 4
| AA	| Authoritative answer in response	| 1
| TC	| Message was truncated | 1
| RD	| Recursion desired (request) | 1
| RA	| Recursion available (response) | 1
| Z	| Zero, reserved for future use | 3
| RCODE	| Response code.<br>0: NOERROR - No error<br>1: FORMERR - Format error<br>2: SERVFAIL<br>3: NXDOMAIN - Nonexistant domain<br>...<br>https://en.wikipedia.org/wiki/Domain_Name_System#cite_note-32 | 4

| Resource Record Field | Description | Length (bytes)
| --- | --- | ---
| Name | Name of requested resource<br>(eg. www, ns1.example.net)| Variable
| Type | Type of RR (A, AAAA, TXT…) | 2
| Class | Class code | 2
| TTL | Seconds that RR stays valid | 4
| RDLENGTH | Length of RDATA field in bytes | 2
| RDATA | RR specific data | Variable

```bash
# Example: dig www.google.com
; <<>> DiG 9.10.6 <<>> www.google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59489
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 4, ADDITIONAL: 9

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.google.com.            IN  A

;; ANSWER SECTION: <- A record for google
www.google.com.     23  IN  A   69.171.247.71

;; AUTHORITY SECTION: <- Authoritative nameservers found
google.com.     124333  IN  NS  ns4.GOoGLE.cOM. 
google.com.     124333  IN  NS  ns1.GOoGLE.cOM.
google.com.     124333  IN  NS  ns3.GOoGLE.cOM.
google.com.     124333  IN  NS  ns2.GOoGLE.cOM.

;; ADDITIONAL SECTION: <- IPs of authoritative nameservers
ns1.google.com.     300146  IN  A   216.239.32.10
ns1.google.com.     308505  IN  AAAA    2001:4860:4802:32::a
ns2.google.com.     303470  IN  A   216.239.34.10
ns2.google.com.     124333  IN  AAAA    2001:4860:4802:34::a
ns3.google.com.     303470  IN  A   216.239.36.10
ns3.google.com.     124333  IN  AAAA    2001:4860:4802:36::a
ns4.google.com.     302836  IN  A   216.239.38.10
ns4.google.com.     302716  IN  AAAA    2001:4860:4802:38::a
```	


## DNS Query

General steps
- Query begins at client computer and is passed to resolver (DNS client service)
- When query cannot be resolved locally, DNS servers are queried.
	
### Query resolution

**Local cache**

Resolver answers from local cache. 

Mappings from previous DNS answers are added to the cache. Static mappings pre-loaded from host file on startup.

- Windows: `systemroot\System32\Drivers\Etc`
- Linux: `/etc/hosts`

**Recursion**

Resolver queries other DNS servers on behalf of the client.

Disable recursion to limit DNS server to resolve only intranet addresses. Cannot use forwarders if recursion is disabled.

Root hints used to locate authoritative root DNS servers. Root hints should be deleted on internal networks.
- Windows: `systemroot\System32\Dns\Cache.dns`

**Iteration**

Resolver replies best answer without querying other DNS servers. Client will query other DNS servers as necessary. Referrals can be in form of CNAME or NS with associated A records of NS.


### Answer type

**Authoritative:** DNS server has A record of the requested resource in its zone file.

**Non-authoritative:** DNS server obtained the answer from another DNS server.


### Record type
**CNAME: Canonical record**
- Points to another domain address
- Allows querying from different domains, but maintain only 1 set of A records

**NS: Nameserver record**
- Provides the authoritative nameserver domain address for the requested resource


## DNS Zones

DNS zone is the authoritative source of information for domain names listed in it.
	
Subdomains can be:
- Managed as part of the same zone 
- Delegated to another zone 
	
High availability:
- Zone Transfers used to replicate and synchronize zones to allow more than one DNS server to host a zone.
- Secondary Zones are read-only copies of a master zone.
- Caching DNS servers do not perform zone transfers.
- Incremental zone transfers can be used to save bandwidth. Incremental versions are tracked via serial number on SOA record.
- By default, zone transfers can only be done to authoritative DNS servers listed by the NS record for the zone.


## Reverse DNS Lookup

- Too costly to map all IPs to domain names without hierarchy
- Inverse queries are outdated. Use PTR query type instead.
- **in-addr.arpa** domain defined to perform practical reverse lookups
  - Reverse the IP to order from most specific information to general information
  - Eg. PTR request for `20.1.168.192.in-addr.arpa` to search hostname of IP `192.168.1.20`
- **ip6.arpa** used for IPv6


## Forwarders

**Forwarder** is a DNS server used to forward queries for external DNS names to DNS servers outside the network.
- Designate a specific DNS server in internal network as a forwarder
  - Prevent exposing all internal DNS servers to the internet.
  - Better use of caching
- DNS servers will send a recursive query to the forwarder. If it times out, they will then contact DNS servers specified in the root hints.
	
**Conditional forwarder** forwards queries according to specific domain names.
- Performance improvement due to not needing to contact root servers.
- Forwarded zone can be any zone.
- Forwarded DNS server does not need to be authoritative on the forwarded zone.
		
**Delegation**
- Delegation can only be done on a child of a domain that you are authoritative on.
- Delegated servers are authoritative on their delegated zones.
		

## Stub Zones

**Stub zone** is a copy of a zone, containing only RRs used to identify the authoritative DNS server for the zone.
- SOA
- NS
- Glue A records (IP address of a name server at a domain name registry)
    
**Glue Records**
- Your domain is example.com and you use the following DNS servers
  - ns1.example.com
  - ns2.example.com
- A circular dependency is created because resolving ns1.example.com requires example.com which requires ns1.example.com
- To break this circular dependency, the parent (com) domain can add the records for the authoritative name servers and their IPs. The A records are the glue records.
  - example.com NS ns1.example.com
  - example.com NS ns2.example.com
  - ns1.example.com A 1.1.1.1 (glue record)
  - ns2.example.com A 2.2.2.2 (glue record)


## Commands
```batch
:: Delete cache (Windows)
ipconfig /flushdns
```