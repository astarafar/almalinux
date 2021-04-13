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
> :information_source: PLP: Closing all ports except those intended to be externally-accessible prevents unprivileged users from starting listening processes on ports that have been left open but do not have a service attached.

In the default `firewalld` configuration, 9090/tcp (`cockpit`) is left open as part of the default configuration under the assumption that the standard Server installation includes the `cockpit` packages and interface. If `cockpit` is not installed (as is the case in a minimal installation), or this functionality is not desired, remove this port from the `firewalld` configuration.

```
# firewall-cmd --remove-service=cockpit
success
# firewall-cmd --remove-service=cockpit --permanent
success
```
> :information_source: `firewalld` operates using two configuration sets: the running configuration, and the persistent, or permanent, configuration. The first command above modifies the running configuration, but this will be lost upon restarting the `firewalld` service or the system itself. The second command commits this change to the permanent configuration so that it remains upon service or system restart.

> :warning: `firewalld` implicitly allows ICMP/ICMPv6 traffic inbound to the host. While filtering ICMP/ICMPv6 has often been stated to have security benefits, this assertion has been disproven in innumerable real-world scenarios, and disabling ICMP/ICMPv6 is known to cause a number of functional issues. **ICMP/ICMPv6 SHOULD NOT be filtered by the firewall.**

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
