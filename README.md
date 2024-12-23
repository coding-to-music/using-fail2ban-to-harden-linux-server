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

### How to whitelist IP in Fail2Ban

https://linuxhandbook.com/fail2ban-basic/

It won’t be a good thing if you ban yourself, right? To ignore an IP address from being banned by the current session of Fail2Ban, you can whitelist the IP using a command like this:

```java
fail2ban-client set <JAIL_NAME> addignoreip <IP_Address>
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

These IP addresses/networks are ignored:

```java
`- 203.93.83.113
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
fail2ban-client get <JAIL_NAME> ignoreip
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
fail2ban-client set <JAIL_NAME> delignoreip <IP_Address>
```

If you want to permanently remove the IP, you should edit the /etc/fail2ban/jail.local file.
