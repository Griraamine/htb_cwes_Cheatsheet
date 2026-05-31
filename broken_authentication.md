## Default creds

Many web applications are set up with default credentials to allow access after installation. However, these credentials need to be changed after the initial setup of the web application. so its worth trying to login using the default creds (even TT setted the JWT secret to 'secret')

## Brute forcing

when I encounter a login form it can have brute forcing attack, not only sql 

#### username brute forcing:

to brute force username, I can see the different responses in the login/register page, the goal is to enumerate existing users by brute forcing usernames and filter for example:  every response has `user already exists` -> bingo 

#### password brute forcing:

after enumerating users, its time to brute force passwords, but first and the only step before brute forcing, filtering the wordlist if needed to match the format of the passwords, for example this command 

```shell
awk 'length($0) >= 10 && /[a-z]/ && /[A-Z]/ && /[0-9]/' /wordlist /new_wordlist
```

#### brute forcing Reset Tokens:

when there exists a reset password functionality, the reset token can be bruteforced 

> REMARK: in the challenge  he directed me to the link `http://website/reset_password.php` with get parameter, so the token is not provided in the request thats why I added the parameter `?token=0000` its the common or the intended way to provide the token 

### Weak Brute-Force Protection

brute forcing can have sort of protections, for example: 

#### Rate limits

Many rate limit implementations rely on the IP address to identify the attacker which can be bypassed using `X-Forwarded-For` header (randomnizing IPs)



### Basic HTTP Authentication

this header: `Authorization: Basic YWxpY2U6c2VjcmV0MTIz` specificallyt the `Basic` keyword marks the usage of basic authentication. the encoded string is simply **base64(username:password)**  or simply: 

![](file:///home/kira/Downloads/image(15).png) 

this popup indicates the usage of the HTTP auth instead of custom login form  

 

this is a vulnerable php code, it checks if the session is active `if(!$_SESSION['active']) {header("Location: index.php");}`  if not it will redirect to `index.php` again, but before it will render the admin.php bexause there is no `exit;`  PHP continues executing the rest of `admin.php`

![](/home/kira/.config/marktext/images/2026-05-25-18-33-44-image.png)  

this endpoint will render whole /admin.php code, the browser will follow the 302 redirection to index.php but  that means when I visit its response using Burpsuite I will find the admin page 

## Attacking Session Tokens(cookies):

#### Brute Force attack:

a session token can be brute forced if it does not provide sufficient randomness and is cryptographically weak. to see that we try to send a request multiple times and compare the sessions :

2c0c58b27c71a2ec5bf2b4**b6e8**92b9f9
2c0c58b27c71a2ec5bf2b4**5460**92b9f9
2c0c58b27c71a2ec5bf2b4**97f5**92b9f9
2c0c58b27c71a2ec5bf2b4**8bcf**92b9f9

out of 32 characters , 28 are the same and 4 are random, this is easy to brute force 

#### Attacking Predictable Session Tokens:

sometime bruteforcing is useless, because sessions token provides sufficient randomness. In fact its not random but encoded, so it can be predicted **(for example base64 encoded)**, so the idea is to decode it and modify it and reencode it 

```shell
echo -n 'user=username;role=admin' | base64 
```

(the -n flag prevents adding new line \n to the output)

#### Session Fixation:

sesssion fixation is a vulnerability that arises when a web application does not update a cookie after a successful authentication, but it uses the existing one. 

**so the normal case**: 

- a session cookie may exist before authentication 

- after successful login, the server should:
  
  1. deletes / destroys old session ID
  
  2. generates a new session ID 
  
  3. bind the new session to the authenticated user 

**vulnerable version**: 

- after successful login:  the web application does not destroy the old session ID and uses it instead
  
  -> the attack idea is deliver a session ID to a victim, for example: 
  
  ```url
  https://example.com/login?PHPSESSID=abc123
  ```
  
  so now the victim's browser is using `PHPSESSID=abc123` and its linked to that user 
  
  so if attacker reuses the same token he will log to the admin account













## Tools



#### Hydra

tool for brute forcing, basic syntax: 

```shell
hydra -l [suername] -p [password] -t [number_of_parallel_threads] [service_options]
```

to change usrname/password to 'file_of_users'/'file_or_passwords' upper the letter
