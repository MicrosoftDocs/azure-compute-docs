---
 title: include file
 description: include file
 services: virtual-machines-linux
author: jushiman
 ms.service: virtual-machines
 ms.topic: include
 ms.date: 04/17/2018
ms.author: jushiman
 ms.custom: include file
# Customer intent: As a system administrator, I want to understand the supported SSH key formats for virtual machines, so that I can configure secure connections effectively.
---

## Supported SSH key formats

Azure currently supports following key types:
* SSH protocol 2 (SSH-2) RSA (Rivest, Shamir, Adleman) with a minimum length of 2048 bits  
* ED25519 Keys with a fixed length of 256 bits 

Other key formats such as Elliptic-curve Diffie–Hellman (ECDH) and Elliptic Curve Digital Signature Algorithm (ECDSA) are currently not supported. 
