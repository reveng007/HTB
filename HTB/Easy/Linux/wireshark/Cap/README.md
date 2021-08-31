1. Port scanning:

![](Scan1.png?raw=true)

![](Scan2.png?raw=true)

![](Scan3.png?raw=true)

![](Scan4.png?raw=true)

![](Scan5.png?raw=true)

![](Scan6.png?raw=true)

2. website

At first I got this webpage:
![](web1.png?raw=true)

But after downloading it, `4.pcap` file, I couldn't find anything...

Then after cahnaging manually in decreasing manner, when I chnaged to **0**:
![](web2.png?raw=true)

It contained a `0.pcap` file which contained username and password

3. wireshark view

![](wireshark1.png?raw=true)

4. ssh login

Used ***linpeas.sh*** to get _priv_esc_ vector:

![](linpeas.sh?raw=true)

python3.8 has _CAP_SETUID_ capabilty

![](priv_esc1.png?raw=true)

Then went to [GTFOBin](https://gtfobins.github.io/gtfobins/python/)

![](root_shell.png?raw=true)

see:[capability vs SUID - hackingarticles](https://www.hackingarticles.in/linux-privilege-escalation-using-capabilities/)


