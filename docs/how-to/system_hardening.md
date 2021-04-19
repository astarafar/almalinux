# **AlmaLinux 8 System Hardening Guide**

**This is a work in progress.** If things look unfinished, they probably are.

## **Table of Contents**
  - [**Purpose and Scope**](#purpose-and-scope)
  - [**Conventions**](#conventions)
  - [**Definitions**](#definitions)
  - [**Introduction**](#introduction)
    - [**Confidentiality**](#confidentiality)
    - [**Integrity**](#integrity)
    - [**Availability**](#availability)
    - [**System Security: What It Is**](#system-security-what-it-is)
    - [**System Security: What It Isn't**](#system-security-what-it-isnt)
    - [**Principle of Least Privilege**](#principle-of-least-privilege)
  - [**Hardening AlmaLinux 8**](#hardening-almalinux-8)
    - [**System Updates**](#system-updates)
    - [**SELinux**](#selinux)
    - [**Default Configuration**](#default-configuration)
    - [**Restrict Open Firewall Ports**](#restrict-open-firewall-ports)
    - [**Harden `sshd` Configuration**](#harden-sshd-configuration)
  - [**Additional Resources**](#additional-resources)
  - [**Glossary**](#glossary)
  - [**License**](#license)

------

## **Purpose and Scope**
This document is intended to guide a system administrator through the process of hardening the operating system and core services beyond the default measures in place after initial installation.

This document does not cover physical or administrative security concepts or controls, nor does it cover application-specific (httpd, postfix, etc.) concerns. (Yet. I plan to expand into **very** common application-specific things once the initial phase of this document is complete.)

------

## **Conventions**
Definitions, explanations, etc. will be written in plain text.

<br />

The names of commands (including full path, if specified), users, groups, services, network protocol names, or other command-related information will be displayed in-line in monospaced type. Examples: `/bin/bash`, `root`, `systemctl`, `dhcpv6-client`.

<br />

Commands to enter will be displayed on their own line(s) in monospaced type, as follows. Following typical shell conventions, commands that SHOULD be executed as an unprivileged user will be prefixed with a `$` symbol, while commands that MUST be executed as the root user will be prefixed with a `#` symbol.

```    
$ printf 'This command should be run as an unprivileged user.'
```
```
# printf 'This command should be run as the root user.'
```

<br />

Shell scripts will be displayed as a fenced code block with syntax highlighting (where appropriate), as follows:

```bash
#!/usr/bin/env bash

printf 'Hello world!'
```

<br />

Example output will be displayed as a fenced code block, beginning with the command that generated the output, as follows:

```
$ cat /etc/os-release
NAME="AlmaLinux"
VERSION="8.3 (Purple Manul)"
ID="almalinux"
ID_LIKE="rhel centos fedora"
VERSION_ID="8.3"
PLATFORM_ID="platform:el8"
PRETTY_NAME="AlmaLinux 8.3 (Purple Manul)"
ANSI_COLOR="0;34"
CPE_NAME="cpe:/o:almalinux:almalinux:8.3:GA"
HOME_URL="https://almalinux.org/"
BUG_REPORT_URL="https://bugs.almalinux.org/"

ALMALINUX_MANTISBT_PROJECT="AlmaLinux-8"
ALMALINUX_MANTISBT_PROJECT_VERSION="8.3"
```

<br />

Explanatory notes, additional information, or references to external resources will be displayed in quote blocks, as follows:

> :information_source: For more information on GitHub Markdown syntax and capabilities, refer to [Writing on Github](https://docs.github.com/en/github/writing-on-github).

<br />

Notes with additional information that has a security or functionality impact that should be considered before implementing will be displayed in quote blocks with a warning symbol, as follows:

> :warning: Take everything you read on the Internet with a grain of salt.

<br />

Notes with additional information describing a danger to the confidentiality, integrity, or availability of the system will be displayed in quote blocks with an exclamation symbol, as follows:

> :exclamation: Do not pipe `curl` directly into `bash`, as this may result in the execution of malicious shell scripts on the system.

<br />

Network ports will be specified in the following form: port/protocol (`service name`). The service name will be omitted if the port in question does not have a service name specified in `/etc/services`, or if the port is being used for a purpose other than that specified in `/etc/services`. 

<br />

Unordered lists will be used to provide lists of relevant information.

<br />

Ordered lists will be used to specify an order-dependent series of actions.

------

## **Definitions**
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.
> :information_source: Specific definitions of these terms are provided in [RFC 2119 - Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119).

------

## **Introduction**

First, let's talk about what security is in the context of a network-connected multi-user operating system. We're also going to discuss what it **isn't**.

Security in a general sense is the intersection between the following three core concepts:

### **Confidentiality**
Confidentiality, in a system security context, is the assurance that information is not disclosed to unauthorized users, processes, or devices. This is the concept of what we traditionally think of as "secrecy," and what most people go to first when they think of security. 

### **Integrity**
Integrity is the assurance that neither the system nor the data contained within has been modified by an unauthorized entity or in an unauthorized way. Compromising this is actually the goal of most non-targeted intrusion attempts. The attacker wants to bring the system under their control so they can use it for their own goals while hiding their activities from the system owner, which leads to the use of rootkits or some other modification to the system; this unauthorized modification is a loss of integrity.

### **Availability**
Availability is the assurance that the system can be accessed by an authorized user and perform its intended functions as needed. A denial of service attack that exhausts the system's resources or otherwise makes it so that the system can't be used is an example of loss of availability. An attacker exploiting a recently-disclosed vulnerability and causing the system to panic and halt would be another example of loss of availability.

<br />

### **System Security: What It Is**
Proper system security is finding the right balance of confidentiality, integrity, and availability based on the purpose of the system and a realistic evaluation of what threats it may face, and putting measures in place to provide the level of protection required.

<br />

### **System Security: What It Isn't**
Proper system security is not flipping every switch and turning every dial up to its absolute maximum. It is easy to make the system either completely unusable or so cumbersome that it might as well be, or to expend so much time and resources that the cost of securing the system has exceeded anything you might lose should the system be compromised.

<br />

### **Principle of Least Privilege**
One of the core principles behind system security is the *principle of least privilege*, which is the concept that every user, process, service, and so on has only that level of access required to perform its given task, and no more. Where appropriate, notes will be provided indicating how this principle has been applied to a given recommendation. Such notes will be prefixed with the characters "PLP."

<br />

### **Security Through Obscurity**
Famous mathematician Claude Shannon, widely known as "the father of information theory," put forth an assertion (referred to generally as Shannon's Maxim) that lies at the foundation of modern information security theory and practice: "The enemy knows the system."

In a more concrete formulation, this means that the system administrator - you - should always assume that any implementation details about your system are immediately known to any attacker. Only ACTUAL secrets (cryptographic keys, passwords, etc.) are expected to remain secret. Anything that can be trivially discovered simply by looking for it, such as open ports and protocols, which services are running, their versions, etc.) cannot be treated as "secret." As a consequence, measures taken to conceal - also known as **obscure** - these implementation details from an attacker have a real-world security value of **absolutely zero**, either as an individual measure or as part of a defense-in-depth approach.

Any recommendation in this guide that is commonly the subject of zero-value obscuring measures will include a note, prefixed with the characters "STO," discussing what those obscuring measures are and why they provide no value.

------

## **Hardening AlmaLinux 8**
### **System Updates**
While this may not be a specific hardening step, the following guidance is probably the single best thing you can do to protect your system:

***KEEP YOUR SYSTEM UP TO DATE.***

This encompasses a number of points:

* System updates MUST be performed regularly.
* All available updates SHOULD be installed.
* Packages SHOULD NOT be excluded or version-locked.
* Keep abreast of security vulnerability disclosures relevant to the operating system, core services, and anything else you may be running on the system so that you are able to address high-priority issues in a timely manner.

Vendors may tell you that their software is only compatible with a certain point release. This is almost universally a complete fabrication, because there is nothing special about a given point release; a point release is just a snapshot of what versions were in the repo at a given point in time, and nothing more.

> :information_source: As always, exceptions exist. Example: The vendor ships a package that relies on a very specific kernel version. In this case, updating the kernel would render that package nonfunctional; this would be a sane reason to version-lock the kernel. However, version-locking packages SHOULD be done as a last resort, as this can have follow-on effects for other packages on the system.

<br />

### **SELinux**
SELinux is an extremely powerful mandatory access control (MAC) layer that is enabled by default within AlmaLinux.

SELinux is capable of blocking a large number of actions that would otherwise result in system compromise, and therefore SHOULD NOT be disabled.

It is acceptable to place the system in `Permissive` mode to perform troubleshooting, but the system SHOULD be returned to `Enforcing` mode as soon as troubleshooting is completed.

There are a litany of resources available on the Internet that describe the concepts and internal workings of SELinux, as well as tools and techniques for troubleshooting issues and adjusting system policies. Use them!

There are a number of software packages whose installation instructions state to disable SELinux. This is nearly always due to the software author not making the effort to create a SELinux policy for their application, and is not indicative of an incompatibility with SELinux in general. It is often possible to create a policy or set of policies that allow the software to function correctly without removing the protections that SELinux provides.

<br />

### **Default Configuration**

After OS installation is complete and the system restarts into the new environment, the following default configuration is present:

* Running services accepting inbound connections:
    
    * `sshd` (22/tcp)
* Open firewall ports (via `firewalld`):
    * 22/tcp (`ssh`)
    * 9090/tcp (`cockpit`)
    * 546/udp (`dhcpv6-client`)
        
        > :information_source: This port is only open to IPv6 peers on the same subnet, and is not reachable by Internet hosts or hosts on other subnets.
* Pertinent `sshd` configuration elements (`/etc/ssh/sshd_config`):
    * `PermitRootLogin yes`
    * `PasswordAuthentication yes`
    * `ChallengeResponseAuthentication no`

<br />

### **Restrict Open Firewall Ports**

The system's firewall MUST only permit connections to services that are intended to be accessible from outside the system.
> :information_source: **PLP**: Closing all ports except those intended to be externally-accessible prevents unprivileged users from starting listening processes on ports that have been left open but do not have a service attached.

In the default `firewalld` configuration, 9090/tcp (`cockpit`) is left open as part of the default configuration under the assumption that the standard Server installation includes the `cockpit` packages and interface. If `cockpit` is not installed (as is the case in a minimal installation), or this functionality is not desired, remove this port from the `firewalld` configuration.

```
# firewall-cmd --remove-service=cockpit
success
# firewall-cmd --remove-service=cockpit --permanent
success
```

> :information_source: `firewalld` operates using two configuration sets: the running configuration, and the persistent, or permanent, configuration. The first command above modifies the running configuration, but this will be lost upon restarting the `firewalld` service or the system itself. The second command commits this change to the permanent configuration so that it remains upon service or system restart.

<br />

### **Allow ICMP/ICMPv6 Traffic**

`firewalld`, by default, blocks ICMP/ICMPv6 traffic inbound to the host. While filtering ICMP/ICMPv6 has often been stated to have security benefits, this assertion has been disproven in innumerable real-world scenarios, and disabling ICMP/ICMPv6 is known to cause a number of functional issues. To avoid these issues, it is strongly recommended to allow ICMP/ICMPv6 traffic:
```
# firewall-cmd --add-protocol=icmp
success
# firewall-cmd --add-protocol=icmp --permanent
success
# firewall-cmd --add-protocol=ipv6-icmp
success
# firewall-cmd --add-protocol=ipv6-icmp --permanent
success
```

> :information_source: For the pedantic, the protocol is correctly referred to as IPv6-ICMP, but ICMPv6 is a common shorthand.

<br />

### **Drop All Non-Permitted Traffic**
`firewalld`, by default, actively rejects inbound connection attempts to ports or via protocols that are not listed in the zone's configuration. It is strongly recommended to change this behavior to silently drop non-permitted traffic.

```
# firewall-cmd --set-target=DROP --permanent
success
```

> :information_source: The `set-target` action does not have a runtime configuration option, and will not take effect until `firewalld` is restarted, either via service restart or system restart.

<br />

### **Harden `sshd` Configuration**

The default configuration of `sshd` allows `root` to log in to the system via password authentication. Given that password-only (single factor) authentication has long been known to be the weakest form of authentication, this presents a significant vulnerability. The `root` user SHOULD NOT be permitted to login via `sshd` using only password authentication.

<br />

There are three options available to remediate this:
* Restrict `root` login via `sshd` to permit only key-based authentication.
    > :warning: Remote monitoring or management solutions may require `root` access via `sshd` to client systems. If this applies to you, this is the correct solution. Prohibiting `root` login via `sshd` entirely may render these solutions nonfunctional.
* Prohibit `root` login via `sshd` entirely.
* Implement a second-factor requirement for `root` login via `sshd`.
    > :information_source: There are many options available for multi-factor authentication (MFA) on Linux operating systems. Selecting and configuring such a solution is outside the scope of this guide.

<br />

To restrict `root` login via `sshd` to permit only key-based authentication, edit `/etc/ssh/sshd_config` via your preferred  editor, and change
```
PermitRootLogin yes
```
to
```
PermitRootLogin prohibit-password
```
and restart `sshd`:
```
# systemctl restart sshd
```

<br />

To prohibit `root` login via `sshd` entirely, edit `/etc/ssh/sshd_config` via your preferred editor, and change
```
PermitRootLogin yes
```
to
```
PermitRootLogin no
```
then restart `sshd`:
```
# systemctl restart sshd
```

<br />

Although compromise of an unprivileged user account does not present the same risk as compromise of the `root` user, similar concerns exist.

If stronger authentication for unprivileged users logging in via `sshd` is desired, password-based authentication MAY be disabled completely, thereby forcing all users to log in via ssh key authentication.
> :exclamation: Be aware that if password authentication is disabled, a user's authorized keys file (`$HOME/.ssh/authorized_keys`) MUST be populated for that user to log in. If this is not completed, the user's account will not be accessible remotely. **You can lock yourself out of a remote system completely, so verify that ssh key authentication works for at least one user with administrative access BEFORE completing the following steps.**

<br />

To disable password authentication within `sshd` completely, edit `/etc/ssh/sshd_config` via your preferred editor, change
```
PasswordAuthentication yes
```
to
```
PasswordAuthentication no
```
and change 
```
ChallengeResponseAuthentication yes
```
to
```
ChallengeResponseAuthentication no
```
then restart `sshd`:
```
# systemctl restart sshd
```
> :warning: Both `PasswordAuthentication` and `ChallengeResponseAuthentication` must be set to `no` in order to disable password-based login. Either mechanism will permit password-based login if left enabled.

> :information_source: **STO**: It is a common misconception that changing the `sshd` service port away from its default (22/tcp) provides a measure of security by concealing the presence of a listening `sshd` service on the system. The presence of a listening `sshd` service, and the port to which that service is attached, **is an implementation detail of the system, not a secret**. It is trivial to probe the system over the network (e.g. via `nmap`) and find the `sshd` service and its associated port. Internet-connected systems are constantly subjected to these kinds of probes, and any listening services will be discovered very quickly.
>
>```
># time nmap -Av -T4 -p1-65535 192.88.99.100
>[...]
>PORT      STATE SERVICE VERSION
>43763/tcp open  ssh     OpenSSH 8.0 (protocol 2.0)
>| ssh-hostkey:
>|   3072 c7:17:fb:fb:ff:91:e8:ba:6b:fd:9c:27:40:ae:3e:0f (RSA)
>|   256 3f:13:c9:bc:c4:1b:08:ec:c3:5c:21:25:11:84:ab:1f (ECDSA)
>|_  256 a7:c9:25:67:e6:dc:7f:13:fd:73:3a:20:a0:2c:c8:5d (ED25519)
>[...]
>real    17m43.575s
>user    0m5.436s
>sys     0m7.276s
>```
>
> As you can see, changing the `sshd` service port to a non-default value provides no security benefit.

<br />

### **Limit Authentication Attempts**
As noted previously, any system in which `sshd` is exposed to the Internet will be subject to a constant bombardment of attempts to gain access to the system. The overwhelming majority of these attempts are the result of botnet activity. The attacker's objective is to gain access to poorly-secured systems and expand the pool of available zombies for the botnet. Obviously, we have no intention of being a poorly-secured system - you are reading this document, after all - so these are of relatively little concern. However, they **are** annoying and can cause a great deal of noise in the `sshd` service log.

We can significantly reduce the volume of these attempts by utilizing `fail2ban`.

The general concept of `fail2ban` is that a given service's log (`fail2ban` is capable of monitoring multiple services) is monitored for events corresponding to failed authentication attempts. If a given number of failed authentication attempts from the same source are observed within a certain window of time, that source is blocked (typically at the firewall level) for a given amount of time.

> :information_source: `fail2ban` is not available in the default AlmaLinux package repositories. Before continuing, you will need to enable the Extra Packages for Enterprise Linux (EPEL) repository provided by the Fedora project, located here: https://fedoraproject.org/wiki/EPEL

The most common use of `fail2ban` is to limit `ssh` connection attempts and significantly reduce the volume of automated connection attempts of the kind discussed previously.

While the global configuration file for `fail2ban` is located at `/etc/fail2ban/jail.conf`, we will not be modifying this file. This will prevent configuration issues in the event that the file is modified or overwritten by a package upgrade. Instead, we will define our `fail2ban` configuration in `/etc/fail2ban/jail.local`.

Our configuration file will contain two sections: a default configuration common to all protected services, and one or more services to protect.

The following is an example configuration file that would protect `sshd`, and a definition of the most common fields in a functional `fail2ban` configuration:

#### **Example Configuration File**

```
[DEFAULT] 
ignoreip = 198.51.100.0/24
findtime  = 900
maxretry = 5
bantime  = 3600
banaction = nftables[type=multiport]
backend = systemd

[sshd] 
enabled = true
```

#### **Field Definitions**
`ignoreip` - Any IP within this range will be ignored by `fail2ban`. Authentication failures originating from clients within this range will not be tallied, and no ban actions will be taken.

`findtime` - The rolling window of time, measured in seconds, within which to monitor for and count authentication failures.

`maxretry` - The number of authentication failures from a single source that, if all observed within the previous `findtime` seconds, will trigger the `banaction`.

`banaction` - The action to perform upon observing `maxretry` authentication failures from a single source within the last `findtime` seconds.

`bantime` - After the `banaction` has been triggered, wait for this amount of time (measured in seconds) before reversing the `banaction`.

`backend` - Use the specified mechanism to monitor the service log for authentication failures. Typically, this will be `systemd`, causing `fail2ban` to monitor the service journal.

<br />

### **To Do**
* Finish `fail2ban`
* ...

------

## **Additional Resources**

------

## **Glossary**

* Confidentiality: 
* Integrity: 
* Availability: 
* Threat:
* Risk:
* Vulnerability:
* Exploit (noun):
* Exploit (verb):

------

## **License**

*AlmaLinux 8 System Hardening Guide* is licensed under a
Creative Commons Attribution-ShareAlike 4.0 International License.

You should have received a copy of the license along with this
work. If not, see <http://creativecommons.org/licenses/by-sa/4.0/>.
