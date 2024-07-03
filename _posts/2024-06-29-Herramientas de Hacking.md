---
title: "Herramientas en Bash y Python para Hacking"
date: 2024-06-28 12:13:02 - 5000
categories: [Hacking, Herramientas]
tags: [Hacking, Herramientas, Python, Bash]
image: assets/img/posts/2024/Herramientas/bash_python.jpg
alt: "Image alt text"
description: Recopilatorio de scripts en Bash y Python para automatizar procesos.
render_with_liquid: false
---

Bash y Python son 2 lenguajes de scripting poderosos que, prácticamente permiten hacer todo en cuanto a tareas de automatización, ciberseguridad y redes. En este post traemos una serie de herramientas que permiten automatizar tareas o reemplazar a algunas herramientas automatizadas en caso se requiera.

# Escaner de equipos activos
## Escaner de equipos activos vía ICMP
```console
#!/bin/bash

for i in $(seq 1 254); do
    timeout 1 bash -c "ping 192.168.18.$i" &>/dev/null && echo "[+] Host 192.168.18.$i - ACTIVE" &
done; wait
```
