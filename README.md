[![Typing SVG](https://readme-typing-svg.herokuapp.com?color=%2336BCF7&lines=Proxmox+8+Fail2Ban+Setup)]()

# 1. Disable paid repository
```
nano /etc/apt/sources.list.d/pve-enterprise.list
```
add '#' symbol before it
```
#deb https://enterprise.proxmox.com stretch pve-enterprise
```

## 2. Add free no-subscription repository
```
nano /etc/apt/sources.list
```
add to file:
```
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
deb http://security.debian.org/debian-security bookworm-security main contrib
```

## 3. Update
```
apt update && apt upgrade -y
```


## 4. Installation
```
apt install fail2ban
```

## 5. Add ssh protection
```
nano /etc/fail2ban/jail.d/sshd.local
```
add to file:
```
[sshd]
enabled   = true
filter    = sshd
banaction = iptables
backend   = systemd
maxretry  = 5
findtime  = 1d
bantime   = 2h

```

## 6. Add proxmox web-ui protection
```
nano /etc/fail2ban/filter.d/proxmox.conf
```
add to file:
```
[INCLUDES]
before = common.conf

[Definition]
failregex = pvedaemon\[.*authentication failure; rhost=<HOST> user=.* msg=.*

ignoreregex =
journalmatch = _SYSTEMD_UNIT=pvedaemon.service
```

## 7. Enable ssh&proxmox rules(Enabling jails)

```
nano /etc/fail2ban/jail.local
```
add:
```
[DEFAULT]
bantime = 2h
banaction = route

[sshd]
enabled = true

[proxmox]
enabled = true
port = 8006
filter = proxmox
backend   = systemd
maxretry  = 5
findtime  = 2h
bantime   = 2h

```


## 8. Edit base ban rules (manually)

```
nano /etc/fail2ban/jail.conf
```
You can set custom default `bantime` (seconds while ban is active) and `maxretry` (wrong tries count)


## 9. Activation and checking

```
systemctl restart fail2ban
```
check proxmox:
```
fail2ban-client -v status proxmox
```
check ssh:
```
fail2ban-client -v status sshd
```
