![](0.profile_pic.png?raw=true)

Hey, this is actually my 1st HTB box write-up as well as box. It is actually an _easy_ marked box. I thought that of getting used to writing reports. As I once heared by one of my interviewer, "Testers are not payed for hacking. They have to hack stuff nearly for free, they get payed only for their report writing skills." For this reason, I came up with this easy box, so that we can move our focus more on ***report writing*** rather than tersting.

So, what are you waiting for?

As always, I started testing by scanning the ports with nmap and found out this.

#### 1. nmap:
--------
Port: 22/tcp - ssh and 80/tcp - http is open

![](Scan1.png?raw=true)

![](Scan2.png?raw=true)

![](Scan3.png?raw=true)

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

Now sending the request to inspector, to check out the response

![](4.sending_request_to_inspector.png?raw=true)

Now, we can see: ***X-Powered-By: PHP/8.1.0-dev*** in reponse.

I googled it and came up with this blog post: [PHP verion 8.1.0-dev](https://flast101.github.io/php-8.1.0-dev-backdoor-rce/)

![](5.PHP_verion_8.1.0-dev.png?raw=true)

So, we can probably set the **User-Agentt** header like this:

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

Link to my report: [Report](https://docs.google.com/document/d/1aHbwQkNBxTsbXFTdsa9iHAaOxjY7efREyHJbgDGJmkE/edit?usp=sharing)

This is it for now, if you have any queries, suggestions (tbh, I'm open to all type of comments), please contact me here:
- [linkedin](https://www.linkedin.com/in/soumyanil-biswas/)
- [twitter](https://twitter.com/reveng007)

Meet, all of you in the next blog, until then, Bye 👋..!!!

