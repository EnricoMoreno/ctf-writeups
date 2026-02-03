# Report — Máquina: `Legacy` — `10.129.10.40`

- **Fonte:** Hack The Box (HTB)
    
- **Data:** 2025-12-18
    
- **Autor:** Enrico Moreno
    
- **Nível:** Easy
    
---

## Executive Summary

A máquina **Legacy** foi comprometida explorando uma vulnerabilidade crítica no **SMBv1 (CVE-2008-4250 / MS08-067)** presente no Windows XP. A exploração permitiu execução remota de código e acesso direto como **NT AUTHORITY\SYSTEM**, resultando na obtenção das flags de usuário e administrador.

---

## Planning & Scoping

- **Alvo:** `10.129.10.40`
    
- **Sistema operacional:** Windows XP
    
- **Ferramentas utilizadas:** nmap, Metasploit
    
- **Limitações:** sem brute force externo, sem pivoting
    

---

## Discovery / Recon

### Nmap — Varredura completa

```bash
sudo nmap -sS -T4 -p- -sV -Pn --min-rate 5000 10.129.10.40 -oN ~/Desktop/Nmap_Legacy.txt
```

**Resultado relevante:**

```
PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Microsoft Windows XP microsoft-ds
```

**Arquivos:**

![](Nmap_Legacy.txt)

---

## Enumeration & Analysis

### Enumeração de protocolo SMB

```bash
nmap -p 445 --script smb-protocols 10.129.10.40
```

**Resultado:**

```
SMBv1 (NT LM 0.12) [dangerous, but default]
```

A presença de **SMBv1** indica um sistema legado vulnerável ao **MS08-067 (CVE-2008-4250)**, uma falha crítica de execução remota de código amplamente explorada em ambientes Windows XP.

---

## Attack / Exploitation

### Exploração MS08-067 (CVE-2008-4250)

Utilizado o módulo Metasploit:

```text
exploit/windows/smb/ms08_067_netapi
```

A exploração resultou em uma **sessão meterpreter** com privilégios máximos.

---

## Post-Exploitation

### Contexto da Sessão

- **Usuário:** `NT AUTHORITY\\SYSTEM`
    
- **Tipo de acesso:** Meterpreter
    

---

## Privilege Escalation

Não aplicável. A vulnerabilidade permite acesso direto como **SYSTEM**, dispensando técnicas adicionais de escalonamento.

---

## Impact & Root Access

- **Impacto:** Comprometimento total do sistema
    
- **Nível de severidade:** Crítico
    
- **Motivo:** Execução remota de código via SMBv1 sem autenticação
    

---

## Flags Obtidas

### User Flag

```
c:\Documents and Settings\john\Desktop\user.txt
```

### Root Flag

```
c:\Documents and Settings\Administrator\Desktop\root.txt
```

---

## Remediation Recommendations

- Desabilitar **SMBv1** imediatamente
    
- Aplicar patches de segurança (MS08-067)
    
- Atualizar sistemas legados para versões suportadas
    
- Restringir acesso às portas 139/445 via firewall
    

---

## Lessons Learned

- SMBv1 representa risco crítico e não deve ser utilizado
    
- Sistemas legados expostos são alvos triviais
    
- Enumeração de protocolo é decisiva para identificação de exploits clássicos
    

---

## Summary

A máquina **Legacy** exemplifica o impacto de serviços obsoletos expostos à rede. A exploração foi direta, confiável e resultou em comprometimento total, reforçando a importância de descontinuação de protocolos inseguros como o SMBv1.
