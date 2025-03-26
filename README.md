# using-fail2ban-to-harden-linux-server

# 🚀 use fail2ban to ban and put into jail the ip address of bots and repeated failures to log into the linux server 🚀

https://github.com/coding-to-music/using-fail2ban-to-harden-linux-server

## GitHub

```java
git init
git add .
git remote remove origin
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:coding-to-music/using-fail2ban-to-harden-linux-server.git
git push -u origin main
```

## Environment variables:

```java

```

# What is Fail2Ban?

If you have enabled SSH, please check the login history of your Linux server or view the SSH logs. You’ll be surprised to see a huge number of IPs that try to log in to your server via SSH.

https://linuxhandbook.com/linux-login-history/

If you have no mechanism in place to deter these login attempts, your system is susceptible to bruteforce attack. Basically, a script/bot will keep on attempting SSH connection your system by trying various combination of username and passwords.

This is where a tool like Fail2Ban comes into picture. Fail2Ban is a free and open source software that helps in securing your Linux server against malicious logins. Fail2Ban will ban the IP (for a certain time) if there is a certain number of failed login attempts.

Fail2Ban works out of the box with the basic settings but it is extremely configurable as well. You can tweak it to your liking and create filters and rules as per your need.

Sounds interesting? Why not test Fail2Ban? Read and follow the rest of the article and try Fail2Ban yourself.

# Viewing Linux login history

https://linuxhandbook.com/linux-login-history/

Linux is very good at keeping logs of everything that goes on your system. Quite naturally, it also stores logs about login and login attempts. The login information is stored in three files, wtmp, utmp and btmp:

- `/var/log/wtmp` – Logs of last login sessions
- `/var/run/utmp` – Logs of the current login sessions
- `/var/log/btmp` – Logs of the bad login attempts

Let’s see these things in a bit detail.

### View history of all logged users

To view the history of all the successful login on your system, simply use the command last.

```java
last
```

The output should look like this. As you can see, it lists the user, the IP address from where the user accessed the system, date and time frame of the login. pts/0 means the server was accessed via SSH.

```java
abhi pts/0 202.91.87.115 Wed Mar 13 13:31 still logged in
root pts/0 202.91.87.115 Wed Mar 13 13:30 - 13:31 (00:00)
servesha pts/0 125.20.97.117 Tue Mar 12 12:07 - 14:25 (02:17)
servesha pts/0 209.20.189.152 Tue Mar 5 12:32 - 12:38 (00:06)
root pts/0 202.91.87.114 Mon Mar 4 13:35 - 13:47 (00:11)

wtmp begins Mon Mar 4 13:35:54 2019
```

The last line of the output tells you the when was the wtmp log file was created. This is important because if the wtmp file was deleted recently, last command won’t be able to show history of the logins prior to that date.

You may have a huge history of login sessions so it’s better to pipe the output through less command.

### View login history of a certain user

If you just want to see the login history of a particular user, you can specify the user name with last command.

```java
last <username>
```

You’ll see the login information of only the selected user:

```java
last servesha
servesha pts/0 125.20.97.117 Tue Mar 12 12:07 - 14:25 (02:17)
servesha pts/0 209.20.189.152 Tue Mar 5 12:32 - 12:38 (00:06)

wtmp begins Mon Mar 4 13:35:54 2019
```

### Display IP addresses in login history instead of hostname

You couldn’t see it in the previous output but by default, last command shows the hostname instead of the IP address of the user. If you are on a sub-network, you’ll probably see only the hostnames.

You can force to display the IP addresses of the previously logged users with the -i option.

```java
last -i
```

### Display only last N logins

If your system has a good uptime, perhaps your login history would be huge. As I mentioned earlier, you can use the less command or other file viewing commands like head or tail.

Last command gives you the option to display only certain number of login history.

```java
last -N

example:

last -100
```

Just replace N with the number you want. You can also combine it with the username.

### View all the bad login attempts on your Linux server

Now comes the important part: checking the bad login attempts on your server.

You can do that in two ways. You can either use the last command with the btmp log file:

```java
last -f /var/log/btmp
```

or you can use the lastb command:

```java
lastb
```

Both of these commands will yield the same result. The lastb is actually a link to the last command with the specified file.

```java
root     ssh:notty    218.92.0.158     Wed Mar 13 14:34 - 14:34  (00:00)
sindesi  ssh:notty    59.164.69.10     Wed Mar 13 14:34 - 14:34  (00:00)
root     ssh:notty    218.92.0.158     Wed Mar 13 14:34 - 14:34  (00:00)
sindesi  ssh:notty    59.164.69.10     Wed Mar 13 14:34 - 14:34  (00:00)
root     ssh:notty    218.92.0.158     Wed Mar 13 14:34 - 14:34  (00:00)
```

Bad logins could be an incorrect password entered by a legitimate user. It could also be a bot trying to brute force your password.

You have to analyze here and see if you recognize the IPs in the log. If there has been too many login attempts from a certain IP with user root, probably someone is trying to attack your system by bruteforcing.

You should deploy Fail2Ban to protect your server in such cases. Fail2Ban will ban such IPs from your server and thus giving your server an extra layer of protection.

## Installing Fail2Ban on Linux

You can guess the popularity of Fail2Ban from the fact that it is available in the official repositories of all the major Linux distributions. This makes installing Fail2Ban a simple task.

https://linuxhandbook.com/fail2ban-basic/

## Install Fail2Ban on Ubuntu & Debian

First, make sure your system is updated:

```java
sudo apt update && sudo apt upgrade -y
```

Now, install Fail2Ban with this command:

```java
sudo apt install fail2ban
```

## Understanding Fail2Ban configuration file

There are two main configuration files in Fail2Ban: /etc/fail2ban/fail2ban.conf and /etc/fail2ban/jail.conf. Let me explain what they do.

`/etc/fail2ban/fail2ban.conf`: This is the configuration file for the operational settings of the Fail2Ban daemon. Settings like loglevel, log file, socket and pid file is defined here.

`/etc/fail2ban/jail.conf`: This is where all the magic happens. This is the file where you can configure things like default ban time, number of reties before banning an IP, whitelisting IPs, mail sending information etc. Basically you control the behavior of Fail2Ban from this file.

Now, before you go and change these files, Fail2Ban advise making a copy with .local file for these conf files. It’s because the default conf files can be overwritten in updates and you’ll lose all your settings.

```java
sudo cp /etc/fail2ban/fail2ban.conf /etc/fail2ban/fail2ban.local

sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Now let’s understand the jail.conf file. If you use the less command to read this big file, it may seem quite confusing. The conf file tries to explain everything with way too many comments. So, let me simplify this for you.

The `jail.conf` file is divided into services. There is a `[Default]` section and it applies to all services. And then you can see various services with their respective settings (if any). All these services are in brackets. You’ll see sections like `[sshd]`, `[apache-auth]`, `[squid]` etc.

If I remove the comments, the default section looks like this:

```java
[DEFAULT]
ignorecommand =
bantime = 10m
findtime = 10m
maxretry = 5
backend = auto
usedns = warn
logencoding = auto
enabled = false
mode = normal
filter = %(name)s[mode=%(mode)s]
destemail = root@localhost
sender = root@
mta = sendmail
protocol = tcp
chain =
port = 0:65535
fail2ban*agent = Fail2Ban/%(fail2ban_version)s
banaction = iptables-multiport
banaction_allports = iptables-allports
action_abuseipdb = abuseipdb
action = %(action*)s
```

Let me tell you the meaning of some of these parameters.

- `bantime`: Set the length of the ban. Default is 10 minutes.
- `findtime`: The window in which the action on an IP will be taken. Default is 10 minutes. Suppose a bad login was attempted by a certain IP at 10:30. If the same IP reaches the maximum number of retries before 10:40, it will be banned. Otherwise, the next failed attempt after 10:40 will be counted as first failed attempt.
- `maxretry`: The number of failed retries before an action is taken
- `usedns`: The “warn” setting attempts to use reverse-DNS to look up the hostname and ban it using hostname. Setting it to no will ban IPs, not hostname.
- `destemail`: The email address to which the alerts will be sent (needs to be configured)
- `sender`: The sender name in the notification email
- `mta`: Mail Transfer Agent used for notification email
- `banaction`: This parameter uses the /etc/fail2ban/action.d/iptables-multiport.conf file to set the action after maximum failed retries
- `protocol`: The type of traffic that will be dropped after the ban

If you want to make any changes for any jail (or for all the jail), like the maximum retries, ban time, find time etc., you should edit the `jail.local` file.

## edit the defaults to be more strict

```java
sudo vi /etc/fail2ban/jail.local
```

about line 90 you will see these entries, set them to these more strict values

```java
# "bantime" is the number of seconds that a host is banned.
bantime  = 48h

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
findtime  = 10m

# "maxretry" is the number of failures before a host get banned.
maxretry = 2
```

restart the fail2ban service and verify it is operating correctly

```java
sudo systemctl restart fail2ban

sudo systemctl status fail2ban.service

sudo fail2ban-client status

sudo fail2ban-client status sshd

sudo tail -f /var/log/auth.log

sudo tail -f /var/log/fail2ban.log
```

## How to use Fail2Ban to secure Linux server

Let me show you some of the ways you can use Fail2Ban to harden Linux security.

Note that you need to be root user or have sudo access to run the fail2ban commands.

## Enable Fail2Ban on your server and check all running jails

You can use systemd commands to start and enable Fail2Ban on your Linux server:

```java
sudo systemctl start fail2ban
sudo systemctl enable fail2ban

sudo systemctl status fail2ban.service
```

Once Fail2Ban is enabled, you can see the status and the active jails with fail2ban-client command:

```java
sudo fail2ban-client status
```

```java
Status
|- Number of jail: 1
`- Jail list: sshd
```

In case you were wondering, sshd jail is enabled by default.

## See Fail2Ban log

Fail2Ban log is located at `/var/log/fail2ban.log`. The log files are in the following format:

```java
2019-03-25 07:09:08,004 fail2ban.filter [25630]: INFO [sshd] Found 139.59.69.76 – 2019-03-25 07:09:07
2019-03-25 07:09:36,756 fail2ban.filter [25630]: INFO [sshd] Found 159.89.205.213 – 2019-03-25 07:09:36
2019-03-25 07:09:36,757 fail2ban.filter [25630]: INFO [sshd] Found 159.89.205.213 – 2019-03-25 07:09:36
2019-03-25 07:09:36,774 fail2ban.actions [25630]: NOTICE [sshd] Ban 159.89.205.213
2019-03-25 07:09:36,956 fail2ban.filter [25630]: INFO [sshd] Found 182.70.253.202 – 2019-03-25 07:09:36
2019-03-25 07:09:36,957 fail2ban.filter [25630]: INFO [sshd] Found 182.70.253.202 – 2019-03-25 07:09:36
2019-03-25 07:09:36,981 fail2ban.actions [25630]: NOTICE [sshd] Ban 182.70.253.202
2019-03-25 07:09:37,247 fail2ban.filter [25630]: INFO [sshd] Found 112.64.214.90 – 2019-03-25 07:09:37
2019-03-25 07:09:37,248 fail2ban.filter [25630]: INFO [sshd] Found 112.64.214.90 – 2019-03-25 07:09:37
2019-03-25 07:09:37,589 fail2ban.actions [25630]: NOTICE [sshd] Ban 112.64.214.90
```

You can see that it identifies the IPs and bans them when they cross the threshold of maximum retry.

## See banned IPs by Fail2Ban

One way is to check the status of a certain jail. You can use the Fail2Ban client for this purpose.

```java
sudo fail2ban-client status <jail_name>
```

For example, if you have to see all the bad ssh logins banned by Fail2Ban, you can use it in the following manner. The output would show the total failed attempts and the total banned IPs.

```java
sudo fail2ban-client status sshd
```

```java
Status for the jail: sshd
|- Filter
| |- Currently failed: 14
| |- Total failed: 715
| `- File list: /var/log/auth.log
`- Actions
|- Currently banned: 7
|- Total banned: 17
`- Banned IP list: 177.47.115.67 118.130.133.110 68.183.62.73 202.65.154.110 106.12.102.114 61.184.247.3 218.92.1.150
```

The system that is try to login via SSH from the failed login should get an error like this

```java
ssh: connect to host 93.233.73.133 port 22: Connection refused
```

## How to permanently ban an IP with Fail2Ban

By now you know that the ban put on an IP by Fail2Ban is a temporary one. By default it’s for 10 minutes and the attacker can try to login again after 10 minutes.

This poses a security risk because attackers could use a script that tries logging in after an interval of 10 minutes.

So, how do you put a permanent ban using Fail2Ban? There is no clear answer for that.

Starting Fail2Ban version 0.11, the ban time will be automatically calculated and the persistent IPs will have their ban time increased exponentially.

But if you check your Fail2Ban version, you probably are running the version 0.10.

```java
sudo fail2ban-server --version
```

```java
Fail2Ban v0.10.2
Copyright (c) 2004-2008 Cyril Jaquier, 2008- Fail2Ban Contributors
Copyright of modifications held by their respective authors.
Licensed under the GNU General Public License v2 (GPL).
```

In earlier versions, you could use a negative bantime (bantime = -1) and that would have been equivalent to a permanent ban but if you try this method, you’ll probably see an error like ‘Starting fail2ban: ERROR NOK: (‘database disk image is malformed’,)’.

One not so clean workaround would be to increase the bantime to something like 1 day, 1 week, 1 month or 1 year. This could circumvent the problem until the new version is available on your system.

### How to unban IP blocked by Fail2Ban

First check if the IP is being blocked or not. Since Fail2Ban works on the iptables, you can look into the iptable to view the IPs being banned by your server:

```java
sudo iptables -n -L
```

You may have to use grep command if there are way too many IPs being banned.

If you find the specified IP address in the output, it is being banned:

So, the next step is to find which ‘jail’ is banning the said IP. You’ll have to use Grep command with the fail2ban logs here.

As you can see in the output below, the IP is being banned by sshd jail.

```java
sudo grep -E ‘Ban.*61.184.247.3’ /var/log/fail2ban.log
```

```java
2019-03-14 13:09:25,029 fail2ban.actions [25630]: NOTICE [sshd] Ban 61.184.247.3
2019-03-14 13:52:56,745 fail2ban.actions [25630]: NOTICE [sshd] Ban 61.184.247.3
```

Now that you know the name of the jail blocking the IP, you can unban the IP using the fail2ban-client:

```java
sudo fail2ban-client set <jail_name> unbanip <ip_address>
```

### How to whitelist IP in Fail2Ban

https://linuxhandbook.com/fail2ban-basic/

It won’t be a good thing if you ban yourself, right? To ignore an IP address from being banned by the current session of Fail2Ban, you can whitelist the IP using a command like this:

```java
sudo fail2ban-client set <JAIL_NAME> addignoreip <IP_Address>
```

These IP addresses/networks are ignored:

```java
`- 203.93.83.113
```

You can find your IP address in Linux easily. In my case, it was

```java
sudo fail2ban-client set sshd addignoreip 203.93.83.113
```

### find your hostname so you can whitelist it

```java
hostname -I

or

ip address
```

Mismatched IPs: It’s possible your outgoing IP isn’t what you expect. Action: From your laptop (in any shell), check your public IP:

```java
curl ifconfig.me
```

Find all whitelisted ip addresses

```java
sudo fail2ban-client get sshd ignoreip
sudo fail2ban-client get <JAIL_NAME> ignoreip
```

If you want to permanently whitelist the IP, you should edit the jail configuration file. Go to the said jail section and add the ignoreip line like this:

```java
ignoreip = 127.0.0.1/8 <IP_TO_BE_WHITELISTED>
```

If you want to whitelist an IP from all the jails on your system, it would be a better idea to edit the `/etc/fail2ban/jail.local` file and add a line under the DEFAULT section like what we saw above.

You’ll have to restart Fail2Ban to take this change into effect.

### How to see the IP whitelist by a jail

You can see all the IPs whitelisted by a jail using this command:

```java
sudo fail2ban-client get <JAIL_NAME> ignoreip
```

It should show all the IPs being ignored by Fail2Ban for that jail:

```java
sudo fail2ban-client set sshd addignoreip 203.93.83.113
```

These IP addresses/networks are ignored:

```java
|- 127.0.0.0/8
|- ::1
`- 203.93.83.113
```

### How to remove an IP from Fail2Ban whitelist

If you are removing the IP from a certain jail’s whitelist, you can use this command:

```java
sudo fail2ban-client set <JAIL_NAME> delignoreip <IP_Address>
```

If you want to permanently remove the IP, you should edit the `/etc/fail2ban/jail.local` file.
