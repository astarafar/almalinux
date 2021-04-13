# **AlmaLinux 8 System Hardening Guide**
## **Table of Contents**
`To be completed when finished.`

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

> For more information on GitHub Markdown syntax and capabilities, refer to [Writing on Github](https://docs.github.com/en/github/writing-on-github).

<br />

Network ports will be specified in the following form: port/protocol (`service name`). The service name will be omitted if the port in question does not have a service name specified in `/etc/services`, or if the port is being used for a purpose other than that specified in `/etc/services`. 

<br />

Unordered lists will be used to provide lists of relevant information.

<br />

Ordered lists will be used to specify an order-dependent series of actions.

------

## **Definitions**
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.
> Specific definitions of these terms are provided in [RFC 2119 - Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119).

## **Introduction**

First, let's talk about what security is in the context of a network-connected multi-user operating system. We're also going to discuss what it **isn't**.

Security in a general sense is the intersection between the following three core concepts:

### **1. Confidentiality**
Confidentiality, in a system security context, is the assurance that information is not disclosed to unauthorized users, processes, or devices. This is the concept of what we traditionally think of as "secrecy," and what most people go to first when they think of security. 

### **2. Integrity**
Integrity is the assurance that neither the system nor the data contained within has been modified by an unauthorized entity or in an unauthorized way. Compromising this is actually the goal of most non-targeted intrusion attempts. The attacker wants to bring the system under their control so they can use it for their own goals while hiding their activities from the system owner, which leads to the use of rootkits or some other modification to the system; this unauthorized modification is a loss of integrity.

### **3. Availability**
Availability is the assurance that the system can be accessed by an authorized user and perform its intended functions as needed. A denial of service attack that exhausts the system's resources or otherwise makes it so that the system can't be used is an example of loss of availability. An attacker exploiting a recently-disclosed vulnerability and causing the system to panic and halt would be another example of loss of availability.

### **System Security: What It Is**
Proper system security is finding the right balance of confidentiality, integrity, and availability based on the purpose of the system and a realistic evaluation of what threats it may face, and putting measures in place to provide the level of protection required.

### **System Security: What It Isn't**
Proper system security is not flipping every switch and turning every dial up to its absolute maximum. It is easy to make the system either completely unusable or so cumbersome that it might as well be, or to expend so much time and resources that the cost of securing the system has exceeded anything you might lose should the system be compromised.

### **Principle of Least Privilege**
One of the core principles behind system security is the *principle of least privilege*, which is the concept that every user, process, service, and so on has only that level of access required to perform its given task, and no more. Where appropriate, notes will be provided indicating how this principle has been applied to a given recommendation. Such notes will be prefixed with the characters "PLP."

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

> As always, exceptions exist. Example: The vendor ships a package that relies on a very specific kernel version. In this case, updating the kernel would render that package nonfunctional; this would be a sane reason to version-lock the kernel. However, version-locking packages SHOULD be done as a last resort, as this can have follow-on effects for other packages on the system.

### ****Default Configuration****

After OS installation is complete and the system restarts into the new environment, the following default configuration is present:

* Running services accepting inbound connections:
    
    * `sshd` (22/tcp)
* Open firewall ports (via `firewalld`):
    * 22/tcp (`ssh`)
    * 9090/tcp (`cockpit`)
    * 546/udp (`dhcpv6-client`)
        
        > Note: this port is only open to IPv6 peers on the same subnet, and is not reachable by Internet hosts or hosts on other subnets.
* Pertinent `sshd` configuration elements (`/etc/ssh/sshd_config`):
    * `PermitRootLogin yes`
    * `PasswordAuthentication yes`
    * `ChallengeResponseAuthentication no`

### **Restrict Open Firewall Ports**

The system's firewall MUST only permit connections to services that are intended to be accessible from outside the system.
> PLP: Closing all ports except those intended to be externally-accessible prevents unprivileged users from starting listening processes on ports that have been left open but do not have a service attached.

In the default `firewalld` configuration, 9090/tcp (`cockpit`) is left open as part of the default configuration under the assumption that the standard Server installation includes the `cockpit` packages and interface. If `cockpit` is not installed (as is the case in a minimal installation), or this functionality is not desired, remove this port from the `firewalld` configuration.

```
# firewall-cmd --remove-service=cockpit
success
# firewall-cmd --remove-service=cockpit --permanent
success
```
> `firewalld` operates using two configuration sets: the running configuration, and the persistent, or permanent, configuration. The first command above modifies the running configuration, but this will be lost upon restarting the `firewalld` service or the system itself. The second command commits this change to the permanent configuration so that it remains upon service or system restart.

> `firewalld` implicitly allows ICMP/ICMPv6 traffic inbound to the host. While filtering ICMP/ICMPv6 has often been stated to have security benefits, this assertion has been disproven in innumerable real-world scenarios, and disabling ICMP/ICMPv6 is known to cause a number of functional issues. **ICMP/ICMPv6 SHOULD NOT be filtered by the firewall.**

### **Harden `sshd` Configuration**

The default configuration of `sshd` allows `root` to log in to the system via password authentication. Given that password-only (single factor) authentication has long been known to be the weakest form of authentication, this presents a significant vulnerability. There are three options available to remediate this:
* Restrict `root` login via `sshd` to permit only key-based authentication.
    > Remote monitoring or management solutions may require `root` access via `sshd` to client systems. If this applies to you, this is the correct solution. Prohibiting `root` login via `sshd` entirely may render these solutions nonfunctional.
* Prohibit `root` login via `sshd` entirely.
* Implement a second-factor requirement for `root` login via `sshd`.
    
    > There are many options available for multi-factor authentication (MFA) on Linux operating systems. Selecting and configuring such a solution is outside the scope of this guide.

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

To prohibit `root` login via `sshd` entirely, edit `/etc/ssh/sshd_config` via your preferred  editor, and change
```
PermitRootLogin yes
```
to
```
PermitRootLogin no
```
and restart `sshd`:
```
# systemctl restart sshd
```

Although compromise of an unprivileged user account does not present the same risk as compromise of the `root` user, similar concerns exist. 

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
