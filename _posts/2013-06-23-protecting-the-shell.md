---
layout: post
title: "Protecting The Shell"
description: ""
category: network security
tags: [shell, network security]
---
{% include JB/setup %}
Secure Shell ( ssh ) is a tool that is used on a regular basis for a nix system. However, this protocol while being a very powerful tool also brings security risks to a network. If an attacker is able to brute-force, crack, etc a password to a system this could potentially be very bad to a network if it goes unnoticed. On a large network this risk increases and could allow more systems to be taken by an attacker. I work on a Class B network and some of the users to these systems do not have an understanding of the security issues involved. Sure you could try to inform all the users but that isn't as feasible when you allow guests to a network and users are constantly coming/going.

So let's explore some of the security options a network like this has and some possible implementations to secure your own nix system. One of the easiest things to do is to go into your ssh daemon and change the default port that ssh uses. Instead of port 22 you could use something like 22222. Just like that you have adverted an attacker scanning across the net for port 22 and are now less likely to have brute-force attacks on your system.
<br />
Now while this is easy to implement in your sshd_config that does not mean you are secure. If an attack isn't targeted then the attacker will more then likely miss that port as it is not one of the privileged ports. A targeted attack would be more likely to check some of the dynamic ports and what services are listed. This can be done with a tool like nmap ( a security scanner ) which stands for network mapper.<br />
<div></div>
**Example Scans**
- p denotes port(s) to look at
- PN denotes no ping
- sV denotes service scan
<br /><br />

1. nmap -PN -p 22,222,2222,22222 &lt;host IP(s)&gt;
2. nmap -PN -p 1000-6000 &lt;host IP(s)&gt;
3. nmap -PN -p 1000-65535 &lt;host IP(s)&gt;
4. namp -PN -p 1000-65535 -sV --script=banner &lt;host IP(s)&gt;
<br /><br />

Now these are just very basic scans and there are a ton of options to add on but lets look at the first example. This scan is looking for variations on a host for port 22. Like I said this option is the easy way and sometimes the lazy way. The second example shows how an attacker can scan some of the dynamic ports and the third example shows how an attacker can scan all the non standard ports for anything that may be open for an attack. Now scanning like this isn't as common because if you are doing this for a large network it would take a fair amount of time and might become noticeable. The fourth example shows a variation of the third but implements a service scan which would allow an attacker to grab the banner and list the service listening on that port.
<div></div>
**Example Ouput**
{% highlight bash  %}
$ nmap -PN -p 2222 192.168.0.168 -sV --script=banner
Starting Nmap 6.00 ( http://nmap.org ) at 2013-03-26 21:48
Nmap scan report for bleh (192.168.0.168)
Host is up (0.000065s latency).
PORT		STATE SERVICE VERSION
2222  tcp	open  ssh     OpenSSH 
{% endhighlight %}

As you can see this means that although we may have deterred one attack we are still susceptible to another. So what else can we do to protect the shell against attackers? Well a user could add tools such as <span style="color: blue;"><a href="http://www.fail2ban.org/wiki/index.php/Main_Page" target="_blank"><span style="color: blue;">fail2ban</span></a></span>and <span style="color: blue;"><a href="http://www.cyberciti.biz/faq/block-ssh-attacks-with-denyhosts/" target="_blank"><span style="color: blue;">DenyHosts</span></a></span>. These tools allow a system to block an IP if too many attempts to login occur. While this adds security this once again does not guarantee against better attacks.<br />
<br />
Something I was going to do and may be an option for you is to write a script to check your logs every once in awhile. In the sshd_config you can enable <a href="https://help.ubuntu.com/community/SSH/OpenSSH/Configuring" target="_blank"><span style="color: blue;">logs</span></a> that keep track of who attempts what for ssh on your system. Reading these logs is somewhat painful so install logwatch or some other log reader/formatter. With these two you could write a script to parse the logs and give you updates on possible things to look out for.<br />
<br />
Debian based systems 
{% highlight bash  %}
$ apt-get install logwatch
{% endhighlight %}
RHEL based systems:
{% highlight bash  %}
$ yum install logwatch
{% endhighlight %}

<br />
Now the final option I would like to talk about is two factor authentication. I recently started using this system which allows me to login with a password or an RSA key. After that authentication is successful, a command is called to a program which gives me options to verify this connection via my phone. Now this adds a cool factor because I can see the IP and where it is coming from with the options to allow or deny the connection ( using the mobile app ). You could also receive a phone call or SMS based on your preferred method of delivery ( costs a telephony credit ).<br />
<div></div>
**Example Output**
{% highlight bash  %}
$ ssh bleh.com
Duo two-factor login for <username>
Enter a passcode or select one of the following options:
1. Duo Push to XXX-XXX-****
2. Phone call to XXX-XXX-****
3. SMS passcodes to XXX-XXX-****
Passcode or option (1-3): 1
Pushed a login request to your phone...
Success. Logging you in...
{% endhighlight %}
&lt;username&gt; - username you establish with the company<br />
\*\*\*\* - are the last four digits of your cellphone<br />
<br />
This can be all yours for free under the personal use option from <span style="color: blue;"><a href="https://www.duosecurity.com/" target="_blank"><span style="color: blue;">duosecurity</span></a>. </span>They also have options for different sized companies. This two factor security is also not just limited to ssh but other applications as well. The website also provides <a href="https://www.duosecurity.com/" target="_blank"><span style="color: blue;">documentation</span></a> to setup everything after an account has been created.<br />
<br />
I think I have ranted long enough about ssh so I think I will leave this discussion here for now. Please feel free to comment/ask questions on anything.<br />
<br />
