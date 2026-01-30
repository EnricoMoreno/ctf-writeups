# Report — Máquina: Devel — 10.129.9.61

- **Fonte:** Hack The Box
    
- **Data:** 2025-12-17
    
- **Autor:** Enrico Moreno
    
- **Nível:** easy
    
---

## Executive Summary

A máquina **Devel** apresenta FTP com login anônimo e permissão de escrita, permitindo o upload de um payload **ASPX** que resulta em execução remota de código via IIS 7.5. Após obter acesso inicial, foi possível realizar **privilege escalation** utilizando a vulnerabilidade **MS10-015 (KiTrap0D)**, culminando em acesso **SYSTEM** e obtenção das flags de usuário e administrador.

---

## Planning & Scoping

- **Alvo:** 10.129.9.61
    
- **Sistema Operacional:** Windows
    
- **Serviços expostos:** FTP (21/tcp), HTTP (80/tcp)
    
- **Ferramentas utilizadas:** Nmap, FTP client, msfvenom, Metasploit Framework
    
- **Limitações:** Sem brute force externo
    

---

## Discovery / Recon

### Nmap

```bash
nmap -Pn -sS -p- -T4 -sV 10.129.9.61 -oN ~/Desktop/Nmap_Devel.txt
```

**Resultados relevantes:**

- `21/tcp` — FTP — Microsoft ftpd
    
- `80/tcp` — HTTP — Microsoft IIS 7.5
    

Indicação clara de ambiente **Windows + IIS + ASP.NET**.

**Arquivos:**

![](Nmap_Devel.txt)

---

## Enumeration & Analysis

### FTP

- Login anônimo habilitado (`anonymous`)
    
- Permissão de escrita ativa (`put` permitido)
    
- Diretório raiz do FTP corresponde à webroot do IIS
    

Isso permite o upload direto de arquivos executáveis pelo servidor web.

---

## Attack / Exploitation

### Acesso inicial

Foi gerado um payload **ASPX** e enviado via FTP para a webroot. Ao acessar o arquivo pelo navegador, obteve-se uma shell reversa no sistema alvo.

Resultado: **acesso inicial como usuário de baixo privilégio**.

---

## Post-Exploitation

Após a obtenção da sessão, foi realizada enumeração local para identificar possíveis vetores de elevação de privilégio.

---

## Privilege Escalation

### Módulo utilizado

```
exploit/windows/local/ms10_015_kitrap0d
```

A vulnerabilidade **MS10-015 (KiTrap0D)** afeta versões antigas do Windows e permite elevação de privilégio local.

### Resultado

```text
meterpreter > getuid
Server username: NT AUTHORITY\\SYSTEM
```

Privilégios elevados com sucesso.

---

## Impact & Root Access

### Flags obtidas

- **User flag:** `C:\Users\babis\Desktop\user.txt`
    
- **Root flag:** `C:\Users\Administrator\Desktop\root.txt`
    

Impacto total: **comprometimento completo do sistema**.

---

## Remediation Recommendations

- Desabilitar login FTP anônimo
    
- Remover permissões de escrita na webroot
    
- Atualizar o sistema operacional e aplicar patches de segurança
    
- Restringir execução de scripts no IIS
    

---

## Lessons Learned

- FTP com escrita + IIS é um vetor crítico de RCE
    
- ASP.NET (`.aspx`) é o principal alvo em IIS moderno
    
- Sistemas Windows desatualizados permanecem altamente vulneráveis a exploits locais conhecidos
    

---

## Summary

A exploração da máquina Devel demonstra como serviços legados e configurações inseguras podem levar rapidamente ao comprometimento total do sistema, reforçando a importância de hardening básico e atualizações regulares.