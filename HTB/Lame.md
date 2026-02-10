# Report — Máquina: `Lame` — `10.129.10.2`

- **Fonte:** Hack The Box (HTB)
    
- **Data:** 2025-12-18
    
- **Autor:** Enrico Moreno
    
- **Nível:** Easy
    
---

## Executive Summary

A máquina **Lame** foi comprometida por meio de uma vulnerabilidade conhecida no **Samba (CVE-2007-2447)**, permitindo execução remota de comandos como **root**. O acesso inicial e final foi obtido diretamente com privilégios administrativos, resultando na captura das flags de usuário e root.

---

## Planning & Scoping

- **Alvo:** `10.129.10.2`
    
- **Sistema operacional:** Linux
    
- **Ferramentas utilizadas:** nmap, smbclient, Metasploit
    
- **Limitações:** sem brute force externo, sem pivoting
    

---

## Discovery / Recon

### Nmap — Varredura completa

```bash
sudo nmap -sS -T4 -p- -sV -Pn --min-rate 5000 10.129.10.2 -oN ~/Desktop/Nmap_Lame.txt
```

**Resultado relevante:**

```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X
3632/tcp open  distccd     distccd v1
```

**Arquivos:**

![](Nmap_Lame.txt)

---

## Enumeration & Analysis

### Enumeração SMB

```bash
smbclient -L //10.129.10.2/ -N
```

**Resultado:**

```
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
tmp             Disk      oh noes!
opt             Disk
IPC$            IPC       IPC Service
ADMIN$          IPC       IPC Service
```

O banner do serviço SMB indicou a versão **Samba 3.0.20-Debian**, vulnerável ao **CVE-2007-2447**, que permite execução remota de comandos via script `usermap`.

---

## Attack / Exploitation

### Exploração do Samba (CVE-2007-2447)

Utilizado o módulo Metasploit:

```text
multi/samba/usermap_script
```

O módulo retornou uma **shell direta como root**, sem necessidade de escalonamento de privilégios adicional.

---

## Post-Exploitation

### Upgrade para Meterpreter

```text
multi/manage/shell_to_meterpreter
```

Sessão convertida com sucesso para **meterpreter**, facilitando navegação e coleta de artefatos.

---

## Privilege Escalation

Não aplicável. O acesso inicial já foi obtido como **root** devido à vulnerabilidade crítica no Samba.

---

## Impact & Root Access

- **Impacto:** Comprometimento total do sistema
    
- **Nível de severidade:** Crítico
    
- **Motivo:** Execução remota de comandos com privilégios administrativos sem autenticação
    

---

## Flags Obtidas

### User Flag

```
/home/makis/user.txt
```

### Root Flag

```
/root/root.txt
```

---

## Remediation Recommendations

- Atualizar imediatamente o **Samba** para versões corrigidas
    
- Restringir ou desabilitar SMB anônimo
    
- Bloquear portas SMB (139/445) externamente
    
- Monitorar logs para execuções suspeitas
    

---

## Lessons Learned

- Serviços legados expostos representam alto risco
    
- Enumeração de versões é crítica para identificação rápida de exploits conhecidos
    
- Exploits públicos podem resultar em comprometimento total sem etapas intermediárias
    

---

## Summary

A máquina **Lame** demonstra um cenário clássico de falha crítica por software desatualizado. A exploração foi direta, reprodutível e com impacto máximo, sendo um excelente laboratório introdutório para identificação de serviços vulneráveis e uso do Metasploit.