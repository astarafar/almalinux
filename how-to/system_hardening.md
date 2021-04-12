# AlmaLinux 8 System Hardening Guide
## Table of Contents
`To be completed when finished.`

## Purpose and Scope
This document is intended to guide a system administrator through the process of hardening the operating system and core services beyond the default measures in place after initial installation.

This document does not cover physical or administrative security concepts or controls, nor does it cover application-specific (httpd, postfix, etc.) concerns.

## Definitions
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

(Yes, I am that guy.)

## Introduction
First, let's talk about what security is in the context of a network-connected multi-user operating system. We're also going to discuss what it **isn't**.

Security in a general sense is the intersection between the following three core concepts:

### 1. Confidentiality
Confidentiality, in a system security context, is the assurance that information is not disclosed to unauthorized users, processes, or devices. This is the concept of what we traditionally think of as "secrecy," and what most people go to first when they think of security. 

### 2. Integrity
Integrity is the assurance that neither the system nor the data contained within has been modified by an unauthorized entity or in an unauthorized way. Compromising this is actually the goal of most non-targeted intrusion attempts. The attacker wants to bring the system under their control so they can use it for their own goals while hiding their activities from the system owner, which leads to the use of rootkits or some other modification to the system; this unauthorized modification is a loss of integrity.

### 3. Availability
Availability is the assurance that the system can be accessed by an authorized user and perform its intended functions as needed. A denial of service attack that exhausts the system's resources or otherwise makes it so that the system can't be used is an example of loss of availability. An attacker exploiting a recently-disclosed vulnerability and causing the system to panic and halt would be another example of loss of availability.

### System Security: What It Is
Proper system security is finding the right balance of confidentiality, integrity, and availability based on the purpose of the system and a realistic evaluation of what threats it may face, and putting measures in place to provide the level of protection required.

### System Security: What It Isn't
Proper system security is not flipping every switch and turning every dial up to its absolute maximum. It is easy to make the system either completely unusable or so cumbersome that it might as well be, or to expend so much time and resources that the cost of securing the system has exceeded anything you might lose should the system be compromised.

## Hardening AlmaLinux 8
### System Updates
While this may not be a specific hardening step, the following guidance is probably the single best thing you can do to protect your system:

***KEEP YOUR SYSTEM UP TO DATE.***

This encompasses a number of points:
* Perform system updates **regularly**.
* Install **all** available updates.
* Do **not** hold/pin package versions or exclude packages.
* Keep abreast of security vulnerability disclosures relevant to the operating system, core services, and anything else you may be running on the system so that you are able to address high-priority issues in a timely manner.

Vendors may tell you that their software is only compatible with a certain point release. This is almost universally a complete fabrication, because there is nothing special about a given point release; a point release is just a snapshot of what versions were in the repo at a given point in time, and nothing more.

## Glossary

* Confidentiality: 
* Integrity: 
* Availability: 
* Threat:
* Risk:
* Vulnerability:
* Exploit (noun):
* Exploit (verb):

# Credits
Written by Connor Sheridan for the AlmaLinux Foundation.