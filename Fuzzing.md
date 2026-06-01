# Web Fuzzing

So its not that complicated, first thing fuzz 

#### Directories

ez 

#### Files

ez 

- if nothing found add extensions 

#### Parameters & Values

after identifying the endpoints, its useful to fuzz for parameters (both in GET and POST requests)

or there can be hint like : 

```shellsession
$ curl http://IP:PORT/get.php

Invalid parameter value
x:
```

- the response tells us that the parameter x is missing, so here fuzzing the value of the parameter x if enough
  
  but sometimes a parameter can exist and require a specific value without displaying any error. For example `GET /admin.php?debug=1` and `GET /admin.php?debug=0` may look identical because the normal response is `GET /admin.php` but `GET /admin.php?debug=1` may reveal extra functionality

- If no parameter names are disclosed, parameter fuzzing can be used to discover hidden parameters. A parameter can exist and require a specific value without displaying any error

#### Vhosts & Subdomains

> Concepts: one webserver can host multiple web apps. so its divided to Vhosts, (like VMs inside the server) , subdomains are a way to distinguish between Vhosts for example app.example.com -> application VHost and blog.example.com -> blog VHost



The server knows which domain is requested in the URL and but it knows which host(or Vhost) is requested in the `Host` Header





**VHost fuzzing** 

- **using gobuster** : (after linking the given ip to the domain in /etc/hosts)

```shellsession
$ gobuster vhost -u http://domain:port -w wordlist --append-domain
```

 `vhost` is to activate the vhost fuzzing mode instead of focusing on files and directories. 

`--append-domain`: This crucial flag instructs `Gobuster` to append the base domain (`domain`) to each word in the wordlist. This ensures that the `Host` header in each request includes a complete domain name (e.g., `admin.domain.com`), which is essential for vhost discovery.

- **using ffuf** : fuzz the hsotname itself 
  
  ```shellsession
  $ ffuf -u http://FUZZ.domain/ -w wordlist.txt
  
  
  admin.example.com
  dev.example.com
  test.example.com
  ```
  
   

**Subdomain fuzzing** 

- **using gobuster** : 

```shellsession
$ gobuster dns --domain example.com -w /usr/share/wordlists/SecLists/Discovery/DNS//subdomains-top1million-5000.txt
```

`dns` activates gobuster's DNS fuzzing mode, directing it to focus on discovering subdomains.

add `--timeout 5s` if it gives error **[ERROR] lookup alpha.inlanefreight.com.: i/o timeout**  because the DNS server sometimes takes longer than default (1s) to respond, i fit persists, reduce number of threads ( default 10 ) with the flag `-t 5`

- **using ffuf** : usually keep the URL fixed and fuzz the `Host` header
  
  ```shellsession
  ffuf -u http://domain/ -H "Host: FUZZ.domain" -w wordlist.txt
  ```
  
  



#### API fuzzing

yk how (almost the same as directories and files)

### Tools

#### ffuf

basic syntax: 

```shellsession
$ ffuf -u <url> -w <wordlist> 
```

#### Gobuster
