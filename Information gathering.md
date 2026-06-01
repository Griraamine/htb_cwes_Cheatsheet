# Information gathering

This is the first thing to do against a given IP address, 

### Subdomain & VHost enumeration

**see [fuzzing cheatsheet](./Fuzzing.md) for bruteforcing enumeration**

### DNS Zone Transfers (AXFR)

> CONCEPTS FIRST: 
> 
> - primary and seconday DNS servers: servers that hold the same addressing (dns to ip) , its like a backup(if one dies the other can resolves the dns and the domain remains accessible) and load balanciing(many users query same domain) 
> 
> - only the primary is edited, the secondary takes a copy (zone transfer)
> 
> - DNS Zone transfer vuln: is when the DNS server **allows unauthorized AXFR (zone transfer) requests** 

Zone transfer vuln is done to obtain all DNS records (especially hidden subdomains and their IP addresses) in a single request.

1. Find the domain  
   
   if you are provided with domain , skip this 

2. Query its NS records  
   
   simply with the command: 
   
   ```shellsession
   $ dig NS  zonetransfer.me
   
   ;; ANSWER SECTION:
   zonetransfer.me.    7200    IN    NS    nsztm2.digi.ninja.
   zonetransfer.me.    7200    IN    NS    nsztm1.digi.ninja.
   ```

3. Identify the authoritative DNS servers
   
   the intended output is **nsztm2.digi.ninja.** and **nsztm1.digi.ninja.**  

4. Attempt an AXFR (zone transfer)  
   
   syntax: `dig  axfr @dns_server  domain_name`
   
   ```shell-session
   $ dig axfr @nsztm2.digi.ninja. zonetransfer.me
   ```

5. If allowed, receive the entire zone file  

6. Extract subdomains, IPs, mail servers, etc.

## Tools

#### WHOIS

it is like a giant phonebook for the internet, letting you look up who owns or is responsible for various online assets( this may contain some secrets in a ctf/challenge)

Syntax: 

```shellsession
$ whois example.com <OPTION> 
```

the options can be found in HTB module

## Remarks:

in the skill exercise of **DNS Zone Transfers**:

> he gave me an IP addr and told me to perform  zone transfer for a domain example.com on the targeted machine 
> 
> **Q:  since I have a domain why he gave me an IP addr I will do the 5 steps above and no IP was needed**
> 
> **R:here he means by 'on the targeted machine' that the targeted machine is the DNS server itself, so no need to search for server name,  the command of the solution is `dig AXFR @TARGETED_SYSTEM domain`**
