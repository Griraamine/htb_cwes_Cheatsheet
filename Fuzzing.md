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

the response tells us that the parameter x is missing 

#### Vhosts & Subdomains

**VHost fuzzing** using gobuster: (after linking the given ip to the domain in /etc/hosts)

```shellsession
$ gobuster vhost -u http://domain:port -w wordlist --append-domain
```

 `vhost` is to activate the vhost fuzzing mode instead of focusing on files and directories. 

`--append-domain`: This crucial flag instructs `Gobuster` to append the base domain (`domain`) to each word in the wordlist. This ensures that the `Host` header in each request includes a complete domain name (e.g., `admin.domain.com`), which is essential for vhost discovery.

**Subdomain fuzzing** using gobuster: 

```shellsession
$ gobuster dns -d example.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

`dns` activates gobuster's DNS fuzzing mode, directing it to focus on discovering subdomains.

#### API fuzzing

yk how (almost the same as directories and files)

### Tools

#### ffuf

basic syntax: 

```shellsession
$ ffuf -u <url> -w <wordlist> 
```

#### Gobuster
